# Rotary Text Input — Design Record

Status: concept. No hardware, no firmware, no key map yet. Recorded 2026-11-16. This document records the mechanism, the arithmetic that fixes its numbers, the biomechanics rules it must obey, the calibration model, and the alternatives rejected along the way.

Five keys per hand. The layer/modifier channel is mechanical rotation: the key platter rolls ±45° about a rod aligned with the forearm, through seven detents at 15°. Fingers supply the character; the forearm selects the layer. The two channels use disjoint muscle groups, and neither ever displaces the fingers from the keys.

## Concept

- One hand: 5 keys + 7 rotary layers.
- Comfortable gestures per layer: 5 singles + 10 two-key chords = 15.
- 7 × 15 = 105 distinct outputs per hand — full ASCII plus navigation, with headroom.
- Two hands: one platter is the character plane, the other the modifier plane. 7 × 7 = 49 joint states, structured as layer × modifier so it composes like a keyboard's modifier stack, not a flat 49-way switch.
- The lineage is Baudot: five keys is five bits, and Baudot's letters/figures shift was a two-position version of this dial. This design promotes the shift key to a first-class, continuously present mechanical channel.

## The arithmetic that fixes the numbers

Target: ~100 outputs (95 printable characters plus navigation and functions).

| gesture set per layer | count | layers needed |
| --- | --- | --- |
| singles only | 5 | 20 |
| singles + two-key chords | 15 | ≈ 7 |
| full chord space (2⁵ − 1) | 31 | 4 |

- Chosen: 7 layers × 15 gestures = 105; `log2(105) ≈ 6.7` bits per stroke.
- The two channels split a keystroke's entropy almost evenly: 7 positions ≈ 2.8 bits, 15 gestures ≈ 3.9 bits.
- Detents sit at 15° across ±45° of travel, 0° in the middle.
- 7 is at the proprioceptive ceiling for blind position selection — the same reason fretboards mark 5/7/12 and stop there.

## Biomechanics rules

Constraints, not preferences. Each one rejects a specific earlier idea (see Design history).

- **The layer motion is pronation/supination** — forearm rotation. Never wrist ulnar/radial deviation (small-arc, small-muscle, RSI-flagged) and never shoulder rotation (slow, proximal, drags every finger through a wide arc — the "petting a cat" motion).
- "Rotating the forearm from the elbow" is anatomically literal: pronator teres originates at the medial epicondyle.
- **Doorknob, not key.** Torque transmits through palm friction with zero grip. A key clamps grip and twist together, coupling forearm rotators to finger flexors. The doorknob decouples them: layer bits (forearm rotators) and chord bits (finger flexors) are disjoint muscle groups — neither fatigues the other, and one hand can hold its own twist while its own fingers chord.
- **The keys ride the rotating part.** Hand + platter + keys form one rigid body through every detent. Nothing re-homes; the finger map never separates from the keys — the property that makes holding a doorknob while it turns feel like nothing.
- **The palm rides the platter** — it is what transmits the roll. Fingertips ride the keys.
- **Key 3 sits over the axis**: the middle finger is the pivot of the rock. Outer keys sweep ±(w/2)·sin 45° ≈ ±3 cm, and the fingers ride it.
- **The roll axis stays parallel to the forearm** in both horizontal and vertical projection. Misalignment reintroduces exactly the wrist/shoulder compensation the mechanism exists to avoid — which is why the pitch and roll-bias calibrations exist.

## Mechanism

A fixed rod along the forearm line, anchored at both ends in uprights on a base plate. The chunk — the key platter — rides the rod through bushings at both of its ends and rolls ±45°.

### Parts

