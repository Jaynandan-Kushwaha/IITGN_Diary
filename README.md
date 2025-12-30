# Task 3: Removal of On-Chip POR and Final GLS Validation (SCL-180)


## 🚀 Executive Summary and Objective

The objective of this work is to remove the internal Power-On Reset (POR) logic from the VSD Caravel–based RISC-V SoC and validate a simplified reset architecture based on a single external active-low reset signal.
Earlier implementations targeting the SKY130 PDK relied on a behavioral module commonly known as dummy_por to emulate power sequencing during simulation. While suitable for early RTL validation, this approach does not represent real silicon behavior and is not appropriate for a production-level physical design flow.
For the SCL 180 nm technology, the internal POR circuitry has been completely removed. Reset control is instead handled by an external reset pad provided by the IO library. These reset pads are asynchronous and become available immediately after power stabilization, eliminating the need for any internal reset generation logic.
The RTL was refactored accordingly, followed by synthesis using DC Topographical mode and full gate-level simulation. The results confirm that the design functions correctly without any internal POR logic and remains fully synthesizable and timing-clean.
This repository documents the complete engineering justification for POR removal, along with the implementation and verification steps, demonstrating that the design is physically realistic, simpler, and ready for downstream physical design and tapeout.

## 📘 Context and Engineering Rationale

### Why the Internal POR Was Removed

The transition to an external-only reset architecture is driven by fundamental silicon-level considerations. The decision is supported by three key technical reasons, outlined below.

### 1. Practical Silicon Reality

A true Power-On Reset circuit is inherently analog in nature. It depends on voltage references, comparators, hysteresis behavior, and carefully controlled thresholds. Any POR logic written purely in Verilog using counters, delays, or clock-based constructs does not reflect real silicon behavior. Such digital approximations can introduce synthesis ambiguities and unpredictable behavior during actual power-up, making them unsuitable for a physically accurate design.

### 2. Reset Pad Behavior in SCL 180 nm

Detailed inspection of the SCL 180 nm I/O library confirms that the reset input pads are fully functional without any internal enable or POR-controlled gating. The reset signal propagates directly from the pad into the core logic once power is applied. This means no internal POR circuitry is required to activate or qualify the reset path.

### 3. System-Level Robustness

Modern SoC designs rely on board-level reset supervisors rather than internal POR generators. An externally controlled reset ensures that reset deassertion occurs only after the entire power delivery network has stabilized. This approach avoids the risk of premature reset release caused by local on-chip voltage fluctuations during ramp-up.

---

## 📑 Phase 1: Pre-Design Analysis

Before making any RTL changes, a structured dependency analysis was performed to fully understand how POR-related signals were used in the legacy design. This ensured that POR removal would not introduce functional regressions.

### POR Usage Analysis Document

The POR usage analysis traces every occurrence of por-related signals such as porb_h, porb_l, and dummy_por within the original vsdcaravel design.

This analysis includes:
- A complete audit of module instantiations
- Fan-out tracing of POR-related signals
- Identification of blocks dependent on POR versus generic reset
- Clear documentation of safe removal points

### Pad Reset Analysis Document

A separate pad-focused study was conducted to validate reset behavior at the I/O boundary.

This document covers:
- Detailed review of SCL 180 nm I/O pad datasheets
- Power-up and reset signal availability analysis
- Comparison between SCL 180 nm and SKY130 reset requirements
- Risk assessment and mitigation through board-level reset control

## 🛠️ Phase-2: RTL Refactoring Strateg
 search for every instance in the design where dummy_por and its corresponding signals are used

### Modifications Implemented

The `dummy_por` module, which previously utilized non-synthesizable delays to mimic startup, was completely removed. The top-level `vsdcaravel.v` and housekeeping logic were refactored to accept a single global input: `input reset_n`.

### Legacy Signal Compatibility

To maintain compatibility with the internal SoC hierarchy without rewriting every sub-module, a direct wire-mapping strategy was implemented. Legacy POR signal names are now aliases for the external reset pin.
 I just take reset_n as a input from testbench and assign reset_n to all the internal signal like porb_h, porb_l, and inverse of por_l we just use them as a wire they will not perform any behaviour like por module we just use them they are just name but it will work according to the outseide reset signal
 
