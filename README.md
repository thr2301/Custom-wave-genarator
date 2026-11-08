# Costum Signal Generator ⚡

An open-source, fully analog function generator built around the **TL074 quad operational amplifier**. This project generates simultaneous Triangle, Square, and Sine waves. It features a discrete, mathematical piecewise-linear diode wave shaper, adjustable output buffering, and a low-noise split-rail power design.

Designed and routed in **KiCad 8**, with complete circuit simulation available for **LTspice**.

---

## 📑 Table of Contents
1. [Key Features](#-key-features)
2. [Theory of Operation](#-theory-of-operation)
3. [LTspice Simulation](#-ltspice-simulation)
4. [Component Selection Guide](#-component-selection-guide)
5. [PCB Design & Layout](#-pcb-design--layout)
6. [Calibration & Tuning](#-calibration--tuning)


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

## 💻 LTspice Simulation

Before modifying the PCB, you can test resistor changes in the provided LTspice `.asc` file. 
<img width="1858" height="899" alt="Image" src="https://github.com/user-attachments/assets/ecf07521-2cf6-45b9-b70a-f437f7def819" />
**Pro-Tips for Simulation:**
* ** Voltage divivers are used instead of trimmmers in some cases
* ** Transient analysis is used
  <img width="1900" height="923" alt="Image" src="https://github.com/user-attachments/assets/4e8b6cf4-2759-4584-9320-aedef695a974" />

---

## 🔬 Component Selection Guide

To achieve a clean, low-distortion waveform, component quality is critical:
* **Resistors:** Use **1% tolerance metal film** resistors. The wave shaper relies on precise math; carbon resistors with 5% variance will result in a lopsided sine wave.
* **Capacitors (Timing):** `C1` must be a **Film Capacitor** (Polyester/Mylar or Polypropylene). Ceramic capacitors suffer from microphonics and temperature drift, which will cause your frequency to wander.
* **Capacitors (Bypass):** The 0.1µF decoupling caps placed near the TL074 power pins should be standard **MLCC ceramics** for the fastest high-frequency noise rejection.
* **Diodes:** You *must* use **1N4148** or **1N914** small-signal fast-switching diodes. Do not substitute with power rectifiers like the 1N4007 or 1N4937, as their forward voltage curves and junction capacitance will ruin the high-frequency response.
  <img width="1172" height="814" alt="Image" src="https://github.com/user-attachments/assets/122a2829-971f-4915-bca5-73577761836d" />

---


## 📐 PCB Design & Layout

If modifying the KiCad `.kicad_pcb` layout, adhere to these analog routing guidelines:
1. **Copper Pours:** The board features grounded copper fills on both the `F.Cu` and `B.Cu` layers to shield the sensitive wave shaper from EMI.
   <img width="838" height="850" alt="Image" src="https://github.com/user-attachments/assets/5193e8b9-2b2f-4915-b57c-0fbfec2ddd5d" />
3. **Decoupling Placement:** The bypass capacitors are physically routed millimetres away from Pin 4 (+15V) and Pin 11 (-15V) of the TL074 to absorb transient switching spikes from the square wave generator.
   <img width="1144" height="848" alt="Image" src="https://github.com/user-attachments/assets/43324d7e-386f-4cb2-a3b4-8cc3c0b5c06d" />
5. **Trace Widths:** Analog signal traces are routed at `0.25mm` (10 mil), while the power rails (`+15V`, `-15V`) are thickened to `0.5mm` (20 mil) to minimize voltage drop.

---

## ⚙️ Calibration & Tuning

Once assembled, you will need an oscilloscope to calibrate the device:
1. **Power Up:** Connect a ±15V lab supply to the power terminal.
2. **Check the Triangle:** Probe the output of U2. You should see a sharp, symmetrical triangle wave. 
3. **Tune the Volume:** Probe the final output of U4. Slowly turn the `RV1` Bourns trimmer clockwise with a flathead screwdriver. The sine wave should grow in amplitude without flat-lining (clipping) at the top or bottom.
4. **THD Minimization (Advanced):** If the sine wave has sharp "points" at the peaks, the input triangle wave is too large. You can drop the value of `R2` slightly to lower the Schmitt trigger threshold until the wave rounds out perfectly.

---

## TODO
Upload the pcb with soldered components when is ready
