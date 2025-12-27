# RISC-V SoC Research Task  
## Synopsys VCS + DC_TOPO Flow using SCL180 PDK

---

## 📌 Task Overview

This task focuses on migrating the complete RISC-V SoC verification flow from open-source tools to **industry-grade Synopsys EDA tools**.  
The goal is to demonstrate independent research capability, correct RTL-to-GLS verification, and professional-level documentation.

All simulations and synthesis are performed on the **vsdcaravel SoC** using the **SCL180 technology**.

---

## 🎯 Objective

- Transition from open-source tools to **Synopsys VCS and DC_TOPO**
- Maintain functional correctness across RTL and Gate-Level Simulation (GLS)
- Use **SCL180 standard cell libraries**
- Develop industry-standard debugging and documentation skills

---

## 🔁 Key Changes from Previous Tasks

### 1. Research-Driven Execution
- No step-by-step guide provided
- Tool manuals and error logs were studied
- Synopsys SolvNet used for issue resolution
- Blind copy-paste execution strictly avoided

### 2. Mandatory Toolchain Migration
The following tools were **completely removed**:
- `iverilog`
- `gtkwave`

They do **not** appear in:
- Scripts
- Makefiles
- Logs
- Documentation

---

## 🧰 Tools Used

| Tool | Purpose |
|-----|--------|
| Synopsys VCS | RTL & Gate-Level Simulation |
| Synopsys DC_TOPO | RTL Synthesis |
| SCL180 PDK | Standard Cell Library |
| Synopsys SolvNet | Debugging & Documentation Reference |

---

## 🧪 Functional Simulation (RTL)

### Tool Used
**Synopsys VCS**

### Description
- RTL-level simulation of `vsdcaravel`
- Same intent as earlier `hkspi` functional test
- Verified correctness of core SoC functionality

### Outputs Generated
- Simulation log
- FSDB / VPD waveform
- Pass/fail confirmation from testbench

### 📷 Evidence
![RTL Simulation Waveform](images/rtl_simulation.png)

---

## 🏗️ Synthesis Flow

### Tool Used
**Synopsys DC_TOPO**

### Description
- Synthesized `vsdcaravel` using **SCL180 standard cells**
- Timing and area constraints applied
- Clean constraint handling ensured

### Important Design Constraint
- **POR module and memory modules kept as RTL**
- Treated as blackboxes where required
- No macro replacement done at this stage

### Outputs Generated
- Synthesized gate-level netlist
- Area report
- Timing report
- Power report

### 📷 Evidence
![DC_TOPO Synthesis Report](images/dc_topo_report.png)

---

## 🔍 Gate-Level Simulation (GLS)

### Tool Used
**Synopsys VCS**

### Description
- Simulated post-synthesis netlist
- Linked with:
  - SCL180 standard cell functional models
  - RTL versions of POR and memory blocks

### Verification Goals
- RTL and GLS functional equivalence
- No unexpected `X` propagation
- Correct clock and reset behavior

### 📷 Evidence
![GLS Waveform](images/gls_waveform.png)

---

## 🧠 Use of Synopsys SolvNet

Synopsys SolvNet was actively used for:
- Understanding VCS compile/simulation flags
- Debugging DC_TOPO constraint warnings
- Resolving GLS linking and library issues

OTP access was coordinated as per instructions.

---

## ⚠️ Issues Faced and Resolution

### Issue 1: DC_TOPO Blackboxing Warnings
**Problem:**  
POR and memory blocks were initially flagged during synthesis.

**Resolution:**  
Explicitly treated them as RTL/blackbox modules as per task constraints.

---

### Issue 2: GLS X-Propagation
**Problem:**  
Initial GLS showed `X` values on reset-related signals.

**Resolution:**  
Reset sequencing and library linking order were corrected after reviewing SolvNet documentation.

---

### Issue 3: Library Mismatch Errors
**Problem:**  
Functional mismatch due to incorrect standard cell model linkage.

**Resolution:**  
Correct SCL180 functional models were linked during GLS elaboration.

---

## ✅ Final Observations

- RTL simulation and GLS behavior matched correctly
- Timing constraints were met without critical violations
- Synthesis results were clean and reproducible
- Full migration to Synopsys tools achieved successfully

---

## 🏁 Conclusion

This task successfully demonstrates the ability to work in an **industry-grade SoC design and verification environment** using Synopsys tools and SCL180 technology.

Key learnings include:
- Professional RTL-to-GLS verification flow
- Independent debugging using tool documentation
- Proper constraint handling during synthesis
- Importance of clean documentation and reproducibility

This task establishes readiness for **real-world VLSI and SoC research work**.

---

## ⏰ Deadline Compliance

**Submitted before:**  
14th December, 11:59 PM IST

✔ All mandatory requirements satisfied  
✔ Documentation updated as required