```verilog
input reset_n;  // Single External Active-Low Reset

// Mapping legacy POR names to the external pin
assign porb_h = reset_n;  // Power-on-Reset Bar (High Voltage Domain)
assign porb_l = reset_n;  // Power-on-Reset Bar (Low Voltage Domain)
assign por_l  = ~reset_n; 
```

### RTL Verification (VCS)


Verification was performed to ensure that removing the internal delay logic did not break the reset sequence.

#### Simulation Command

```bash
vcs -full64 -sverilog -timescale=1ns/1ps -debug_access+all \
    +incdir+../ +incdir+../../rtl +incdir+../../rtl/scl180_wrapper \
    +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/6M1L/verilog/tsl18cio250/zero \
    +define+FUNCTIONAL +define+SIM \
    hkspi_tb.v -o simv

./simv -no_save +define+DUMP_VCD=1 | tee sim_log.txt
```
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/rtl%20simu%20without%20dummy-por.png)

### GTK Wave without por 

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gtkwave%20rtl%20simu%20without%20dummy-por.png)

#### Results

- **Reset Timing**: The waveform confirms `reset_n` releases at 1000ns exactly as driven by the testbench
- **State Integrity**: Register read/write operations (Reg 0 to 18) function correctly immediately after reset release

## 🧩 Phase 3: DC_TOPO Synthesis Results

The Raven SoC was synthesized using Synopsys Design Compiler in Topographical mode (DC_TOPO) with the SCL 180 nm standard cell library. This phase validates that removing the internal POR logic does not introduce synthesis issues such as unresolved references, latch inference, or hidden reset structures.

The primary objective of this stage was to confirm that the design remains structurally clean and fully reset-driven using only the external active-low reset signal.

### Synthesis Execution Flow

The synthesis flow was executed from the designated synthesis working directory using the standard DC script. The run completed successfully without fatal errors or warnings related to reset logic.

### Post-Synthesis Design Statistics

Key characteristics of the generated netlist are summarized below:

- Total number of ports: 12,749  
  This confirms that the full I/O boundary is preserved after POR removal.

- Total number of nets: 37,554  
  Indicates stable connectivity across the design hierarchy.

- Sequential cells: 6,882  
  All sequential elements are driven exclusively by the external reset signal.

- Combinational cells: 18,422  
  Reflects expected logic density for the Raven SoC.

- Macros or black boxes: 16  
  No POR-related black boxes are present in the synthesized netlist.

- Total standard cell area: 773,088.68 square microns  
  Demonstrates an optimized footprint using SCL 180 nm cells.

### Synthesis Quality Validation

The following quality checks were explicitly verified after synthesis:

- No unresolved module or cell references were reported  
  All instances are correctly bound to SCL 180 nm library cells.

- No inferred latches were detected  
  Removal of POR logic did not introduce unintended storage elements.

- Reset structure is clean and uniform  
  Reports confirm that the external reset signal directly controls the reset pins of all flip-flops.

These results confirm that the design is synthesis-safe, reset-consistent, and structurally correct without any internal Power-On Reset circuitry.

#### command 

````
dc_shell -f ../dc_shell.tcl | tee dc_shell.log
````

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/synthesis.png)

---

> Note:- In last i solved these errors but didnt take ss you can find error free synthesis in my synthesis folder there were no single error in synthesis file even not a linking error 

## 🧪 Phase-4: Final Gate-Level Simulation (GLS)

### Objective

GLS acts as the **"Final Proof"**. It validates that the synthesized netlist (which now lacks the behavioral POR) functions correctly when the reset is applied externally in a simulation environment using SCL-180 functional models.

### Checklist & Observations

- **Reset Assertion**: The `reset_n` signal is held low during the initialization phase (0 to 1000ns). No X-propagation was observed on output pins.

- **Reset De-assertion**: Upon release, the internal clock tree activates immediately.

