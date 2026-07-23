# The Tempest Rev 1 "High-Score Cheat": The Authoritative Guide

## The lore, the complete effect table, the exact fault, and the full anatomy of Atari's misfiring copy protection

**Project:** Tempest ROM analysis (rev 1 program chips vs. rev 2/3, checked against Atari's original program source)<br>
**Document date:** July 22, 2026<br>
**Supersedes:** `tempest_rev1_cheat_analysis.md` and `_v2` (both retained)

## Executive summary

The first production version of Tempest (Atari, 1981) contains the most famous "cheat" in early arcade history: finish a game with a high enough score, and the last two digits of that score select an effect — 40 free credits, 255 lives, a level select, a self-steering blaster, a drifting screen. This guide establishes, from the game's actual program code and Atari's original source files, exactly what every last-two-digit value from 00 to 99 does, why the phenomenon exists at all, and why it behaves in the peculiar ways players remember: the required score, the wait through attract mode before anything happens, and the related mystery resets at levels 11 and 14.

The short version. Tempest ships with six independent anti-counterfeiting booby traps, written by programmer Dave Theurer to punish bootleg boards with delayed, deniable malfunctions. One trap verifies a checksum of the on-screen "© MCMLXXX ATARI" copyright line; its punishment, once the score passes 180,000, is to repeatedly increment whichever internal game variable is named by the last two digits of the score. That punishment was never supposed to reach the public — but a single wrong byte in the checksum's list of acceptable answers (address $B1F1: the constant $2A where $29 was intended, making the verifier accept a value no real machine produces while rejecting one every real machine produces) armed it on genuine boards. Every effect in forty years of folklore is just the map of what happens to live in the game's first 256 memory slots. Atari's revision 2, shipped December 1981, corrects the one byte and nothing else in the security system; the cheat exists only on revision 1.

This guide also settles the folklore's loose ends. The middle-digit ("BB") conditions in some circulated lists have no basis in the code. The score threshold is exactly 180,000 (Atari's own service bulletin said 170,000; the code says otherwise). The level 11 and level 14 resets come from two *different* traps that are not part of the score cheat, cannot fire on genuine unmodified hardware, and are the expected symptom of running modified or copied ROMs. And the wait through an attract cycle or two after your game ends is real, with a specific cause: the warning flag is recomputed every time the copyright line is drawn, and the screen layout that trips the bug comes around only partway through the attract rotation.

## 1. The lore: forty years of circulating descriptions

No single "canonical" description of the cheat exists. What exists is a family of accounts — a primary paper trail from Atari itself, one first-person account from the programmer, two modern technical write-ups, and a cloud of player-lore variants that agree on the core and contradict each other at the edges. Surveying them is worth doing before touching the code, because the disagreements are themselves diagnostic: the claims that appear in every independent lineage turn out to be exactly the ones the code supports.

### 1.1 The primary sources

**Atari's own service bulletin (December 4, 1981)** is the earliest document. It tells operators: "If the score on your Tempest is greater then 170,000, there is a 12% chance that a program bug may award 40 credits for one quarter." It offered a replacement ROM (part 136002-217, swapped at board location J1), noted that upright cabinets before serial number 17426 were affected while cabaret and cocktail cabinets shipped already fixed, and framed the whole thing purely as a "program bug" — no effect list, no mention of lives or level select, and certainly no mention of copy protection. Two details are notable in hindsight: the threshold is stated as 170,000 where the code says 180,000, and the odd "12% chance" figure is about what you get if six of the roughly fifty reachable two-digit endings award credits — Atari was summarizing statistically what players would later enumerate.

**Dave Theurer's first-person account** (given decades later) is the only source that explains the mechanism from the inside. He describes building "about 6 levels of protection to prevent things such as removal of 'Atari' from the various screens by unscrupulous... counterfeiters," with failure responses that would "trash RAM at random (sometimes using the last few digits of the score as the random number), sometimes after a delay of a few minutes." On the bug itself: "Just before shipping, I rearranged some of the screens, moving the Atari logo more towards the center on one of them, and I forgot to change the 3rd or 4th level checksum protecting that location." Asked whether the fiasco was his fault: "Yes... But it was intended as protection against piracy." He also recounted that when operator complaints arrived within weeks, he first blamed hardware, until an Atari lab technician with a hardware analyzer caught the software itself corrupting the coin counter. More than 17,000 upright machines had already shipped.

**Modern technical write-ups** — a detailed 2023 video analysis and its companion wiki page, plus an independent confirmation from the author of a Vectrex-hardware Tempest emulator — correctly identify the broad architecture: multiple checks covering the copyright text, the code that draws it, and the POKEY sound chips' random-number hardware, with one check misfiring on genuine boards and thereby giving players "a way to overwrite Tempest variables in a defined way (based on the last score digits)." These accounts are accurate as far as they go; this guide goes further (the complete effect table, the exact fault arithmetic, the gate semantics, and the timing behavior).

### 1.2 The player-lore variants

The folk descriptions, spread through 1980s magazines (a UK player traces his recipe to Computer & Video Games around 1984), tip books, BBS and Usenet posts, and later websites, share a stable core: score of the form AABBCC; AA high enough; ending digits CC = 06, 11, 12, 16, 17, or 18 for 40 free credits; 46 for a level select reaching level 81; 48 for 255 lives; 05 to play during attract mode; assorted "drift" effects in the 60s. Around that core, the variants disagree on almost everything else:

- **The threshold.** Over 160,000; over 170,000 (Atari's bulletin); over 180,000; over 190,000; "between 189,999 and 208,000"; "you must complete level 8 first." One number is right (Section 8).
- **The middle digits.** One lineage of sources — evidently copying a common ancestor — adds "BB must be between 30 and 59." No first-hand account and no technical write-up includes it, and nothing in the code tests the middle digits. It is folklore accretion, probably an over-generalization from the particular scores on which someone happened to see effects.
- **Intent.** Most accounts call it a bug; one widely-copied database entry asserts the codes "were intentionally programmed in as a security measure" — wrong in the sense intended (they were not a deliberate backdoor), right in a sense its author didn't mean (the machinery is deliberate; the codes are its shrapnel).
- **Embellishments.** A 99-credit variant; credits awarded on ending 40; "255 shooters in memory (only 6 display)"; a claim that 1982-83 machines carry an "anti-cheat chip" (they carry a fixed ROM). One practical detail recurs in the good first-hand recipes and turns out to be load-bearing: *end the game, then wait an attract cycle before the effects show.* Section 9 explains it.
- **Adjacent lore.** The score 179,976 sometimes attached to "free games" stories resolves to garbled retellings of the 170,000/180,000 threshold, not a separate phenomenon. And the cheat reached fiction: the novel Ready Player One turns ending a Tempest game on 189,412 (a ...12 code) into a plot point, faithfully applying the folk rule.

Operators, for their part, remember the cheat asymmetrically: word spread, kids drained credit counters, and operators responded by refusing play or swapping in the fixed ROM — which is why surviving rev 1 boards are the minority today.

### 1.3 What the lore gets right and wrong: the scorecard

| Claim | Verdict (from code, Sections 4-9) |
|---|---|
| Last two digits of final score select an effect | Correct, mechanism confirmed |
| Credits for 06/11/12/16/17/18; level select at 46; lives at 48; attract play at 05; drifts in the 60s | Correct in every listed case; the full 00-99 table is in Section 5 |
| Score must exceed 170,000 / 160,000 / 190,000 | All wrong; the gate is exactly 180,000 |
| "BB must be 30-59" | No basis in code |
| "Complete level 8 first" | No basis in code |
| Must wait through attract mode after game over | Correct, and now explained |
| 40-credit cap | Correct in effect (credit counter saturates) |
| Level select "up to 81" | The real limit is the level table's end; 81 is where the game's own level-start logic tops out |
| Works only on rev 1 | Correct; one byte separates rev 1 from rev 2 |

The rest of this guide is the code's own story, told in plain language: the protection system (Sections 3-4), the complete effect table (Section 5), the exact fault and why Theurer missed it (Section 6), the level 11/14/21 traps (Section 7), the thresholds (Section 8), the attract-mode delay (Section 9), and the revision history and verdict (Sections 10-11).

## 2. Evidence base and method

Everything below rests on primary evidence, cross-checked three ways:

- **The revision 1 program chips** (the ten ROMs of an original board, matching MAME's `tempest1` set byte for byte), assembled into the complete 64 KB memory image the game's 6502 processor sees, and disassembled. All addresses cited are processor addresses in that image.
- **The revision 2 and 3 chips**, diffed byte-by-byte against revision 1. Only two chips differ between rev 1 and rev 2 (board locations J1 and P1); the complete classified diff is in Section 10.
- **Atari's original source code** — Theurer's assembly-language files (`tempest/*.MAC` in this repository, from the August 27, 1981 build), which supply the programmers' own variable names and comments, including the security system's remarkably candid annotations: "SECURITY," "VERIFY ATARI LITERAL," "ATARI BETTER BE ON SCREEN," "KILL STACK." The archived variable-layout file is a slightly stale variant of the shipped build, so every memory assignment used in Section 5 was verified against the shipped binary (the layout turns out to match with zero drift, anchored on a dozen independently confirmed variables).
- **A survey of the circulating lore** (Section 1), used as background and as a list of claims to confirm or refute — never as evidence about the code.

One point of translation used throughout: the game numbers its levels ("waves") internally from 0, while the screen displays them from 1. Internal wave 10 is displayed level 11. Lore speaks in displayed levels; the code speaks in internal waves; this guide gives both where it matters.

## 3. The protection system: six booby traps

Arcade bootlegging in 1981 was industrial-scale: counterfeiters copied a hit board outright, often erasing Atari's name and copyright to muddy infringement claims. Tempest's defense is a suite of six independent traps, each split into two halves that run at different times so that no single moment of inspection reveals cause and effect:

- A **verifier** runs quietly during normal operation and computes an arithmetic fingerprint — a checksum — of something every genuine machine has. It stores the result in a flag byte: zero for "authentic," nonzero for "tampered."
- A **consumer** runs continuously but acts only when two things are true at once: its flag is nonzero, *and* the player has progressed past a trigger point (a level or score threshold). Then it sabotages the game in a way engineered to look like an ordinary malfunction.

What the six verifiers check falls into three groups:

1. **The copyright line as drawn on screen.** Two verifiers (call them the display checks) sum the 40 bytes of vector-drawing instructions that the game generates, live, each time it draws "© MCMLXXX ATARI." Erase or alter the on-screen copyright and the sum changes. These two arm the score-steered trap — the cheat.
2. **The program code that draws Atari's name.** Two verifiers checksum stretches of the program itself — the routines and text ("the ATARI literal," in the source's jargon: the letters stored verbatim as data) responsible for putting Atari's name on screen. Patch the name out of the ROM and these fire. They arm the level 11 and level 21 traps.
3. **The POKEY sound chips.** Atari's proprietary POKEY chip contains a hardware random-number generator that steps in a precisely defined sequence. One verifier checks that the generators are actually running (frozen or absent POKEYs mean substitute hardware); another hashes paired reads of both chips' generators under interrupt lockout, where the fixed instruction timing makes the relationship between successive reads exactly predictable on genuine silicon. These arm the level 14 trap and the second score-steered trap respectively.

The six flag/consumer pairs, with their triggers and payloads:

| # | Verifier checks | Flag | Trigger | Sabotage |
|---|---|---|---|---|
| 1 | Copyright line as drawn (all non-play screens) | QT3 | score >= 180,000 | Increment the memory slot named by the score's last two digits, every frame — **the cheat** |
| 2 | Copyright line as drawn (attract mode) | QT6 | (shares trap 1's consumer) | same |
| 3 | POKEY paired-read hash | QT5 | score >= 150,000 | Same increment, aimed at a different memory region |
| 4 | Code that draws the attract-screen logo | QT1 | wave >= 10 (displayed level 11) | Jam the display frame timer every frame — game hangs, watchdog resets it |
| 5 | POKEY generators running at all | QT4 | wave >= 13 (displayed level 14) | Write into the processor's stack every frame — return address corrupted, crash, reset |
| 6 | Code that draws the copyright on the credits screen | QT2 | wave >= 20 (displayed level 21) | Silently switch the processor into decimal arithmetic mode — subtle, spreading corruption |

Note the design logic of the triggers: nothing fires until a machine has been earning money on location for a while in the hands of decent players. A counterfeiter's bench test — power it up, play a few levels, ship it — passes clean. The failures arrive weeks later, scattered, unreproducible, and indistinguishable from the cloner's own workmanship. Theurer's traps were, in the narrow sense, excellent engineering; that is exactly why the one that misfired was so hard to diagnose.

## 4. The cheat's engine: two digits become an address

The consumer for traps 1 and 2 is four short instructions, and they are the entire cheat. Once per frame — sixty times a second — the game runs, in effect:

```text
if (copyright-check flag OR attract-copyright flag) is nonzero:   ; board "tampered"
    if score's top two digits > 17 (i.e., score >= 180,000):
        X = score's last two digits
        add 1 to memory slot number X
```

(In the disassembly, at $A581: `LDA $0455 / ORA $011B / BEQ skip / LDA #$17 / CMP $42 / BCS skip / LDX $40 / INC $00,X`.)

Two facts of the 6502's architecture turn this from vandalism into a steerable cheat:

**The score digits are the address, literally.** Tempest stores the score in binary-coded decimal: each byte holds two decimal digits exactly as displayed. The byte at slot $40 holds the score's last two digits, so a score ending in 46 puts the value $46 into the X register, and `INC $00,X` then increments memory slot $46. No translation, no scrambling: the digits you see on screen *are* the slot number that gets corrupted.

**Slot numbers 00-99 are the game's most important real estate.** The 6502's first 256 memory slots ("page zero") are the fastest to access, so that is where Tempest keeps its live game state: the master state machine, the credit counter, the coin plumbing, the score itself, lives, wave numbers, the spinner input, the 3D camera. The score's last two digits can only name slots $00 through $99 (both digits are 0-9), and that range happens to blanket everything interesting.

The result: the trap's author got the "random, unattributable RAM trashing" he designed — during play, the score's last digits change constantly, so the increments spray across dozens of slots, and the visible symptom on a bootleg is generic flakiness. But the moment a game *ends*, the score freezes, the spray becomes a beam pointed at one slot, and the beam stays on through attract mode (Section 9). Players discovered that aiming the beam — by shooting one last spike to fine-tune the final digits — selected repeatable, useful effects. The cheat is Theurer's weapon, held steady.

The second score-steered trap (flag QT5, threshold 150,000) is the same idea aimed at the next region of memory (slots $0200-$0299 — enemy, cursor, and bookkeeping state), producing overlapping oddities at lower scores on boards where the POKEY hash check failed. On genuine rev 1 boards it stayed silent — the misfiring check was the copyright one — so the famous cheat is specifically the page-zero trap.

Section 5 now does what no circulating account has done: reads off, for all one hundred possible endings 00-99, what lives at that slot and what the beam does to it.

## 5. The complete table: every ending, 00 through 99

How to read this section: "ending" is the last two digits of the final score; the effect described is what one increment per frame does to the variable at that slot, sustained for as long as the trap conditions hold (typically: game over with score >= 180,000, attract mode running — see Section 9 for exactly when the beam is on). Variable names are Atari's own, from the source. This is a code-derived map, not a rumor list; where an identification carries any residual uncertainty it is flagged.

### Endings 00-09: the state machine and the credit counter

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 00 | QSTATE | Master state code; used (doubled) as an index into a table of routine addresses | An odd or out-of-range state sends execution through a garbage address: crash within a frame, then watchdog reset. Folklore's "freeze the screen" |
| 01 | QDSTATE | The same kind of state code for the display system | Misaligned display dispatch: screen garbage, and depending on where the broken lookup lands, other display states' screens — the operator statistics display among them, which is what folklore reports as "view bookkeeping totals." Usually ends in a crash/reset |
| 02 | QNXTSTA | The state to resume after a pause | A time bomb: nothing happens until the next pause ends (new life, wave transition), then the game resumes into a corrupted state and crashes |
| 03 | QFRAME | Free-running frame counter | Counts double-speed. Subtle: blinking text and animation cadences run at twice the rate. Essentially harmless |
| 04 | QTMPAUS | Pause countdown timer | The countdown never reaches zero: the game soft-locks at the next between-wave or between-life pause |
| 05 | QSTATUS | Mode flags; the top bit distinguishes attract mode from a paid game | In attract, the repeated +1s carry into the top bit and flip it: the attract demonstration becomes a controllable game — folklore's "play during attract mode." (If it fires mid-game instead, the game falls into attract logic and malfunctions severely) |
| 06 | CRDT | The credit counter itself | Credits climb at ~60 per second until the counter saturates: the classic "40 free credits" |
| 07 | INTCT | Interrupt tick counter for the coin module | Slight timing skew only; nothing visible |
| 08 | COINA | Raw coin/slam switch snapshot, rewritten ~250x/second by the interrupt handler | Overwritten before it can matter; at most a rare phantom coin |
| 09 | CMODE | Coin-mode option byte, refreshed from the DIP switches every interrupt | Overwritten before use; nothing |

### Endings 10-19: the coin plumbing

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 10, 11, 12 | PSTSL (3 slots) | Per-mechanism coin-validation timers (left/center/right) | Interferes with the validation window a real coin must pass through. On its own: nothing visible (folklore credits 11/12 with free credits — see the note below the table) |
| 13, 14, 15 | CCTIM (3 slots) | Timers that pulse the cabinet's electromechanical coin-tally meters | No screen effect and no credit; the coin meters inside the coin door click over continuously. Tempest has no credit sound — the sound module contains no coin effect at all — so folklore's "credit sound without credit" is this mechanical clicking from the coin door, or embellishment |
| 16 | BCCNT | Coins-counted-toward-bonus counter | Repeatedly crosses the bonus-coin threshold: bonus credits trickle in free |
| 17 | CNCT | Coin units accrued toward a credit | Credits climb: free credits |
| 18 | BC | Bonus coins accrued | Converted into credits: free credits |
| 19 | COLRAM+0 | Zero-page shadow of a color-table entry, loaded once per wave | The shadow is never re-read between waves; nothing visible |

A note on 11 and 12: folklore consistently lists them among the credit codes, but the slots they hit are validation *timers*, not counters — incrementing them does not by itself mint a credit. Two readings are possible: the folk lists absorbed 11/12 by analogy with their neighbors, or sustained corruption of the validation timers combined with switch noise occasionally produces spurious coin acceptance. The three certain credit codes from the coin module are 16, 17, and 18, with 06 hitting the credit counter directly. That four-to-six spread is worth remembering next to Atari's "12% chance" arithmetic.

### Endings 20-39: the silent zone

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 20-28 | COLRAM+7...15 | More per-wave color-table shadows | Nothing visible |
| 29, 30 | TEMP0, TEMPZ | General scratch | Nothing: rewritten before every use |
| 31-34 | MTEMP | Macro-internal scratch | Nothing |
| 35, 36 | SAVEX, SAVEY | Register save temporaries | Nothing |
| 37-39 | INDEX1-3 | Loop counters, initialized before each loop | Nothing |

Twenty consecutive duds — and a first glimpse of why the folk lists look the way they do. Players mapped this territory blind, one high-scoring game per data point; a stretch where nothing ever happened simply never entered the lore.

### Endings 40-49: score, waves, and lives

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 40 | LSCORL | The score's own last two digits (the trap's steering register) | Self-modifying: the ending climbs once per frame, so the beam sweeps forward through slots 41, 42, 43... The +1 is binary, not decimal, so the digits pass 9 and display as non-numeric glyphs |
| 41 | LSCORM | The score's middle two digits | Middle digits churn upward with garbage glyphs — folklore's "last two digits switch" is this churn, misdescribed |
| 42 | LSCORH | The score's top two digits | +10,000 points per frame — "score increases quickly" — and, since this is the byte the trap's own gate tests, the beam keeps itself switched on |
| 43, 44, 45 | RSCORL/M/H | Player 2's score, three bytes | Player 2's score display scrambles and climbs (it is drawn even in a one-player game) |
| 46 | WAVEN1 | Player 1's wave-number ledger (the per-player record; the game works from a separate working copy) | The famous level select — by a two-step relay, not directly: the beam inflates the attract demo's wave ledger, the demo's end-of-game bookkeeping records the inflated value as "highest wave reached," and that record unlocks every rung of the next game's starting-level ladder up to the game's own built-in ceiling, displayed level 81. Section 9.3 traces the full mechanism, including the garbled attract-demo level that players learned to read as the "cheat ready" signal |
| 47 | WAVEN2 | Player 2's wave number | Same effect, player 2 only |
| 48 | LIVES1 | Player 1's lives count | One life per frame. The byte wraps at 255 (folklore's number); on screen the life-icon row floods, bloating the display list with flicker and slowdown — effectively infinite lives |
| 49 | LIVES2 | Player 2's lives | Same, player 2 only |

