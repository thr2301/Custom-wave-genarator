# TL074 Analog Signal Generator ⚡

An open-source, fully analog function generator built around the **TL074 quad operational amplifier**. This project generates simultaneous Triangle, Square, and Sine waves. It features a discrete, mathematical piecewise-linear diode wave shaper, adjustable output buffering, and a low-noise split-rail power design.

Designed and routed in **KiCad 8**, with complete circuit simulation available for **LTspice**.

---

## 📑 Table of Contents
1. [Key Features](#-key-features)
2. [Theory of Operation](#-theory-of-operation)
3. [Component Selection Guide](#-component-selection-guide)
4. [Bill of Materials (BOM)](#-bill-of-materials-bom)
5. [PCB Design & Layout](#-pcb-design--layout)
6. [Calibration & Tuning](#-calibration--tuning)
7. [LTspice Simulation](#-ltspice-simulation)
8. [License](#-license)

---

## ✨ Key Features
* **Tri-Wave Output:** Generates Square, Triangle, and Sine waves from a single TL074 IC.
* **Low-THD Sine Wave:** Utilizes a custom multi-stage diode clipping network with precise bias voltages to perfectly round off the triangle wave.
* **Master Volume Control:** Integrated Bourns 3296P multi-turn trimmer for fine amplitude adjustment (0% to 100%) without degrading waveform symmetry.
* **Output Isolation:** The 4th op-amp acts as a unity-gain output buffer, protecting the delicate internal wave shaper from external loads and impedance mismatches.
* **High-Fidelity Audio Range:** Tuned for the ~1kHz audio spectrum (easily modifiable for LFO or higher-frequency RF applications).
* **Split-Rail Power:** Optimized for standard ±15V dual power supplies for maximum headroom and signal clarity.

---

## 🧠 Theory of Operation

### 1. The Core Oscillator (Square & Triangle)
The foundation of the generator uses **U1** and **U2** in a classic relaxation oscillator topology:
* **U1 (Schmitt Trigger):** Acts as a comparator with hysteresis. It outputs a square wave that slams between the +15V and -15V rails.
* **U2 (Integrator):** Takes the harsh square wave and mathematically integrates it using a feedback capacitor (`C1`). A constant positive voltage turns into a downward-sloping ramp, and a constant negative voltage turns into an upward-sloping ramp—yielding a perfect Triangle wave.
* The Triangle wave is fed back into the Schmitt Trigger. The threshold at which the trigger reverses direction is set by the ratio of the feedback resistors (`R1` and `R2`), which directly determines the Triangle wave's peak-to-peak amplitude.

### 2. The Diode Wave Shaper (Sine)
Because building a pure analog sine wave oscillator is inherently unstable, this circuit uses a **Piecewise Linear Approximation Shaper** around **U3**:
* The raw Triangle wave is passed through a voltage divider network containing parallel branches of **1N4148 small-signal diodes**.
* Each branch is tied to a specific DC bias voltage (e.g., ±1.5V and ±3.0V). 
* As the Triangle wave's voltage rises, it sequentially turns on these diodes. Each activated branch drops the equivalent resistance of the network, gently reducing the gain and "bending" the straight slope of the triangle wave to approximate the curve of a pure sine wave.

### 3. The Output Stage
* **U4 (Unity Gain Buffer):** To ensure the sine wave doesn't collapse when plugged into an oscilloscope or audio interface, a 100kΩ volume potentiometer divides the final signal and feeds it into the Non-Inverting (`+`) input of U4. The output is a robust, low-impedance signal capable of driving external gear.

---

## 🔬 Component Selection Guide

To achieve a clean, low-distortion waveform, component quality is critical:
* **Resistors:** Use **1% tolerance metal film** resistors. The wave shaper relies on precise math; carbon resistors with 5% variance will result in a lopsided sine wave.
* **Capacitors (Timing):** `C1` must be a **Film Capacitor** (Polyester/Mylar or Polypropylene). Ceramic capacitors suffer from microphonics and temperature drift, which will cause your frequency to wander.
* **Capacitors (Bypass):** The 0.1µF decoupling caps placed near the TL074 power pins should be standard **MLCC ceramics** for the fastest high-frequency noise rejection.
* **Diodes:** You *must* use **1N4148** or **1N914** small-signal fast-switching diodes. Do not substitute with power rectifiers like the 1N4007 or 1N4937, as their forward voltage curves and junction capacitance will ruin the high-frequency response.

---

## 🛒 Bill of Materials (BOM)

| Reference | Component | Value | Footprint / Package | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **U1, U2, U3, U4** | Quad Op-Amp | TL074 | `DIP-14_W7.62mm` | Solder a 14-pin DIP socket to the board. |
| **D1 - D4** | Signal Diode | 1N4148 | `D_DO-35_SOD27_P7.62mm_Horizontal` | Low capacitance, DO-35 glass package. |
| **RV1** | Multi-Turn Trimmer | 100kΩ | `Potentiometer_Bourns_3296P_Vertical` | Bourns 3296P or SR Passives T67W (Top adjust). |
| **R1 - R8** | Fixed Resistors | Various | `R_Axial_DIN0207_L6.3mm` | 1/4W, 1% Metal Film recommended. |
| **C1** | Timing Cap | 0.1µF | `C_Disc_D5.0mm_W2.5mm_P5.00mm` | Box/Film type preferred for stability. |
| **C2, C3** | Bypass Caps | 0.1µF | `C_Disc_D5.0mm_W2.5mm_P5.00mm` | Ceramic disc. |
| **J1** | Power Input | 3-Pin | `TerminalBlock_Phoenix_MKDS-1,5-3` | 5.08mm pitch (+15V, GND, -15V). |
| **J2, J3, J4** | Signal Outputs | 2-Pin | `PinHeader_1x02_P2.54mm_Vertical` | Standard 2.54mm headers for scope probes. |

---

## 📐 PCB Design & Layout

If modifying the KiCad `.kicad_pcb` layout, adhere to these analog routing guidelines:
1. **Copper Pours:** The board features grounded copper fills on both the `F.Cu` and `B.Cu` layers to shield the sensitive wave shaper from EMI. 
2. **Decoupling Placement:** The bypass capacitors are physically routed millimetres away from Pin 4 (+15V) and Pin 11 (-15V) of the TL074 to absorb transient switching spikes from the square wave generator.
3. **Trace Widths:** Analog signal traces are routed at `0.25mm` (10 mil), while the power rails (`+15V`, `-15V`) are thickened to `0.5mm` (20 mil) to minimize voltage drop.

---

## ⚙️ Calibration & Tuning

Once assembled, you will need an oscilloscope to calibrate the device:
1. **Power Up:** Connect a ±15V lab supply to the power terminal.
2. **Check the Triangle:** Probe the output of U2. You should see a sharp, symmetrical triangle wave. 
3. **Tune the Volume:** Probe the final output of U4. Slowly turn the `RV1` Bourns trimmer clockwise with a flathead screwdriver. The sine wave should grow in amplitude without flat-lining (clipping) at the top or bottom.
4. **THD Minimization (Advanced):** If the sine wave has sharp "points" at the peaks, the input triangle wave is too large. You can drop the value of `R2` slightly to lower the Schmitt trigger threshold until the wave rounds out perfectly.

---

## 💻 LTspice Simulation

Before modifying the PCB, you can test resistor changes in the provided LTspice `.asc` file. 

**Pro-Tips for Simulation:**
* **Testing Wave Shaper Resistors:** Use the `.step param` directive to automatically test hundreds of resistor values. For example, to find the perfect bend for the 3V diode branch:
  `.step param R_test 1k 5k 100`
* **Checking Distortion:** Use the `.four` command to analyze the exact Total Harmonic Distortion (THD) of your sine wave:
  `.four 1kHz V(sine_out)`
* View the results by pressing `Ctrl+L` to open the SPICE Error Log.

---

## 📄 License
This project is licensed under the **MIT License**. You are free to use, modify, and distribute this hardware design for both personal and commercial projects. See the [LICENSE](LICENSE) file for details.