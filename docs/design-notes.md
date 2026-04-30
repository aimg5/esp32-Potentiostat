# Design Notes — ESP32 Potentiostat v1

## Project goal
Build a single-channel potentiostat capable of cyclic voltammetry on 
electrochemical cells, designed end-to-end (analog circuit → PCB → firmware 
→ PC software → testing) over 4 days as a self-directed portfolio project. 
Domain alignment: matches the analog front-end class found in commercial 
biosensor products such as Embio Diagnostics' B.EL.D.

## Application context
[Brief paragraph: what a potentiostat does, three-electrode cell, why it's 
the analog kernel of every electrochemical biosensor]

## Design specifications

| Spec | Target | Driving requirement |
|---|---|---|
| Cell voltage range | ±1.0 V around midpoint | Aqueous electrochemistry window |
| Voltage resolution | ≤1 mV | CV peak feature width (~30-60 mV) |
| Scan rate range | 10–200 mV/s | Standard analytical CV |
| Current range (full-scale) | ±10 µA at default gain | Typical biosensor currents |
| Current resolution | ≤10 nA | Resolve baseline noise |
| Current noise floor target | ≤50 nA RMS | SNR for typical peaks |
| Supply | 3.3 V single from USB | Practical, ESP32-compatible |

## Topology choice
Adder topology (working electrode at virtual ground, control amp drives 
counter electrode). Single-supply 3.3 V with virtual ground at 1.65 V. 
Fixed-gain TIA with three swappable feedback resistors (10 kΩ / 100 kΩ / 
1 MΩ) covering 3 decades of current range manually. Differs from CheapStat 
in omitting the gain-switching multiplexer and reference-electrode buffer; 
both are explicit v2 upgrades.

## Component selection rationale

- **Op-amp: MCP6002** — rail-to-rail I/O, 1 pA input bias current (CMOS), 
  1.8 V minimum supply, DIP-8, ~€1.50. Alternative: TLV2372. Future v2: 
  OPA2333 for zero-drift offset performance.
- **DAC: MCP4725** — 12-bit I²C, 0.8 mV resolution at 3.3 V, breadboard-
  friendly breakout. Sufficient for ≤1 mV voltage resolution spec.
- **MCU: ESP32-DevKitC** — 12-bit ADC with calibration, dual-core 
  enables real-time sampling on dedicated FreeRTOS task, future WiFi/cloud 
  upgrade path for IoT-style data logging.
- **R_f: 10 kΩ default** — chosen to match the dummy cell's expected 
  current range (~50–100 µA). 100 kΩ and 1 MΩ alternates available for 
  lower-current applications.

## Sign convention
IUPAC: oxidation currents (electron flow into the working electrode 
terminal) are positive. Because the TIA inverts, firmware applies 
`current = (V_REF - V_adc) / R_f` to recover physical sign.

## Day 1 simulation results
- Stage A: TIA standalone — gain confirmed at 100 kV/A (with 100 kΩ R_f), 
  bidirectional current handling demonstrated.
- Stage B: full potentiostat with resistive dummy cell — control loop 
  enforces V_REFERENCE = V_TARGET, linear I-V response on dummy cell.
- Stage C: AC analysis of TIA — −3 dB bandwidth ~100 kHz, no peaking, 
  smooth phase rolloff indicating phase margin >70°. 100 pF compensation 
  cap appropriate for chosen R_f and op-amp GBW.

## Future work (v2)
- Switched-gain TIA via analog multiplexer (e.g., ADG704) for autoranging
- Buffered reference electrode input to eliminate bias-current loading
- Dedicated low-noise LDO (LP5907) for analog supply rail
- Cell isolation switch between scans
- Real cell testing with ferri/ferrocyanide redox standard
- v2 op-amp upgrade to zero-drift OPA2333 for improved offset performance