### Endings 50-59: the controls

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 50 | TBHD | Accumulated spinner-knob count | The game computes knob motion as (current minus previous reading); inflating the current reading fakes steady rotation: the blaster creeps around the rim — "auto pilot" |
| 51 | CURSPO | The blaster's position on the rim | The position itself is pushed one step per frame: same visible creep, applied directly |
| 52 | OTB | The *previous* spinner reading | Fakes rotation the other way: the blaster creeps in the opposite direction. A genuine effect no folk list ever found — the counterpart to 50 that nobody discovered because nobody had a reason to try 52 |
| 53 | FRTIMR | The display frame-pacing timer | Runs double-rate: pacing disturbance, skipped frames, flicker (this is the same timer that trap 4 jams to kill the game at level 11) |
| 54 | BUFRDY | A buffer-status flag the shipped game never reads (vestigial) | Nothing |
| 55-59 | OBJIND, PXL/PYL/PZL, LINSCA | Per-object scratch for the 3D math | Nothing: recomputed before every use |

### Endings 60-69: the camera

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 60 | EZL | The 3D eye (camera) depth position | The viewpoint slides steadily down the tube; the scene warps and periodically snaps — "objects drift down" |
| 61-64 | SXL/SXH/SZL/SZH | Screen-coordinate scratch | Nothing |
| 65 | EYEFAC | Eye-to-screen distance (perspective strength) | Flagged uncertain: the disassembly shows almost no live references. If active, a continuous zoom distortion; more likely nothing |
| 66 | XADJL | Horizontal position of the scene's vanishing point, fine byte | The whole rendered scene — well, enemies, shots — slides sideways one fine step per frame: "objects drift right." Nothing in the game world moves; the picture does |
| 67 | XADJH | The same offset, coarse byte | Each +1 is worth 256 fine steps: the scene relocates in violent hops — "objects jump" |
| 68 | ZADJL | Vertical offset, fine byte | Steady vertical drift — "drift up" |
| 69 | ZADJH | Vertical offset, coarse byte | 256-step vertical strobing. Folklore lists 68 and 69 as the same effect; the code shows why they are adjacent — two bytes of one value — but also that 69 is far more violent than 68 |

