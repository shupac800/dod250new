# Peavey Classic 60/60 as the 1tube Bench Power Amplifier

## Signal interface, schematic-derived supply voltages, and B+/heater tap plan

**Project:** labamp modular 12AX7 preamp rack<br>
**Document date:** July 17, 2026<br>
**Schematic:** Peavey Classic Series 60/60, drawing dated 4-23-90, P/N 81503250 (archived copy of the Mooselander scan)

## Executive summary

The Peavey Classic 60/60 is the chosen interim power amplifier for hearing the `1tube` board before the labamp PI and power-amp units exist. It is a purpose-built stereo slave amp: 250 kOhm input impedance, 1 V RMS input sensitivity for full rated output, one push-pull 6L6GC pair and one dedicated output transformer per channel. The 1tube's plate-coupled output (roughly 38 kOhm source impedance) drives it with no interface hardware at all: instrument cable from the 1tube output jack to a Power Amp In jack, speaker on the matching impedance tap, channel level up.

The 60/60 can also serve as an interim B+ and heater source for the 1tube board. The factory schematic prints no DC voltages, so the supply-node values below are calculated from the schematic's component values and cross-checked against service-forum measurements; they are inference, not manufacturer data, and must be verified by measurement before the tap is wired. The best tap point is the **V1 node** (the last, most-filtered preamp supply node, estimated 375-400 VDC), dropped through a dedicated series RC in the umbilical to the 1tube's working B+. The heater tap is the 6.3 VAC winding legs `FILA`/`FILB`, which the amp references to ground through a 100 Ohm resistor pair, so the tapped heater is already ground-referenced. The tap does not supply the 1tube's 12 V relay or 3.3 V logic power; those still require an external low-voltage supply, and the relay bank's de-energized state must be confirmed to leave a plate-resistor path closed before first power-up.

The community has also published a small set of modifications for this amp - adjustable bias, a 4x negative-feedback increase, filter-cap renewal, and output-tube substitution (Section 8). Of these, only the adjustable-bias mod earns a recommendation for the bench rig; the others are catalogued with the reasons to leave them alone.

## 1. What the 60/60 is

The Classic 60/60 is a 2U rack, two-channel, all-tube power amplifier: two complete mono amps sharing one power transformer, heater winding, bias supply, and chassis. Per channel: input jack, level pot, half of a shared 12AX7 as input gain stage, a full 12AX7 long-tailed-pair phase inverter, and a fixed-bias push-pull 6L6GC pair into a dedicated output transformer with 16/8/4 Ohm taps.

| Parameter | Value | Provenance |
|---|---|---|
| Rated output | 60 W RMS/channel, both driven (65 W one side) | Peavey spec |
| Input impedance | 250 kOhm | Peavey spec |
| Input sensitivity | 1 V RMS for full rated output | Peavey spec |
| Tube complement | 4x 6L6GC, 3x 12AX7 | schematic |
| Bias | Fixed, factory-set cold; no user adjustment | spec + forum reports |
| Damping factor | ~1 (design intent, guitar voicing) | Peavey spec |

Tube designators from the schematic: `V7A`/`V7B` are the two halves of the shared input 12AX7 (one half per channel); `V1A/V1B` and `V4A/V4B` are the channel 1 and channel 2 phase-inverter 12AX7s; `V2/V3` and `V5/V6` are the 6L6GC pairs.

## 2. Signal interface to the 1tube board

The 1tube output is plate-coupled (no cathode follower): source impedance approximately `R_plate || r_a` = 35-40 kOhm with a 100 kOhm plate load, and it varies as plate resistors are relay-switched. Against the 60/60's 250 kOhm input this is a ~13% divider loss - negligible. The 1 V full-power sensitivity is trivially met; a 12AX7 stage driven by a guitar delivers several volts RMS, so the practical issue is attenuating, not boosting. Overdriving the 60/60 input simply clips its input stage, which may be a desired experimental condition.

Hookup:

1. Guitar to 1tube INPUT; 1tube OUTPUT to 60/60 Power Amp In (either channel), ordinary unbalanced instrument cables.
2. Speaker cabinet on the matching 16/8/4 Ohm output jack of the same channel. Never operate off standby without a load.
3. Channel level down, power on, warm up, standby off, level up.

Keep the signal cable to 1-2 m: the 38 kOhm source against ~100 pF/m of cable capacitance gives f(-3dB) around 8-9 kHz at 5 m. The second channel is free for A/B against a second module or the future PI board.

## 3. Power supply chain from the schematic

The schematic (two pages, both channels plus supply) prints component values but **no DC operating voltages**. The chain, read from the drawing:

```text
HV secondary -- bridge CR5-CR8 (1N4007) -- B+ node
  B+ filter: C17 + C20 in series (100 uF/350 V each = 50 uF at 700 V rating),
             balanced by R31/R34 220 kOhm
  B+ --> STANDBY switch --> R27 (400 Ohm, 10 W FP) --> SCRN node
  SCRN filter: C18 + C19 series (100 uF/350 V each), balanced by R29/R32 220 kOhm
  SCRN --> R28 (4 kOhm, 5 W) --> V2 node   (C16, 22 uF/450 V)
  V2   --> R62 (22 kOhm)     --> V1 node   (C32, 22 uF/450 V)
```

`V2` feeds both phase inverters (68 kOhm plate loads R2/R24 and R1/R23; LTP tail 470 Ohm + 22 kOhm). `V1` feeds both input half-triodes (56 kOhm plate loads, 1.5 kOhm cathodes). Global negative feedback returns from each output secondary (`FBK1`/`FBK2`) through a **1 MOhm series resistor** (R53 channel 1, R58 channel 2) into the junction of the input stage's 1.5 kOhm cathode resistor (R54/R59) and the 16 kOhm resistor (R55/R60) that completes the cathode path to ground. The feedback fraction is therefore set by the 16k/(1M + 16k) divider - roughly 1.6%, a very light loop consistent with the published damping factor of ~1. (An earlier draft of this document described the return as "through 16 kOhm into the cathode"; the schematic shows the 16 kOhm as the grounded lower leg and the 1 MOhm as the series feedback element.)

Heaters: one 6.3 VAC winding, legs `FILA`/`FILB`, referenced to ground by an artificial center tap of two 100 Ohm resistors (R41/R42). The heater string is at ground potential, not elevated.

A [freestompboxes.org thread](https://www.freestompboxes.org/viewtopic.php?t=22598) reports measured power-transformer secondary voltages, which anchor the schematic's winding entry points:

| Winding | Wire colors | Board pins | Measured | Role |
|---|---|---|---|---|
| HV | red - red/yellow (CT) - red | J23 - nc - J29 | 185 - 0 - 185 VAC | Feeds bridge CR5-CR8 across the full 370 VAC winding; the center tap is unconnected |
| Bias | orange - white/orange - orange | J30 - J32 - J35 | 20 - 0 - 20 VAC | Negative bias supply (CR9/CR11 into C22) |
| Heater | yellow - yellow | J37 - J38 | 6.3 VAC | `FILA`/`FILB` through fuse F4 |

The HV figure is a useful cross-check on Section 4: 370 VAC end-to-end into a full bridge gives about 523 V peak, consistent with the ~480-500 VDC loaded B+ estimate.

Bias: a negative supply (CR9/CR11, C22 1000 uF/35 V, R36 47 K, R38 470, R39 4.7 K) fixed at the factory; not relevant to the tap but do not disturb it.

## 4. Estimated node voltages

These are calculated operating points, cross-checked against service-forum measurements ("400-500 VDC at pin 3; screens a few volts less"). Measure before relying on any of them; a healthy unit on modern 120-125 V mains will likely sit at the upper end.

| Node | Estimate | Basis |
|---|---|---|
| B+ (6L6 plates) | ~480-500 VDC | series 350 V cap stack sized for ~700 V peak; forum-measured 400-500 V |
| SCRN (6L6 screens) | B+ minus ~5 V | ~10-15 mA total screen idle current through R27 400 Ohm |
| V2 (PI plate supply) | **~455-470 VDC** | ~5-7 mA total preamp draw through R28 4 kOhm |
| V1 (input stage supply) | **~375-400 VDC** | ~3-4 mA through R62 22 kOhm |
| PI plates (V1A/B, V4A/B) | ~390-400 VDC | ~0.6-0.7 mA per triode through 68 kOhm from V2 |
| Input stage plates (V7A/B) | ~300 VDC | ~1.3-1.5 mA through 56 kOhm from V1 |
| LTP tail top | ~25-30 V | ~1.3 mA through 22 kOhm tail |

The controlling fact for the tap plan: **every preamp supply node in the 60/60 sits well above the 1tube's working B+.** Even V1, the lowest and best-filtered node, is around 390 V.

## 5. B+ and heater tap plan

The tap is four wires out the back through a grommeted hole: two B+ (feed and dedicated return) and the two heater legs. The 1tube's few-mA draw is invisible to the 60/60; the engineering content is in the dropper, the fault limiting, and the grounding.

### 5.1 B+ tap

- **Tap point: the V1 node** (junction of R62 and C32, eyelet J42 on the board). It is the most-filtered node, the lowest voltage available, and downstream of the standby switch - so the 60/60's standby also kills the tap, which is desirable.
- **Series RC dropper at the amp end of the umbilical.** For a 280 V target at ~3 mA from a measured 390 V node: `R = (390 - 280) / 3 mA = 37 kOhm`; use 33-39 kOhm, 3 W flameproof, then 22-47 uF / 450 V at the 1tube end. Recompute from the measured V1 voltage and the board's actual draw; the 1tube tolerates a wide B+ range by design (the labamp PSU spec's preamp output range is 100-400 VDC).
- **The dropper is also the fault limiter.** Placed at the amp end, a short anywhere on the external run draws only ~11 mA (390 V / 36 kOhm) and dissipates ~4 W in a resistor rated to survive it. No unlimited conductor leaves the chassis. A 100 mA fuse ahead of the dropper is optional insurance.
- If a connector is used, the energized side must present sockets, not pins.

