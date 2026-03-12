---
title: "Game Design Notes"
subtitle: "Ideation, Level Design Heuristics, Systems Thinking, and Spatial Readability"
author: "Niall McGuinness"
institute: "Dundalk Institute of Technology"
programme: "BSc (Hons) in Computing in Games Development"
module_code: "COMP I8015"
module_title: "3D Game Development"
stage: 4
version: "1.0"
generated_at: "2026-03-06"
format:
  html:
    toc: true
    toc-depth: 2
    number-sections: true
  pdf:
    toc: true
    number-sections: true
---
# Game Design Notes
## Ideation, Level Design, Systems Thinking, and Spatial Readability

These notes are designed as **standalone game design notes** for analysing, planning, and improving game experiences. They focus on transferable principles that apply across genres, team sizes, and production contexts.

They are written to support both **conceptual understanding** and **practical decision-making**: not just *what* a principle is, but *why it matters to the player*, *what good implementation looks like*, and *what common failure modes to watch for*.

---

## Part 1: Ideation and Assessing Game Ideas

Good ideas are rarely the result of inspiration alone. Strong game concepts usually emerge from **structured exploration**, **constraints**, and **iterative refinement**.

### 1.1 Creative Generation Techniques

#### Inspiration from Mundane Life
Many distinctive games are built from ordinary activities, environments, or social situations rather than from overused genre conventions. Looking at everyday routines (retail, logistics, maintenance, caregiving, commuting, paperwork, local rituals) can produce more original mechanics because the designer must invent systems rather than inherit them.

**Why it matters to the player:**  
Unusual source material often produces fresh interactions and memorable situations.

**Common failure mode:**  
Using an unusual theme but falling back on generic mechanics that do not express that theme.

**What good implementation looks like:**  
The player’s actions feel inseparable from the premise.

**Design question:**  
What does the player *do* in this concept that they do not commonly do in other games?

---

#### Non-Game Interactions as Design Seeds
Interfaces, tools, and physical objects can inspire mechanics. A game can borrow interaction patterns from forms, apps, devices, or manual processes and reinterpret them as play systems.

**Why it matters to the player:**  
Borrowed interaction logic can make systems intuitive because players already understand the pattern.

**Common failure mode:**  
Copying the surface style of a real interface without capturing what makes it meaningful or interactive.

**What good implementation looks like:**  
The player quickly understands the interaction model, but the game adds tension, stakes, or interpretation.

**Design question:**  
Which real-world interaction pattern is this design borrowing, and what does play add to it?

---

#### Designing for Memorable Moments
A useful ideation strategy is to imagine a player later describing a moment from the game to someone else. This shifts focus from abstract features toward **lived experience**.

**Why it matters to the player:**  
Players often remember events, reversals, discoveries, and consequences more than feature lists.

**Common failure mode:**  
Designing around systems that are technically complex but produce no emotionally meaningful moments.

**What good implementation looks like:**  
The design creates opportunities for surprise, relief, mastery, panic, cleverness, or social retelling.

**Design question:**  
What is one moment in this game that a player might describe enthusiastically afterward?

---

#### “Find the Fun” Through Retheming
A mechanic may become stronger when placed in a different setting or aesthetic frame. Retheming can reveal the mechanic’s real identity.

**Why it matters to the player:**  
A mechanic and theme that fit each other improve clarity, tone, and emotional impact.

**Common failure mode:**  
Becoming attached to a theme before verifying whether it supports the gameplay.

**What good implementation looks like:**  
The theme reinforces what the player is already doing, rather than fighting it.

**Design question:**  
If the current theme were removed, what kind of experience does the mechanic naturally want to become?

---

### 1.2 Assessment and Refinement Frameworks

#### Skills Audit and Constraints
Assess the available skills, tools, time, and production capacity. Constraints are not only limitations; they are design-shaping forces that help define scope and originality.

**Why it matters to the player:**  
A smaller, coherent game usually provides a stronger experience than an over-scoped one with unfinished systems.

**Common failure mode:**  
Planning content or systems that cannot be built, tested, or polished within the available capacity.

**What good implementation looks like:**  
The concept is aligned with what can actually be executed well.

**Design question:**  
What hard constraints (time, art skill, code complexity, testing access, hardware) should shape this design from the start?

---

#### The Single-Sentence Pitch
A workable game idea should be describable in one sentence that includes:
- **who/what the player is**
- **what they are trying to achieve**
- **what they do repeatedly to achieve it**