### Endings 70-99: the display pipeline

| Ending | Variable | What it is | Effect of the beam |
|---|---|---|---|
| 70-73 | YCOMP, VGSIZE, VGBRIT | Per-vector scratch: size and brightness staging | Nothing, or an occasional one-frame flicker |
| 74 | VGLIST (low) | The write pointer used to build each frame's vector display list | The increment lands mid-build nearly every frame, shifting subsequent drawing commands one byte out of register: persistent sparkle, stray vectors, broken shapes |
| 75 | VGLIST (high) | The same pointer, high byte | Drawing commands land 256 bytes away mid-build: severe screen breakup, and the display processor can run off into garbage — likely watchdog reset |
| 76, 77 | SVGLIST | Saved copy of the list pointer, used to close each frame's list | The list is closed at the wrong address and left unterminated: display runaway — garbage or blank screen, likely reset |
| 78-97 | X0L...Z7H | Unit-vector components for the well's sixteen rim lines | Recomputed each time the well is drawn; at most intermittent one-frame skewing of a well line. (Residual uncertainty: if recomputation is per-wave rather than per-frame, corruption would accumulate as progressive skewing of the well between waves) |
| 98 | INTENS | Per-object draw intensity | Nothing observed reachable |
| 99 | SUBCOU | Point-counter scratch | Nothing |

### What the full map explains about the folklore

Laid out end to end, the map answers a question the folk lists never could: why *these* codes? Of one hundred endings, roughly sixty hit scratch space, shadows, or per-frame-rewritten staging and do nothing a player could ever notice. The lore's fourteen-odd entries are not a curated menu — they are precisely the survivors: the slots that are both *live* (read after the beam writes them) and *persistent* (not rewritten before being read). The map also yields three genuine discoveries the lore missed: ending 52 (autopilot in the reverse direction), ending 02 (the pause-exit time bomb), and endings 74-77 (the display-list corruptions, which players who hit them would have written off as a dying monitor). And it resolves the one systematic error: 11 and 12 sit on validation timers, not credit counters — the certain credit endings are 06, 16, 17, 18.

## 6. The fault, exactly: one byte at $B1F1

Everything in this guide traces back to a single wrong byte. This section pins it down completely — what the check computes, what the correct answers are, what revision 1 accepts instead, and why the error survived to production.

### 6.1 What the copyright verifier actually computes

Tempest draws its screen with Atari's vector hardware: each frame, the program writes a fresh list of drawing instructions ("move here, draw this letter at this scale and color") into a shared memory buffer, and the display processor executes the list. When the game is about to write the instructions for the "© MCMLXXX ATARI" line, it quietly saves a pointer to the spot where those instructions will begin. The verifier later returns to that spot and adds up the first 40 bytes of the drawing instructions as actually generated that frame — not the text in ROM, but the live commands, position and all.

The sum is then tested by subtraction and a peel-off chain of comparisons. Compute R = $0E minus the 40-byte sum (all arithmetic in one byte, wrapping). Then:

- if R = 0, the check passes (this accepts a layout whose bytes sum to $0E);
- otherwise scramble R with the constant $E5 and pass if the result is zero (accepting a layout summing to $29);
- otherwise scramble again with a second constant and store the result as the flag — zero passes, anything else arms the trap.

That second constant is the fault byte, at address $B1F1. Revision 1 has **$2A** there; revision 2 has **$29**. Unwinding the arithmetic: rev 2's $29 makes the third accepted sum **$42**; rev 1's $2A makes it **$3F**.

### 6.2 Why there are three right answers — and why rev 1's third is wrong

Why does the verifier accept three different sums at all? Because the copyright line is drawn, by the same shared routine, on several different screens at different positions: the credits/info screen and the high-score initials screen (one position), the player-rating screen (another), and the attract-mode logo screen (a third). The lettering, scale, and color commands are identical every time; the *position* command differs. Three distinct layouts, three distinct 40-byte sums: **$0E, $29, and $42**. A correct verifier must accept exactly those three.

Revision 2 accepts $0E, $29, $42 — exactly right. Revision 1 accepts $0E, $29, and **$3F** — and $3F is a phantom. No screen the game can produce sums to $3F; that acceptance slot is wasted on a layout that does not exist, while the real third layout, summing to $42, is *rejected*. Every time a genuine, untouched revision 1 board draws the copyright line on that third screen, the verifier computes a nonzero flag and arms the sabotage. The protection's threat model — "someone has tampered with the copyright" — is triggered by the copyright, intact, at one of its own legitimate positions.

The arithmetic distance between right and wrong is diagnostic. $2A versus $29 differ by two low bits; equivalently, the accepted sum $3F versus the required $42 differ by 3. This is not a design mistake — the three-position scheme, the peel-off chain, the choice to checksum live display data are all coherent — it is a clerical error, one hex digit off in one hand-entered constant.

### 6.3 Why Theurer didn't catch it