### 5.2 Heater tap

Tap `FILA` and `FILB` (both legs of the 6.3 VAC winding, e.g. at a preamp socket) and run them as a tightly twisted pair. One added 12AX7 (300 mA) is negligible on a winding feeding four 6L6s plus three 12AX7s (~4 A). The winding is already ground-referenced through the 100 Ohm pair, so the 1tube heater arrives referenced to the 60/60's ground; do not add a second reference on the 1tube side. AC heater hum is accepted for this interim rig.

### 5.3 Grounding

Return B+ on its own umbilical conductor to the tap-point ground; never through the audio cable shield. The umbilical plus signal cable still form a loop, so bundle them physically. If residual hum matters, lift the audio cable shield at one end - never the supply return.

### 5.4 What the tap does not provide

The 12 V relay supply and 3.3 V logic supply still come from an external low-voltage source. Before first power-up, verify that the relay bank's de-energized (NC) state leaves at least one plate-resistor path closed: a plate node with no DC path to B+ is the failure mode the 1tube design rules already prohibit, and it must hold in the all-relays-unpowered condition this bench rig will start in.

## 6. Why the 60/60 over the Fryette PS-2

Both work as the interim power amp. The 60/60 is preferred because it is exactly a slave power amp (published 250 kOhm / 1 V RMS input, per-channel level, no attenuator/load logic in the path), its schematic is available so the tap can be engineered rather than guessed, and its second channel supports A/B testing. The PS-2 stays in reserve for its unique role: reactive-load attenuation of the complete amplifier once the labamp power unit exists. The PS-2's Line In accepts the same signal (line level, INPUT LEVEL switch HI; Fryette publishes no sensitivity or impedance figure), so it remains a fallback.

## 7. Direct injection at the phase inverter (optional modification)

The stock signal path enters through the input stage `V7A`/`V7B`, which is both the channel's gain stage and its global-NFB summing point (feedback from the output secondary returns through the 1 MOhm series resistor R53/R58 into the 1.5 kOhm + 16 kOhm cathode network). An alternative is to inject the 1tube's post-coupling-cap output directly at the PI grid stopper (100 kOhm), reducing the Peavey to a bare "PI + power stage" with no foreign gain stage ahead of the board under test.

DC conditions are compatible: the 1tube output is AC-coupled, and the PI grid keeps its ~+25 V reference through the 470 kOhm return to the tail top, so LTP bias is untouched. Inject at the driven grid only; the opposite grid stays AC-grounded through its 0.047 uF cap. Drive requirement is about 1.2-1.5 V RMS for full power (LTP gain ~25-30x, ~35 V RMS needed per 6L6 grid) - essentially the same as the front jack, because the NFB loop was consuming the input stage's gain.

Conditions and consequences:

- **The coupling cap from the input-stage plate (C6 or its channel twin) must be lifted.** Paralleling a source onto the live node fails twice over: V7's ~38 kOhm plate impedance shunts the injected signal, and the still-closed NFB loop actively opposes it. With the cap lifted, V7 idles harmlessly and stays in place.
- **Lifting that cap opens the global NFB loop** (output -> V7 cathode -> V7 plate -> C6 -> PI). The power amp then runs open-loop: more gain, several-times-higher output impedance and looser damping, more distortion, more response variation with speaker impedance. Do not A/B jack-input against direct-injection results and attribute the difference to the 1tube - the Peavey itself changes character between the modes.
- **The channel level pot is bypassed**; all level control moves to the 1tube side.
- A switched rear jack that restores the coupling-cap connection when nothing is plugged in preserves both modes.

