# Caravel RTL vs GLS Verification – HKSPI Test

## Overview
This project focuses on **functional verification of the Caravel SoC** by comparing **RTL simulation** and **Gate-Level Simulation (GLS)** results for the **hkspi (Housekeeping SPI) test**.  
The objective is to ensure that both RTL and synthesized gate-level netlists produce **identical functional behavior** using **open-source EDA tools** and the **Sky130 PDK**.


---

## Objective
- Verify that **RTL simulation** and **GLS** outputs for the `hkspi` test match exactly
- Confirm that both simulations end with a **“Passed”** status
- Identify and resolve any mismatches between RTL and GLS

---

## Tools & Environment
- **HDL Simulators**:  
  - Icarus Verilog (`iverilog`, `vvp`)  
  - Verilator (optional)
- **Synthesis & PnR**:  
  - Yosys  
  - OpenLane / OpenROAD
- **PDK**:  
  - Sky130
- **Layout Tools** (optional):  
  - Magic  
  - KLayout
- **Platform**:  
  - Local Linux system or GitHub Codespaces

---

## Task 1: Environment Setup
### Clone the official Caravel Github Repository:

```bash
git clone https://github.com/efabless/caravel.git
```

### Install the Necessary PDKs:

Install and use the PDK managing tool volare

```bash
pip install volare
```

>[!NOTE]
>If you encounter an error, try installing inside a virtual environment

Define the following environment variable

```bash
export PDK_ROOT="/home/<your user name>/<desired PDK directory>"
```

Example:

```bash
export PDK_ROOT="/home/vsd/VLSI/pdks"
```

List remote the PDK you want to install (e.g. sky130 or gf180mcu) as follows:

```
volare ls-remote --pdk sky130
```

Select one release for installation.

```bash
volare enable --pdk sky130 cd1748bb197f9b7af62a54507de6624e30363943
# or enter the most recent stable version
```
The necessary PDKs will be installed.


### Install RISC-V toolchain (If you dont have)

Go to https://github.com/riscv-collab/riscv-gnu-toolchain/releases/ and download the riscv32-elf-ubuntu-24.04-gcc.tar.xz (Or the one which matches your OS version).

---

### Verified tool installation:
- `iverilog`
- `vvp`
- `verilator`

### Verification
All tools were successfully installed and accessible from the terminal.

📷 **Screenshot: Tool version check**  

![Task1_Tool_Check](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Caraval_SOC/Images/Screenshot%20from%202025-09-19%2023-28-54.png)
![images](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Caraval_SOC/Images/Screenshot%20from%202025-12-04%2016-33-00.png)

---
