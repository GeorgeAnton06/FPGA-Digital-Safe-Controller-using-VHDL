# FPGA-Based Digital Safe Controller (VHDL)

A modular digital safe/access-code controller developed in **VHDL** as part of a 60-hour ADC and FPGA prototyping training program at the **Electronics Research Institute (ERI)** in July 2026. The design captures three successive 2-bit input symbols to form a **6-bit code**, supports storing a new code, and compares a subsequent entry against the stored value. The provided design was developed in **Xilinx Vivado** and assembled as the schematic shown below.

![System schematic](docs/system-schematic.png)

## How it works

1. **Encode:** `enc4x2` transforms three button signals into two code bits. It assumes a valid button-input pattern; it does not independently reject simultaneous button presses.
2. **Capture:** `staging_buffer` contains two 3-bit shift registers. Whenever its `dp` enable is asserted on a rising clock edge, it shifts in a new 2-bit symbol. After three captures, its six outputs represent the three most recently sampled symbols.
3. **Select mode:** `DeMUX1x2Layer` routes those six bits to the stored-code path when `sel = '1'`, or the comparison path when `sel = '0'`.
4. **Store:** `SafeRamProject` stores six bits on a rising clock edge when its `ce` input is high. Despite its name, this module implements a **single six-bit storage register**, not an addressable RAM.
5. **Compare:** `comparator_6bit` uses six XNOR gates and an AND reduction to generate complementary `is_match` and `not_match` indications for binary inputs.
6. **Indicate:** `SafeProjectPB` synchronizes the Enter button through two flip-flops, detects a rising transition, issues a store-enable pulse in storage mode, and updates the green/red LED outputs in comparison mode.

**Mode control:** `sel = '1'` requests code storage; `sel = '0'` requests comparison. The behavior of the complete system also depends on the schematic wiring and timing of the capture-enable signal (`dp`).

## VHDL modules

| File | Entity | Purpose |
| --- | --- | --- |
| [`src/enc4x2.vhd`](src/enc4x2.vhd) | `enc4x2` | Three-button to two-bit encoder |
| [`src/staging_buffer.vhd`](src/staging_buffer.vhd) | `staging_buffer` | Three-symbol / six-bit shift-register buffer |
| [`src/DeMUX1x2Layer.vhd`](src/DeMUX1x2Layer.vhd) | `DeMUX1x2Layer` | Selects storage versus comparison data path |
| [`src/SafeRamProject.vhd`](src/SafeRamProject.vhd) | `SafeRamProject` | Clock-enabled six-bit stored-code register |
| [`src/comparator_6bit.vhd`](src/comparator_6bit.vhd) | `comparator_6bit` | Equality / inequality detection |
| [`src/SafeProjectPB.vhd`](src/SafeProjectPB.vhd) | `SafeProjectPB` | Enter-event processing and LED control |

## Tools and concepts

- **Language:** VHDL (`IEEE.STD_LOGIC_1164`)
- **Design environment:** Xilinx Vivado
- **Concepts:** Modular RTL, combinational encoding, shift registers, clock-enabled storage, equality comparison, synchronous control, button-edge detection, schematic integration

## Repository structure

```text
fpga-digital-safe-vhdl/
├── README.md
├── src/
│   ├── enc4x2.vhd
│   ├── staging_buffer.vhd
│   ├── DeMUX1x2Layer.vhd
│   ├── SafeRamProject.vhd
│   ├── comparator_6bit.vhd
│   └── SafeProjectPB.vhd
├── docs/
│   ├── system-schematic.png
│   └── verification-checklist.md
├── simulation/
│   └── README.md
└── .gitignore
```

## Reproducing the design

1. Create a **Vivado RTL project** targeting your FPGA board.
2. Add the six VHDL files from `src/` as design sources.
3. Re-create the top-level connections according to `docs/system-schematic.png`, or add your original top-level block-design source if available.
4. Add the appropriate board-specific pin/clock constraints (`.xdc`) for your hardware.
5. Run simulation with a testbench, then synthesis, implementation, and programming once the complete project is available.

**Important:** This repository currently contains the six supplied VHDL modules and the schematic **image**, not a complete top-level VHDL or Vivado block-design project, constraints, or testbenches. The screenshot is not itself an executable design source. The source files are included unchanged.

## Verification status and limitations

The author's project description reports on-board FPGA implementation. This repository does **not yet contain independently reproducible testbench results, implementation logs, or a board-specific constraint file**. Suggested tests are documented in [`docs/verification-checklist.md`](docs/verification-checklist.md).

Current RTL considerations:
- The stored code is initialized to `000000` and is **not cleared by** the controller's `reset` signal.
- The Enter input has a two-stage synchronizer and pulse detector, but **no mechanical-switch debounce circuit**.
- `staging_buffer` has no reset or explicit full-code counter: it always retains the **three most recent enabled samples**.
- The encoder has no invalid-input/error detection.
- A proper simulation should confirm that store-enable timing and data routing behave as intended at the complete-system level.

## Next improvements

- [ ] Add original Vivado project/block-design source and board model
- [ ] Add `.xdc` pin/clock constraints (only if authorized to publish)
- [ ] Add VHDL testbenches and waveform screenshots
- [ ] Document correct-code, incorrect-code, overwrite, and reset test cases
- [ ] Add FPGA photographs or a short demo video
- [ ] Add an explicit debouncer and define desired reset behavior if the design is extended

## Author and context

**George Anton Ibrahim** — Electronics and Communications Engineering undergraduate, Egypt-Japan University of Science and Technology (E-JUST).

Developed during ERI FPGA prototyping training, July 2026.