For ordinary listening, the unmodified front jack remains the better path - it needs no work and the NFB makes the amp more neutral, not less. Direct injection earns its keep when minimizing foreign stages between a labamp board and the speaker matters more than flat response, e.g. when the labamp PI unit later needs a known power stage behind it.

## 8. Published modifications

The 60/60 has a small but consistent body of published mods, developed mostly on diyAudio, the Peavey Forum, and pedal/amp DIY boards. Four recur: adjustable bias, increased global negative feedback, filter-capacitor renewal, and output-tube substitution. Each is summarized below with its designators verified against the P/N 81503250 schematic, and assessed against this document's bench use. None is a prerequisite for the interface or tap plan.

### 8.1 Adjustable bias (the most-published mod)

The factory bias is fixed and set cold: the negative supply (CR9 rectifier, C22 1000 uF/35 V, filtered by C23/C24 200 uF/75 V) feeds the `BIAS` rail through a divider whose series element is **R38 (470 Ohm)** and whose shunt leg is **R36 (47 kOhm)** to ground. With only 470 Ohm on top of 47 kOhm, nearly the full negative supply reaches the grids - the divider has almost no authority, which is why the amp cannot be biased hotter without surgery.

The canonical recipe is Enzo's, posted on [diyAudio in 2005](https://www.diyaudio.com/community/threads/peavey-classic-60-60-bias-mod-advice.54050/) from a 60/60 he had modified for per-channel adjustment:

- **Raise the series resistor from 470 Ohm to 3.3 kOhm** so the divider can actually divide.
- **Replace the 47 kOhm shunt with a 6 kOhm fixed resistor in series with a 10 kOhm pot**, wired as a variable resistance. Enzo deliberately makes the *lower* (shunt) leg the variable one: if the pot wiper opens, the divider ratio rises toward unity, bias goes maximally negative, and the tubes run cold - the failure-safe direction. A pot in the series position fails the other way.
- **For independent per-channel bias**, duplicate the divider and filter (two shunt networks, two filter caps), cut the board trace that distributes the single bias rail, and wire each channel's bias feed separately. Enzo's post gives his specific trace cuts (under C32; the C13-R23 jumper) and feed points (the R6/R16 pad, the pad near R15); treat those as a guide to intent rather than gospel, because his designators came from a different Peavey drawing - a later poster in the same thread notes that his "R68 (470 Ohm)" corresponds to **R38** on this schematic, and the correspondence of the layout references should be re-derived from the actual board before cutting.

