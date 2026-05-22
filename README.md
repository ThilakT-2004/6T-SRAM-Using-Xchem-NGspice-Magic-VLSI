# 6T SRAM Cell Design — SKY130 PDK

A complete 6-transistor (6T) SRAM cell design and characterization suite using the **SkyWater SKY130A** open-source PDK, implemented and simulated with **ngspice / xschem**. The project covers the full SRAM read/write path: core cell, stability analysis, write driver, tristate buffer, precharge circuit, and differential sense amplifier.

---

## 📐 Cell Architecture

The 6T SRAM cell uses two cross-coupled CMOS inverters (M1–M4) for storage, and two NMOS access transistors (M5, M6) controlled by the Word Line.

| Signal | Description |
|--------|-------------|
| `WL`   | Word Line — enables read/write access |
| `BL`   | Bit Line |
| `BLB`  | Bit Line Bar (complementary) |
| `Q`    | Storage node |
| `QB`   | Complementary storage node |
| `VDD`  | Supply voltage (1.8 V) |

**Transistor sizing:** L = 150 nm, W = 1 µm (PMOS & access NMOS), W = 420 nm (pull-down NMOS)

---

## 📁 Project Structure

```
6t-sram/
├── spice/
│   ├── 6T_sram.spice              # Core 6T SRAM cell transient testbench
│   ├── sram_dc.spice               # DC VTC analysis
│   ├── sram_ncurve.spice           # N-curve characterization
│   ├── sram_snm.spice              # Hold Static Noise Margin
│   ├── sram_snmR.spice             # Read Static Noise Margin
│   ├── sram_snmW.spice             # Write Static Noise Margin
│   ├── sram_tran.spice             # Full read/write transient testbench
│   ├── sram_write.spice            # Write operation testbench
├── results/
│   ├── schematics/                 # xschem schematic screenshots
│   └── Simulation_waveforms/                  # ngspice simulation waveforms
└── README.md
```

---

## 🔬 Simulations & Results

### 1. 6T SRAM Core Cell — Transient Analysis

Basic read/write transient over 20 ns showing WL, BL, BLB, Q, QB toggling correctly.

| Schematic | Waveform |
|-----------|----------|
| ![6T SRAM Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/6T_sram.png) | ![6T SRAM Transient](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/6T_sram.png) |

---

### 2. Read & Write Transient (`sram_tran`, `sram_write`)

Full transient testbench with piecewise-linear (PWL) WL stimulus and initial conditions V(Q)=0, V(QB)=1.8. Captures both write-0 and write-1 cycles across a 20 µs window.

| Schematic | All Signals |
|-----------|-------------|
| ![sram_tran Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_read&write.png) | ![sram_tran Waveform](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_tran.png) |

---

### 3. DC Analysis — Voltage Transfer Characteristic

DC sweep of the storage node showing the bistable VTC (Q and QB crossing at ~0.9 V), confirming correct inverter operation at the TT corner.

| Schematic | VTC Result |
|-----------|------------|
| ![DC Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_dc.png) | ![DC VTC](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_dc.png) |

---

### 4. Hold Static Noise Margin (SNM)

Butterfly curve obtained by double-sweeping both inverter VTCs. The SNM is the side length of the largest square inscribed in the eye opening.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![SNM Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_snm_hold.png) | ![SNM Result](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_snm_hold.png) |

---

### 5. Read Static Noise Margin (Read SNM)

SNM evaluated with WL = VDD (access transistors ON). Read disturb degrades the noise margin compared to hold SNM — the eye opening shrinks visibly.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![Read SNM Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_snmR.png) | ![Read SNM Result](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_snmR.png) |

---

### 6. Write Static Noise Margin (Write SNM)

The butterfly loop collapses (no eye opening), confirming the cell can be flipped during a write operation.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![Write SNM Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_snmW.png) | ![Write SNM Result](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_write.png) | ![Write SNM Result](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_write_.png) |

---

### 7. N-Curve Analysis

Current vs. voltage sweep for combined read stability and writeability analysis. The positive-current peak gives the Static Current Margin (SINM); the zero-crossing separation gives the Static Voltage Margin (SVM).

| Schematic | N-Curve |
|-----------|---------|
| ![N-Curve Schematic](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Schematics/sram_ncurve.png) | ![N-Curve Result](6T-SRAM-Using-Xchem-NGspice-Magic-VLSI/Simulation_waveforms/sram_ncurve.png) |

---

## ⚙️ Requirements

- [ngspice](http://ngspice.sourceforge.net/) (v37+)
- [xschem](https://xschem.sourceforge.io/) (for schematic editing)
- [SkyWater SKY130A PDK](https://github.com/google/skywater-pdk) with `open_pdks`

> **Note:** Update the `.lib` path in each SPICE file to match your local PDK installation:
> ```spice
> .lib /path/to/open_pdks/sky130/sky130A/libs.tech/combined/sky130.lib.spice tt
> ```

---

## ▶️ Running Simulations

```bash
# Core cell transient (20 ns)
ngspice spice/6T_sram.spice

# Full read/write transient (20 µs)
ngspice spice/sram_tran.spice

# Write operation
ngspice spice/sram_write.spice

# DC VTC
ngspice spice/sram_dc.spice

# Hold / Read / Write SNM
ngspice spice/sram_snm.spice
ngspice spice/sram_snmR.spice
ngspice spice/sram_snmW.spice

# N-curve
ngspice spice/sram_ncurve.spice

```

---

## 📊 Design Summary

| Parameter            | Value                  |
|----------------------|------------------------|
| Supply Voltage (VDD) | 1.8 V                  |
| Technology           | SkyWater SKY130 (180nm)|
| Min Channel Length   | 150 nm                 |
| PMOS Width           | 1 µm                   |
| NMOS Pull-down Width | 420 nm                 |
| Access NMOS Width    | 1 µm                   |
| Simulation Corner    | TT (Typical-Typical)   |
| Simulation Tool      | ngspice + xschem       |

---

## 📜 License

This project is open-source. PDK device models are governed by the [SkyWater SKY130 PDK License](https://github.com/google/skywater-pdk/blob/main/LICENSE).
