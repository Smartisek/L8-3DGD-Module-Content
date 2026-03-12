---
title: "Interior Environment Optimisation in Unreal Engine 5"
subtitle: "Lighting Performance, Geometry, Profiling, and Level Streaming"
author: "Niall McGuinness"
institute: "Dundalk Institute of Technology"
programme: "BSc (Hons) in Computing in Games Development"
module_code: "COMP I8015"
module_title: "3D Game Development"
stage: 4
academic_year: "2025–26"
version: "1.1"
format:
  html:
    toc: true
    toc-depth: 2
    number-sections: true
  pdf:
    toc: true
    number-sections: true
---

# Interior Environment Optimisation 

> These notes are optimisation-first, not feature-first. Every technique described here exists because interior environments in Unreal Engine 5 have specific, predictable failure modes that will surface before submission if you do not address them early. Read the accompanying **Project Companion** for direct mapping of these principles to your specific game. Profiling is not a final-week activity. Start it at greybox.

---

## 1. Why Interior Environments Demand Specific Optimisation

An exterior environment has natural occlusion helpers: terrain blocks distant geometry, draw distances are managed by atmospheric haze, and the sky provides a cheap, consistent ambient base. An interior environment has none of these. You are responsible for every light bounce, every shadow, and every draw call in a fully enclosed space. Without deliberate design, Lumen will attempt to resolve global illumination across an entire multi-room structure, shadow-casting lights will double and triple their cost through reflections and bounces, and draw calls will accumulate across every modular wall tile and prop in line of sight.

Interior optimisation is not an end-of-production polish task. It is a design discipline that must begin at greybox — because the spatial decisions you make in the first week of building (room size, corridor length, sightline management, wall enclosure) have direct and non-trivial consequences for frame time. A room with a long, unobstructed sightline into three adjacent spaces will cost significantly more to render than an equivalent room where doorframes break the view into discrete occlusion zones.

The target frame time for a shipping interior environment on a mid-range student development machine is **16.67ms per frame (60fps)**. Every technique in these notes exists to help you stay within that budget without compromising the visual quality your project brief requires.

---

## 2. Lighting Optimisation

### 2.1 Photometric Units: Formal Definitions

Unreal Engine 5 uses physically based lighting. Every light actor exposes its intensity in real-world photometric units rather than arbitrary engine values. Understanding what these units measure is not an academic exercise — it is a prerequisite for placing lights with predictable results and for reading the lighting references (photographs, film stills, architectural specifications) that professional level artists use to calibrate their scenes.

---

**Candela (cd)**

The candela is the SI base unit of **luminous intensity** — the power of a light source in a specific direction, weighted by the sensitivity of the human eye to different wavelengths. One candela is formally defined as the luminous intensity of a source emitting monochromatic radiation at 540 × 10¹² Hz (approximately green light) with a radiant intensity of 1/683 watts per steradian in that direction.

In practical terms, candela measures *how bright a light appears when you look directly toward it from a given angle*. A standard candle produces approximately 1 cd. A household 60W incandescent bulb produces roughly 60–80 cd. A car headlamp on high beam produces approximately 20,000 cd.

In Unreal Engine 5, Point Lights and Spot Lights can be set to use candela as their intensity unit. This is the correct unit when you are modelling a specific real-world fixture whose photometric data sheet specifies intensity in candela — a chandelier pendant, a wall sconce, a practical lamp. Candela intensity does not attenuate with distance in the energy sense; it is the intrinsic directional brightness of the source.

---

**Lumen (lm)**

The lumen is the SI unit of **luminous flux** — the total quantity of light emitted by a source in all directions, integrated over the full sphere, weighted by the human visual response curve. It measures the *total light output* of a source, not its output in any particular direction.

The relationship between lumens and candela is: **1 lm = 1 cd · sr**, where sr is steradian (the unit of solid angle). A point source emitting 1 candela uniformly in all directions produces a total flux of 4π ≈ 12.57 lumens.

In practical terms, a household 60W incandescent bulb produces approximately 800 lm. An LED tube replacement for a fluorescent strip (1.2m) produces approximately 2,000–3,500 lm. These figures appear on the packaging of every commercial light fitting, making lumens the most useful unit for matching real-world fixtures in an architectural interior.

In Unreal Engine 5, the default intensity unit for Point Lights, Spot Lights, and Rect Lights is lumens. When set to lumens, UE5 internally distributes the total flux across the light's shape and solid angle — a Rect Light set to 2,000 lm will distribute that output across the area of the emitting rectangle, producing lower intensity at the edge and higher intensity at the centre, consistent with physical area light behaviour.

> **Note on naming conflict.** Unreal Engine 5's *Lumen* global illumination system takes its name from the photometric unit but is a separate concept. The unit measures physical light quantity; the system is a software implementation of real-time global illumination. The naming can cause confusion — when these notes refer to *Lumen*, the capitalised form refers to the GI system; the lower-case *lumen* (lm) refers to the photometric unit.

---

**Nit (cd/m²)**

The nit is the informal name for the SI unit of **luminance**, measured in candela per square metre (cd/m²). Where candela measures the directional intensity of a source and lumen measures its total output, luminance measures *how bright a surface appears to an observer* — the amount of light arriving at the eye from a specific surface area in a specific direction.

Luminance is what the camera and the eye actually perceive. A surface with high luminance appears bright; a surface with low luminance appears dark. The relationship between the illuminance falling on a surface (lux, see below) and the luminance reflected toward an observer depends on the surface's reflectance (albedo) and its BRDF.

Typical luminance reference values:

| Surface or source | Luminance (cd/m²) |
|:---|---:|
| Clear sky at noon | ~8,000 |
| White paper in office lighting | ~100–250 |
| Computer monitor (SDR) | ~200–300 |
| Computer monitor (HDR peak) | 400–1,000+ |
| Candle flame | ~10,000 |
| Bare incandescent filament | ~10,000,000 |

In Unreal Engine 5, luminance is the primary unit used by the **Auto Exposure** (Eye Adaptation) system and the **Post Process Volume**. The `Min` and `Max Brightness` parameters in the Post Process Volume are expressed as EV100 values derived from scene luminance. When you disable Auto Exposure and set a fixed exposure, you are telling the camera to treat a specific luminance value as middle grey — which is precisely what a physical camera's exposure setting does. Emissive material intensity in UE5 is specified in nits; the physically correct range for an emissive surface representing a lit practical (a lamp shade, a neon tube) is typically 50–5,000 cd/m² depending on fixture type.