Theurer's own account supplies the first half of the answer: "Just before shipping, I rearranged some of the screens, moving the Atari logo more towards the center on one of them, and I forgot to change the 3rd or 4th level checksum protecting that location." The third accepted sum protected a screen whose layout changed at the last minute. Move the copyright line, and its position bytes — hence its sum — change; the tolerance constant in the verifier must be recomputed by hand and re-entered. That maintenance step, on that one constant, went wrong in the final pre-ship shuffle. (Whether he recomputed the new sum and mistyped the constant, or mis-derived it, is unknowable from the bytes; the two-bit distance says "slip," not "misunderstanding.")

The second half of the answer is structural: this bug was nearly impossible to catch in 1981-style testing, for reasons Theurer had engineered himself.

1. **Failure is silent by design.** A failed check prints nothing and halts nothing; it writes a nonzero byte into an obscure flag. The whole point of the trap architecture was that verification and consequence are separated in time and mechanism, so that a bootlegger probing for the protection cannot see cause and effect. That same separation blinds the author's own testing.
2. **The consequence sits behind a 180,000-point gate.** To observe any symptom, a tester must play — legitimately, on the final ROMs, after the last-minute screen shuffle — to 180,000, roughly level 15-plus territory for a good player, *and then* notice subtle memory corruption whose visible form depends on the score's incidental last digits. A factory burn-in or a programmer's quick loop through the screens shows nothing.
3. **The wrong constant looks exactly like a right one.** The verifier's obfuscated peel-off chain means the constants are not readable quantities ("the sum should be $42") but scrambled intermediates ($E5, $2A). Nothing about $2A looks wrong on inspection; the error is invisible without redoing the hand arithmetic against the final screen layouts.
4. **The check verifies runtime data, not ROM.** A static comparison of the ROM against a reference would not reveal it; the sum exists only when the actual machine draws the actual screen. Only end-to-end play on final hardware exercises it — see point 2.
5. **Six traps, one author, ship deadline.** The protection suite was Theurer's own side project inside a game he was simultaneously finishing. When field reports of misbehaving machines arrived weeks later, he looked first at hardware — the traps were *designed* to make software sabotage look like flaky hardware, and they fooled their own creator until an Atari lab technician, watching with a hardware analyzer, caught the program red-handed corrupting the coin counter.