- **Functional Equivalence**: The SPI register tests passed, matching the RTL behavior exactly.

### command 

```
vcs -full64 -sverilog -timescale=1ns/1ps \
    -debug_access+all \
    +define+FUNCTIONAL+SIM+GL \
    +notimingchecks \
    hkspi_tb.v \
    +incdir+../synthesis/output \
    +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/verilog/tsl18cio250/zero \
    +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/stdcell/fs120/4M1IL/verilog/vcs_sim_model \
    -o simv

./simv
```

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gls%20simulation.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gls%20simv%20log.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gls%20vcs%20log.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gtkwave%20gls.png)

## 📌 Engineering Deliverables and Design Justification

As part of the task requirements, a complete set of supporting documents has been prepared to justify the architectural decision to remove the internal Power-On Reset logic. These documents collectively provide traceability, risk analysis, and verification evidence for design review.

### POR Removal Justification Document

The file POR_Removal_Justification.md serves as the final decision record for this change. It formally captures the engineering rationale behind eliminating the internal POR mechanism.

This document explains:
- Why Power-On Reset is fundamentally an analog function and cannot be safely modeled in RTL
- Why RTL-based POR implementations are unsafe for silicon tapeout
- The risks evaluated during the decision process and the mitigation strategies applied
- Alignment of the chosen approach with standard industry practices
- Architectural differences between SCL 180 nm and SKY130 technologies

### Core Technical Arguments

#### POR as an Analog Function

A true POR circuit relies on analog behavior that cannot be represented in synthesizable logic. Such circuits require voltage comparators, well-defined hysteresis, and noise immunity mechanisms, along with timing references independent of the system clock. These properties are intrinsic to analog design and cannot be recreated reliably using digital RTL constructs.

#### Limitations of RTL-Based POR

Implementing POR purely in RTL introduces multiple risks:
- Circular dependency, where POR logic depends on a clock that may not be stable during power-up
- Lack of hysteresis, making the reset signal vulnerable to supply noise and voltage ramp fluctuations
- Unverifiable timing behavior, as reset release depends on process- and temperature-sensitive delays
- Undefined synthesis behavior, since power-edge detection is not supported by synthesis tools

#### Risk Mitigation Strategy

The identified risks are mitigated through a system-level reset strategy:
- Early reset release is prevented using a board-level reset supervisor with built-in hysteresis
- Power-up unknown states are avoided by holding reset low throughout VDD ramp-up
- Reset pin noise is reduced using external pull-down resistors and RC filtering
- Missing or stalled reset conditions are handled using supervisor watchdog functionality

The overall logic and reset control flow of the design follows a clean external-reset-driven model.

---

## ✔ Verification Summary

### RTL Simulation

RTL-level simulations completed successfully. Reset assertion and de-assertion behavior was verified, all register operations functioned correctly, and no functional or timing-related issues were observed.

### Synthesis Validation

Synthesis using DC Topographical mode completed without errors. All modules were mapped correctly to SCL 180 nm standard cells, no unresolved references were reported, no dummy POR instances were found in the netlist, and no inferred latches were introduced.

### Gate-Level Simulation

Gate-level simulation confirmed functional equivalence with the RTL design. Reset propagation was clean, no glitches were observed, all test vectors passed, and waveform inspection showed expected behavior across the design.

---

## 📎 Quick Reference Summary

Phase 1: POR usage analysis document – completed  
Phase 1: Pad reset analysis document – completed  
Phase 2: POR-free RTL implementation – verified  
Phase 3: DC_TOPO synthesis – clean netlist generated  
Phase 4: Gate-level simulation – functional match confirmed  
Phase 5: POR removal justification document – completed  

---

## 🔍 Key Conclusions

1. Power-On Reset is inherently an analog problem and should not be implemented in digital RTL
2. SCL 180 nm IO pads fully support an external reset without internal POR assistance
3. The synthesized netlist contains no POR-related logic or hidden reset structures
4. Functional correctness is preserved, as proven by gate-level simulation
5. The adopted reset strategy aligns with established industry ASIC design practices