---

**Lux (lx)**

The lux is the SI unit of **illuminance** — the luminous flux arriving at (or incident on) a surface per unit area. One lux equals one lumen per square metre (1 lx = 1 lm/m²). Where luminance (nits) describes what the eye receives from a surface, illuminance (lux) describes how much light is *landing on* that surface from the environment.

The inverse-square law governs how illuminance falls off with distance from a point source: doubling the distance from the source quarters the illuminance. This is why a chandelier positioned 8m above a marble floor produces dramatically less illuminance at floor level than the same fixture positioned 3m above it, and why accurate placement height matters when trying to match a real-world reference photograph.

Typical illuminance reference values:

| Environment | Illuminance (lux) |
|:---|---:|
| Direct summer sunlight | ~100,000 |
| Overcast outdoor daylight | ~1,000–10,000 |
| Well-lit office or classroom | ~300–500 |
| Domestic living room (ambient) | ~50–150 |
| Candlelit dining room | ~10–30 |
| Moonlit outdoor scene | ~0.1–1 |

In Unreal Engine 5, the **Directional Light** (sun or sky) exposes its intensity in lux. A physically accurate noon sun is approximately 100,000 lux. Interior scenes receiving natural light through windows will have interior illuminance values of 300–2,000 lux depending on glazing area, orientation, and sky condition — which matches the domestic and office values in the table above. Understanding the lux scale allows you to set interior Directional Light contributions (for windows with skylights or light shafts) to physically plausible values without guesswork.

---

**Relationship between the four units**

The four units describe different stages of the same physical process:

```
Light source emits  →  luminous flux (lm, total output)
                    →  in a direction with intensity (cd)
Landing on a surface  →  illuminance (lx = lm/m²)
Reflected toward eye  →  luminance (cd/m² = nits)
```

A professional lighting workflow moves through these stages in sequence: specify the source output in lumens (matching the fixture datasheet), position it so the illuminance at key surfaces matches a reference photograph, then verify the resulting luminance in the rendered frame matches the target exposure value. Unreal Engine 5's physically based pipeline supports this workflow directly.

---

### 2.2 Lumen Configuration for Interior Spaces

Lumen is UE5's fully dynamic global illumination and reflection system. In interior environments it is powerful and expensive. It will produce physically accurate bounce light, colour bleed, and reflections automatically — but only if you configure it deliberately. The default project settings are not tuned for interior performance at game frame rates.

At project creation, verify these settings in **Edit → Project Settings → Rendering**:

- **Dynamic Global Illumination Method**: Lumen
- **Reflection Method**: Lumen
- **Shadow Map Method**: Virtual Shadow Maps
- **Hardware Ray Tracing**: enabled only if your target spec has an RTX-class GPU — otherwise leave disabled for broader compatibility

**Lumen quality tuning.** Lumen's quality is controlled through a Post Process Volume placed in the level with **Infinite Extent** checked. The key parameters to adjust in its Lumen GI and Lumen Reflections sections are:

- **Final Gather Quality** — controls how many rays are used to resolve indirect lighting. At 1.0 (default) this is already significant. For enclosed interiors drop this to 0.5–0.75 and test for visual acceptability.
- **Max Trace Distance** — the maximum distance Lumen traces rays. Interior rooms rarely need more than 600–800 units. Set this to match your largest room diagonal, not the full level extent.
- **Scene Capture Cache Resolution** — increase to 128 or 256 only for reflective surfaces (marble, glass, polished metal) that will be seen at close range. For general use, leave at default.

> **Common mistake.** Leaving the Post Process Volume on default settings means Lumen is tracing rays across the entire level distance. In a multi-room interior this is immediately visible in the GPU Visualiser as an oversized Lumen GI pass. Add the volume and cap Max Trace Distance at week two of production.

<!-- Search: Unreal Engine 5 GPU Visualiser VSM shadow pass breakdown screenshot | Source: Epic Games documentation or UE5 editor screenshot -->

*The GPU Visualiser (Ctrl + Shift + ,) breaks the frame into individual passes. ShadowDepths and the Lumen GI pass are the most common performance bottlenecks in interior environments. Identify these passes first before making any changes.*

---

### 2.3 Virtual Shadow Maps

Virtual Shadow Maps (VSM) replace the traditional cascade shadow map system and are the correct shadow method for UE5 interior environments. They provide sub-pixel-accurate contact shadows under railings, columns, and mezzanine overhangs that traditional shadow maps cannot achieve without a prohibitive increase in shadow resolution.

VSM cost scales primarily with **the number of shadow-casting lights** and the **density of geometry** within each light's influence radius. For a large interior environment such as a grand foyer with a mezzanine:

- Only your hero light fixtures — central chandelier, primary skylight — should cast dynamic VSM shadows.
- Secondary wall sconces and fill lights should have **Cast Shadows** disabled. The visual loss is negligible for secondary fixtures; the performance saving is significant.
- Set `r.Shadow.Virtual.ResolutionLodBiasLocal` to 1.0 for lights beyond 4–5 metres from the player's primary navigation path to halve their shadow tile resolution without visible quality loss.

**What to profile.** Open the GPU Visualiser (**Ctrl + Shift + ,**) and locate the **ShadowDepths** and **VSMProject** entries. These two passes together should not exceed 4–5ms in an interior at target spec. If they do, the cause is almost always too many shadow-casting lights or a light with an unnecessarily large radius.

---

### 2.4 Light Mobility and Motivated Sources

Every light in your interior should have a **motivated source** — a visible or strongly implied real-world origin (a ceiling fixture, a window, a neon sign). This is not only an artistic principle; it has direct performance consequences. Motivated sources constrain where lights are placed, which naturally limits their count and radius, which reduces VSM and Lumen cost.

Assign light mobility with performance in mind:

