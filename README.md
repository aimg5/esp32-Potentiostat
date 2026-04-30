# ESP32 Potentiostat v1

A single-channel potentiostat for cyclic voltammetry, designed end-to-end as 
a self-directed 4-day project. Analog front-end, Altium PCB, ESP32 firmware, 
Python GUI, and characterization.

**Status:** in active development. Day 1: simulation complete.

## Project structure
- `docs/` — design notes, specifications, test procedures
- `simulation/` — LTspice schematics and validation plots
- `pcb/` — Altium schematic and PCB layout (in progress)
- `firmware/` — ESP32 source code (planned)
- `software/` — Python GUI for live CV plotting (planned)

## Day 1 progress
LTspice simulation of the analog front-end completed. Validated:
- TIA gain and bidirectional current handling
- Closed-loop control on a 3-resistor dummy cell
- AC stability with phase margin >70°

See `docs/design-notes.md` for full design rationale and `simulation/screenshots/` 
for validation plots.

## Future work
Switched-gain TIA, buffered reference, low-noise LDO, real-cell testing 
with ferri/ferrocyanide. See `docs/design-notes.md`.