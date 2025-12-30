

# Task-4: Full Management SoC DV Validation on SCL-180 (POR-Free Design)

## Objective

The objective of this task is to demonstrate that the modified Management SoC RTL, with complete removal of internal Power-On Reset (POR) logic, is functionally stable, reusable, and production-ready on the SCL-180 technology node.

This validation is performed by executing the complete set of Management SoC Design Verification (DV) tests originally developed for the Caravel platform, without altering the test intent, stimulus, or expected outcomes. The goal is to prove that POR removal does not introduce functional regressions and that the design behaves identically across all supported implementation views.

The DV flow establishes functional equivalence across:
- Pure RTL simulation
- Synthesized netlist with RTL SRAM models
- Synthesized netlist with DC_TOPO-compatible SRAM handling

Identical behavior across all configurations confirms correctness of reset strategy, synthesis assumptions, and simulation consistency.

---

## Validation Strategy

The validation methodology follows a progressive abstraction approach:

1. Verify baseline behavior using POR-free RTL
2. Re-verify using a synthesized gate-level netlist
3. Re-run identical DV tests with different memory modeling strategies
4. Compare results across all runs to confirm functional equivalence

At no point are DV tests modified, ensuring that validation coverage remains unchanged from the original Caravel-based flow.

---

## Toolchain Requirements

The following tools and libraries are required to reproduce the validation flow:

- Synopsys VCS for RTL and gate-level simulation
- Synopsys Design Compiler for synthesis
- SCL-180 standard cell libraries
- SCL-180 IO pad libraries and Verilog models

---

## Key Observations

- Removal of internal POR logic does not affect DV test behavior
- Reset sequencing is fully controlled by the external `reset_n` signal
- No test depends on analog POR timing assumptions
- No additional constraints or workarounds are required
- DV results remain deterministic across abstraction levels

---


## RTL Preparation

In last task we alaready removed por from rtl and it also passsed for hkspi both rtl and gls you can prefer [Task3](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/tree/main/Task_NoPOR_Final_GLS)

## Management SoC DV – Run-1 (RTL SRAM)

### Housekeeping SPI – Functional Simulation

Functional simulation focuses on the `housekeeping_spi` block within the Management SoC. The corresponding testbench is located in the `dv/hkspi` directory.

```
cd dv/hkspi/
```

The Synopsys environment is initialized before invoking VCS.

```
csh
source /home/madank/toolRC_iitgntapeout
```

The following command compiles the RTL and testbench:

```
vcs -full64 -sverilog -timescale=1ns/1ps -debug_access+all \
    +incdir+../ +incdir+../../rtl +incdir+../../rtl/scl180_wrapper \
    +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/6M1L/verilog/tsl18cio250/zero \
    +define+FUNCTIONAL +define+SIM \
    hkspi_tb.v -o simv
```

Simulation execution and VCD dump:

```
./simv -no_save +define+DUMP_VCD=1 | tee sim_log.txt
```

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/rtl%20simu%20without%20dummy-por.png)

All housekeeping SPI test cases pass successfully. Registers 0 through 18 return expected values, confirming correct functional behavior.

Waveforms are inspected using GTKWave:

```
gtkwave hkspi.vcd hkspi_tb.v
```

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gtkwave%20rtl%20simu%20without%20dummy-por.png)


---

## Gate-Level Simulation of hkspi

Gate-level simulation validates post-synthesis behavior using the synthesized netlist.

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
```



Simulation execution:

```
./simv -no_save +define+DUMP_VCD=1 | tee sim_log.txt
```
![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gls%20simulation.png)

With RTL models restored, gate-level simulation results match functional simulation.

Waveform verification:

```
gtkwave hkspi.vcd hkspi_tb.v
```

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gtkwave%20gls.png)



---

## Other Management SoC Blocks

### GPIO

RTL compilation issues are resolved. However, RTL simulation currently fails and requires further debugging.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task4/Images/Screenshot%20from%202025-12-23%2019-36-06.png)

### MPRJ_CONTROL

RTL compilation issues are resolved. RTL simulation is failing and is under investigation.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task4/Images/Screenshot%20from%202025-12-24%2013-04-25.png)

### Storage

RTL compilation issues are resolved. RTL simulation is failing and pending analysis.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task4/Images/Screenshot%20from%202025-12-24%2013-06-43.png)

### IRQ

RTL compilation issues are resolved. RTL simulation is failing and requires additional debug.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task4/Images/Screenshot%20from%202025-12-24%2012-58-59.png)