- **Static lights** are baked to lightmaps and have no runtime cost. Use for any fixed-position ambient fill that will never change state — ceiling fill lights in a stable environment, for example. Note that baked lighting and Lumen are not mutually exclusive; you can use Static secondary lights while keeping your hero lights Movable under Lumen.
- **Stationary lights** bake their indirect contribution but resolve direct shadowing at runtime. Appropriate for fixtures that could theoretically move in a scripted event but are stable in normal gameplay.
- **Movable lights** are fully dynamic and carry the full VSM and Lumen cost. Reserve Movable mobility for lights that *must* change state during gameplay — a chandelier that dims during a power failure, a flickering fluorescent during an alarm sequence.

> **Design rule.** Every Movable light in your scene should correspond to a specific gameplay state change that the player will observe. If a light is Movable but its state never changes during play, switch it to Stationary or Static. Movable lights that do not move are the most common avoidable performance cost in student interior environments.

---

### 2.5 IES Profiles and Rect Lights

**IES profiles** are photometric data files that define the real-world light distribution pattern of a fixture. Applying an IES profile to a Point or Spot Light produces physically accurate falloff and directionality that matches the actual fixture type — a chandelier IES profile will cast radially upward and downward in the correct proportions; a recessed downlight IES will produce a tight cone with characteristic falloff. UE5 ships with a library of IES profiles accessible from the Details panel of any light actor.

IES profiles do not add GPU cost. They improve visual accuracy at no performance penalty and are the single fastest way to make a set of identical Point Lights read as different fixture types.

**Rect Lights** should be used in preference to rows of Point Lights for any linear architectural source: LED strips, long fluorescent fittings, cove lighting above a mezzanine rail. A single Rect Light configured to match the physical dimensions of the cove is cheaper than four or five Point Lights approximating the same area, and produces more accurate area shadow quality under Lumen.

---

## 3. Physical Camera Parameters and Exposure

Unreal Engine 5's camera and Post Process Volume are modelled directly on a real-world photographic camera. Every exposure parameter in the Post Process Volume maps to a physical camera control. Understanding those controls as a photographer understands them — not as arbitrary sliders — is the prerequisite for setting scene exposure intentionally, for matching photographic reference, and for understanding why the Auto Exposure system behaves as it does in a given lighting scenario.

This section describes each parameter in its physical form first, then states its role in UE5.

---

### 3.1 Aperture (f-stop / f-number)

**Physical definition.** The aperture is the opening in the camera lens through which light passes to reach the sensor. Its size is expressed as the **f-number** (also called the f-stop), denoted *N*, which is the ratio of the lens focal length *f* to the diameter of the aperture opening *D*:

$$N = \frac{f}{D}$$

A larger f-number means a *smaller* aperture opening; a smaller f-number means a *larger* opening. This counterintuitive relationship exists because the f-number is a ratio in which the aperture diameter is the denominator — as the opening widens, *N* falls. The standard f-stop sequence is:

**f/1.0, f/1.4, f/2, f/2.8, f/4, f/5.6, f/8, f/11, f/16, f/22**

Each step in this sequence halves or doubles the *area* of the aperture opening (because area scales with the square of the diameter), which halves or doubles the amount of light reaching the sensor. Moving from f/2.8 to f/5.6 is a two-stop reduction — one quarter the light.

**Aperture controls two independent visual properties simultaneously:**

1. **Exposure** — a wider aperture (lower *N*) admits more light, producing a brighter image for a given shutter speed and ISO.
2. **Depth of field** — a wider aperture produces a *shallower* depth of field (less of the scene is in sharp focus); a narrower aperture produces a *deeper* depth of field (more of the scene is sharp). This follows from the geometry of lens optics: a wider opening produces a larger circle of confusion for any out-of-focus point.

Typical real-world aperture values by context:

| Context | Typical f-stop |
|:---|:---|
| Portrait (shallow, subject separation) | f/1.4 – f/2.8 |
| Street / documentary | f/5.6 – f/8 |
| Landscape (maximum depth) | f/8 – f/11 |
| Architecture (full depth of field) | f/11 – f/16 |
| Interior, tripod, low light | f/2.8 – f/5.6 |

**In Unreal Engine 5.** The Post Process Volume exposes aperture as **Aperture (f-stop)** under the Camera section. It drives two independent effects:

- **Depth of field** — the Depth of Field section uses the aperture value to compute the circle of confusion and produce lens-accurate focus falloff. f/1.4 produces a cinematic shallow-focus look; f/11 produces a sharp, deep-focus image across the full scene depth.
- **Bokeh shape and size** — out-of-focus highlight discs are computed from the aperture value and the number of aperture blades set in the Camera Actor.

> **Note.** Aperture in UE5 does **not** affect scene brightness directly when Auto Exposure is active — the Auto Exposure system compensates for the change in light quantity. To use aperture as an exposure control, disable Auto Exposure and use Manual exposure mode.

---

### 3.2 Shutter Speed (Exposure Time)

**Physical definition.** The shutter is the mechanical curtain or electronic gate that controls how long the camera sensor is exposed to light. **Shutter speed** is the duration of that exposure, denoted *t*, measured in seconds. It is conventionally expressed as a fraction: 1/125s, 1/60s, 1/30s, and so on.

A longer shutter speed admits more light (brighter exposure) but records more motion blur — any subject movement during the exposure is smeared across the frame. A shorter shutter speed admits less light but freezes motion. The relationship is linear: doubling *t* doubles the light, producing one stop brighter exposure.

**The 180-degree shutter rule** is the cinematographic convention for selecting shutter speed to produce natural-looking motion blur: set the shutter speed to approximately twice the frame rate. At 24fps this gives 1/48s; at 30fps it gives 1/60s; at 60fps it gives 1/120s. Deviating significantly upward (very fast shutter) produces stroboscopic motion. Deviating downward (very slow shutter) produces excessive or nauseating blur.

Typical real-world shutter speeds by context:

| Context | Typical shutter speed |
|:---|:---|
| Sports / action freeze | 1/1000s – 1/2000s |
| Film (24fps, 180° rule) | 1/48s |
| Handheld photography, daylight | 1/125s – 1/500s |
| Interior, tripod, low light | 1/15s – 2s |
| Light trails / long exposure | 5s – 30s |