A milder variant posted on the [Peavey Forum](https://forums.peavey.com/viewtopic.php?t=52705) keeps the stock value in range: series 470 Ohm -> 3 kOhm, shunt 47 kOhm -> **27 kOhm + 25 kOhm pot** in series (27-52 kOhm span, bracketing the factory 47 kOhm), which allows both colder- and hotter-than-stock settings.

**Bench relevance: high.** This rig will run long A/B sessions and possibly non-factory output tubes (Section 8.4), and the one thing every service thread agrees on is that the factory setting is very cold. The mod is independent of the B+/heater tap - it touches only the negative supply - but do it with the same discipline: measure the stock bias voltage and per-tube idle current first (the schematic prints neither), and use a matched quad or, with the per-channel version, matched pairs.

### 8.2 Increased negative feedback: R53/R58 1 MOhm to 250 kOhm

A circulated tone mod (evidenced by [Reverb listings of modded units](https://reverb.com/item/2547567-peavey-classic-series-60-60-stereo-tube-power-amp-modded); its original forum write-up was not located, so its provenance is the thinnest of the four) replaces the two 1 MOhm series feedback resistors **R53 and R58** with **250 kOhm**, one per channel. Against the fixed 16 kOhm lower leg (R55/R60) this raises the feedback fraction from about 1.6% to about 6% - roughly 4x more NFB. The published claim is that the amp gets sweeter and better damped; circuit analysis agrees on the mechanism: lower gain, lower output impedance, flatter response into a reactive speaker load, at the cost of the loose, damping-factor-~1 voicing Peavey designed in.

**Bench relevance: leave stock, at least initially.** The 60/60's value in this project is as a *known* reference power stage; changing its loop gain changes its input sensitivity (more drive needed for full power - the 1tube has it to spare) and its interaction with every speaker used for evaluation. If the mod is applied later, note that Section 7's direct-injection option already bypasses this loop entirely - injected-at-PI results are unaffected by R53/R58 either way, which also makes the mod easy to A/B against memory-free: front-jack channel modded, direct-injection path as control.

### 8.3 Filter capacitor renewal

Two published positions exist. The aggressive one: double the filter capacitance (at minimum the first two stages - the C17/C20 series stack on B+ and C18/C19 on the screen node) for a claimed "profound" tightening. The conservative one, from the [diyAudio refresh thread](https://www.diyaudio.com/community/threads/peavey-classic-60-60-cleaning-refreshing-upgrading.336054/): on an amp this age, inspect for bulged electrolytics, bring it up slowly on a variac, and **replace nothing that is not damaged**.

For this project the conservative position is the right one, with one addition: these amps are 30+ years old, and the B+ tap plan (Section 5) depends on the V1 node being quiet, so measure ripple at V1/V2 during tap commissioning - if it is high, the 22 uF C32/C16 and the main stacks are the suspects, and like-for-like replacement (100 uF/350 V, with the R31/R34 220 kOhm balance resistors retained) is maintenance, not modification. If capacitance is ever increased, remember the standby switch then closes onto a larger uncharged bank - the inrush and the rectifier margin, not the audio, are the constraint.

### 8.4 Output tube substitution

[Eurotubes' retube kits](https://www.eurotubes.com/product/peavey-classic-60-60-power-amp-full-retube-kits/) document the two standard choices: JJ 6L6GC quads preserve the factory headroom and breakup point, while JJ 5881s drop output roughly 20% and move power-stage breakup earlier. Either substitution strengthens the case for Section 8.1: with fixed cold bias, a retube is otherwise at the mercy of the new quad's grid-voltage-to-idle-current mapping.

**Bench relevance: none until the stock quad fails.** For a reference amp, tube consistency across sessions matters more than tube flavor.

### 8.5 What was not found

No published mod addresses the two things this project actually adds - the B+/heater tap (Section 5) and PI direct injection (Section 7). The closest historical artifact is a "peavey_classic_6060_mods" zip once hosted on GeoCities (linked in the 2005 diyAudio thread, long dead), which per the thread's discussion contained the single-pot bias mod that Enzo's per-channel version supersedes. The tap and injection plans remain original engineering, which is why their verification steps (measure before wiring; lift C6 before injecting) stay in this document rather than resting on community precedent.

## Conclusion

The 60/60 needs nothing from the 1tube but a standard line-level signal - about 1 V RMS nominal into 250 kOhm - delivered on an instrument cable into a Power Amp In jack, and any working 1tube configuration exceeds that. As a supply donor, the schematic supports a clean four-wire tap: B+ from the V1 node (~390 V estimated, measure first) through a 33-39 kOhm/3 W dropper that doubles as fault limiting, heaters from the ground-referenced 6.3 VAC winding legs. The remaining constraints are that all node voltages here are calculated, not factory-printed, so measurement precedes wiring; and the tap leaves the relay/logic supplies and the de-energized relay-state verification as separate prerequisites.

Sources: [Peavey Classic 60/60 schematic scan](https://dod250.shupac.com/files/Peavey-Classic-60_60-schematic.pdf) ([archived original](http://web.archive.org/web/20220812202057/https://mooselander.com/wordpress/wp-content/uploads/Peavey-Classic-60_60-schematic.pdf)), [Peavey Classic Series manual](https://assets.peavey.com/literature/manuals/theclassic.pdf), [Audiofanzine spec listing](https://en.audiofanzine.com/guitar-power-amplifier/peavey/classic-series-discontinued-classic-60-60/), [Peavey Forum tube-layout thread](https://forums.peavey.com/viewtopic.php?t=46250), local copy: `C:\Users\david\labamp\reference\Peavey-Classic-60_60-schematic.pdf`.

Modification sources (Section 8): [diyAudio bias-mod thread (Enzo, 2005)](https://www.diyaudio.com/community/threads/peavey-classic-60-60-bias-mod-advice.54050/), [Peavey Forum bias-mod thread](https://forums.peavey.com/viewtopic.php?t=52705), [freestompboxes.org 60/60 thread](https://www.freestompboxes.org/viewtopic.php?t=22598) (PT secondary voltages only), [diyAudio cleaning/refreshing thread](https://www.diyaudio.com/community/threads/peavey-classic-60-60-cleaning-refreshing-upgrading.336054/), [Eurotubes 60/60 retube kits](https://www.eurotubes.com/product/peavey-classic-60-60-power-amp-full-retube-kits/), [Reverb modded-unit listing](https://reverb.com/item/2547567-peavey-classic-series-60-60-stereo-tube-power-amp-modded). The R53/R58, R54/R59, R55/R60, R36, and R38 designators and values were verified against the P/N 81503250 schematic.