**Why it matters to the player:**  
Clarity at the concept level often improves clarity in mechanics, onboarding, and progression.

**Common failure mode:**  
A pitch that describes worldbuilding, story premise, or aesthetic only, but not player action.

**What good implementation looks like:**  
The core loop is visible in the sentence.

**Design question:**  
Does the sentence describe actions and goals, or only setting and mood?

---

#### Fixing Genre Problems
A productive ideation method is to identify a frustration in an existing genre and design around that pain point.

**Why it matters to the player:**  
This orients the concept toward a real player need rather than novelty for its own sake.

**Common failure mode:**  
“Fixing” a genre by removing the very tension that makes it compelling.

**What good implementation looks like:**  
The redesign solves a friction point while preserving the genre’s emotional core.

**Design question:**  
What problem in a familiar genre is being addressed, and what must be preserved so the genre still feels like itself?

---

#### SCAMPER for Iteration
SCAMPER is a structured ideation checklist:
- **Substitute**
- **Combine**
- **Adapt**
- **Modify / Magnify / Minify**
- **Put to another use**
- **Eliminate**
- **Rearrange / Reverse**

**Why it matters to the player:**  
It helps designers move beyond the first version of an idea.

**Common failure mode:**  
Using SCAMPER to generate novelty without checking whether the result improves player experience.

**What good implementation looks like:**  
Each variation changes how the player thinks, acts, or feels.

**Design question:**  
Which SCAMPER variation most meaningfully changes the player’s decisions, not just the game’s appearance?

---

## Part 2: Level Design Principles and Workflow

Level design is the discipline of shaping **space, pacing, choice, and readability** so the player can act meaningfully inside the game.

### 2.1 Workflow and Process

#### Grayboxing First
Grayboxing separates **spatial function and gameplay readability** from final art and polish. It enables rapid testing of flow, sightlines, scale, timing, and interaction before aesthetic commitments make change expensive.

**Why it matters to the player:**  
Readable, well-paced spaces are usually the result of iteration, not decoration.

**Common failure mode:**  
Polishing environment art before verifying whether the space plays well.

**What good implementation looks like:**  
The space is understandable and playable in simple forms before art pass.

**Design question:**  
If all art were removed, would the intended path, challenge, and interaction opportunities still read clearly?

---

#### Motivation and Task Balance
A practical workflow often mixes “necessary but less exciting” tasks with “motivating” tasks to maintain momentum.

**Why it matters to the designer (and indirectly the player):**  
Sustained momentum improves iteration quality and reduces stalled production.

**Common failure mode:**  
Long stretches of invisible technical work with no visible progress, or constant polish work with no functional progress.

**What good implementation looks like:**  
The workflow alternates between structural progress and morale-sustaining wins.

**Design question:**  
What small visible milestone can be paired with the next difficult technical task?

---

#### Beautiful Corners (Micro-Vertical Slice)
A “beautiful corner” is a small, polished area used to test the game’s visual language, lighting approach, material style, and environmental tone while the rest remains in graybox.

**Why it matters to the player:**  
Visual cohesion supports mood, legibility, and trust in the world.

**Common failure mode:**  
Polishing a showcase area so heavily that it becomes unrepresentative of the rest of the project.

**What good implementation looks like:**  
The polished sample informs style rules for broader production rather than becoming a one-off.

**Design question:**  
What visual rules does this polished slice establish that can scale across the project?

---

#### Build Introductory Spaces Late
The first area (or earliest player-facing content) is often best designed later, once the mechanics, pacing, and failure patterns are understood.

**Why it matters to the player:**  
Good onboarding depends on understanding what players actually struggle with.

**Common failure mode:**  
Designing onboarding before the systems stabilize, resulting in tutorials that teach the wrong things.

**What good implementation looks like:**  
The introductory space is designed around observed player needs and core mechanics.

**Design question:**  
What does a new player need to understand first, and what evidence supports that?

---

### 2.2 Strategic Design and Internal Rules

#### Top-Level Content Planning
Plan the large-scale structure of the experience:
- progression arc
- feature introduction order
- pacing variation
- intensity changes
- environmental contrast
- difficulty rhythm

**Why it matters to the player:**  
A strong macro-structure prevents monotony and supports learning.

**Common failure mode:**  
Designing spaces in isolation with no clear progression logic.

**What good implementation looks like:**  
Each section has a clear role in the overall experience.

**Design question:**  
What is this space teaching, testing, rewarding, or changing in the player’s understanding?