**In Unreal Engine 5.** The Post Process Volume exposes shutter speed as **Shutter Speed (1/s)** under the Camera section — the value is entered as the *denominator* of the fraction, not the duration itself. Entering `120` means 1/120s. This value contributes to the EV100 calculation and controls **motion blur intensity**: the longer the shutter (lower 1/s value), the more motion blur is applied to moving objects. A physically accurate interior with a stationary camera typically uses a shutter speed between 1/30s and 1/125s depending on the lighting level.

---

### 3.3 ISO (Sensor Sensitivity)

**Physical definition.** ISO (formerly ASA in American standards; standardised by the International Organization for Standardization) is the measure of a camera sensor's **sensitivity to light**. In film photography it corresponded to the chemical sensitivity of the emulsion. In digital photography it is an amplification factor applied to the electrical signal from the sensor after exposure.

ISO follows a linear scale: ISO 200 is twice as sensitive (one stop brighter) as ISO 100. ISO 800 is eight times as sensitive (three stops brighter).

**ISO has one significant cost: noise.** Amplifying the sensor signal also amplifies the random electrical noise inherent in the sensor. At low ISO values (100–400) this noise is negligible. At high values (3200+) it manifests as visible grain — random luminance and chrominance variation across the frame. The usable range of a camera is determined by its sensor size and engineering quality.

The standard ISO sequence: **100, 200, 400, 800, 1600, 3200, 6400**. Each step is one stop of additional sensitivity.

Typical real-world ISO values by context:

| Context | Typical ISO |
|:---|:---|
| Bright daylight exterior | 100 |
| Overcast outdoor | 200–400 |
| Indoor, good artificial light | 400–800 |
| Indoor, dim / candlelit | 800–3200 |
| Night, handheld | 3200–12800 |

**In Unreal Engine 5.** ISO appears under the Camera section of the Post Process Volume. In UE5, ISO does not produce visual noise directly — the engine does not simulate sensor grain from ISO amplification. ISO functions purely as a multiplier in the EV100 calculation. Film grain, if desired, must be added separately through **Film Grain Intensity** in the Post Process Volume, independently of ISO. The practical use of ISO in UE5 is to set the sensitivity baseline that, combined with aperture and shutter speed, produces the correct EV100 for the scene's designed luminance range.

---

### 3.4 Focal Length and Focal Distance

These are two distinct parameters that are frequently confused because both use the word "focal."

---

**Focal Length**

**Physical definition.** Focal length is the distance in millimetres between the optical centre of a lens and the camera sensor when the lens is focused at infinity. It is a fixed optical property of the lens.

Focal length determines the **field of view** and **perspective compression** of the image:

- **Short focal lengths (wide angle, 14–35mm)** capture a wider field of view and produce noticeable perspective distortion — objects close to the camera appear disproportionately large relative to distant objects. This exaggerates perceived depth and spatial scale. Used in architecture photography to capture full rooms, and in game cameras to emphasise environmental scale.
- **Standard focal lengths (~50mm)** approximate the field of view and perspective of the human eye. They produce minimal distortion and feel naturalistic. The conventional choice for documentary, portrait, and general-purpose work.
- **Long focal lengths (telephoto, 85–400mm)** capture a narrower field of view and compress perspective — objects at different distances appear closer in scale to each other than they are in reality. This produces a flattened, painterly quality. Used in portraiture to isolate subjects, and in establishing shots to collapse the apparent distance between a foreground subject and a background landmark.

| Focal length | Range | Typical use |
|:---|:---|:---|
| Ultra-wide | 8–24mm | Architecture, environment, VR |
| Wide | 24–35mm | Interior, landscape, photojournalism |
| Standard | 45–55mm | Street, documentary, general |
| Short telephoto | 85–105mm | Portrait, detail work |
| Telephoto | 135–300mm | Sport, wildlife, compression effects |
| Super-telephoto | 400mm+ | Wildlife, astronomy |

**In Unreal Engine 5.** The **CineCameraActor** exposes focal length directly as **Current Focal Length (mm)**. The default is 35mm — slightly wide, appropriate for third-person games. A first-person interior walkthrough typically uses 60–80mm to approximate natural human peripheral vision without distortion. The standard Camera Actor uses **Field of View** (degrees) rather than focal length. For a full-frame sensor (36mm width), the conversion is approximately:

$$\text{Horizontal FOV} = 2 \times \arctan\!\left(\frac{18}{\text{focal length (mm)}}\right)$$

---

**Focal Distance (Focus Distance)**

**Physical definition.** Focal distance is the distance from the camera to the subject that is in **sharpest focus** — the plane of focus. It is not a property of the lens; it is set by the photographer (or autofocus system) for each shot, by adjusting the lens elements.

Objects at the focal distance are rendered at maximum sharpness. Objects closer or further away are rendered with increasing blur, with the blur radius determined jointly by the distance from the focal plane and the size of the aperture. The range of distances that appear acceptably sharp on either side of the focal plane is the **depth of field** — a property jointly determined by focal length, aperture, and focal distance.

**In Unreal Engine 5.** The CineCameraActor exposes **Current Focal Distance** in Unreal units (1 unit = 1cm by default). In the Post Process Volume, depth of field is configured through the **Depth of Field** section, which uses this distance to compute the near and far focus planes. UE5 supports two DOF implementations:

- **Cinematic DOF** — physically based, lens-accurate bokeh discs, correct circle of confusion falloff. Expensive. Use with awareness of its PostProcessing pass cost, especially at low f-stop values where the circle of confusion is large.
- **Gaussian DOF** — non-physical, cheap, soft-edged approximation. Acceptable for distant background softening but not for close-range bokeh effects.

---

### 3.5 Exposure Value (EV100)

**Physical definition.** Exposure Value (EV) is a single number that encodes a combination of aperture *N* and shutter speed *t* that produces a defined exposure on a sensor of standard sensitivity (ISO 100). It is defined as:

$$\text{EV} = \log_2\!\left(\frac{N^2}{t}\right)$$

where *N* is the f-number (e.g., *N* = 2.8 for f/2.8) and *t* is the shutter speed in seconds (e.g., *t* = 0.008 for 1/125s). This definition assumes ISO 100.

To generalise across any ISO value *S*, the formula is scaled by the sensor sensitivity relative to the standard:

$$\text{EV}_{100} = \log_2\!\left(\frac{N^2}{t}\right) - \log_2\!\left(\frac{S}{100}\right) = \log_2\!\left(\frac{100 \cdot N^2}{S \cdot t}\right)$$

The two forms are algebraically equivalent. The second form is more convenient when working with non-standard ISO values, since it folds all three exposure triangle parameters into one expression.

**Reading the formula.** EV is logarithmic — each integer step represents a one-stop change, a doubling or halving of exposure. The formula makes explicit that exposure depends on the *ratio* of aperture area (proportional to *N*²) to exposure duration (*t*). An infinite number of (N, t) combinations produce the same EV. These are **equivalent exposures** — identical brightness, but different depth of field and motion blur:

| Aperture (*N*) | Shutter speed (*t*) | EV100 |
|:---|:---|---:|
| f/1.4 | 1/60s | 3 |
| f/2.0 | 1/30s | 3 |
| f/2.8 | 1/15s | 3 |
| f/4.0 | 1/8s | 3 |
| f/8.0 | 1/2s | 3 |

**Worked example.** A photographer shoots an interior scene at f/2.8, 1/60s, ISO 400:

$$\text{EV}_{100} = \log_2\!\left(\frac{100 \times 2.8^2}{400 \times \tfrac{1}{60}}\right) = \log_2\!\left(\frac{100 \times 7.84}{6.67}\right) = \log_2(117.5) \approx 6.9$$

An EV100 of approximately 7 corresponds to a dim, warm interior — consistent with a domestic living room in the evening, without photographic flash or supplementary lighting. This is the correct exposure baseline for a candlelit escape room or a torch-lit dungeon interior.

**EV100 reference scale for common scene types:**

| Scene | EV100 (approx.) |
|:---|---:|
| Moonless night sky | −6 |
| Moonlit exterior | 0 |
| Candlelit interior | 2–3 |
| Domestic living room (ambient) | 5–7 |
| Office with fluorescent lighting | 8–10 |
| Overcast outdoor daylight | 12 |
| Full sun on a clear day | 15–16 |
| Snow scene in direct sun | 17–18 |

**In Unreal Engine 5.** EV100 is the primary exposure currency of UE5's physically based camera and appears in three places in the Post Process Volume:

1. **Auto Exposure → Min/Max Brightness** — these parameters clamp the range of EV100 values within which the Auto Exposure system will adapt. Setting Min = 6 and Max = 10 constrains the camera to the exposure range of a typical well-lit interior, preventing it from brightening aggressively in dark corners or blowing out near windows.

2. **Auto Exposure → Exposure Compensation** — an EV100 offset applied on top of the measured scene exposure, equivalent to the +/− dial on a physical camera. A value of +1 brightens the image by one stop; −1 darkens it by one stop.

3. **Manual exposure mode** — when Auto Exposure Method is set to **Manual**, the engine uses the aperture, shutter speed, and ISO values in the Camera section directly to compute EV100 and fix the exposure at that value for the entire scene. This is the correct mode for pre-authored, non-adaptive lighting in a designed interior where you have deliberately calibrated the brightness of each zone.

> **Production recommendation.** For a designed interior environment with deliberate lighting — an escape room, a narrative investigation space, a tavern — disable Auto Exposure or tightly clamp its Min/Max range. Auto Exposure is designed for scenes with large luminance variation (an exterior moving from deep shade into direct sun). In a designed interior, it will fight your lighting decisions: the intentional brightness contrast between a foyer and a service corridor is a design choice, and Auto Exposure will reduce it as the player approaches the darker zone. Use Manual mode or a narrow EV100 clamp range to preserve that contrast.

---

### 3.6 The Exposure Triangle: Summary and UE5 Implications

The three parameters — aperture (*N*), shutter speed (*t*), and ISO (*S*) — are collectively called the **exposure triangle**. Each controls scene brightness and one independent secondary quality:

| Parameter | Exposure effect | Secondary effect in UE5 |
|:---|:---|:---|
| Aperture (*N*) | ↑ N = less light | Depth of field: ↑ N = deeper focus, smaller bokeh discs |
| Shutter speed (*t*) | ↑ t = more light | Motion blur: ↑ t = more blur on moving objects |
| ISO (*S*) | ↑ S = more sensitivity | Grain: ↑ S = more noise (must be added manually via Film Grain) |

For an interior environment specifically:

- Use a **moderate aperture** (f/2.8–f/5.6) to produce shallow-to-moderate depth of field — sharp within the interaction zone, softening toward distant geometry. This directs attention and reduces the visual complexity of background surfaces that are not the focus of play.
- Use a **shutter speed consistent with the 180-degree rule** for your target frame rate. At 30fps, use 1/60s. At 60fps, use 1/120s. Faster shutters produce uncomfortably sharp motion on player locomotion; slower shutters produce excessive blur.
- Set **ISO to the lowest value** that achieves your target EV100 with your chosen aperture and shutter speed. In UE5, higher ISO carries no noise benefit over adjusting the other parameters, so there is no reason to increase it beyond what the EV100 calculation requires.

---

## 4. Geometry and Draw Call Optimisation

### 4.1 Nanite for Interior Architecture

Nanite is UE5's virtualised geometry system. It replaces manual LOD chains for Static Meshes, automatically streaming only the triangle detail visible at the current camera distance and angle. For interior architecture — decorative mouldings, balustrade detail, column capitals, coffered ceiling panels — Nanite is the single most significant optimisation available. You can import and place high-poly architectural meshes from the Fab marketplace or your own modelling pipeline without manually managing LODs.

Enable Nanite on any Static Mesh whose polygon count exceeds approximately 50,000 triangles. Right-click the asset in the Content Browser and select **Nanite → Enable**, or check the Nanite settings in the Static Mesh editor.

Nanite is **not** appropriate for:

- Skeletal meshes (character assets, animated props)
- Meshes with translucent or masked materials (glass panels, chain-link, foliage) — Nanite does not support non-opaque materials
- Very small props (sub-30cm items in the scene) where the Nanite overhead per-draw exceeds the cost of a simple low-poly mesh
- Meshes with World Position Offset in their material — WPO disables Nanite's rasterisation path

