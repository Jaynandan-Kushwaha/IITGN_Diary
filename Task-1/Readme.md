# Task 1 – RISC-V Reference SoC Functional & GLS Replication (SCL180)

## Overview

This task focuses on **replicating the complete RTL (functional) simulation and Gate-Level Simulation (GLS)** of the **vsdcaravel RISC-V SoC** using the **SCL180 PDK** on a local IITGN machine. The goal is to verify that the synthesized gate-level netlist behaves **functionally equivalent** to the RTL design.

This work follows the official reference repository provided by VSD and demonstrates understanding of SoC integration, simulation flow, and synthesis readiness.

---

## Reference Repository

* **Repository:** vsdRiscvScl180
* **Reference Path:** `iitgn/`

The following directories were studied in detail:

* `rtl/` – Complete RTL of vsdcaravel SoC
* `dv/` – Design verification (hkspi testbench)
* `synthesis/` – Synthesis outputs
* `gl/` – Gate-level netlists
* `gls/` – Gate-level simulation environment
* `images/` – Reference waveforms and logs

---

## System Setup

**Machine:** Local IITGN Linux Machine
**PDK:** SCL180
**Processor Core:** VexRiscv
**SoC:** vsdcaravel (RISC-V Reference SoC)

---

## Task Flow Summary

1. Study SoC architecture and reference flow
2. Run **RTL Functional Simulation** using hkspi test
3. Verify waveforms and console output
4. Run **Gate-Level Simulation (GLS)** using synthesized netlist
5. Compare RTL vs GLS waveforms
6. Document results with logs, VCDs, and screenshots

---

## 1. Functional (RTL) Simulation

### Objective

To verify correct functionality of the hkspi block using RTL simulation.

### Testbench Location

```
dv/hkspi/
```

### Key Setup

* RISC-V GCC path configured
* SCL IO models linked
* hkspi firmware compiled and loaded

### Expected Results

* Successful console output
* No simulation errors
* Clean waveform behavior

### Evidence

**Console Output:**

![RTL Console Output](images/rtl_console_output.png)

**RTL Waveform:**

![RTL Waveform](images/rtl_waveform.png)

---

## 2. Gate-Level Simulation (GLS)

### Objective

To verify that the **synthesized gate-level netlist** behaves identically to the RTL design.

### GLS Directory

```
gls/
```

### Netlist Modifications

The following black-box modules were replaced with behavioral models:

* `dummy_por`
* `RAM128`
* `housekeeping`

Included at the top of the synthesized netlist:

```
`include "dummy_por.v"
`include "RAM128.v"
`include "housekeeping.v"
```

### Power Pin Fix

In `vsdcaravel.v`, power pin connections were corrected by replacing:

```
1'b0 → vssa
```

This avoids floating ground issues during GLS.

### Expected Results

* GLS simulation completes successfully
* No unknown (X) states on critical signals
* GLS waveform closely matches RTL waveform

### Evidence

**GLS Console Output:**

![GLS Console Output](images/gls_console_output.png)

**GLS Waveform:**

![GLS Waveform](images/gls_waveform.png)

---

## 3. RTL vs GLS Comparison

### Comparison Criteria

* SPI transaction timing
* Reset behavior
* Clock alignment
* Register read/write correctness

### Observation

* RTL and GLS waveforms are **functionally equivalent**
* Minor delays observed in GLS due to gate-level timing
* No functional mismatches detected

**RTL vs GLS Waveform Comparison:**

![RTL vs GLS](images/rtl_vs_gls.png)

---

## Repository Structure

```
Day1_Task_Replication/
│── Functional_Simulation/
│   ├── logs/
│   ├── vcd/
│   └── waveforms/
│
│── GLS_Simulation/
│   ├── netlist/
│   ├── logs/
│   ├── vcd/
│   └── waveforms/
│
│── images/
│── README.md
```

---

## Issues Faced & Fixes

| Issue                          | Resolution                         |
| ------------------------------ | ---------------------------------- |
| Black-box module errors in GLS | Added behavioral includes          |
| Power pin mismatch             | Replaced constant ground with vssa |
| Missing IO models              | Corrected SCL IO library paths     |

---

## Deliverables Checklist

* RTL simulation log ✔
* RTL VCD ✔
* RTL waveform screenshot ✔
* GLS netlist ✔
* GLS simulation log ✔
* GLS VCD ✔
* GLS waveform screenshot ✔
* README documentation ✔

---

## Conclusion

This task successfully demonstrates **end-to-end verification** of the vsdcaravel RISC-V SoC using both RTL and gate-level simulations on the **SCL180 technology**. Functional equivalence between RTL and GLS confirms synthesis correctness and SoC readiness for tapeout-level flows.

---

**Status:** ✅ Completed
**Deadline:** Met (EOD)