---

#### Internal Rules for Consistency
Projects benefit from self-imposed design rules for geometry, traversal cues, material transitions, interaction signifiers, and visual hierarchy.

**Why it matters to the player:**  
Consistency helps players learn the game’s language and predict what matters.

**Common failure mode:**  
Inconsistent signposting or visual semantics (e.g., objects that look interactive but are not, or important paths that visually blend into the background).

**What good implementation looks like:**  
The environment communicates in a stable visual language.

**Design question:**  
What recurring visual patterns teach the player what is traversable, interactable, dangerous, or decorative?

---

#### Telegraphing and Visibility
Telegraphing means communicating important spatial or interaction information in advance using shape, contrast, motion, framing, lighting, sound, or placement.

**Why it matters to the player:**  
Telegraphing reduces confusion while preserving challenge.

**Common failure mode:**  
Hiding critical information and mistaking obscurity for difficulty.

**What good implementation looks like:**  
The player notices key opportunities before they need them, even if they do not fully understand them yet.

**Design question:**  
What can the player see, hear, or infer *before* making the next important decision?

---

## Part 3: Mechanics, Rules, Systems, and Player Experience

Games are not only collections of features; they are **systems of rules** that generate player behaviour and emotional outcomes.

### 3.1 Core Concepts

#### Rules
Rules define what happens under specific conditions (“if X, then Y”). In digital games, many rules are implemented in code and may be invisible to players.

**Why it matters to the player:**  
Consistent rules make the world feel learnable and fair.

**Common failure mode:**  
Hidden or inconsistent rule behaviour that feels arbitrary.

**What good implementation looks like:**  
Players can form accurate mental models through play.

**Design question:**  
Can players learn the rule through feedback, or does it remain opaque?

---

#### Mechanics
Mechanics are the player’s actionable verbs and interactions (move, inspect, combine, shoot, trade, persuade, build, dodge, assign, etc.).

**Why it matters to the player:**  
Mechanics define how the player participates in the system.

**Common failure mode:**  
Adding many mechanics that do not meaningfully interact or support the intended experience.

**What good implementation looks like:**  
A small set of mechanics generates rich decisions through context and system interaction.

**Design question:**  
Which mechanics are truly central, and which are decorative or redundant?

---

#### Systems
Systems are interlocking rule structures that produce behaviour over time (economy, AI response, crafting, weather, suspicion, faction relations, power routing, wave spawning, etc.).

**Why it matters to the player:**  
Systems create depth, consequence, and emergent outcomes.

**Common failure mode:**  
Systems that exist in parallel but do not influence each other.

**What good implementation looks like:**  
Player actions in one system have visible consequences in another.

**Design question:**  
What changes elsewhere in the game when the player interacts with this system?

---

### 3.2 The Game as a Machine (MDA Perspective)

A useful design perspective is to view the game as a machine designed to produce an intended **aesthetic experience**.

- **Mechanics**: rules, actions, systems as implemented
- **Dynamics**: what happens when players use them in context
- **Aesthetics**: the emotional / experiential result (tension, mastery, curiosity, relief, dread, wonder, cleverness)

**Why it matters to the player:**  
Players do not experience the code directly; they experience the dynamics and feelings the code generates.

**Common failure mode:**  
Designing mechanics in isolation without testing whether they produce the intended aesthetic outcome.

**What good implementation looks like:**  
There is a plausible link between what the player does repeatedly and what they are meant to feel.

**Design question:**  
What aesthetic outcome is this design aiming for, and which mechanics are carrying that outcome?

---

### 3.3 Types of Mechanics (Useful Distinctions)

#### Functional Mechanics
Core interactions required for progression or success.

**Examples:** traversal, combat actions, item use, puzzle manipulation, resource allocation.

**Risk if neglected:**  
The game may be visually interesting but not playable or strategically meaningful.

---

#### Expressive Mechanics
Optional interactions that support character, tone, role-play, or playful curiosity.

**Examples:** inspecting props, petting animals, emotes, harmless object interactions, ambient interactions.

**Why they matter:**  
They add texture, humanity, and identity to the world.

**Risk if overused without structure:**  
The game can feel unfocused if expressive interactions overwhelm the core loop.

---

#### Cognitive Mechanics
Actions performed primarily in the player’s mind: deducing, remembering, predicting, prioritising, comparing, inferring, planning.

**Why they matter:**  
Many of the most satisfying moments in games come from cognition, not animation.