- **Rod** (fixed): the roll axis, anchored in two uprights. Doubles as spring seat and encoder axle.
- **Chunk** (rotates): the drop-in button assembly. Bushings at both ends ride the rod. Five keys in a transverse row on top, key 3 over the axis. C2-symmetric footprint and connector so one SKU serves both hands by a 180° flip; handedness is one bit and firmware mirrors the chord table on it.
- **Bias collar** (rotates once, then locks): on the rod at the palm end. Carries the detent crown and the hall sensor together — see Calibration.
- **Detents**: 7 × 15°, harder click at 0°. Ball plunger in the chunk's end face rides the crown. At the palm end because torque enters there — shortest load path, and the click is felt in the heel.
- **End stops**: rubber strips on the base plate at the ±z edges; the platter's low edge kisses them at ±45°. Travel limits and shock absorption in one part.
- **Sensing**: magnet ring on the chunk end, hall sensor in the collar. Bounded travel means a flex loop suffices — no slip ring.
- **Spring** (optional): torsion spring on the rod beside the collar. Unsprung = latched layers (turn and stay). Sprung = momentary modifiers (hold to shift). Axial click while sprung = latch-on-demand. One BOM, three behaviors.

### Loading

- Two bearings turn every moment into a couple spread over the span: each bearing sees M/L, and L is a whole hand length. A one-sided pivot would instead take all moments at one fat cantilevered joint.
- Worst spurious roll moment: a one-sided chord, a couple of newtons at a couple of centimeters. Intentional palm torque: the forearm's weight behind a ~5 cm lever — an order of magnitude more. Detent breakaway lives in that gap. It is a number, not a feel.
- Stiffness check that killed the earlier trough design: 2 N mid-span over a 15–20 cm span on a small rod deflects hundredths of a millimeter — continuous edge support buys nothing real.

### Side view

Camera on z; the key row runs into the page; drawn at 0° roll.

```text
  elbow
    \    forearm slope 5-15 deg — the PITCH lock matches this
     \
      \            palm rides the platter — heel sits just inboard of the
       \           detents, so the 0-deg click lands under the heel
        \      _______________________________________________
         \____|                        A                      |______
              |   row of 5 keys ->  [#####] (z: into page)    |       \___
              |                       |                       |          \ fingertips
              |                       |                       |           | drape here
              |                       |        C H U N K       |           v
              |                       |  (bore + bushings     |
              |                       |    ride the rod)      |
              '-----------------------|-----------------------'
                                      A
    [crown] [##]=====================[#]====================[##]
       ^      ^bearing                   ROD (fixed)      bearing^
       |
  bias collar: crown + hall sensor, on the rod here.
  Rotate + lock once = ROLL BIAS (0 deg = zero-muscle rest).
  (Optional torsion spring seats here too — sprung = momentary)

    NEAR UPRIGHT                                FAR UPRIGHT
       |                                            |
       |     height: set once                       |     height: set once
       |                                            |
  [X]  |                                      (o)   |
 pitch hinge (z-axis):                  curved slot + lock screw
 uprights + rod + chunk pitch            (serrated quadrant — no drift)
 as a unit, to match the forearm slope
       |____________________________________________|
       |               BASE PLATE                   |
       |    yaw = placement on the desk — free      |
       |____________________________________________|
        '  - - - - - - - - - - - - - - - - - - - '
         rubber strips hide at the ±z edges (out of this view):
         the platter's low edge kisses them at ±45 deg = soft end stops

  ROLL = the interactive channel: 7 detents × 15 deg, ±45 deg about the rod,
  harder click at 0 — pronation/supination, the layer selector.
```

### Section A–A

At the key row, looking toward the elbow (y up, z across the hand).

```text
                 fingertips: five pads over five keys
                  ( )   ( )   ( )   ( )   ( )
                  _     _     _     _     _
                 |1|   |2|   |3|   |4|   |5|          keys across the hand
 pinky   _______/                                           \_______  thumb
  side  |                     C H U N K                            |  side
        |                      .-.                                |
 -45 <- |                     ( o )   ROD: fixed (roll axis)        | -> +45
        |                      '-'                                |
        |          key 3 sits over the axis: barely travels       |
        |___________________________________________________________|
                    |                         |
                    x                         x      low corners at ±45 deg:
                    '------ w = hand width ---'      sweep ±(w/2)*sin45 ~ ±3 cm
          ________/\______________________________/\________       (fingers
         |  rubber                          rubber          |        ride it;
         |           BASE PLATE (spans under all)           |  nothing re-homes)
         |__________________________________________________|

  palm is nearer the wrist — off this cut. Hand + chunk rotate about
  the rod as one rigid body, through every detent.
```

