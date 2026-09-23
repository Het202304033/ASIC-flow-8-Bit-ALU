
Full-Custom ASIC Design & Tape-Out of an 8-Bit Adder/Subtractor ALU :

<img width="1710" height="540" alt="Your paragraph text" src="https://github.com/user-attachments/assets/97c7c81f-7b7b-44c5-a7bb-6124d3f9f090" />

# Full-Custom ASIC Design of an 8-Bit ALU

## 1. Project Overview
This repository contains the details of an 8-bit Arithmetic Logic Unit (ALU), designed and validated for tape-out[cite: 4]. The ALU performs addition and subtraction, utilizing tri-state buffers for safe integration into larger shared-bus architectures[cite: 4]. 

## 2. Pin Configuration & Functionality
* **Inputs:** `A7-A0` and `B7-B0` (hardwired to register A and register B), with `A0` and `B0` acting as the Least Significant Bits (LSB)[cite: 4].
* **Outputs:** `BUS7-BUS0` (connected to the main bus) and `C_O` (Carry Out)[cite: 4].
* **Operation Control (SU Pin):** Switches the ALU between mathematical operations. When `SU=0`, the ALU performs addition[cite: 4]. When `SU=1`, the input from register B passes through an XOR gate alongside the `SU` signal, passing the inverted output into the ALU to perform subtraction[cite: 4]. 
* **Chip Select (CS Pin):** An active-high configuration controlling a tri-state buffer[cite: 4]. When `CS=1`, the buffer actively drives the calculated results onto the `BUS`[cite: 4]. When `CS=0`, the ALU enters a high-impedance state, isolating it from the bus to prevent electrical contention or short circuits[cite: 4].

## 3. Post-Layout Performance Metrics
* **Propagation Delay:** 1.62 ns[cite: 4].
* **Drive Capability:** Successfully charges a 50 pF load capacitor in 0.2 µs[cite: 4].
* **Maximum Current:** 535 µA maximum dynamic current supplied by the ALU[cite: 4].

## 4. Worst-Case Delay Analysis
The critical path delay of 1.62 ns represents a scenario where a carry bit ripples through every single 1-bit full adder stage[cite: 4].
* **Test Vector:** Triggered during an addition operation (`SU=0`) by applying `A = 00000001` and `B = 11111111`[cite: 4]. 
* **Critical Path Flow:** Arrival at `A0/B0` $\rightarrow$ Input XOR gate $\rightarrow$ Sequential propagation through Carry Chains (Bit 0 to Bit 7) $\rightarrow$ Tri-state buffer $\rightarrow$ Final output pin (`BUS7`)[cite: 4].
* **Alternative Bottleneck:** If mathematical calculation finishes before the Chip Select (`CS`) is driven high, the primary delay shifts to the turn-on time of the output tri-state buffers driving the bus load[cite: 4].

## 5. Arithmetic Overflow & The XOR Solution
* **Unsigned vs. Signed:** For unsigned numbers (0 to 255), the `C_out` pin correctly flags an overflow[cite: 4]. However, for signed 2's complement numbers (-128 to 127), `C_out` produces false errors (e.g., `-2 + -4 = -6` generates a carry out, but is a valid result)[cite: 4]. 
* **Design Limitation:** The current design lacks a dedicated Signed Overflow Flag (V), meaning hardware cannot natively detect signed overflow (which only occurs when adding two numbers of the same sign yields an opposite sign)[cite: 4].
* **Proposed Logic Solution:** Implementing an XOR gate to compare the carry entering the MSB against the carry leaving it (`V = C_in7 XOR C_out7`)[cite: 4]. For example, adding `64 + 64` yields `128` (an overflow, interpreted as `-128`)[cite: 4]. This generates a carry-in to Bit 7 (`1`), but no carry-out (`0`), evaluating to `1 XOR 0 = 1`, successfully flagging the signed overflow[cite: 4].

## 6. Physical Design & DFM: Antenna Effect Prevention
During fabrication (specifically plasma etching), long unbroken floating metal traces can act as antennas, accumulating static charge that can exceed the breakdown voltage of a transistor's thin gate oxide, destroying the device[cite: 4]. 
* **Metal Jumpering (Layer Hopping):** To resolve "Antenna ratio exceeded" violations in the layout, long horizontal data lines are not drawn as a single continuous trace[cite: 4]. 
* **Implementation:** Traces are deliberately severed into shorter segments by dropping down (or popping up) to an adjacent metal layer using vias[cite: 4]. Because etching is done layer-by-layer, this ensures no single layer is physically long enough to accumulate lethal plasma charge[cite: 4]. The isolation protects sensitive gates until the final upper-level metals bridge the connections[cite: 4].