**Common failure mode:**  
Designing cognitive challenge without sufficient information, feedback, or clarity.

**What good implementation looks like:**  
The player has enough signals to think productively, not randomly guess.

**Design question:**  
What information is available to support the thinking this design is asking the player to do?

---

## Part 4: Navigation, Wayfinding, and Cognitive Mapping

Players build mental models of game spaces. Navigation becomes satisfying when the environment supports that mental model.

### 4.1 Cognitive Maps

A **cognitive map** is the player’s internal representation of the environment. Getting lost often means the player’s mental map does not match the actual space.

**Why it matters to the player:**  
Navigation quality affects pacing, confidence, and whether challenge feels meaningful or frustrating.

**Common failure mode:**  
Assuming the layout is clear because the designer already knows it.

**What good implementation looks like:**  
Players can explain where they are, where they have been, and where they think they need to go.

**Design question:**  
What evidence shows that players understand the space rather than merely stumbling through it?

---

### 4.2 Egocentric vs Allocentric Navigation

#### Egocentric Navigation
The player navigates relative to their immediate viewpoint (“turn left here, then right”). UI aids like minimaps can support this.

**Strengths:**  
Fast local orientation, short-term path following.

**Risk:**  
Players may complete routes without forming durable spatial understanding.

---

#### Allocentric Navigation
The player builds a broader map-like understanding of the environment (“that room is above the corridor,” “the tower is east of the courtyard”).

**Strengths:**  
Better route memory, stronger place identity, improved world coherence.

**Risk if unsupported:**  
If landmarks and structure are weak, players cannot form this map.

**Design question:**  
Which aspects of the environment help the player understand the *whole* space, not just the next turn?

---

### 4.3 Five Elements of Wayfinding (Environmental Legibility)

#### 1. Paths
Channels of movement such as corridors, roads, rivers, bridges, trails, catwalks.

**Design use:**  
Paths shape pacing, expectation, and encounter rhythm.

**Failure mode:**  
Too many visually equivalent paths with no hierarchy.

**Design question:**  
Which path reads as primary, and why?

---

#### 2. Landmarks
Distinct reference points used for orientation (tower, statue, broken sign, unique lighting feature, unusual silhouette).

**Design use:**  
Landmarks anchor memory and support navigation decisions.

**Failure mode:**  
Landmarks that are not visually distinct, not visible when needed, or repeated too often.

**Design question:**  
What landmarks are visible from key decision points?

---

#### 3. Districts
Areas with consistent character (material palette, lighting mood, architecture style, soundscape, density, gameplay type).

**Design use:**  
Districts create spatial identity and help players chunk the world.

**Failure mode:**  
Uniform environments where all areas feel interchangeable.

**Design question:**  
How can a player tell they have entered a different area without UI text?

---

#### 4. Edges
Boundaries between areas (walls, cliffs, fences, canals, thresholds, strong lighting transitions).

**Design use:**  
Edges define territory, containment, and transitions.

**Failure mode:**  
Invisible or ambiguous boundaries that confuse whether a space is accessible.

**Design question:**  
What clearly separates one zone from another?

---

#### 5. Nodes
Decision points or convergences (junctions, plazas, atriums, control rooms, crossroads).

**Design use:**  
Nodes are important orientation and pacing moments.

**Failure mode:**  
Nodes that lack identity, causing players to revisit them without recognizing them.

**What good implementation looks like:**  
Nodes feel like places, not just intersections.

**Design question:**  
What makes this node memorable on sight?

---

## Part 5: Interaction Verbs, Affordances, and Readability

A player’s experience depends not only on what actions exist, but on whether the world clearly communicates **where**, **when**, and **how** those actions are possible.

### 5.1 Verbs and Action Design

A useful way to analyse a game is by listing its player verbs:
- movement verbs (move, jump, crouch, sprint, climb)
- observation verbs (look, inspect, scan, listen)
- manipulation verbs (pick up, rotate, place, combine, modify)
- social verbs (talk, persuade, threaten, gift)
- strategic verbs (assign, trade, route, commit resources)

**Why it matters to the player:**  
Verbs determine agency. They define what kinds of problems the player can solve.

**Common failure mode:**  
Adding objects and spaces without defining what the player can actually do with them.

**What good implementation looks like:**  
The environment is designed around the verbs, not the other way around.

**Design question:**  
Which verbs are supported repeatedly and meaningfully by the current space/system?

---

