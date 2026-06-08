# DOD 250 — Assembly Notes

## Footswitch: 3PDT vs. DPDT substitution

The board is laid out for a **3PDT latching footswitch** (SW1, footprint `SW_FSW800_3PDT`).
A 3PDT provides three switched poles:

1. Pole 1 — input signal routing (effect input vs. bypass)
2. Pole 2 — output signal routing (effect output vs. bypass)
3. Pole 3 — status LED switching (LED on only when the effect is engaged)

Neither Mouser nor the major catalog distributors stock the pedal-style 9-pin
3PDT stomp switch; it must be ordered from a guitar-pedal specialty supplier
(Love My Switches, StompBox Parts, Tayda).

### Substituting a DPDT footswitch — **TODO: document in assembly instructions**

A **DPDT** latching footswitch can be used in place of the 3PDT, but it has only
**two poles**. Both poles are consumed by the true-bypass signal routing
(input + output), leaving **no third pole to switch the status LED**.

Assembly instructions MUST explain, when a DPDT is fitted instead of the 3PDT:

- Use pole 1 for input routing and pole 2 for output routing exactly as the two
  signal poles of the 3PDT (same in/out lugs).
- The status LED can no longer indicate bypass/engaged state. Choose one:
  - **Omit** the LED, or
  - Wire it as an **always-on power indicator** (LED + its series resistor R9
    tied across +9V), accepting that it stays lit whenever the pedal is powered
    regardless of bypass state.
- Map the DPDT lug pattern to the SW1 footprint pads (the LED-pole pads on the
  3PDT footprint are left unpopulated / rewired per the LED choice above).

The SW1 footprint (`edaFactory:SW_FSW800_3PDT`) is the standard Taiwan-Alpha 3PDT
pin grid: **9 pins, columns at ±5.3 mm, rows at ±4.8 mm, 1.5 mm holes**. A DPDT
switch has **6 pins (2 columns × 3 rows)** at the same 4.8 mm row pitch; fit the
two DPDT columns to two of the three footprint columns, leaving the third column
(3 holes — the LED pole) unpopulated.

Primary 3PDT part (what the footprint is modeled on):
- StompBox Parts — *3PDT Footswitch PRO, PCB Pin* (**FSW-800-1028**), ~$5.25
- Genuine Taiwan Alpha **SF17010F-0302-21R-L** (same part)
- Budget equivalents, same pin grid: **Tayda A-1842** ($2.99), Love My Switches
  *3PDT Latched – PCB* ($4.29)

Candidate DPDT parts (specialty suppliers — PCB-pin, latching; verify the column
spacing maps to the SW1 footprint before ordering):
- **Tayda A-361** — *2PDT/DPDT Latching Stomp Foot Switch, PCB* ($1.89)
- Love My Switches — *DPDT Latched Foot Switch, PCB Mount*
