# The Mausoleum Crypts — Detailed Minigame Design Document

## Table of Contents

1. [Overview & Lore](#1-overview--lore)
2. [Requirements & Entry](#2-requirements--entry)
3. [Arena Layout & Environment](#3-arena-layout--environment)
4. [NPC Definitions](#4-npc-definitions)
5. [Wave System](#5-wave-system)
6. [Boss Mechanics](#6-boss-mechanics)
7. [Between-Wave Phase](#7-between-wave-phase)
8. [Point System](#8-point-system)
9. [Reward Shop](#9-reward-shop)
10. [Item Definitions](#10-item-definitions)
11. [Crypt Keeper Dialogue](#11-crypt-keeper-dialogue)
12. [Implementation File Structure](#12-implementation-file-structure)
13. [Technical Patterns](#13-technical-patterns)

---
Melee Skeleton — Bone Rattle
  Every 5th hit, it rattles its bones and boosts its attack speed from 4 ticks to 3 for the next 2 attacks. Message: "The skeleton fights in a frenzy!". Just a counter in ai_applayer2 that temporarily changes 
  the attack rate. Keeps melee skeletons feeling relentless.
 Ranger Skeleton — Marked Target
  After 3 consecutive hits on the same player, the ranger "marks" them — the next ranged hit deals double damage. Visual: spotanim_npc when the mark is applied, message: "The skeleton archer has you in its    
  sights!". The counter resets if the player moves 3+ tiles from their position (you dodged the lock-on) or if they get behind a pillar (line of sight break).
  Elite Skeleton — Shield Wall
  When another crypt skeleton (any type) is adjacent to the elite, the elite gets +20 to all defences. When the last nearby skeleton dies, it loses the bonus. Message on activation: "The Skeleton Champion     
  rallies the undead!". Checked each ai_applayer2 tick with npc_huntall counting nearby skeletons. This rewards the player for killing the regular skeletons first instead of tunneling the elite, and makes the
  fight order matter.
## 1. Overview & Lore

### Concept

A repeatable, wave-based combat minigame set beneath the Mausoleum near Paterdomus. Players descend into burial crypts overrun with restless undead and fight through 10 waves of escalating difficulty. Between waves, they loot coffins for supplies and choose branching paths. Performance earns Crypt Tokens exchangeable for unique rewards.

### Lore

The crypts beneath the Mausoleum were once the resting place of Saradomin's fallen templars — warriors who gave their lives defending the Salve barrier from Morytania's horrors. For centuries, the holy wards of Paterdomus kept the dead at rest.

But since Lord Drakan's forces breached the River Salve's blessings (during Priest in Peril), dark energy has seeped into the crypt network. The templars' bones stir. Shades coalesce from shadows. Ancient burial guardians, once protectors of the dead, have turned violent.

**Brother Cedwyn**, a Saradominist monk stationed at the Mausoleum, has been trying to contain the situation alone. He's realized that while the dead cannot be permanently laid to rest until the Salve's full power is restored, regularly clearing the crypts slows the corruption's spread. He seeks adventurers willing to descend repeatedly and push back the darkness.

### Design Goals

- **Repeatable engagement** — Not a one-time quest, but an activity players return to for tokens and rewards
- **Scalable challenge** — Waves 1-4 are accessible at 50 combat, waves 5-7 require decent gear, waves 8-10 are genuinely dangerous
- **Gold sink** — 5,000 coin entry fee
- **Resource sink** — Players burn through food, potions, prayer points; coffin loot only partially resupplies
- **Thematic consistency** — All undead models, dungeon tilesets, and locs already exist in the 2004 cache
- **Solo-focused** — Designed for one player (no group scaling needed)

---

## 2. Requirements & Entry

### Requirements

| Requirement | Details |
|------------|---------|
| Combat level | 50+ (recommended 60-70 for full clears) |
| Quest | Priest in Peril complete |
| Entry fee | 5,000 coins per run |

### How to Enter

1. Travel to the Mausoleum (east of Varrock, near Paterdomus)
2. Speak to **Brother Cedwyn** at the Mausoleum entrance
3. Pay 5,000 coins
4. Descend via the crypt staircase into Wave Room 1

### Leaving

- **Death**: Player dies normally (loses items per standard death mechanics). Run ends. No tokens awarded for incomplete waves.
- **Forfeit**: Player can climb the exit stairs in any room to leave. Tokens earned so far are kept.
- **Completion**: After wave 10, a blessed staircase appears. Player exits with all earned tokens.
- **Logout**: Treated as forfeit. Player respawns at the Mausoleum entrance on next login. Tokens earned so far are kept.

---

## 3. Arena Layout & Environment

### Map Region

The crypts occupy a new underground map region. The dungeon uses the standard dungeon tileset (`o42` floor, `o42 f16` blocked/wall) with crypt-specific decoration.

### Room Structure

The dungeon consists of 5 distinct rooms connected by short corridors. The player progresses linearly through them, but some rooms offer a left/right path choice affecting enemy composition.

```
                    [ENTRY]
                      |
              +----- ROOM 1 -----+        Waves 1-2
              |    (Burial Hall)  |        8x8 with 4 alcoves
              +--------+----------+
                       |
              +--------+----------+
              |    ROOM 2         |        Waves 3-4
              |  (Ossuary)        |        10x8 with bone piles
              +---+----------+----+
                  |          |
           [LEFT PATH]  [RIGHT PATH]       (choice corridor)
                  |          |
              +---+----------+----+
              |    ROOM 3         |        Wave 5 (mini-boss)
              |  (Guardian Hall)  |        12x10 with pillars
              +--------+----------+
                       |
              +--------+----------+
              |    ROOM 4         |        Waves 6-9
              |  (Deep Crypts)    |        10x12, L-shaped
              +--------+----------+
                       |
              +--------+----------+
              |    ROOM 5         |        Wave 10 (final boss)
              | (Sanctum of the   |        14x14 octagonal
              |    Entombed)      |
              +-------------------+
```

### Room Details

#### Room 1 — The Burial Hall (Waves 1-2)

- **Size**: 8x8 central area with 4 small 2x3 alcoves (one per cardinal side)
- **Floor**: `o42` dungeon stone
- **Walls**: `1434 10 1` solid rock fill + `1417 0 rotation` wall faces
- **Decoration**:
  - 4 coffins (`coffin`, model_loc_398) — one per alcove, lootable between waves
  - 4 wall torches (`196`) flanking the alcoves
  - Ground rubble (`320`, `321`) scattered across floor (shape 22)
  - Entry staircase (`cryptstairsup`, model_loc_1730) on the north wall — exit point
  - South passage to Room 2

#### Room 2 — The Ossuary (Waves 3-4)

- **Size**: 10x8 rectangular
- **Theme**: Bone storage — shelves of skulls and remains
- **Decoration**:
  - 6 bookshelves (`380`, `381`) along east and west walls (repurposed as bone shelves)
  - 2 coffins in alcoves
  - Skull torches (`207`) on all four walls
  - Bone piles — ground decorations (`336`, shape 22)
  - 2 pillars (`1868`) breaking up the center
- **South wall**: Two passages (left and right) — player chooses one
  - **Left path**: Narrow 3-wide corridor, 6 tiles long. Leads to Room 3 from the west side. Next wave has more skeletons (ranged-heavy).
  - **Right path**: Wide 5-wide corridor, 4 tiles long. Leads to Room 3 from the east side. Next wave has more zombies (melee-heavy).

#### Room 3 — The Guardian Hall (Wave 5 — Mini-boss)

- **Size**: 12x10 with cut corners (octagonal-ish)
- **Theme**: Ancient templar burial chamber — grander than previous rooms
- **Decoration**:
  - 4 stone pillars (`1868`) in a square formation around center
  - 2 sarcophagi (`ibantomb_left`, `ibantomb_right`, model_loc_3353) on east and west walls
  - Large candelabras (`209`) in each corner
  - Wall torches (`196`) every 3 tiles
  - Faded magic circle on floor (`740`, shape 22) in center — the Guardian spawns here
  - 2 coffins for between-wave looting
  - South passage to Room 4

#### Room 4 — The Deep Crypts (Waves 6-9)

- **Size**: 10x12, L-shaped (main 10x8 area + 6x4 eastern extension)
- **Theme**: Deeper, darker — shade territory
- **Decoration**:
  - Skull torches (`207`) replace normal torches — dimmer, more sinister
  - 4 coffins spread across the L-shape
  - Crates (`355`) and barrels (`362`) — old supplies left by previous explorers
  - Stalactites (`3826`, shape 22) hanging from ceiling in several spots
  - Ground: more rubble (`320`, `321`), scattered bones (`336`)
  - Portcullis (`portcullis_upass`, model_loc_3303) on the eastern extension — opens after wave 7
  - Castle door (`1517`) at the south exit — opens after wave 9
  - 2 pillars (`1868`) in main area

#### Room 5 — Sanctum of the Entombed (Wave 10 — Final Boss)

- **Size**: 14x14 with octagonal cut corners (same technique as the Apprentice's Betrayal inner vault)
- **Theme**: Grand burial sanctum — the most powerful templar's resting place
- **Decoration**:
  - 8 pillars (`1868`) in an octagonal ring around the arena
  - Central sarcophagus (`ibantomb_left` + `ibantomb_right` side by side) — The Entombed rises from this
  - 4 large candelabras (`209`) at compass points
  - 4 skull torches (`207`) on outer walls
  - Chandelier (`210`) overhead at center
  - Faded magic circles (`740`) at 4 points around the sarcophagus
  - Altar (`410`) on the north wall — non-functional, decorative only
  - Blessed staircase (`cryptstairsup`) appears on the north wall AFTER the boss is defeated
  - No coffins — no resupply before the boss

---

## 4. NPC Definitions

### 4a. Crypt Keeper NPC — Brother Cedwyn

```
[brother_cedwyn]
name=Brother Cedwyn
desc=A Saradominist monk guarding the Mausoleum crypts.
model1=model_230_npc          // monk body (same as monks of Entrana)
model2=model_246_npc          // monk head
recol1s=2340                  // robe primary → dark grey
recol1d=8
recol2s=14724                 // robe secondary → dark grey
recol2d=8
walkanim=human_walk_f
readyanim=human_ready
op1=Talk-to
op3=Trade
vislevel=hide
wanderrange=3
```

**Location**: Standing near the Mausoleum entrance, coordinates near the existing mausoleum area.

### 4b. Wave Enemies

#### Crypt Zombie (Waves 1-2)

Three tiers for variety within the early waves:

```
[crypt_zombie_weak]
name=Crypt Zombie
desc=A shambling corpse risen from the crypts.
model1=model_2931_npc
walkanim=zombie_walk
readyanim=zombie_ready
op2=Attack
vislevel=24
hitpoints=25
attack=18
strength=16
defence=14
attackbonus=0
strengthbonus=0
stabdefence=5
slashdefence=3
crushdefence=8
magicdefence=5
rangedefence=5
attack_anim=zombie_attack
defend_anim=zombie_block
death_anim=zombie_death
attack_sound=zombie_attack
defend_sound=zombie_hit
death_sound=zombie_death
damagetype=^crush_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_zombie
```

```
[crypt_zombie_mid]
name=Crypt Zombie
desc=A shambling corpse risen from the crypts.
model1=model_2931_npc
model2=model_2932_npc         // armed variant (has weapon model)
recol1s=4526
recol1d=6050                  // slightly greener tint
walkanim=zombie_walk
readyanim=zombie_ready
op2=Attack
vislevel=32
hitpoints=35
attack=25
strength=22
defence=20
attackbonus=5
strengthbonus=7
stabdefence=10
slashdefence=8
crushdefence=12
magicdefence=10
rangedefence=10
attack_anim=zombie_attack
defend_anim=zombie_block
death_anim=zombie_death
attack_sound=zombie_attack
defend_sound=zombie_hit
death_sound=zombie_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_zombie
```

```
[crypt_zombie_strong]
name=Crypt Zombie
desc=A large, rotting corpse. It looks stronger than the others.
model1=model_2931_npc
model2=model_2932_npc
recol1s=4526
recol1d=12777                 // darker, more decayed
resizeh=115
resizev=115                   // slightly larger
walkanim=zombie_walk
readyanim=zombie_ready
op2=Attack
vislevel=44
hitpoints=48
attack=36
strength=34
defence=28
attackbonus=10
strengthbonus=12
stabdefence=12
slashdefence=10
crushdefence=15
magicdefence=12
rangedefence=12
attack_anim=zombie_attack
defend_anim=zombie_block
death_anim=zombie_death
attack_sound=zombie_attack
defend_sound=zombie_hit
death_sound=zombie_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_zombie
```

#### Crypt Skeleton (Waves 3-4)

```
[crypt_skeleton_melee]
name=Crypt Skeleton
desc=The reanimated bones of a fallen templar.
model1=model_2944_npc
model2=model_2946_npc         // armed (sword)
walkanim=skeleton_walk
readyanim=skeleton_ready
op2=Attack
vislevel=45
hitpoints=42
attack=40
strength=38
defence=35
attackbonus=15
strengthbonus=14
stabdefence=9
slashdefence=11
crushdefence=-5               // weak to crush (standard skeleton weakness)
magicdefence=1
rangedefence=4
attack_anim=skeleton_attack
defend_anim=skeleton_block
death_anim=skeleton_death
attack_sound=skeleton_attack
defend_sound=skelly_hit
death_sound=skeleton_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_skeleton
```

```
[crypt_skeleton_ranger]
name=Crypt Skeleton
desc=A skeleton archer. Its aim is unnervingly precise.
model1=model_2944_npc
model2=model_2947_npc         // bow/ranged weapon variant
walkanim=skeleton_walk
readyanim=skeleton_ready
op2=Attack
vislevel=50
hitpoints=45
attack=45
strength=40
defence=30
rangeattack=55
attackbonus=0
strengthbonus=0
rangestrengthbonus=25
stabdefence=5
slashdefence=5
crushdefence=-5
magicdefence=1
rangedefence=5
attack_anim=skeleton_attack
defend_anim=skeleton_block
death_anim=skeleton_death
attack_sound=skeleton_attack
defend_sound=skelly_hit
death_sound=skeleton_death
damagetype=^ranged_style
param=undead,^true
huntmode=aggressive_ranged
huntrange=6
category=crypt_skeleton
```

```
[crypt_skeleton_elite]
name=Crypt Skeleton Champion
desc=A skeleton clad in ancient templar armour. It fights with deadly precision.
model1=model_2944_npc
model2=model_2945_npc         // heavy weapon variant (legends quest style)
recol1s=29562
recol1d=26515                 // darker bone color
walkanim=skeleton_trans_walk
readyanim=skeleton_trans_ready
op2=Attack
vislevel=55
hitpoints=55
attack=52
strength=50
defence=48
attackbonus=25
strengthbonus=20
stabdefence=20
slashdefence=22
crushdefence=5
magicdefence=10
rangedefence=15
attack_anim=skeleton_trans_attack
defend_anim=skeleton_trans_block
death_anim=skeleton_trans_death
attack_sound=skeleton_attack
defend_sound=skelly_hit
death_sound=skeleton_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_skeleton
```

#### Crypt Ghost (Waves 6-7)

```
[crypt_ghost]
name=Restless Spirit
desc=The tormented spirit of a buried templar.
model1=model_2961_npc
model2=model_2964_npc
model3=model_2965_npc
walkanim=ghost_walk
readyanim=ghost_ready
op2=Attack
vislevel=48
hitpoints=40
attack=42
strength=40
defence=42
stabdefence=5
slashdefence=5
crushdefence=5
magicdefence=-10              // weak to magic
rangedefence=5
attack_anim=ghost_attack
defend_anim=ghost_block
death_anim=ghost_death
attack_sound=ghost_attack
defend_sound=ghost_hit
death_sound=ghost_death
damagetype=^crush_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_ghost
```

```
[crypt_ghost_strong]
name=Vengeful Spirit
desc=A powerful, wrathful ghost. Its presence chills you to the bone.
model1=model_2961_npc
model2=model_2964_npc
model3=model_2965_npc
resizeh=130
resizev=130                   // noticeably larger
walkanim=ghost_walk
readyanim=ghost_ready
op2=Attack
vislevel=62
hitpoints=58
attack=55
strength=52
defence=55
stabdefence=10
slashdefence=10
crushdefence=10
magicdefence=-5
rangedefence=10
attack_anim=ghost_attack
defend_anim=ghost_block
death_anim=ghost_death
attack_sound=ghost_attack
defend_sound=ghost_hit
death_sound=ghost_death
damagetype=^crush_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_ghost
```

#### Crypt Shade (Waves 8-9)

```
[crypt_shade]
name=Crypt Shade
desc=A dark, humanoid shadow. It drains your energy.
model1=model_226_npc
model2=model_306_obj_wear
model3=model_174_npc
model4=model_265_obj_wear
recol1s=24075
recol1d=0                     // all-black recolors
recol2s=14724
recol2d=0
recol3s=10570
recol3d=0
recol4s=15855
recol4d=0
recol5s=2340
recol5d=0
walkanim=ghosthuman_walk_forward
readyanim=ghosthuman_ready
op2=Attack
vislevel=65
hitpoints=62
attack=60
strength=58
defence=55
attackbonus=20
strengthbonus=18
stabdefence=15
slashdefence=15
crushdefence=15
magicdefence=5
rangedefence=15
attack_anim=human_sword_transslash
defend_anim=human_sword_transdef
death_anim=human_transdeath
attack_sound=stabsword_slash
defend_sound=ghost_hit
death_sound=ghost_death
damagetype=^crush_style
param=undead,^true
huntmode=aggressive_melee
category=crypt_shade
```

```
[crypt_shade_strong]
name=Shadow Wraith
desc=A powerful shade radiating darkness. Your prayers feel weaker near it.
model1=model_226_npc
model2=model_306_obj_wear
model3=model_174_npc
model4=model_265_obj_wear
recol1s=24075
recol1d=4226                  // dark purple tint instead of pure black
recol2s=14724
recol2d=4226
recol3s=10570
recol3d=4226
recol4s=15855
recol4d=4226
recol5s=2340
recol5d=4226
resizeh=120
resizev=120
walkanim=ghosthuman_walk_forward
readyanim=ghosthuman_ready
op2=Attack
vislevel=75
hitpoints=72
attack=68
strength=65
defence=62
attackbonus=25
strengthbonus=22
stabdefence=20
slashdefence=20
crushdefence=20
magicdefence=10
rangedefence=20
attack_anim=human_sword_transslash
defend_anim=human_sword_transdef
death_anim=human_transdeath
attack_sound=stabsword_slash
defend_sound=ghost_hit
death_sound=ghost_death
damagetype=^crush_style
param=undead,^true
param=drain_prayer,^true      // custom param — drains 1-3 prayer per hit
huntmode=aggressive_melee
category=crypt_shade
```

**Prayer drain mechanic**: Shadow Wraiths drain 1-3 prayer points on each successful hit. Implemented in `[ai_applayer2,crypt_shade_strong]` — after damage calc, `stat_sub(prayer, calc(1 + random(3)), 0)` on the player.

### 4c. Bosses

#### Crypt Guardian (Wave 5 Mini-boss)

```
[crypt_guardian]
name=Crypt Guardian
desc=An ancient skeleton warrior bound to protect the crypts. Its armour glows faintly.
model1=model_2944_npc
model2=model_2948_npc         // heavy weapon variant (Ranalph Devere style)
recol1s=29562
recol1d=22218                 // lighter bone — "blessed" appearance
resizeh=140
resizev=140                   // noticeably larger than regular skeletons
walkanim=skeleton_trans_walk
readyanim=skeleton_trans_ready
op2=Attack
vislevel=70
hitpoints=85
attack=62
strength=60
defence=65
attackbonus=45
strengthbonus=44
stabdefence=35
slashdefence=35
crushdefence=10               // still weak to crush
magicdefence=25
rangedefence=30
attack_anim=skeleton_trans_attack
defend_anim=skeleton_trans_block
death_anim=skeleton_trans_death
attack_sound=skeleton_attack
defend_sound=skelly_hit
death_sound=skeleton_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
maxhit=12
```

**Mechanics**: See [Section 6a](#6a-crypt-guardian-wave-5).

#### The Entombed (Wave 10 Final Boss)

Phase 1 — Skeleton Form:

```
[the_entombed_skeleton]
name=The Entombed
desc=An immense skeleton rises from the sarcophagus. Ancient power radiates from its bones.
model1=model_2944_npc
model2=model_2946_npc         // armed skeleton
recol1s=29562
recol1d=15200                 // golden-tinted bones
resizeh=190
resizev=190                   // giant
walkanim=skeleton_trans_walk
readyanim=skeleton_trans_ready
op2=Attack
vislevel=95
hitpoints=100
attack=80
strength=78
defence=75
attackbonus=50
strengthbonus=48
stabdefence=40
slashdefence=40
crushdefence=15
magicdefence=30
rangedefence=35
attack_anim=skeleton_trans_attack
defend_anim=skeleton_trans_block
death_anim=skeleton_trans_death
attack_sound=skeleton_attack
defend_sound=skelly_hit
death_sound=skeleton_death
damagetype=^slash_style
param=undead,^true
huntmode=aggressive_melee
maxhit=16
```

Phase 2 — Shade Form:

```
[the_entombed_shade]
name=The Entombed
desc=The skeleton dissolves into a terrible shadow. Dark energy pulses outward.
model1=model_226_npc
model2=model_306_obj_wear
model3=model_174_npc
model4=model_265_obj_wear
recol1s=24075
recol1d=6273                  // dark purple/indigo
recol2s=14724
recol2d=6273
recol3s=10570
recol3d=6273
recol4s=15855
recol4d=6273
recol5s=2340
recol5d=6273
resizeh=160
resizev=160
walkanim=ghosthuman_walk_forward
readyanim=ghosthuman_ready
op2=Attack
vislevel=95
hitpoints=80
attack=75
strength=72
defence=68
attackbonus=35
strengthbonus=30
stabdefence=25
slashdefence=25
crushdefence=25
magicdefence=-10              // vulnerable to magic in this form
rangedefence=25
attack_anim=human_sword_transslash
defend_anim=human_sword_transdef
death_anim=human_transdeath
attack_sound=otherworld_attack
defend_sound=otherworld_hit
death_sound=otherworld_death
damagetype=^crush_style
param=undead,^true
param=drain_prayer,^true
huntmode=aggressive_melee
maxhit=14
```

Phase 3 — Ghost Form:

```
[the_entombed_ghost]
name=The Entombed
desc=A colossal spirit wails in fury. This is its final form.
model1=model_2961_npc
model2=model_2964_npc
model3=model_2965_npc
resizeh=200
resizev=200                   // massive ghost
walkanim=ghost_walk
readyanim=ghost_ready
op2=Attack
vislevel=95
hitpoints=60
attack=82
strength=80
defence=60
attackbonus=40
strengthbonus=42
stabdefence=15
slashdefence=15
crushdefence=15
magicdefence=-15              // very weak to magic
rangedefence=15
attack_anim=ghost_attack
defend_anim=ghost_block
death_anim=ghost_death
attack_sound=ghost_attack
defend_sound=ghost_hit
death_sound=ghost_death
damagetype=^crush_style
param=undead,^true
huntmode=aggressive_melee
maxhit=18
```

**Mechanics**: See [Section 6b](#6b-the-entombed-wave-10).

---

## 5. Wave System

### Wave Overview

| Wave | Room | Enemies | Total HP | Points | Notes |
|------|------|---------|----------|--------|-------|
| 1 | Burial Hall | 3x Crypt Zombie (weak) | 75 | 10 | Tutorial wave |
| 2 | Burial Hall | 2x Crypt Zombie (mid) + 2x Crypt Zombie (weak) | 120 | 15 | Slightly harder |
| 3 | Ossuary | 3x Crypt Skeleton (melee) + 1x Crypt Skeleton (ranger) | 171 | 25 | Ranged threat introduced |
| 4 | Ossuary | 2x Crypt Skeleton (melee) + 2x Crypt Skeleton (ranger) + 1x Crypt Skeleton (elite) | 229 | 35 | Mixed styles |
| 5 | Guardian Hall | **Crypt Guardian** + 2x Crypt Skeleton (melee) | 169 | 60 | Mini-boss |
| 6 | Deep Crypts | 3x Restless Spirit + 2x Crypt Zombie (strong) | 216 | 40 | Ghosts introduced |
| 7 | Deep Crypts | 2x Vengeful Spirit + 3x Crypt Zombie (strong) + 1x Restless Spirit | 300 | 50 | Harder ghost mix |
| 8 | Deep Crypts (east) | 4x Crypt Shade + 1x Crypt Zombie (strong) | 296 | 60 | Shades + prayer drain |
| 9 | Deep Crypts (east) | 3x Shadow Wraith + 2x Crypt Shade | 340 | 70 | Heavy prayer drain |
| 10 | Sanctum | **The Entombed** (3 phases) | 240 total | 150 | Final boss |

**Total possible points per full clear**: 515

### Wave Spawning

Enemies spawn with visual effects:

- **Zombies**: Rise from the ground — `spotanim_map(smokepuff, coord, 0, 0)` + `sound_synth(zombie_attack, 0, 0)`, spawn after 1 tick delay
- **Skeletons**: Bones assemble — `spotanim_map(smokepuff_large, coord, 0, 0)` + `sound_synth(skeleton_attack, 0, 0)`
- **Ghosts**: Fade in — `spotanim_map(teleport_casting, coord, 92, 0)` + `sound_synth(ghost_attack, 0, 0)`
- **Shades**: Emerge from shadow — `spotanim_map(smokepuff, coord, 0, 0)` + `sound_synth(otherworld_attack, 0, 0)`

Enemies spawn staggered across 3-5 ticks (not all at once) so the player can see them appear. Spawn positions are pre-defined per room at various tiles around the perimeter.

### Wave Completion Detection

When an enemy dies, its `[ai_queue3,crypt_*]` handler:
1. Calls `gosub(npc_death)` for standard death behavior
2. Checks `npc_findhero` — if true, player gets kill credit
3. Uses `npc_huntall(room_center_coord, 15, 0)` to count remaining enemies with `category=crypt_zombie|crypt_skeleton|crypt_ghost|crypt_shade`
4. If count = 0, triggers `queue(crypt_wave_complete, 0)` on the killing player

### Wave Transition

On wave completion:
1. Player receives message: `"Wave [N] complete! [points] Crypt Tokens earned."`
2. `%crypt_wave` incremented
3. `%crypt_tokens = add(%crypt_tokens, wave_points)`
4. If wave < 10: 50-tick (30 second) intermission begins — coffins become lootable, passage to next room opens
5. If wave = 10: Boss defeated sequence plays

### Path Choice (After Wave 4)

After completing wave 4 in the Ossuary, two passages open (south-west and south-east). The player walks to one:

- **Left path** → Wave 5 skews toward ranged skeleton escorts with the Guardian
- **Right path** → Wave 5 skews toward zombie escorts with the Guardian

The path choice is tracked in `%crypt_path` (0=left, 1=right) and subtly affects enemy compositions in waves 6-9:

| Path | Wave 6-7 bias | Wave 8-9 bias |
|------|---------------|---------------|
| Left | More ghosts, fewer zombies | More shades, fewer wraiths |
| Right | More zombies, fewer ghosts | More wraiths, fewer shades |

This gives minor replayability — the overall difficulty is similar, but the combat feel differs.

---

## 6. Boss Mechanics

### 6a. Crypt Guardian (Wave 5)

**Stats recap**: Level 70, 85 HP, max hit 12, high slash/stab defence, weak to crush.

**Phase 1 (85-40 HP) — Standard Combat**:
- Melee-only, standard attack speed (4 ticks)
- Hits up to 12 with slash
- No special mechanics — straightforward but hard-hitting

**Phase 2 (Below 40 HP) — Empowered**:
- At 40 HP: `npc_say("You will not defile these crypts!")` + `spotanim_npc(teleport_casting, 92, 0)`
- Attack speed increases from 4 to 3 ticks
- Max hit increases to 14
- Every 8th attack: Bone Shield — Guardian gains +40 to all defences for 2 attacks, then reverts
  - Visual: `spotanim_npc(prot_from_magic_icon, 92, 0)` — prayer-like shield glow
  - Message: `"The Crypt Guardian raises a magical shield!"`
  - After 2 hits received: `"The Guardian's shield shatters!"`

**Death**: Standard `skeleton_trans_death` animation. Drops nothing (rewards are token-based). `spotanim_map(smokepuff_large, coord, 0, 0)`.

### 6b. The Entombed (Wave 10)

A 3-phase boss fight using the Nazastarool pattern — each phase is a different NPC that spawns when the previous one dies.

**Phase 1 — Skeleton Form (100 HP)**

- Max hit: 16
- Attack speed: 4 ticks
- Attack style: Slash melee
- **Special — Bone Volley** (every 15 ticks): The Entombed stomps the ground, dealing 3-6 damage to the player regardless of prayer. Visual: `npc_anim(skeleton_trans_attack, 0)` + `spotanim_map(smokepuff_large, player_coord, 0, 0)`.
  - Message: `"The Entombed slams the ground! Bones fly in all directions!"`
- At 0 HP: Death animation plays. After 2 ticks, the skeleton collapses and a shade rises from the remains.

**Transition to Phase 2**:
```
[ai_queue3,the_entombed_skeleton]
gosub(npc_death);
if (npc_findhero = ^false) { return; }
npc_setmode(none);
npc_anim(skeleton_trans_death, 0);
npc_delay(2);
def_coord $coord = npc_coord;
npc_del;
if (p_finduid(uid) = true) { p_delay(1); }
npc_add($coord, the_entombed_shade, 1000);
npc_anim(human_transdeath, 0);     // reverse death = "rising" effect
spotanim_npc(smokepuff_large, 124, 34);
npc_setmode(applayer2);
npc_delay(1);
npc_say("You cannot destroy what is already dead...");
```

**Phase 2 — Shade Form (80 HP)**

- Max hit: 14
- Attack speed: 3 ticks (faster)
- Attack style: Crush melee
- **Weakness**: Magic (magic defence = -10)
- **Special — Prayer Drain**: Every hit drains 2-5 prayer points
- **Special — Shadow Step** (every 20 ticks): The Entombed teleports behind the player
  - `npc_tele(movecoord(player_coord, 0, 0, -1))`
  - Visual: `spotanim_npc(smokepuff, 0, 0)` at old position + new position
  - Message: `"The Entombed dissolves into shadow and reappears behind you!"`
- **Special — Summon Shades** (at 40 HP, once): Spawns 2x Crypt Shade at room edges
  - `npc_say("Rise, my servants...")` + `npc_add` x2
  - These shades have reduced HP (30 each) and normal shade stats
  - They do NOT need to be killed to end the phase — only The Entombed's HP matters

**Transition to Phase 3**:
Same pattern as Phase 1→2. Shade collapses, ghost rises from its position.

**Phase 3 — Ghost Form (60 HP)**

- Max hit: 18 (highest damage, but lowest HP)
- Attack speed: 3 ticks
- Attack style: Crush melee
- **Weakness**: Magic (magic defence = -15)
- **Special — Wail of the Damned** (every 12 ticks): Area effect that hits for 4-8 damage and stuns the player for 1 tick (cannot move/attack for 1 tick)
  - `npc_anim(ghost_attack, 0)` + `spotanim_map(curse_impact, player_coord, 92, 0)`
  - Message: `"The Entombed lets out a devastating wail!"`
  - Sound: `sound_synth(ghost_death, 0, 0)` (repurposed as a wail)
- **Desperation** (below 20 HP): Attack speed increases to 2 ticks, max hit stays 18
  - `npc_say("I... will... NOT... return to that prison!")`
- **No prayer drain in this form** — incentivizes using Protect from Melee for the raw damage
- Any summoned shades from Phase 2 that are still alive despawn when Phase 3 begins

**Final Death**:
- Standard ghost_death animation at full 200% size — dramatic
- `spotanim_map(teleport_casting, coord, 92, 0)` — large flash
- `sound_synth(ghost_death, 0, 0)`
- Message: `"The Entombed finally collapses. Peace returns to the crypt."`
- 3-tick delay, then:
  - Blessed staircase loc spawns on north wall: `loc_add(blessed_exit_stairs_coord, cryptstairsup, ...)`
  - Message: `"A staircase of light appears to the north."`
  - `%crypt_tokens = add(%crypt_tokens, 150)`
  - `%crypt_wave = 11` (marks completion)

### Boss Strategy Summary

| Phase | Style | Weakness | Key Threat | Counter |
|-------|-------|----------|------------|---------|
| Skeleton | Slash | Crush weapons | Bone Volley (unavoidable 3-6) | Tank with food |
| Shade | Crush | Magic spells | Prayer drain + Shadow Step | Magic attacks, bring prayer pots |
| Ghost | Crush | Magic spells | Wail of the Damned (AoE stun) + raw damage | Protect from Melee, sustained magic DPS |

**Recommended gear**: Bring melee (crush weapon like warhammer) for Phase 1, switch to magic for Phases 2-3. Bring prayer potions for shade phase. Total fight duration: roughly 2-3 minutes depending on gear.

---

## 7. Between-Wave Phase

### Timing

After each wave (except wave 10), the player has **50 ticks (30 seconds)** before the next wave begins. A countdown message appears:

- At 50 ticks: `"The next wave begins in 30 seconds. Prepare yourself."`
- At 25 ticks: `"15 seconds until the next wave..."`
- At 8 ticks: `"The dead stir once more..."`
- At 0 ticks: Next wave spawns

### Coffin Looting

Each room has 2-4 coffins. During the between-wave phase, coffins become interactable (op1=Search). Each coffin can only be looted once per run.

**Coffin Interaction**:
```
[oploc1,crypt_coffin_closed]
p_arrivedelay;
anim(human_openchest, 0);
sound_synth(coffin_open, 0, 0);
p_delay(1);
loc_change(crypt_coffin_open, 500);
// Roll loot table
~crypt_coffin_loot;
```

**Coffin Loot Table** (rolled per coffin):

| Roll (d100) | Loot | Notes |
|-------------|------|-------|
| 1-25 | Empty | `"The coffin is empty."` |
| 26-40 | 2x Lobster | Basic food |
| 41-52 | 2x Swordfish | Better food |
| 53-60 | 1x Prayer potion (3) | Crucial for shade waves |
| 61-68 | 15x Fire rune + 10x Air rune | Spell ammo |
| 69-75 | 10x Chaos rune | Better spell ammo |
| 76-82 | 5x Death rune | High-tier spell ammo |
| 83-88 | 1x Super attack (3) or 1x Super strength (3) | 50/50 |
| 89-93 | 1x Shark | Premium food |
| 94-97 | 1x Super restore (3) | Restores all stats + prayer |
| 98-100 | 1x Rune full helm or 1x Rune platelegs | Rare bonus gear (equip or alch) |

Average value per coffin: roughly 1-2 food items or a potion dose. Players can't rely on coffins alone — they need to bring their own supplies.

### Room Transition

After the between-wave phase, if the next wave is in a new room:
1. A door/passage in the south wall opens (loc_change from wall to open doorway)
2. Message: `"The passage to the next chamber has opened."`
3. Player walks through the corridor to the next room
4. The passage behind them seals (loc_change back to wall)
5. Next wave spawns after the player enters the room

---

## 8. Point System

### Token Earning

Crypt Tokens are tracked in `%crypt_tokens` (a permanent varp with `transmit=yes` so it can display on an interface).

| Source | Tokens |
|--------|--------|
| Wave 1 | 10 |
| Wave 2 | 15 |
| Wave 3 | 25 |
| Wave 4 | 35 |
| Wave 5 (mini-boss) | 60 |
| Wave 6 | 40 |
| Wave 7 | 50 |
| Wave 8 | 60 |
| Wave 9 | 70 |
| Wave 10 (final boss) | 150 |
| **Full clear total** | **515** |

### Bonus Tokens

| Bonus | Tokens | Condition |
|-------|--------|-----------|
| Flawless Run | +50 | Complete all 10 waves without dying (no deaths, obviously) |
| Speed Clear | +30 | Complete all 10 waves in under 500 ticks (~5 minutes) |
| Prayer Warrior | +25 | Kill 5+ enemies using only protection prayers active (take no food between waves 6-9) |

Bonuses are checked at the end of the run and added to `%crypt_tokens` before the exit message.

### Token Persistence

Tokens persist across sessions (varp scope=perm). Players accumulate tokens over multiple runs. A typical run where the player reaches wave 7 before dying earns roughly 235 tokens. A full clear earns 515 (+ up to 105 bonus).

### Economy Targets

| Reward | Cost | Runs to Earn (full clear) | Runs to Earn (wave 7 avg) |
|--------|------|---------------------------|---------------------------|
| Blessed Bone Shards | 50 | ~1 | ~1 |
| Crypt Teleport Tab | 25 | ~1 | ~1 |
| Ghostspeak Scroll | 150 | ~1 | ~1 |
| Crypt Archer's Coif | 350 | ~1 | ~2 |
| Sanctified Mace | 900 | ~2 | ~4 |
| Shroud of the Crypt | 1500 | ~3 | ~7 |

---

## 9. Reward Shop

### Shop Interface

Brother Cedwyn's op3=Trade opens a standard shop-style interface. Instead of coins, prices are denominated in Crypt Tokens displayed from `%crypt_tokens`.

### Reward Items

| # | Item | Token Cost | Category |
|---|------|-----------|----------|
| 1 | Blessed Bone Shards (x100) | 50 | Consumable (Prayer XP) |
| 2 | Crypt Teleport Tab | 25 | Consumable (teleport) |
| 3 | Ghostspeak Scroll | 150 | Consumable (utility) |
| 4 | Crypt Archer's Coif | 350 | Equipment (ranged helm) |
| 5 | Sanctified Mace | 900 | Equipment (melee weapon) |
| 6 | Shroud of the Crypt | 1500 | Equipment (magic cape) |

See [Section 10](#10-item-definitions) for full item specs.

---

## 10. Item Definitions

### Blessed Bone Shards

```
[blessed_bone_shards]
name=Blessed bone shards
desc=Fragments of bone imbued with holy energy. Can be buried for enhanced Prayer experience.
model=model_545_obj           // bone model (reuses existing bone item model)
recol1s=29562
recol1d=22218                 // lighter/golden tint
stackable=yes
tradeable=no
members=yes
cost=0
iop1=Bury
```

- **Use**: Bury for 1,500 Prayer XP each (100 shards = one bury, yielding 1,500 XP)
- **Actually**: Sold in packs of 100 for 50 tokens. Each pack is one "Bury" action for 1,500 XP.
- **No**: Individual shards. One item called "Blessed bone shards" that represents a bundle.

### Crypt Teleport Tab

```
[crypt_teleport_tab]
name=Crypt teleport tab
desc=A clay tablet inscribed with a teleportation spell. Teleports you to the Mausoleum.
model=model_591_obj           // existing tablet model
recol1s=6273
recol1d=4226                  // dark purple tint
stackable=yes
tradeable=no
members=yes
cost=0
iop1=Break
```

- **Use**: Break to teleport to the Mausoleum entrance (near Brother Cedwyn). Single use, consumed on use.

### Ghostspeak Scroll

```
[ghostspeak_scroll]
name=Ghostspeak scroll
desc=A scroll of incantation that grants temporary ghostspeak ability.
model=model_587_obj           // existing scroll model
stackable=yes
tradeable=no
members=yes
cost=0
iop1=Read
```

- **Use**: Read to gain 30 minutes (3,000 ticks) of Ghostspeak ability without needing the amulet equipped. Timer tracked via a soft timer on the player. Consumes the scroll.

### Crypt Archer's Coif

```
[crypt_archers_coif]
name=Crypt archer's coif
desc=A coif woven with blessed thread, favoured by the templar rangers of old.
model=model_191_obj_wear      // existing coif wear model
recol1s=9516
recol1d=8                     // dark grey/black
wearpos=head
wearmodel1=model_191_obj_wear
weight=1
tradeable=no
members=yes
cost=25000

// Combat bonuses
astab=0
aslash=0
acrush=0
amagic=3
arange=7
dstab=4
dslash=6
dcrush=8
dmagic=4
drange=4
rstr=0
prayer=1
```

Comparison to existing coifs:
- Leather coif: +2 range attack, +2/3/4 def
- This: +7 range attack, +4/6/8 def, +3 magic attack, +1 prayer — significantly better, appropriate for its 350-token cost

### Sanctified Mace

```
[sanctified_mace]
name=Sanctified mace
desc=A heavy mace blessed by the templars. It strikes with holy fury against the undead.
model=model_1059_obj          // existing mace object model (rune mace style)
model2=model_1060_obj_wear    // wear model
recol1s=6273
recol1d=22218                 // golden/light tint
wearpos=rhand
wearmodel1=model_1060_obj_wear
weight=5
tradeable=no
members=yes
cost=50000

// Combat bonuses (crush weapon — between rune mace and dragon)
astab=-2
aslash=-2
acrush=52
amagic=0
arange=0
dstab=0
dslash=0
dcrush=0
dmagic=0
drange=0
str=48
prayer=5
```

- Speed: 5 (same as other maces)
- Comparable to rune mace (+47 crush, +44 str, +4 prayer) but with +5 crush, +4 str, +1 prayer
- **Special property**: +10% damage bonus against NPCs with `param=undead,^true`. Implemented in the hit calc — if target has undead param, max hit increased by 10%.

### Shroud of the Crypt

```
[shroud_of_the_crypt]
name=Shroud of the crypt
desc=A dark cape woven from blessed cloth. It hums with protective energy.
model=model_322_obj           // existing cape object model
model2=model_323_obj_wear     // cape wear model
recol1s=926
recol1d=4226                  // dark purple
wearpos=back
wearmodel1=model_323_obj_wear
weight=1
tradeable=no
members=yes
cost=75000

// Combat bonuses (defensive magic cape with prayer)
astab=0
aslash=0
acrush=0
amagic=8
arange=0
dstab=3
dslash=3
dcrush=3
dmagic=8
drange=3
str=0
prayer=4
```

- Best-in-slot magic cape for this era (+8 magic attack/defence, +4 prayer)
- Trades offensive capability for prayer bonus — useful for the crypts themselves and general magic training
- High token cost (1,500) ensures it takes dedication to earn

---

## 11. Crypt Keeper Dialogue

### First Meeting (Priest in Peril complete)
e
```runescript
[opnpc1,brother_cedwyn]
if (%priestperil < ^priestperil_complete) {
    ~chatnpc("<p,neutral>I'm sorry, but you cannot help me until the temple is secured. Speak with Drezel first.");
    return;
}
if (%crypt_wave > 0 & %crypt_wave < 11) {
    @cedwyn_in_progress;
}
@cedwyn_main_menu;

[@cedwyn_main_menu]
~chatnpc("<p,worried>Greetings, adventurer. The crypts beneath us grow more restless by the day.");
~chatnpc("<p,sad>Since the Salve's power was disrupted, the dead have begun to stir. Ancient templars, once at peace, now rise as monsters.");
def_int $choice = ~p_choice3("How can I help?", "What's in it for me?", "Sounds dangerous. No thanks.");

if ($choice = 1) {
    @cedwyn_explain;
} else if ($choice = 2) {
    @cedwyn_rewards;
} else {
    ~chatplayer("<p,neutral>Sounds dangerous. No thanks.");
    ~chatnpc("<p,sad>I understand. The crypts are not for the faint of heart. Return if you change your mind.");
}

[@cedwyn_explain]
~chatplayer("<p,neutral>How can I help?");
~chatnpc("<p,neutral>I need someone to descend into the crypts and fight back the undead. They reform eventually, but each cleansing buys us time.");
~chatnpc("<p,neutral>The work is dangerous. I'll need 5,000 coins to prepare blessed supplies and mark the safe passages for you.");
~chatnpc("<p,neutral>For every wave of undead you defeat, I'll reward you with Crypt Tokens. You can exchange them with me for unique equipment and supplies.");
def_int $choice2 = ~p_choice2("I'm ready. Let me in. (5,000 coins)", "Let me prepare first.");

if ($choice2 = 1) {
    @cedwyn_enter;
} else {
    ~chatplayer("<p,neutral>Let me prepare first.");
    ~chatnpc("<p,neutral>Very well. Return when you're ready. Bring food, potions, and your best weapon. Crush weapons work well against the skeletons, and magic is effective against the spirits.");
}

[@cedwyn_rewards]
~chatplayer("<p,neutral>What's in it for me?");
~chatnpc("<p,neutral>For each wave you survive, you'll earn Crypt Tokens. You can trade these tokens with me for blessed equipment and supplies.");
~chatnpc("<p,neutral>I have a sanctified mace that strikes with holy fury, an enchanted shroud, and more. Would you like to see my wares?");
def_int $choice3 = ~p_choice2("Show me the shop.", "Maybe later.");

if ($choice3 = 1) {
    // Open reward shop interface
    @cedwyn_open_shop;
} else {
    ~chatplayer("<p,neutral>Maybe later.");
}

[@cedwyn_enter]
if (inv_total(inv, coins) < 5000) {
    ~chatnpc("<p,sad>You don't have enough coins. I need 5,000 to prepare the passages.");
    return;
}
if (stat_base(hitpoints) < 10) {
    ~chatnpc("<p,worried>You don't look strong enough. Come back when you've trained more.");
    return;
}
inv_del(inv, coins, 5000);
~chatnpc("<p,neutral>May Saradomin protect you down there. Fight well, and return alive.");
// Teleport player into Room 1
p_telenpc(crypt_room1_entry_coord);
%crypt_wave = 1;
%crypt_coffins_looted = 0;
// Begin wave 1 after 5 tick delay
queue(crypt_start_wave, 5);

[@cedwyn_in_progress]
~chatnpc("<p,worried>You're still in the middle of a run! The crypts await you below.");
~chatnpc("<p,neutral>If you've left the crypts, your progress has been preserved. Would you like to re-enter?");
def_int $choice4 = ~p_choice2("Send me back in.", "I'd like to forfeit this run.");

if ($choice4 = 1) {
    @cedwyn_reenter;
} else {
    ~chatplayer("<p,neutral>I'd like to forfeit this run.");
    %crypt_wave = 0;
    ~chatnpc("<p,sad>Very well. Your earned tokens have been kept. Come back when you're ready for another attempt.");
}
```

### Returning Player (has tokens)

```runescript
// Added to main menu if %crypt_tokens > 0
// After standard greeting:
~chatnpc("<p,neutral>You currently have %crypt_tokens Crypt Tokens. Would you like to spend them, or venture into the crypts again?");
```

### After First Full Clear

```runescript
// Special dialogue triggered on first %crypt_wave = 11 completion
~chatnpc("<p,happy>Incredible! You've defeated The Entombed itself! The crypts are cleansed... for now.");
~chatnpc("<p,neutral>The dead will rise again in time. But you've proven yourself a true champion of the light.");
~chatnpc("<p,neutral>You're welcome to descend again whenever you wish. Each cleansing helps.");
```

---

## 12. Implementation File Structure

```
content/scripts/minigames/game_mausoleum_crypts/
├── configs/
│   ├── mausoleum_crypts.constant     // wave definitions, point values, timing
│   ├── mausoleum_crypts.varp         // %crypt_wave, %crypt_tokens, %crypt_path, %crypt_coffins_looted
│   ├── mausoleum_crypts.npc          // all NPC definitions (brother_cedwyn + all enemies)
│   ├── mausoleum_crypts.obj          // reward items
│   ├── mausoleum_crypts.loc          // crypt-specific locs (crypt_coffin_closed, crypt_coffin_open, blessed_exit_stairs)
│   └── mausoleum_crypts.inv          // reward shop inventory
├── scripts/
│   ├── brother_cedwyn.rs2            // NPC dialogue, entry logic, shop
│   ├── crypt_waves.rs2               // wave spawning, completion detection, transitions
│   ├── crypt_bosses.rs2              // Crypt Guardian + The Entombed AI and phase transitions
│   ├── crypt_enemies.rs2             // standard enemy AI (mostly using default combat, shade prayer drain)
│   ├── crypt_coffins.rs2             // coffin interaction, loot tables
│   ├── crypt_rewards.rs2             // shop interaction, token spending, item effects
│   └── crypt_items.rs2               // item use scripts (bone shards, teleport tab, ghostspeak scroll, sanctified mace undead bonus)
└── maps/
    └── (generated .jm2 files for crypt rooms)
```

### Config File Contents

#### mausoleum_crypts.constant

```
// Wave timing
^crypt_intermission_ticks = 50
^crypt_spawn_stagger_ticks = 3

// Point values per wave
^crypt_wave1_points = 10
^crypt_wave2_points = 15
^crypt_wave3_points = 25
^crypt_wave4_points = 35
^crypt_wave5_points = 60
^crypt_wave6_points = 40
^crypt_wave7_points = 50
^crypt_wave8_points = 60
^crypt_wave9_points = 70
^crypt_wave10_points = 150

// Bonus points
^crypt_bonus_flawless = 50
^crypt_bonus_speed = 30
^crypt_bonus_prayer = 25

// Speed clear threshold (ticks)
^crypt_speed_threshold = 500

// Entry fee
^crypt_entry_fee = 5000

// Boss thresholds
^guardian_phase2_hp = 40
^entombed_shade_summon_hp = 40
^entombed_ghost_desperation_hp = 20

// Wave enemy counts
^wave1_zombies = 3
^wave2_zombies_weak = 2
^wave2_zombies_mid = 2
^wave3_skeletons = 3
^wave3_rangers = 1
^wave4_skeletons = 2
^wave4_rangers = 2
^wave4_elites = 1
^wave6_ghosts = 3
^wave6_zombies = 2
^wave7_vengeful = 2
^wave7_zombies = 3
^wave7_ghosts = 1
^wave8_shades = 4
^wave8_zombies = 1
^wave9_wraiths = 3
^wave9_shades = 2

// Item costs (tokens)
^cost_bone_shards = 50
^cost_teleport_tab = 25
^cost_ghostspeak_scroll = 150
^cost_archers_coif = 350
^cost_sanctified_mace = 900
^cost_shroud_of_crypt = 1500
```

#### mausoleum_crypts.varp

```
[crypt_wave]
scope=temp
transmit=yes

[crypt_tokens]
scope=perm
transmit=yes

[crypt_path]
scope=temp

[crypt_coffins_looted]
scope=temp

[crypt_start_tick]
scope=temp

[crypt_ghostspeak_timer]
scope=temp
```

Note: `%crypt_wave` uses `scope=temp` — resets on logout (treated as forfeit). `%crypt_tokens` uses `scope=perm` — persists forever.

---

## 13. Technical Patterns

### Wave Spawning Pattern

Based on the Mage Arena and Infernus patterns:

```runescript
[queue,crypt_start_wave]
// Determine which wave we're on
switch_int (%crypt_wave) {
    case 1: @crypt_spawn_wave1;
    case 2: @crypt_spawn_wave2;
    case 3: @crypt_spawn_wave3;
    // ... etc
    case 10: @crypt_spawn_wave10;
}

[@crypt_spawn_wave1]
mes("Wave 1: The dead begin to rise...");
sound_synth(zombie_attack, 0, 0);
def_coord $room1_center = 0_XX_YY_ZZ_ZZ;  // Room 1 center coord

// Staggered spawning
~crypt_spawn_npc($room1_center, crypt_zombie_weak, 1, -2);
p_delay(^crypt_spawn_stagger_ticks);
~crypt_spawn_npc($room1_center, crypt_zombie_weak, -1, 2);
p_delay(^crypt_spawn_stagger_ticks);
~crypt_spawn_npc($room1_center, crypt_zombie_weak, 2, 0);

[proc,crypt_spawn_npc](coord $center, npc $type, int $offsetX, int $offsetZ)
def_coord $spawn = movecoord($center, $offsetX, 0, $offsetZ);
spotanim_map(smokepuff, $spawn, 0, 0);
p_delay(1);
npc_add($spawn, $type, 0);
npc_setmode(applayer2);
```

### Enemy Death / Wave Completion Detection

```runescript
[ai_queue3,crypt_zombie_weak]
gosub(npc_death);
~crypt_check_wave_complete;

[ai_queue3,crypt_zombie_mid]
gosub(npc_death);
~crypt_check_wave_complete;

// ... same for all crypt enemy types

[proc,crypt_check_wave_complete]
if (npc_findhero = ^false) { return; }
// Count remaining crypt enemies in the room
def_int $remaining = 0;
npc_huntall(npc_coord, 15, 0);
while (npc_findnext = ^true) {
    if (npc_category = crypt_zombie | npc_category = crypt_skeleton | npc_category = crypt_ghost | npc_category = crypt_shade) {
        $remaining = add($remaining, 1);
    }
}
if ($remaining = 0) {
    if (p_finduid(uid) = ^true) {
        queue(crypt_wave_complete, 0);
    }
}
```

### Wave Completion Handler

```runescript
[queue,crypt_wave_complete]
def_int $points = ~crypt_get_wave_points(%crypt_wave);
%crypt_tokens = add(%crypt_tokens, $points);
mes("Wave <tostring(%crypt_wave)> complete! You earned <tostring($points)> Crypt Tokens.");

if (%crypt_wave = 10) {
    @crypt_run_complete;
    return;
}

// Enable coffin looting
// Coffins are already in the room — they become searchable via the wave state check

%crypt_wave = add(%crypt_wave, 1);

// Start intermission timer
mes("The next wave begins in 30 seconds. Prepare yourself.");
settimer(crypt_intermission, ^crypt_intermission_ticks);

[timer,crypt_intermission]
cleartimer(crypt_intermission);
queue(crypt_start_wave, 0);
```

### Coffin Loot Script

```runescript
[oploc1,crypt_coffin_closed]
// Only lootable between waves (check that no enemies are alive)
if (~crypt_enemies_alive > 0) {
    mes("You can't loot coffins while enemies are still attacking!");
    return;
}
// Check if already looted this coffin (bitfield)
def_int $coffin_index = ~crypt_get_coffin_index(loc_coord);
if (testbit(%crypt_coffins_looted, $coffin_index) = ^true) {
    mes("You've already searched this coffin.");
    return;
}
p_arrivedelay;
anim(human_openchest, 0);
sound_synth(coffin_open, 0, 0);
p_delay(1);
loc_change(crypt_coffin_open, 500);
%crypt_coffins_looted = setbit(%crypt_coffins_looted, $coffin_index);
~crypt_coffin_loot;

[proc,crypt_coffin_loot]
def_int $roll = random(100);
if ($roll < 25) {
    mes("The coffin is empty.");
} else if ($roll < 40) {
    inv_add(inv, lobster, 2);
    mes("You find some lobsters in the coffin.");
} else if ($roll < 52) {
    inv_add(inv, swordfish, 2);
    mes("You find some swordfish in the coffin.");
} else if ($roll < 60) {
    inv_add(inv, prayer_potion3, 1);
    mes("You find a prayer potion in the coffin.");
} else if ($roll < 68) {
    inv_add(inv, fire_rune, 15);
    inv_add(inv, air_rune, 10);
    mes("You find some runes in the coffin.");
} else if ($roll < 75) {
    inv_add(inv, chaos_rune, 10);
    mes("You find some chaos runes in the coffin.");
} else if ($roll < 82) {
    inv_add(inv, death_rune, 5);
    mes("You find some death runes in the coffin.");
} else if ($roll < 88) {
    if (random(2) = 0) {
        inv_add(inv, super_attack3, 1);
        mes("You find a super attack potion in the coffin.");
    } else {
        inv_add(inv, super_strength3, 1);
        mes("You find a super strength potion in the coffin.");
    }
} else if ($roll < 93) {
    inv_add(inv, shark, 1);
    mes("You find a shark in the coffin.");
} else if ($roll < 97) {
    inv_add(inv, super_restore3, 1);
    mes("You find a super restore potion in the coffin.");
} else {
    if (random(2) = 0) {
        inv_add(inv, rune_full_helm, 1);
        mes("You find a rune full helm in the coffin!");
    } else {
        inv_add(inv, rune_platelegs, 1);
        mes("You find rune platelegs in the coffin!");
    }
}
```

### Boss Phase Transition (The Entombed)

Following the Nazastarool pattern from `quest_zombiequeen`:

```runescript
[ai_queue3,the_entombed_skeleton]
gosub(npc_death);
if (npc_findhero = ^false) { return; }
def_coord $coord = npc_coord;
def_int $uid = uid;

// Phase 1 death sequence
npc_setmode(none);
npc_anim(skeleton_trans_death, 0);
spotanim_npc(smokepuff_large, 124, 34);
npc_delay(2);
npc_del;

// Spawn Phase 2
if (p_finduid($uid) = ^true) { p_delay(1); }
npc_add($coord, the_entombed_shade, 1000);
spotanim_npc(smokepuff_large, 124, 34);
npc_anim(human_transdeath, 0);  // reverse death as "rising" effect
npc_setmode(applayer2);
npc_delay(1);
npc_say("You cannot destroy what is already dead...");

[ai_queue3,the_entombed_shade]
gosub(npc_death);
if (npc_findhero = ^false) { return; }
def_coord $coord = npc_coord;
def_int $uid = uid;

// Despawn any remaining summoned shades
npc_huntall($coord, 15, 0);
while (npc_findnext = ^true) {
    if (npc_type = crypt_shade) { npc_del; }
}

// Phase 2 death sequence
npc_setmode(none);
npc_anim(human_transdeath, 0);
spotanim_npc(smokepuff_large, 124, 34);
npc_delay(2);
npc_del;

// Spawn Phase 3
if (p_finduid($uid) = ^true) { p_delay(1); }
npc_add($coord, the_entombed_ghost, 1000);
spotanim_npc(teleport_casting, 0, 0);
npc_setmode(applayer2);
npc_delay(1);
npc_say("FOOLS... I TRANSCEND FLESH AND SHADOW!");

[ai_queue3,the_entombed_ghost]
gosub(npc_death);
if (npc_findhero = ^false) { return; }

// Final boss death — run complete
spotanim_map(teleport_casting, npc_coord, 92, 0);
sound_synth(ghost_death, 0, 0);

if (p_finduid(uid) = ^true) {
    queue(crypt_wave_complete, 3);
}
```

### Shade Prayer Drain

```runescript
[ai_applayer2,crypt_shade_strong]
// Standard combat AI
gosub(npc_default_melee_attack);
// After a successful hit, drain prayer
if (%npc_last_hit_damage > 0) {
    if (p_finduid(%npc_aggressive_player) = ^true) {
        def_int $drain = calc(2 + random(4));  // 2-5 prayer drain
        stat_sub(prayer, $drain, 0);
        mes("The Shadow Wraith drains your prayer!");
    }
}
```

### Sanctified Mace Undead Bonus

```runescript
[proc,crypt_mace_bonus](int $max_hit) -> (int)
// Check if player is wielding sanctified mace
if (inv_total(worn, sanctified_mace) < 1) {
    return ($max_hit);
}
// Check if target is undead
if (npc_param(undead) = ^true) {
    def_int $bonus = calc($max_hit / 10);  // 10% bonus, rounded down
    return (add($max_hit, $bonus));
}
return ($max_hit);
```

### Ghostspeak Scroll Effect

```runescript
[opheld1,ghostspeak_scroll]
if (inv_total(inv, ghostspeak_scroll) < 1) { return; }
inv_del(inv, ghostspeak_scroll, 1);
anim(human_readbook, 0);
mes("You read the ghostspeak scroll. You can understand ghosts for 30 minutes.");
%crypt_ghostspeak_timer = add(map_clock, 3000);
// The ghost dialogue scripts check: if worn(ghostspeak_amulet) OR %crypt_ghostspeak_timer > map_clock
```

---

## Appendix A: Combat Difficulty Analysis

### Estimated Damage Per Wave

| Wave | Enemies | Est. DPS to Player | Duration | Total Damage | Food Needed |
|------|---------|--------------------|-----------|--------------|----|
| 1 | 3 weak zombies | ~3/tick | 20 ticks | ~60 | 3 lobs |
| 2 | 4 mixed zombies | ~5/tick | 25 ticks | ~125 | 5 lobs |
| 3 | 4 skeletons | ~6/tick | 30 ticks | ~180 | 7 lobs |
| 4 | 5 mixed skeletons | ~8/tick | 35 ticks | ~280 | 10 swords |
| 5 | Guardian + 2 skeletons | ~10/tick | 45 ticks | ~450 | 15 swords |
| 6 | 3 ghosts + 2 zombies | ~7/tick | 30 ticks | ~210 | 8 swords |
| 7 | 6 mixed ghosts/zombies | ~10/tick | 40 ticks | ~400 | 14 swords |
| 8 | 4 shades + 1 zombie | ~9/tick + prayer drain | 35 ticks | ~315 + prayer | 12 swords + ppot |
| 9 | 3 wraiths + 2 shades | ~12/tick + heavy drain | 40 ticks | ~480 + prayer | 16 swords + 2 ppot |
| 10 | The Entombed (3 phases) | ~8-14/tick varies | 80 ticks | ~800 | 20+ sharks |

**Total resources for a full clear (estimated)**:
- ~20 sharks or equivalent high-tier food
- ~15 lobsters/swordfish (supplemented by coffins)
- 3-4 prayer potions (4-dose)
- Combat potions optional but recommended for waves 8-10
- Runes if using magic for ghost/shade phases

This positions the minigame as a meaningful resource sink. Even with coffin loot, players spend more than they receive.

### Recommended Combat Levels

| Level Range | Expected Performance |
|------------|---------------------|
| 50-60 | Can reach waves 4-5 with effort |
| 60-70 | Can reach waves 6-7 consistently |
| 70-80 | Can reach wave 9, boss is challenging |
| 80+ | Full clears achievable with good gear |
| 90+ | Comfortable full clears, speed clear bonus possible |

---

## Appendix B: Lore Extensions

### Brother Cedwyn's Backstory

Cedwyn was a young monk at the Paterdomus monastery. When the events of Priest in Peril weakened the Salve barrier, Cedwyn was the first to notice the disturbances in the Mausoleum crypts. While older monks debated what to do, Cedwyn established a vigil at the entrance, warning travelers and seeking adventurers who could help.

He has no combat ability himself — his role is purely as a quest giver, shopkeeper, and lore source. He can provide information about the templars, the history of the crypts, and the nature of each type of undead encountered within.

### The Templars

The buried templars were Saradominist warriors who defended the Salve barrier during the God Wars. Their spirits were kept at peace by holy wards embedded in the crypt architecture. The weakening of the Salve has disrupted these wards, and the templars' spirits have become confused and hostile — they no longer recognize the living as allies.

**The Entombed** is the tomb's original champion — a templar commander of great power. In life, he swore an oath to guard the crypts forever. In undeath, that oath has been twisted into a compulsion to attack all intruders. His three-phase transformation represents the stages of his corruption: first his body rises (skeleton), then his spirit darkens (shade), and finally his pure soul — now wrathful — manifests (ghost).

### Morytania Tie-in

The crypts' corruption is directly linked to the events of Priest in Peril. If future content restores the Salve's full power, the crypts could canonically become easier or have reduced waves. This creates a narrative thread that ties the minigame into the broader Morytania storyline.

The Mausoleum's existing holy barrier (referenced in `holy_barrier.rs2`) serves as a lore explanation for why the undead don't spill out into the overworld — the barrier contains them within the crypts but isn't strong enough to suppress them entirely.