> **Production check.** After enabling Nanite on your key architectural assets, switch the viewport to the **Nanite Triangles** visualisation mode (View Mode dropdown). Any mesh displaying in pink/magenta is being rendered via Nanite's virtualised path. Any mesh in dark grey is falling back to standard rasterisation. Identify the fallback meshes and investigate whether Nanite can be enabled on them.

---

### 4.2 Instanced Static Meshes

Modular interior construction generates a large number of identical assets — wall panels, floor tiles, balustrade sections, window frames. Each placed instance of a mesh with a unique position/rotation/scale is processed as a separate draw call by default. In a large interior environment this can produce several thousand draw calls from geometry that is visually and materially identical.

**Hierarchical Instanced Static Mesh (HISM)** components collapse multiple instances of the same mesh and material into a single draw call, regardless of instance count. In an interior environment built from modular assets this is typically the second or third largest draw call reduction available, after Nanite.

UE5 applies auto-instancing for Static Mesh Actors sharing the same mesh and material. Verify it is working by checking `stat SceneRendering` before and after manually grouping a set of modular wall tiles. If the draw call count drops, auto-instancing is active. If it does not, check whether material instances are diverging (different parameter values on the same master material will prevent batching).

---

### 4.3 Occlusion and Sightline Management

UE5's hardware occlusion culling discards geometry that is behind an opaque surface from the player's current view. In interior environments, this system is only effective if your spatial design **actually creates occlusion boundaries** — walls that fully enclose volumes, doorframes with physical depth that block the view into the next room before the player crosses the threshold.

An open-plan foyer with long unobstructed sightlines into three adjacent corridors forces the renderer to process every piece of geometry in all three corridors even when they are only partially visible. The same foyer designed so that each corridor exit is framed by a doorframe with a 40–60cm reveal — short enough to feel architecturally correct, long enough to generate a meaningful occlusion event — reduces the renderer's workload significantly from the primary standpoint position.

Design sightlines deliberately in the greybox. Do not treat occlusion as a post-hoc optimisation. Corridors that are architecturally correct (wall reveals, door recess depths) are also spatially correct — they create the threshold experience players expect from a grand interior.

<!-- Search: Unreal Engine 5 occlusion culling visualisation mode screenshot | Source: Epic Games documentation or UE5 editor, Vis Buffer or Frozen Occluders view -->

*The Frozen Occluders visualisation mode (Show menu in viewport) reveals which meshes are being culled from the current camera position. Green meshes are visible; red meshes have been culled. In a well-designed interior with proper doorframe reveals, the majority of the geometry behind each doorway should read as culled from the primary standpoint.*

---

## 5. Materials and Shader Cost

### 5.1 Master Material Architecture

Every unique material permutation in your scene generates a separate shader compile event and occupies a distinct slot in the GPU's material batch. A common student error is creating many separate material instances that appear identical in the viewport but are technically distinct because they reference different parent materials or carry divergent parameter values. This prevents batching and inflates both compile time and runtime shader cost.

For an interior environment, maintain a **master material per surface category**: one master for stone/marble, one for painted plaster/wall, one for wood, one for metal. All variation within a category — different colours, roughness maps, normal maps — should be driven through **Material Parameter Collections** or **material instances** of the same parent. This ensures that the GPU's material batching system can consolidate all marble surfaces into a single batch regardless of their specific texture inputs.

---

### 5.2 Shader Complexity and Translucency

Open the viewport's **Shader Complexity** view mode to visualise per-pixel shader cost across the scene. The colour scale runs from green (cheap) through yellow and orange to red/white (expensive). Any surface in orange or red in your primary viewing positions is a candidate for material simplification.

The two most common shader complexity hotspots in interior environments are:

- **Reflective floor surfaces.** Polished marble floors with a Lumen reflection request, a normal map, a roughness map, and a parallax height map are common and expensive. Consider whether parallax is perceptible at the player's typical viewing angle on a flat floor; at near-oblique angles the answer is usually no, and the instruction can be removed from the material without visible change.
- **Translucent glass.** Glass panels on chandeliers, mezzanine railings, and windows carry overdraw cost proportional to the area of transparent surface visible from the camera. Each transparent layer requires the renderer to process everything behind it. A chandelier with twelve individually transparent glass pendants in the player's primary view is significantly more expensive than a chandelier where the pendants use a masked or dithered opacity approach.

> **View mode discipline.** Check Shader Complexity and Quad Overdraw at the end of every production week, from the player's primary standpoint positions — not from an elevated editor camera angle. The cost the engine pays is the cost from the camera the player uses, not from the camera you use to dress the scene.

---

## 6. Profiling Workflow

Profiling must be treated as a scheduled production activity, not a reactive response to a performance problem. A frame budget exceeded in week eight is a production emergency. The same budget exceeded in week three is a greybox note.

### 6.1 Console Commands: The Minimum Set

The following commands are the minimum profiling vocabulary for an interior environment. All are entered in the console (~) during Play-In-Editor.

| Command / Shortcut | Pass target | What to look for |
|:---|:---|:---|
| `stat fps` | ≥ 60fps | Baseline frame rate from the player camera position |
| `stat unit` | < 16.67ms total | CPU Game / Draw thread vs GPU thread split — identifies the bottleneck side |
| `stat scenerendering` | < 2000 draw calls | Total draw primitive count and visible mesh element count |
| `profilegpu` | Per-pass budget | Full single-frame GPU breakdown — Lighting, Shadow, Lumen GI, BasePass |
| `stat LightRendering` | < 3ms | Cost of the lighting pass; shadow-casting light count |
| `stat ShadowRendering` | < 4ms | Cost of shadow maps; identifies over-budget shadow-casting lights |

For sustained performance analysis across a full player traversal of the foyer — not just a single frame — use **Unreal Insights** (**Tools → Run Unreal Insights** in the editor). Unreal Insights captures CPU and GPU timeline data across an entire play session and identifies hitches, streaming spikes, and shadow recalculation events that single-frame profiling misses.

---

### 6.2 GPU Visualiser: Reading a Frame

The GPU Visualiser (**Ctrl + Shift + ,**) provides a per-pass breakdown of a single rendered frame. For an interior environment the passes to examine first are:

- **Lumen Diffuse Indirect** — if this exceeds 5–6ms, reduce Final Gather Quality in the Post Process Volume and cap Max Trace Distance.
- **ShadowDepths + VSMProject** — if these exceed 4–5ms combined, audit shadow-casting lights and disable Cast Shadows on any non-hero fixture.
- **BasePass** — if this exceeds 4ms, check Shader Complexity for expensive materials and look for overdraw in glass and translucency.
- **PostProcessing** — if this exceeds 2ms, audit the Post Process Volume for enabled features (AO, Bloom quality, motion blur) that may not be perceptible at your target resolution.

Optimise passes in order of cost, largest first. Making a 0.3ms saving in a 1ms pass while ignoring an 8ms Lumen pass is not productive optimisation.

---

## 7. Level Streaming for Multi-Room Interiors

A large interior environment — a grand foyer with a mezzanine, radiating into corridors and subsidiary rooms — should not be loaded as a single persistent level. Every piece of geometry and every light in a single-level interior is processed regardless of whether it is in the player's current view frustum, because UE5's culling and streaming systems operate at the sub-level asset level, not at the level-file level.

### 7.1 Level Streaming Architecture

The correct architecture for a large interior is a **persistent level** containing only the foyer geometry, lighting, and game logic, with **sub-levels** for each major subsidiary space — north corridor, south corridor, mezzanine upper, library wing — that are streamed in and out as the player approaches and passes through their threshold.

In the World Outliner, create each sub-level via **Levels → Create New Level**, move the relevant actors into it, then add a **Level Streaming Volume** in the persistent level that triggers load and unload. The streaming volume should be placed approximately three to four metres before the doorway threshold — early enough that geometry is resident and rendered before the player can see through the door, late enough that you are not holding multiple sub-levels in memory simultaneously.

For a UE5.4+ project, **World Partition** with streaming cells replaces manual Level Streaming as the preferred architecture for large environments. If your interior is part of a World Partition map, ensure each major room is enclosed within its own streaming cell boundary and that cell size is set to match room scale (approximately 2,000–4,000 units per cell for a large foyer).

<!-- Search: Unreal Engine 5 Level Streaming Volume setup interior level screenshot | Source: Epic Games documentation, UE5 editor Levels panel -->

*Level Streaming Volumes define the spatial trigger for loading and unloading a sub-level. For an interior corridor, the volume should extend from the doorframe threshold to a point several metres into the foyer — covering the full approach path so that the streamed level is always resident before the player has line of sight into the corridor.*

---

### 7.2 Loading and Unloading Discipline

The most common Level Streaming error is setting both load and unload volumes to the same shape, which creates a condition where the sub-level unloads the moment the player steps back across the threshold. A player who briefly retreats from a doorway should not see geometry pop out. The unload trigger should be set several metres further back into the foyer than the load trigger — creating a hysteresis zone in which the sub-level remains loaded even if the player moves slightly away from the threshold.

Test streaming behaviour by enabling **Level Streaming debugging** in the World Outliner Levels panel. Loaded levels are shown in green; levels pending load are yellow; unloaded levels are grey. Walk the full primary path and confirm that no level is loading visibly in front of the player and no level is unloading while geometry from it is still in the player's view frustum.

---

## 8. Quick Reference: Production Checklist

Use this checklist at each major production milestone. It is a profiling discipline tool, not a submission rubric. A failing check is a production task, not an artistic choice.

| Check | Diagnostic question |
|:---|:---|
| **Lumen configured** | Is Max Trace Distance capped in the Post Process Volume? Is Final Gather Quality below 1.0? |
| **VSM shadow audit** | Are Cast Shadows disabled on all non-hero secondary lights? |
| **Light mobility** | Is every Movable light connected to a gameplay state change that requires it to be Movable? |
| **Camera exposure** | Is Auto Exposure disabled or tightly clamped? Have EV100 Min/Max been set for the designed lighting range? |
| **Depth of field** | Is aperture set to produce the intended focus depth? Is Cinematic DOF cost within the PostProcessing budget? |
| **Nanite enabled** | Are all high-poly architectural meshes (>50k triangles) using Nanite? Have fallbacks been checked? |
| **Instanced geometry** | Are modular wall tiles and repeated props using HISM or confirmed as auto-instanced? |
| **Occlusion boundaries** | Do all doorframes and corridor entrances have physical reveal depth that generates occlusion events? |
| **Shader complexity** | Has Shader Complexity view been checked from primary player standpoints? Are any passes in orange/red? |
| **Draw call budget** | Does `stat scenerendering` report fewer than 2000 draw primitives from the foyer standpoint? |
| **GPU Visualiser checked** | Have Lumen GI, ShadowDepths, BasePass, and PostProcessing pass costs been recorded? |
| **Frame rate target** | Is the level sustaining 60fps from the primary player standpoints under Play-In-Editor? |
| **Streaming configured** | Are sub-levels or World Partition cells set up for all major subsidiary spaces? |
| **Streaming tested** | Has the streaming hysteresis zone been tested? No pop-in or pop-out on the primary path? |

---

## 9. Reflective Questions

These questions are for individual reflection after each profiling session. Answer them with reference to your current profiled state, not your intended final state.

1. **Frame budget.** Run `stat unit` from your primary player standpoint. What is your current GPU frame time? What is the single largest pass in the GPU Visualiser? What is the one change most likely to reduce it?

2. **Lumen cost.** What is the current cost of the Lumen Diffuse Indirect pass? Have you capped Max Trace Distance to your largest room size? What does reducing Final Gather Quality from 1.0 to 0.5 do to both visual quality and frame time in your specific scene?

3. **Shadow audit.** List every light in your scene with Cast Shadows enabled. For each one, identify whether it is a hero fixture, a secondary fixture, or a fill light. Which shadow-casting lights are secondary or fill? What happens to ShadowDepths pass cost when their shadows are disabled?

4. **Draw calls.** What is your current draw primitive count from the foyer standpoint? Which asset type contributes the most — modular wall geometry, props, or lights? If draw calls exceed 2000, is it because of a large number of unique mesh/material combinations, or because instancing is not working?

5. **Occlusion effectiveness.** Using the Frozen Occluders view mode, stand at your primary foyer standpoint and take a screenshot. What percentage of the level geometry is culled? If the answer is less than 50%, what sightlines are keeping geometry visible that a player from this position has no need to see?