The deepest irony is that the protection performed exactly to specification. It detected what it classified as tampering, waited, and sabotaged the machine deniably — so deniably that Atari itself needed instrumented debugging to find it. The specification was simply armed with one wrong byte, and over 17,000 machines shipped before the December 1981 replacement ROM (the bulletin's part 136002-217 — the J1 chip carrying corrected byte $B1F1, and nothing else of consequence: Section 10).

## 7. The wave-gated traps: the resets after levels 10 and 13, and the quiet one after level 20

Three of the six traps trigger on level progress instead of score. They are frequently conflated with the score cheat; they should not be. They are armed by *different* verifiers, they were *not* affected by the $B1F1 bug, and — this is the important practical fact — none of them can fire on a genuine, unmodified revision 1 board. When they do fire, they are working as designed: something really is different about the machine, most commonly the ROM contents.

### 7.1 Getting the level numbers straight

The gates are exact, but stating them requires care with two numbering systems. Internally the game counts waves from 0; the screen displays levels from 1, so internal wave N is displayed level N+1. The trap comparisons, read directly from the disassembly:

| Trap | Comparison in code | Fires when internal wave reaches | In displayed-level terms |
|---|---|---|---|
| Frame-timer jam (flag QT1) | wave >= 10 | 10 | the level *displayed as 11* |
| Stack corruption (flag QT4) | wave >= 13 | 13 | the level *displayed as 14* |
| Decimal-mode switch (flag QT2) | wave > 19 | 20 | the level *displayed as 21* |

Now the observable timing. Each trap's consumer runs every frame and tests the current wave counter, so it fires on the very first frame after the counter steps to the fatal value — and the counter steps during the between-level warp sequence, before the new level is playable. So for the first trap: you finish the level displayed as 10 (internal wave 9, still below the gate), the game begins the zoom down the tube, the wave counter ticks from 9 to 10, and the trap fires mid-transition. The machine locks and the watchdog resets it. The player's experience is "the game crashes going from level 10 to 11" — and both descriptions of that crash are simultaneously true: the *gate* is at displayed level 11 (internal wave 10), and the *last level you can actually play* is displayed level 10. Any account that says "the game dies at level 10" or "at level 11" is describing the same event from opposite sides of a transition the game never finishes. Likewise the second trap: level 13 is the last one played; the reset lands in the warp to 14.

### 7.2 What each trap does mechanically

**The level 10-to-11 killer (flag QT1).** Every frame once the gate opens, the trap slams the display frame-pacing timer to a fixed value ($7A). That timer is what the display loop uses to know a frame has elapsed; pinned, the display sequencing never advances. The game hangs, the hardware watchdog (a dead-man circuit that reboots the machine if the program stops checking in) times out, and the machine resets. Armed by: an additive checksum of the block of *program code* that draws Atari's logo on the attract screen. This is a pure ROM-integrity check — genuine chips always sum correctly; it fails only if that region of the program has been altered or mis-copied.

**The level 13-to-14 killer (flag QT4).** Every frame once its gate opens, the trap writes the wave number into address $01FF — the top of the 6502's stack page, the region where the processor keeps its trail of return addresses (its record of "where to resume when the current subroutine finishes"). Whenever the stack happens to be at that depth, the write lands on a live return address; the next return jumps to a garbage location; the program derails and the watchdog resets the machine. The crash is probabilistic within a frame or two — which nicely disguises it as flaky RAM. Armed by: the POKEY liveness check — the verifier reads the sound chips' hardware random-number registers repeatedly and flags them if they are *frozen*. Working POKEYs always pass; the flag means dead or absent sound hardware (or an emulator that fails to model the random-number generator stepping).

**The quiet one after level 20 (flag QT2).** From internal wave 20 — displayed level 21 — onward, the trap executes a single instruction, SED, which switches the 6502 into decimal-arithmetic mode and leaves it there. In that mode, ordinary binary additions and subtractions silently produce wrong answers. There is no hang and no clean reset: timers misbehave, counters skip, score arithmetic garbles, until some routine happens to execute a CLD (clear-decimal) and things briefly normalize before the trap sets it again. This trap is essentially unknown to the lore, and understandably so: reaching displayed level 21 on a compromised board requires surviving the two reset traps first (only possible if their particular verifiers passed while this one failed), and the symptom — creeping arithmetic weirdness — is the least attributable of the six payloads. Armed by: an EOR-based checksum of the program code that draws the copyright on the credits screen. Like QT1, a pure ROM-integrity check that genuine chips always pass.

### 7.3 Why these traps fire in practice — and on whose machines

Revision 2 changed nothing in any of these three verifiers or consumers — the diff is conclusive (Section 10). Atari examined its misfiring protection suite under field pressure, fixed exactly the one constant that was wrong, and left these three armed. That is strong evidence they were believed correct, and the code analysis concurs: the two ROM-checksum verifiers cannot fail on unaltered chips, and the POKEY check cannot fail on working sound hardware.

So a reset at the 10-to-11 or 13-to-14 transition is not the famous rev 1 bug. It is the protection *succeeding*, and it means one of three things:

1. **Modified ROMs.** Any patch that alters the checksummed code regions — the attract-logo drawing code for the level 11 trap, the credits-screen copyright code for the level 21 trap — arms them. This is the trap ROM hackers hit: edit the wrong region of an original Tempest image (or run a bootleg whose copier "cleaned up" the copyright), and the game plays perfectly until it dies in the warp after level 10. The score cheat's own fame compounds the confusion, since people testing cheat behavior are often doing so on patched images.
2. **Dying hardware.** A POKEY whose random-number generator has failed (or a bad socket, or substitute chips on a clone board) arms the level 14 trap while everything else about the game — including sound, sometimes — seems fine.
3. **Imperfect emulation.** An emulator that stubs the POKEY random register, or returns it frozen, arms the level 14 trap on pristine ROM images. Early emulators tripped exactly this; modern MAME models the generator correctly.

On a genuine, unmodified board with healthy POKEYs, all three wave gates stay closed forever, at every level. The only trap a legitimate revision 1 machine ever springs is the score-steered one — through the single wrong byte of Section 6.

## 8. The score thresholds: why 180,000, and why the folklore says something else

### 8.1 What the code actually tests

The score-steered trap has a gate of its own, and it is one instruction long. Tempest keeps the player's score as three bytes of packed decimal — two score digits per byte — at addresses $40 (ones and tens), $41 (hundreds and thousands), and $42 (ten-thousands and hundred-thousands). The trap's dispatcher tests only the top byte:

```
LDA #$17        ; the constant 17 hex = the digit pair "1 7"
CMP $42         ; compare against the score's top two digits
BCS skip        ; if 17 >= top digits, do nothing this frame
LDX $40         ; otherwise: take the score's LAST two digits...
INC $00,X       ; ...and increment that memory address
```

The branch skips the trap whenever $17 is greater than *or equal to* the top byte. So the top digit pair must reach **18** before anything happens: the trap is dormant at every score through 179,999 and active from **exactly 180,000** upward. There is no upper bound in the code — no 190,000 ceiling, no 208,000 window, nothing about the middle digits. Score at least 180,000, and the last two digits of your score select the effect, full stop.

The second score-steered trap — the one that increments in page 2 of memory rather than page 0 — has its own, lower gate, tested the same way against the same byte: it opens at the top digit pair **15**, i.e. **150,000**. (On a genuine board this second trap never runs, because the flag that arms it comes from the POKEY liveness verifier, which healthy hardware always passes. It matters to ROM archaeology, not to players; it is why some disassembly-based writeups mention "150,000" alongside 180,000.)

### 8.2 Why 18? The strongest reading of an uncommented constant

The recovered source code has comments throughout — Atari's programmers annotated freely — but the protection code is the deliberate exception. The gate constants $17 and $15 appear with no explanation at all, in routines whose very labels (innocuous-looking names buried among vector-math helpers) were chosen not to attract attention. That silence is itself informative: the protection was written to be unreadable even to colleagues with source access, because a bootlegger might have source-level tools too. So there is no documented rationale, and any explanation is inference. The inference, though, is strong:

- **High enough to be invisible in testing.** A factory burn-in, a field technician's checkout, a distributor's demo — none of these plays to 180,000. Even Atari's own play-testing would rarely cross it casually. A trap gated this high is effectively undiscoverable by anyone who is not a dedicated player, which is exactly the profile of the intended process: ship, let bootleg boards circulate, and let the sabotage surface weeks later on location, looking like hardware failure.
- **Low enough to be inevitable in the field.** 180,000 is well within a skilled player's session — mid-teens levels. On a popular machine, the gate opens many times a day. The trap was meant to fire in the wild, reliably, just never on a workbench.
- **The two-tier structure is deliberate defense in depth.** Two score-steered traps with different gates (150,000 and 180,000), armed by different verifiers, with different target pages: disabling one does not disable the other, and discovering one gives no hint the other exists.

Within those constraints, the specific values look like round-number engineering judgment — the kind of constant a programmer picks as "clearly past normal play" — not derived quantities. Nothing in the code ties 18 or 15 to anything else. In that limited sense the folklore's instinct that the threshold is "arbitrary" is right; what it misses is that the *band* the number sits in was carefully chosen, even if the exact digit was not.

### 8.3 Reconciling the folklore's numbers

The circulating accounts disagree about the threshold — over 160,000, over 170,000, 179,976, 189,999-to-208,000, even "finish level 8." Against the code, all of them resolve:

- **Atari's own service bulletin says "over 170,000."** The December 1981 field notice describes machines misbehaving above 170,000 with "a 12% chance" of free credits. The 170,000 figure is best read as a field-observed round number, not a code citation — the bulletin was written for operators, from symptom reports, likely before or without a precise code trace. It is *consistent* with a 180,000 gate (every affected game is over 170,000) but imprecise. The "12%" is more interesting: of the hundred possible score endings, the ones that touch credit-related bytes number about six certain (06, 16, 17, 18, and the coin-switch state bytes) — and six-ish out of the roughly fifty endings that do anything observable is in the neighborhood of 12%. The bulletin's percentage reads like honest field statistics on which endings operators actually noticed and reported.
- **"189,999 to 208,000" (the Digital Press lineage)** is a window drawn around scores at which people successfully got credits, then over-fit into a rule. Nothing in the code closes the trap at any upper score.
- **179,976** appears in accounts as a specific score at which the effect was seen; it is one point inside the first thousand-point band above the gate with ending 76 — a display-pointer corruption slot. A memorable single observation, promoted by retelling into a magic number.
- **"Complete level 8"** (the MAME history-file version) is a level-denominated echo of the same fact: 180,000 is reachable around levels 12-16 depending on skill and bonuses, and an eight-level claim is what survives when a score rule is retold as a level rule. There is no wave test anywhere in the score-steered trap.

One more folklore casualty is worth restating from Section 5: no version of the middle-digits rule survives contact with the code. The dispatcher reads two bytes of the score — the top pair (gate) and the bottom pair (target) — and never looks at $41, the thousands digits, at all.

## 9. Why you have to wait: the attract-mode delay, explained

Every detailed recipe for the cheat — most explicitly the one printed in the UK press in the mid-1980s — includes a step that sounds like superstition: after the game ends, *let the attract mode cycle once or twice* before the free credits or the weirdness appear. Players confirmed it; nobody could say why. The code can.

### 9.1 The flag is re-decided every time the copyright is drawn

Recall the architecture: the copyright verifier of Section 6 does not fire the trap. It computes its result and stores it in a flag byte; the dispatcher, on a completely different schedule, reads the flag every frame and acts if it is nonzero. The critical detail is that the verifier *overwrites* the flag with a fresh verdict every time it runs — it does not accumulate or latch. The flag's value at any moment is simply the verdict on the *most recently verified* copyright drawing.

Now add the scheduling rules, both visible in the disassembly:

- The verifier runs only when the copyright line was actually drawn that frame (a helper flag, set by the drawing routine, tells it so), and only in the non-gameplay display states — attract screens, the high-score table, the initials-entry screen. During actual play there is no copyright on screen and the verifier never runs: whatever verdict the flag held when your game started, it holds until your game ends.
- The three copyright layouts of Section 6.2 pass or fail *individually*. Under revision 1's wrong constant, two of the layouts (the ones summing to $0E and $29) are still accepted; only the third — the attract screen whose logo Theurer repositioned before shipping, summing to $42 — is wrongly rejected.

Put together, the flag *toggles* as the machine cycles through its screens: drawn at a passing position, the flag is written zero (trap disarmed); drawn at the failing position, the flag is written nonzero (trap armed); and it keeps flipping as the attract rotation brings each screen around.

### 9.2 The end-of-game sequence disarms; the attract rotation re-arms

Follow a qualifying game through its final minute. You die with 180,000-plus and a chosen ending. The game takes you to the player-rating screen and then, with a top-eight score, the initials-entry screen — and both of those draw the copyright line at *passing* positions. Each drawing overwrites the flag with a clean verdict. At the moment your post-game screens finish, the trap is disarmed, no matter what the flag said earlier.

Then the machine settles into attract mode and begins its rotation of screens. Only when the rotation reaches the failing layout — the repositioned-logo attract screen — does the verifier compute a bad sum and write the flag nonzero. From that frame on, the dispatcher (which also runs during attract) finds the flag set, finds the frozen final score still above 180,000 in the score bytes (attract mode does not clear the last game's score; it is cleared when the *next* game starts), and begins incrementing your chosen address, once per frame, at 60 frames a second.

That is the wait, mechanically: **the delay is the time for the attract rotation to come around to the one screen that fails verification.** Depending on where in the rotation the machine was, that is a fraction of one attract cycle or most of one — and because some effects themselves need time to accumulate (a credit increment at address 06 gives one credit per frame once armed, but a subtle drift needs seconds to become visible), the observed onset spreads across "one or two cycles." The folklore's instruction is not superstition; it is an accurate field observation of a screen-rotation latency plus an accumulation time.

Two corollaries fall out, both matching reported behavior:

- **Starting a new game re-arms nothing and disarms everything.** The new game clears the score below the gate, so the dispatcher goes quiet even if the flag is set; and the flag itself will be rewritten (clean) at the next passing copyright draw. The cheat's effects live only in the window between the end of a qualifying game and the start of the next one — which is why the free credits had to be banked, and the machine left alone, before playing them off.
- **The weirdness can stutter.** As attract continues cycling, the flag keeps flipping — armed on the failing screen, disarmed again at the next passing one. The per-frame increments happen only while the flag is nonzero, so slow-accumulating effects advance in bursts synchronized to the attract rotation. Accounts that describe the corruption as coming in "waves" during attract are describing the toggle.

### 9.3 The out-of-sequence attract level: the "cheat ready" signal, and how ending 46 really selects levels

Players who used the ending-46 trick evolved a field discipline: after banking the qualifying score, watch the attract mode, and do not start the new game until the self-playing demo comes up on a level that is visibly out of sequence — the wrong well shape for that point in the demo. That garbled level was read as the sign that the cheat was "ready." The code confirms the practice is exactly right, and the mechanism is more roundabout — and more interesting — than the folklore's "the wave counter becomes your starting level."

**The demo starts clean every time.** When the attract rotation enters its gameplay segment, the initialization code detects attract mode and deliberately randomizes the starting level: it reads the POKEY random-number register, keeps the low three bits (the source comment reads "CHOOSE FROM 1ST 8 LEVELS"), and stores that into *both* wave variables. Whatever the beam had done to slot 46 before the demo began is discarded at that moment. This is why the demo does not simply start on a huge level number.

**The game keeps two wave counters, and the beam hits only one of them.** Slot $46 (WAVEN1) is the per-player *ledger* — the record of which wave player 1 is on. The game also keeps a working copy (CURWAV, outside the beam's reach at slot $9F) that drives colors, enemy mix, and speeds. The two are synchronized only at discrete moments: the ledger is copied to the working copy at wave and life initialization, and — the critical detail — the *well shape* is rebuilt by reading the ledger directly, at every wave transition and every new life.

**So the garble appears at the demo's first transition.** The dispatcher runs in every machine state, the ended game's score is still frozen above 180,000, and the flag holds whatever verdict it had when the demo's play-like state began (the copyright is not on screen during the demo, so the verifier cannot rewrite it). If that verdict was "failed," the beam adds 1 to the ledger sixty times a second for the whole demo. The demo plays its first wave normally — parameters were loaded before the corruption accumulated — but by the first wave-end or death, the ledger has climbed by hundreds (wrapping every 256). The rebuilt well is then computed from a nonsense wave number, and one more detail makes it unmistakable: for any wave value of 98 or more, the well-shape routine substitutes a *fresh random value* (0-95, from the POKEY register) on every rebuild. Meanwhile the colors and enemy behavior still come from the untouched working copy — the low starting level. The result is exactly what players describe: a well that belongs to no coherent level, in the wrong colors for its shape, appearing partway into a demo that started normally.

**Why the garble predicts the level select.** When the demo's single life ends, the standard end-of-game bookkeeping runs and records the highest wave reached — read from the beam-inflated ledger — into the "best wave" slot that drives Tempest's starting-level feature (the rate-yourself ladder, Atari's SkillStep). The starting-level screen offers rungs from a fixed 28-entry table (displayed levels 1, 3, 5, ... 65, 73, 81), extending as far up the table as the recorded best wave justifies; the table simply ends at internal wave 80, which is why the folklore's ceiling is exactly displayed level 81. An inflated best-wave value — anything at or above 80, reached after less than two seconds of beam time — unlocks the entire ladder. The recorded value survives the attract rotation (attract cycles skip the reset that a paid game performs on its own ledger), so it is still standing when the coin drops: the next game's selection screen offers everything up to 81. The corrupted slot-46 value itself never becomes the starting level — new-game initialization overwrites it — the unlock rides entirely on the recorded "highest wave reached."

**Why waiting for the signal is the correct protocol, not superstition.** Two conditions must both have held during a demo for the unlock to be banked: the flag must have been armed when the demo began (Section 9.2's rotation toggle — some demo cycles run with a clean flag and inflate nothing), and the demo must have reached its end-of-game bookkeeping. The out-of-sequence well is visible proof of the first condition, and the demo's death moments later completes the second. A player who waits for the garbled level before pressing start is, without knowing it, confirming that the beam ran during that specific demo cycle and that the inflated wave count has been posted to the ladder record. The rate-yourself screen then provides a second confirmation before any credit is spent: it draws a miniature well beside each offered rung, so an armed machine shows a conspicuously extended ladder of well icons reaching to 81.

## 10. The revision history: what Atari actually changed

The strongest evidence for every claim in this guide is the fix itself. Comparing complete revision 1 and revision 2 memory images byte by byte shows exactly what Atari believed was broken — and, just as important, what it believed was working.

### 10.1 Revision 1 to revision 2: two chips touched, one byte that matters

The program occupies several ROM chips; between revisions 1 and 2, only two chips changed at all. Every differing byte classifies as follows.

**The chip at board position J1** (part 136002-135, replaced by 136002-235) differs in exactly two bytes:

| Address | Rev 1 | Rev 2 | What it is |
|---|---|---|---|
| $B1F1 | $2A | $29 | **The fix.** The wrong verifier constant of Section 6. This single byte changes the third accepted copyright sum from the phantom $3F to the real $42, and the entire public cheat disappears. |
| $B497 | $1E | $1D | A data byte in a table controlling the score-range color/label display ("Novice"/"Expert" boundary presentation). Cosmetic housekeeping, unrelated to the protection; the kind of one-byte tweak that rides along when a chip is being re-mastered anyway. |

That is the whole gameplay-visible difference between the revisions: one repaired constant and one display-table byte. Atari's field bulletin (December 4, 1981) told operators to replace the single chip at J1 — it names the replacement part 136002-217, Atari's field-service numbering for the same corrected image — on all uprights below serial #17426, free through an exchange program that expired March 15, 1982. Cabaret and cocktail cabinets shipped later and carried the fix from the start.

**The chip holding the self-test code** (part 136002-137, replaced by 136002-237) was substantially reworked — a rewritten span from $D80B to $DD8C, one byte at $DDDC, and a new data table at $DFDC-$DFF7. All of it serves the operator self-test mode (the diagnostics an operator runs from inside the cabinet); none of it executes during play or attract. It is a quality-of-life revision that shared the release, not part of the protection story.

**Every other chip is byte-for-byte identical** between the revisions. That identity is the load-bearing fact: all six trap consumers, the other five verifiers, both score gates ($17 and $15), all three wave gates (10, 13, 19), and the dispatcher itself passed Atari's post-incident review untouched.

### 10.2 Revision 2 to revision 3: cleaning up after the cleanup

There is a third revision, and its content is a small comedy of checksums. Revision 3 differs from revision 2 by exactly two bytes, at $A8AF and $AA6D — both in checksum-compensation slots used by the *self-test's own ROM test*. When revision 2 changed the J1 and self-test chips, the stored reference values the self-test compares against were not fully updated; a revision 2 machine plays perfectly but can flag its own perfectly good ROMs as faulty in the operator's diagnostic screen. Revision 3 adjusts the two compensation bytes so the self-test passes again. Gameplay, protection, everything a player can ever see: identical to revision 2.

The pattern is worth savoring: the original bug was a checksum-maintenance error made while changing a screen; the fix for it introduced a checksum-maintenance error in the self-test; and a third release fixed *that*. Keeping hand-maintained checksums consistent across late edits was evidently a systematic weakness of this codebase, which is the most concrete possible corroboration of how the original mistake happened.

### 10.3 What the diff proves

- **The cheat was one byte.** Not a rewritten routine, not a disabled feature — a single wrong constant, corrected in place.
- **The traps were intended to stay.** Atari, with the field failure in hand and every incentive to rip the protection out, verified the other five trap chains and shipped them again, armed. The wave-gated resets of Section 7 are present in every revision.
- **Nothing else about the cheat was "fixed"** — no score gate moved, no effect neutered — because nothing else was broken. The effect menu of Section 5 still exists, bit-identical, in revision 2 and 3 boards; it is merely unreachable, because the verifier that wrongly armed the dispatcher now returns clean verdicts on all three copyright layouts.

## 11. Intentional or accidental? Both, in layers

The question that has followed this story for forty years — "was the Tempest cheat put there on purpose?" — has a definite answer once it is split into the three questions it actually contains.

**The trap machinery: fully intentional.** Six independent verifier/consumer pairs, split flags, staggered schedules, camouflaged labels, payloads engineered to imitate hardware failure — this is deliberate, sophisticated anti-bootlegging protection, confirmed in its author's own account. Claims that the whole phenomenon was an easter egg or a programmer's gift to skilled players are wrong.

**The specific effects — the free credits, the autopilot, the drifts: emergent, not designed.** Nobody sat down and decided that score ending 06 should award credits or that ending 51 should make the ship play itself. The designed behavior was "increment the memory address named by the score's last two digits" — deliberately *random* sabotage, in Theurer's own description. Which endings do what is an accident of where variables happen to sit in the first hundred bytes of memory, a layout fixed long before the protection was written and chosen for the 6502's addressing efficiency, not for meaning. The famous "cheat codes" are found objects: players mapping an unintended interface onto a memory map.

**The firing on genuine machines: pure defect.** The protection was never supposed to run on a legitimate board. One mistyped constant — a maintenance slip during a last-minute screen rearrangement, of exactly the species the company went on to commit twice more (Section 10.2) — turned an anti-piracy weapon inward onto 17,000-plus of Atari's own machines.

So the folklore's two camps are each half right. The "it's a bug" camp correctly describes why anyone ever saw the behavior; the "it's intentional" camp correctly describes the machinery that produced it. The precise statement is: **an intentional trap, with unintentional targeting, producing effects nobody chose.** The widely copied summary in emulator history files — that the protection was intentional and the free credits its punishment for pirates — captures the first layer and misses the other two.

## Conclusion

Strip away forty years of retelling and the Tempest cheat is three facts stacked on top of each other. Dave Theurer built a six-trap booby-trap suite to make bootlegged Tempest boards fail slowly, randomly, and deniably in the field. One of its verifiers shipped with a single hand-computed constant one hex digit off — $2A where $29 belonged — because a screen layout changed just before shipping and the checksum protecting it was not updated. And the payload behind that verifier was a one-instruction memory corruptor whose target is named by the last two digits of the player's score, turning the low bytes of the 6502's memory map into an accidental menu that players, without ever knowing what they were doing, learned to order from.

Everything else in the lore is refraction through that prism. The score thresholds are real but exactly 180,000, not the bulletin's 170,000 or the window rules. The middle-digits condition is a myth. The ending codes are real, incomplete, and occasionally mislabeled — and the full menu, all one hundred entries, is in Section 5, including three effects the folklore never found. The resets after levels 10 and 13 are a different mechanism entirely, working exactly as designed against modified ROMs and dead hardware. The wait through the attract mode is real, and it is the rotation latency of the one screen whose copyright line fails a check it should pass. And the whole phenomenon vanished in December 1981 when a field technician swapped one chip whose contents differ from the original by two bytes — one of them cosmetic.

It is tempting to call the protection a failure, and as shipped it was: it never caught a bootlegger, it attacked its own installed base, and it cost Atari a chip-exchange program. But as a piece of engineering psychology it succeeded almost too well. It was designed to make deliberate sabotage indistinguishable from failing hardware, and it was so convincing that its own author spent weeks blaming capacitors — while players on location, with no schematic and no source code, were already reverse-engineering its one observable seam into a folk science of magic numbers. The players' collective empirical record, it turns out, was remarkably good: nearly every stable rumor in the lore corresponds to something real in the code. They simply had no way to know that what they had discovered was not a secret feature but the shrapnel pattern of a copy-protection scheme detonating on the wrong target.