### Bias collar detail

Seen down the rod at the near end.

```text
      CHUNK END FACE (rotates)          COLLAR, on the rod (locks)
      +--------------+                 +------------------+
      |  magnet ring  | <- read by hall - detent crown:    |
      |               |                |  7 teeth x 15,   |
      |  (ball        | <- ball clicks  |  harder at 0     |
      |   plunger)    |    the teeth -> |                  |
      +--------------+                 +------------------+
      rotating = layer changes         rotate + lock = ROLL BIAS
                                        (crown + hall move together,
                                         firmware stays blind)
```

## Calibration

Setup, not interaction: one placement, one fit, two locks.

| DOF | mechanism | set |
| --- | --- | --- |
| yaw | none — placement on the desk is free | whenever |
| pitch | z-axis hinge at the base; curved slot, serrated quadrant, lock screw | once |
| height | uprights | once |
| roll bias | bias collar — crown and hall rotate together, then lock | once |

- **Pitch matches the forearm slope.** Elbows sit above wrists in real chairs; forearms decline 5–15°. The bridge — uprights + rod + chunk — pitches as a unit.
- **Pitch and bias compose; neither substitutes.** Pitching the rod to the slope tilts the chunk with it; the bias collar then rotates the rest attitude to bring the key row back level. Pitch aligns the axis; bias levels the buttons.
- **The collar carries the detent teeth and the hall sensor together**, so rotating it shifts the detent pattern and the encoder zero in lockstep. Firmware stays blind to calibration forever.
- **Locks are indexed, never friction.** The failure mode is drift: months of rocking torque walking the zero a degree or two until layers misfire and it presents as typos no one can diagnose. Serrations at 2.5–5° give repeatability and tactile increments.
- **Setup ritual:** relax the hand on the platter; wherever it settles is the operational zero; adjust until the 0° detent asks zero muscle; lock; forget. Order: place (yaw) → pitch → height → bias → lock.

## Layer map (sketch, unassigned)

Seven positions per platter. Suggested families for the character platter: letters-a, letters-b, digits, punctuation, navigation, modifier-lock, macros/mouse; the modifier platter carries the shift/ctrl/cmd families. Actual assignment is open work; the arithmetic above bounds it.

## Design history — rejected alternatives

- **Thumb-operated dial** — steals the thumb and forces re-homing between layer and chord.
- **Three stops at ±45°, spring-centered** — every state unmistakable, momentary semantics for free, but only 45 outputs per hand; needs the two-hand multiply for basic layering. Reverted to 7 × 15° once the character/modifier plane split made the joint space structured.
- **Vertical-axis turntable** ("pizza platter on a column") — lazy-susan motion is humeral rotation: shoulder-driven, slow, and every finger sweeps a wide arc.
- **Flat petri-dish lid on a base** — no depth for real detent hardware, and it walks across the desk under chord + rock torque.
- **Curved rockers in a circular race** ("platter inside a circle", lens-in-barrel) — continuous edge support and detents in the race, but the stiffness check showed support was solving a non-problem; round parts and standard bearings replaced compound curves.
- **One-sided rotational anchor** — all moments at one joint; superseded by the through-rod with bearings at both ends.

## Prior art

- **Baudot/Murray code** — 5-bit telegraphy with letters/figures shift: the dial's direct ancestor.
- **Stenotype number bar** — a binary layer selector already in production.
- **CharaChorder, Twiddler, Tap Strap, Datahand, microwriter/CyKey** — the 5-key chord heritage to raid for gesture design.
- **Safe dials** — detent counting by feel is a trained skill: proof that blind position reading trains.
- **Rocking-chair runners, lens elements in a barrel** — the race-and-cradle mechanism the rod simplified away.

## Open questions

- Detent spring rate: the gap between spurious chord moments and palm torque is estimated, not measured.
- Layer map assignment, gesture frequency ordering, learnability.
- Key switch hardware that survives rocking loads inside a thin chunk.
- Firmware: nothing exists. The contract is small — 5 keys, an L/R bit, angle relative to zero.
- The zero-muscle rest claim is an assumption until a hand rests on a prototype.
- Axial channel (latch-on-demand): keep or cut.
