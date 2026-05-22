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
│   └── write_driver.spice          # Write driver circuit
├── results/
│   ├── schematics/                 # xschem schematic screenshots
│   └── waveforms/                  # ngspice simulation waveforms
└── README.md
```

---

## 🔬 Simulations & Results

### 1. 6T SRAM Core Cell — Transient Analysis

Basic read/write transient over 20 ns showing WL, BL, BLB, Q, QB toggling correctly.

| Schematic | Waveform |
|-----------|----------|
| ![6T SRAM Schematic](results/schematics/6t_sram_schematic.png) | ![6T SRAM Transient](results/waveforms/6t_sram_transient.png) |

---

### 2. Read & Write Transient (`sram_tran`, `sram_write`)

Full transient testbench with piecewise-linear (PWL) WL stimulus and initial conditions V(Q)=0, V(QB)=1.8. Captures both write-0 and write-1 cycles across a 20 µs window.

| Schematic | All Signals | Q & QB Detail |
|-----------|-------------|----------------|
| ![sram_tran Schematic](results/schematics/sram_tran_schematic.png) | ![sram_tran Waveform](results/waveforms/sram_tran_waveform.png) | ![sram_write Q/QB](results/waveforms/sram_write_waveform2_QQB.png) |

Write operation waveform showing WL, BL, BLB, Q, QB across 20 µs:

![sram_write Waveform](results/waveforms/sram_write_waveform1.png)

---

### 3. DC Analysis — Voltage Transfer Characteristic

DC sweep of the storage node showing the bistable VTC (Q and QB crossing at ~0.9 V), confirming correct inverter operation at the TT corner.

| Schematic | VTC Result |
|-----------|------------|
| ![DC Schematic](results/schematics/dc_schematic.png) | ![DC VTC](results/waveforms/dc_vtc_curve.png) |

---

### 4. Hold Static Noise Margin (SNM)

Butterfly curve obtained by double-sweeping both inverter VTCs. The SNM is the side length of the largest square inscribed in the eye opening.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![SNM Schematic](results/schematics/snm_schematic.png) | ![SNM Result](results/waveforms/snm_butterfly.png) |

---

### 5. Read Static Noise Margin (Read SNM)

SNM evaluated with WL = VDD (access transistors ON). Read disturb degrades the noise margin compared to hold SNM — the eye opening shrinks visibly.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![Read SNM Schematic](results/schematics/snmR_schematic.png) | ![Read SNM Result](results/waveforms/snmR_butterfly.png) |

---

### 6. Write Static Noise Margin (Write SNM)

The butterfly loop collapses (no eye opening), confirming the cell can be flipped during a write operation.

| Schematic | Butterfly Curve |
|-----------|-----------------|
| ![Write SNM Schematic](results/schematics/snmW_schematic.png) | ![Write SNM Result](results/waveforms/snmW_butterfly.png) |

---

### 7. N-Curve Analysis

Current vs. voltage sweep for combined read stability and writeability analysis. The positive-current peak gives the Static Current Margin (SINM); the zero-crossing separation gives the Static Voltage Margin (SVM).

| Schematic | N-Curve |
|-----------|---------|
| ![N-Curve Schematic](results/schematics/ncurve_schematic.png) | ![N-Curve Result](results/waveforms/ncurve_result.png) |

---

### 8. Write Driver

Differential write driver that drives BL and BLB to complementary values based on input data (`din`) and write enable (`wen`). Two waveform views shown — zoomed out and zoomed in for transition detail.

| Schematic |
|-----------|
| ![Write Driver Schematic](results/schematics/write_driver_schematic.png) |

| Waveform (wide) | Waveform (zoomed) |
|-----------------|-------------------|
| ![Write Driver 1](results/waveforms/write_driver_waveform1.png) | ![Write Driver 2](results/waveforms/write_driver_waveform2.png) |

---

### 9. Tristate Buffer

Tristate output buffer used in the data bus path. Output is driven high/low when `en` is asserted; floats (high-Z) otherwise. Verified with `en` and `enb` pulse stimulus.

| Schematic | Waveform |
|-----------|----------|
| ![Tristate Schematic](results/schematics/tristate_buffer_schematic.png) | ![Tristate Waveform](results/waveforms/tristate_buffer_waveform.png) |

---

### 10. Precharge Circuit

Two PMOS transistors precharge BL and BLB to VDD before each read cycle. Controlled by active-low precharge clock (`pclk`). BL/BLB discharge is visible after `pclk` goes high.

| Schematic | Waveform |
|-----------|----------|
| ![Precharge Schematic](results/schematics/precharge_schematic.png) | ![Precharge Transient](results/waveforms/precharge_transient.png) |

---

### 11. Differential Sense Amplifier

Latch-based differential sense amplifier that detects the small voltage difference on BL/BLB during a read and resolves it to full-swing digital output. Signals shown: `r` (read enable), `bl`, `blb`, `diffout`, `out`.

| Schematic | Waveform |
|-----------|----------|
| ![Sense Amp Schematic](results/schematics/sense_amp_schematic.png) | ![Sense Amp Result](results/waveforms/sense_amp_transient.png) |

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

# Write driver
ngspice spice/write_driver.spice
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