### 5.2 Affordances, Signifiers, and Constraints

- **Affordance**: what an object/space *appears* to allow  
- **Signifier**: what communicates that possibility (shape, icon, material, animation, prompt)  
- **Constraint**: what limits incorrect actions (locked state, missing part, range limit, physical blocker)

**Why it matters to the player:**  
Clarity in affordances reduces random trial-and-error and improves confidence.

**Common failure mode:**  
Objects that look usable but are decorative, or critical interactables that look like background props.

**What good implementation looks like:**  
Important interactions are visually and behaviourally legible.

**Design question:**  
Can the player identify likely interactables before receiving explicit instructions?

---

## Part 6: Feedback, State Change, and Cause-and-Effect Clarity

Players need to perceive the consequences of their actions. A system can be mechanically correct and still feel broken if its outcomes are not clearly communicated.

### 6.1 Feedback Layers

Good feedback is often layered across multiple channels:

- **Visual feedback** (lights, particles, motion, color/state changes, animation)
- **Audio feedback** (clicks, hums, alarms, confirmation tones, impact sounds)
- **UI feedback** (prompts, icons, progress indicators, text confirmations)
- **Camera feedback** (framing, shake, focus shift, reveal)
- **World-state feedback** (doors open, hazards deactivate, routes appear, NPC behaviour changes)

**Why it matters to the player:**  
Feedback supports learning, confidence, and fair challenge.

**Common failure mode:**  
A change happens, but the player does not notice what changed or why.

**What good implementation looks like:**  
Players can connect action → consequence with minimal ambiguity.

**Design question:**  
If a player performs an action, how will they know (1) it worked, (2) what changed, and (3) what that change enables?

---

### 6.2 State Change Readability

Many game experiences depend on objects, spaces, or systems changing state over time (inactive/active, locked/unlocked, powered/unpowered, safe/unsafe, visible/hidden).

**Why it matters to the player:**  
State changes often form the logic of progression and problem-solving.

**Common failure mode:**  
State transitions that are technically present but perceptually subtle or spatially disconnected.

**What good implementation looks like:**  
Before/after states are distinct, and transitions are legible in context.

**Design question:**  
Can a player reliably distinguish the relevant states of this object/system at a glance?

---

## Part 7: Playtesting for Readability, Flow, and Learning

Playtesting is not only for finding bugs or asking whether the game is “fun.” It is essential for evaluating **clarity**, **navigation**, **feedback**, and **player understanding**.

### 7.1 What to Observe

Useful observations include:
- hesitation before obvious actions
- repeated failed attempts
- missed cues or missed affordances
- wrong assumptions that persist
- backtracking loops
- dead time (no purposeful action)
- over-reliance on prompts
- successful improvisation (often a sign of good agency)

**Why it matters to the player:**  
Observation reveals where the design is unclear, not where the player is “bad.”

**Common failure mode:**  
Explaining the design to testers during the session, which hides readability problems.

**What good implementation looks like:**  
Players can progress by reading the game, not by being coached.

**Design question:**  
Where did the player’s understanding diverge from the intended design, and what in the game contributed to that divergence?

---

### 7.2 Testing Questions for Designers

After a test, useful questions include:
- What did the player think their goal was?
- What did they try first, and why?
- What information did they notice or ignore?
- Where did they feel uncertain?
- What action-result relationships were clear or unclear?
- What part felt satisfying, and what produced that feeling?

These questions help translate observations into design changes.

---

## Part 8: Ten Principles for Good Level Design (Refined as Heuristics)

These principles are best treated as **heuristics**: reliable guides, not rigid laws. Their value depends on genre, pacing goals, camera setup, and intended player experience.

### 1) Good level design is fun to navigate
Navigation itself should produce engagement through spatial rhythm, visual guidance, discovery, and meaningful route choices.

- **Why it matters:** Movement is often the most repeated player action.
- **Failure mode:** Traversal feels like dead time between “real” gameplay.
- **Design question:** What makes moving through this space enjoyable even before combat/puzzles/story events occur?

---

### 2) Good level design does not rely only on words to tell a story
Spaces can communicate history, use, conflict, intent, and mood through layout, objects, damage, lighting, and contrast.

- **Why it matters:** Environmental storytelling supports immersion and interpretation.
- **Failure mode:** Story information is stated externally but not embodied in the space.
- **Design question:** What can the player infer about this place before reading any text?

---

### 3) Good level design tells the player what to do, but not exactly how to do it
The design should clarify goals while preserving room for experimentation and player-authored solutions (where appropriate).