6. **Camera exposure.** Calculate the EV100 for your current Post Process Volume aperture, shutter speed, and ISO settings using the formula EV₁₀₀ = log₂(100·N²/S·t). Does that value match the intended luminance range for the scene? If Auto Exposure is active, is it erasing intentional brightness contrast between zones?

7. **Streaming and pop-in.** Walk the primary navigation path through your environment and watch the Levels panel. At what point does each sub-level become visible? Is there any geometry that appears to pop into view while the player is already looking at it? What adjustment to the streaming volume position would eliminate it?

8. **Production honesty.** What is the one optimisation in this checklist you have not yet applied? What is the actual reason — time constraint, technical difficulty, or expectation that it will not be noticed in a submission build? What is the frame time cost of not applying it?

---

## Appendix: Photometric and Rendering Glossary

| Term | Unit | Definition |
|:---|:---|:---|
| **Candela** | cd | SI base unit of luminous intensity. Measures the directional brightness of a light source — how much light is emitted per unit of solid angle in a specific direction. One candela ≈ the output of a single candle in a given direction. Used in UE5 for Point and Spot Light intensity when modelling specific real-world fixtures from photometric datasheets. |
| **Lumen** | lm | SI unit of luminous flux. Measures the total light output of a source integrated across all directions. 1 lm = 1 cd·sr. A 60W incandescent bulb produces ~800 lm. The default intensity unit for Point, Spot, and Rect Lights in UE5. Distinct from UE5's *Lumen* GI system, which shares the name. |
| **Nit** | cd/m² | Informal name for candela per square metre, the SI unit of luminance. Measures how bright a surface appears to an observer — the light arriving at the eye from a unit of surface area. Used in UE5 for emissive material intensity and as the basis for the Auto Exposure (EV100) system in the Post Process Volume. |
| **Lux** | lx | SI unit of illuminance. Measures the luminous flux arriving at a surface per unit area: 1 lx = 1 lm/m². Governs how lit a surface is, independent of what it reflects. Follows the inverse-square law with distance from the source. Used in UE5 for Directional Light (sun/sky) intensity — a physically accurate noon sun is ~100,000 lx. |
| **Aperture (f-number)** | f/N | The ratio of lens focal length to aperture diameter. Controls the amount of light entering the camera and the depth of field. Lower f-number = wider opening = more light = shallower DOF. In UE5, set in the Camera section of the Post Process Volume; affects DOF and bokeh shape. |
| **Shutter speed** | 1/t (s) | The duration for which the sensor is exposed to light, expressed as a fraction of a second. Longer shutter = more light = more motion blur. In UE5, entered as the denominator (e.g., 120 for 1/120s) in the Camera section; drives motion blur intensity. |
| **ISO** | S | A linear measure of sensor sensitivity. Doubling ISO = one stop brighter; also increases noise in real photography. In UE5, ISO functions purely as a multiplier in the EV100 calculation — film grain must be added separately via Film Grain Intensity. |
| **Focal length** | mm | Distance from lens optical centre to sensor when focused at infinity. Determines field of view and perspective compression. Short focal length = wide FOV and exaggerated perspective. Long focal length = narrow FOV and compressed perspective. In UE5, set on the CineCameraActor; standard Camera Actors use Field of View angle instead. |
| **Focal distance** | cm (UE units) | Distance from the camera to the plane of sharpest focus. Objects closer or further than this distance are blurred, with blur radius determined by the aperture size. In UE5, set as Current Focal Distance on the CineCameraActor; drives Cinematic DOF calculations in the Post Process Volume. |
| **EV100** | — | Exposure Value at ISO 100. Defined as log₂(N²/t) at ISO 100, or log₂(100·N²/S·t) for arbitrary ISO S. A logarithmic scale — each +1 EV step doubles the light required for the same exposure. Used throughout UE5's Post Process Volume for Auto Exposure range clamping, Exposure Compensation, and Manual exposure mode. |
| **Equivalent exposure** | — | Any combination of aperture, shutter speed, and ISO that produces the same EV100 — and therefore the same image brightness — but different depth of field, motion blur, and noise. Choosing between equivalent exposures is the primary creative decision of the exposure triangle. |
| **Depth of field** | — | The range of distances from the camera that appear acceptably sharp. Determined jointly by aperture, focal length, and focal distance. Wider aperture = shallower DOF. In UE5, computed by Cinematic DOF using the aperture value from the Post Process Volume. |
| **Circle of confusion** | mm | The disc of light projected onto the sensor by an out-of-focus point source. Its diameter determines the perceived sharpness of a point at a given distance from the focal plane. When the circle of confusion exceeds the sensor's resolving capability (or a defined perceptual threshold), the point appears blurred. The size of the circle of confusion increases with aperture width and with distance from the focal plane. |
| **Albedo** | — | The proportion of incident light a surface reflects, expressed as a value between 0 (perfect absorber) and 1 (perfect reflector). In UE5's physically based materials, the Base Color input defines albedo. Values above 0.9 or below 0.02 are physically implausible and will produce incorrect Lumen GI. |
| **IES Profile** | — | Photometric data file specifying the angular light distribution of a real-world fixture. Assigned to a Point or Spot Light in UE5 to replicate the precise falloff and directional pattern of the physical fixture. No runtime cost. Available from fixture manufacturers and free online libraries. |
| **VSM** | — | Virtual Shadow Map. UE5's high-resolution shadow technique, replacing traditional cascaded shadow maps. Provides sub-pixel contact shadows at low memory cost. Cost scales with shadow-casting light count and scene geometric complexity within each light's radius. |
| **Lumen (GI system)** | — | Unreal Engine 5's fully dynamic global illumination and reflection system. Uses software and optional hardware ray tracing to resolve indirect lighting, colour bleed, and reflections in real time without pre-baked lightmaps. Shares its name with the photometric unit (lm) but is a distinct concept. |
| **Nanite** | — | UE5's virtualised geometry system. Replaces manual LOD chains for Static Meshes by dynamically streaming only the triangle detail visible at the current camera position and scale. Does not support translucent or masked materials, skeletal meshes, or World Position Offset. |