- **Why it matters:** This balances direction with agency.
- **Failure mode:** Either total ambiguity (confusion) or over-prescription (no ownership).
- **Design question:** Is the objective clear while the method remains meaningfully open?

---

### 4) Good level design constantly teaches
Spaces can introduce, reinforce, combine, and subvert mechanics over time.

A useful pattern is:  
**Introduce → Practice → Challenge → Twist**

- **Why it matters:** Learning is central to player satisfaction.
- **Failure mode:** New demands appear before players have learned the required patterns.
- **Design question:** What is this space teaching, and how is that teaching staged?

---

### 5) Good level design is capable of surprise
Surprise can come from pacing changes, route reveals, shifted rules, spatial reversals, or new interpretations of familiar mechanics.

- **Why it matters:** Surprise refreshes attention and supports memorable moments.
- **Failure mode:** Constant unpredictability that destroys trust in the game’s logic.
- **Design question:** What expectation is being set here, and how might it be meaningfully subverted?

---

### 6) Good level design empowers the player
Players should be able to perceive their impact on the world, whether through traversal mastery, system manipulation, problem-solving, or changed conditions.

- **Why it matters:** Perceived agency supports engagement and investment.
- **Failure mode:** The player performs actions but cannot see meaningful consequences.
- **Design question:** Where does the game show the player that their decisions matter?

---

### 7) Good level design supports varied challenge
Challenge can be layered through risk-reward choices, optional routes, time pressure, resource constraints, enemy positioning, or complexity of interpretation.

- **Why it matters:** Different players seek different intensities and solution styles.
- **Failure mode:** Flat challenge profile with no variation or self-directed difficulty.
- **Design question:** How can players choose a safer, riskier, faster, slower, simpler, or more rewarding approach?

---

### 8) Good level design is efficient
Spaces and assets should serve multiple design purposes where possible (navigation, story, encounter setup, guidance, atmosphere, reuse, return paths).

- **Why it matters:** Efficiency improves production feasibility and coherence.
- **Failure mode:** Expensive content with little gameplay value.
- **Design question:** What else is this space/asset doing besides looking good?

---

### 9) Good level design creates emotion through space
Scale, compression, openness, light, shadow, repetition, clutter, acoustics, and exposure can all shape emotional tone.

- **Why it matters:** Spatial design is an emotional instrument, not just a container.
- **Failure mode:** Mood is delegated entirely to music or story while space remains emotionally neutral.
- **Design question:** What is the intended emotional tone here, and which spatial choices produce it?

---

### 10) Good level design is driven by mechanics
The level should be shaped by the actions and systems the player uses. The environment is the medium through which mechanics become experience.

- **Why it matters:** Mechanics and level design are inseparable in play.
- **Failure mode:** Spaces are visually impressive but mechanically irrelevant.
- **Design question:** Which core mechanic is this space showcasing, testing, or transforming?

---

## Part 9: A Practical Review Lens for Any Game Space or Sequence

When reviewing a level, room, route, or playable sequence, ask:

1. **Clarity:** What does the player think is happening?
2. **Goal:** What does the player think they are trying to do?
3. **Agency:** What actions appear available?
4. **Readability:** What in the environment communicates those actions?
5. **Feedback:** How does the game confirm or deny outcomes?
6. **Flow:** Where does momentum build, pause, or break?
7. **Learning:** What is being taught or reinforced?
8. **Emotion:** What is the intended feeling, and what produces it?
9. **Coherence:** Do the space, mechanics, and tone support each other?
10. **Evidence:** What observed player behaviour suggests the design is working (or not)?

---

## Reference Videos (Suggested Viewing)

- [Game Mechanics & Systems Thinking — Indie Game Clinic](https://www.youtube.com/watch?v=nkLmjJK3vOw)
- [Level Design Approaches for Solo Devs — Indie Game Clinic](https://www.youtube.com/watch?v=OLXn6YYAk7M)
- [Stop Getting Lost: Make Cognitive Maps, Not Levels — (GDC talk)](https://www.youtube.com/watch?v=HpLMFqztV_M)
- [Ten Principles for Good Level Design — Dan Taylor (GDC)](https://www.youtube.com/watch?v=iNEe3KhMvXM)
- [The Secret to GOOD Game Ideas (Practical Ideation Methods Explained) — Indie Game Clinic](https://www.youtube.com/watch?v=LMOCQNcMleg)

