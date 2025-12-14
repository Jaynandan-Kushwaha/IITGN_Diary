
# Caravel RTL & GLS Verification Guide (SKY130)

This document describes the complete environment setup and verification flow for running **RTL** and **Gate-Level Simulation (GLS)** of the **Caravel SoC** using the **SKY130 PDK**.  
The verification flow is demonstrated using the **Housekeeping SPI (HKSPI) Directed Verification (DV)** test and is based on real debugging and simulation experience.

---

## Document Objective

- Establish a reproducible Caravel verification environment
- Execute RTL and GLS simulations successfully
- Compare RTL and GLS results for functional equivalence
- Document common issues and reliable solutions

---

## Target Platform

- Linux (Ubuntu recommended)
- Local machine or Virtual Machine environment

---

## Prerequisites

- Git installed
- Python and pip available
- Make utility present
- GCC / G++ toolchain
- Minimum 100 GB disk space
- Stable internet connection

---
## Task 1: Environment Setup

### Purpose

The purpose of this task is to establish a stable and reproducible verification environment capable of supporting both RTL simulation and gate-level simulation (GLS) for the Caravel SoC. A correctly configured environment is essential to ensure that any observed functional behavior is a result of the design itself and not influenced by tool or library inconsistencies.

### Description

The environment setup begins with the official **Caravel GitHub repository**, which serves as the reference design for this verification activity. The repository was reviewed to confirm the presence of all required RTL sources, design verification (DV) testbenches, and configuration files necessary for simulation and synthesis.

The **Sky130 Process Design Kit (PDK)** was installed and configured to provide access to standard cell libraries, Verilog timing models, and technology-specific files required during synthesis and gate-level simulation. Ensuring correct PDK integration is critical, as it directly impacts technology mapping and the functional accuracy of the gate-level netlist.

Verification tools were then validated to confirm correct installation and compatibility with the Caravel workflow. The following tools were confirmed to be operational:

- **Icarus Verilog (iverilog)** for compiling RTL and gate-level designs  
- **vvp** for executing compiled simulations and observing runtime behavior  
- **Verilator** (optional) for RTL analysis and consistency checks  

Each tool was tested within the configured environment to ensure proper interaction with Caravel source files and Sky130 libraries. This step helps eliminate toolchain-related issues that could otherwise surface during synthesis or long simulation runs.

### Expected Outcome

Upon completion of this task, the following conditions are satisfied:

- The Caravel repository is correctly cloned and structured
- Sky130 PDK paths are properly configured and accessible
- Simulation and synthesis tools execute reliably without configuration errors
- The environment is fully prepared for accurate RTL and GLS verification

Successful completion of this task establishes a solid foundation for all subsequent verification steps.

## Clone Caravel Repository

Clone the official Efabless Caravel repository and keep the directory structure unchanged.  
The repository contains RTL sources, DV environments, OpenLane flows, and Makefile targets required for simulation and synthesis.

---

## Repository Structure Overview

- RTL source files for Caravel and management SoC
- DV testbenches for verification
- OpenLane configuration files
- Top-level Makefile

---

## Install SKY130 PDK Using Volare (Recommended)

Volare provides reproducible and verified SKY130 PDK builds.  
A fixed, known-good PDK version should be enabled to avoid tool and library mismatches.

Ensure the PDK root path is exported permanently so that all tools can locate the PDK.

---

## Install SKY130 PDK Using OpenLane

OpenLane can automatically install the SKY130 PDK as part of its setup flow.  
This approach is useful when OpenLane is already being used for synthesis and place-and-route.

---

## Manual SKY130 PDK Installation Using open_pdks

Manual compilation of open_pdks is possible but slower and more error-prone.  
This method requires additional dependencies and is generally not recommended unless necessary.

---

## Verify Icarus Verilog Version (Mandatory)

Icarus Verilog **version 11** is required for Caravel RTL and GLS simulations.

Using older versions may result in:
- Incorrect signal propagation
- Random X states in GLS
- Parsing or elaboration errors
- Simulation crashes

---

## Verify Verilator Installation (Optional)

Verilator can be used for faster cycle-accurate RTL simulations but is optional for the HKSPI DV flow.

---

## Additional Tools Required for GLS

- Yosys for synthesis
- OpenLane for full RTL-to-GDS flow
- Magic and Netgen for layout verification
- GTKWave for waveform analysis

---

## HKSPI Directed Verification (DV) Overview

The HKSPI DV test validates the functionality of the Housekeeping SPI controller inside the Caravel management SoC.

The test emulates an external SPI master and verifies correct SPI-to-Wishbone transactions and register access.

---

## HKSPI DV Test Location

The HKSPI DV environment is located under the management SoC verification directory inside the Caravel repository.

---

## Key Files in HKSPI DV Test

- hkspi_tb.v – SPI master testbench and result monitor
- housekeeping_spi.v – RTL implementation of HKSPI
- hkspi.hex – SPI command stream and expected readback values

---

## Purpose of HKSPI Verification

- Validate SPI protocol handling
- Verify SPI-to-Wishbone bridge behavior
- Confirm register read/write correctness
- Ensure RTL and GLS functional equivalence

---

## Run HKSPI RTL Simulation

Run the RTL simulation using the provided Makefile target.  
A successful run must end with a clear PASS message from the test monitor.

---

## Save RTL Simulation Log

Save the complete RTL simulation output for later comparison with GLS results.

---

## Generate Gate-Level Netlist Using OpenLane

Generate synthesized SKY130 gate-level netlists using the provided GLS Makefile target.  
This step invokes Yosys and OpenLane automatically.

---

## Location of Generated Gate-Level Netlists

All synthesized gate-level netlists are placed inside the Caravel gate-level directory.  
These netlists are used for GLS execution.

---

## Run Gate-Level Simulation (GLS)

Run the GLS simulation using the Makefile target.  
The same HKSPI testbench is reused to validate gate-level functionality.

---

## Save GLS Simulation Log

Save the GLS simulation output for documentation and comparison with RTL results.

---

## Confirm Correct GLS Output

A successful GLS simulation must end with a PASS message indicating that gate-level behavior matches RTL.

---

## Compare RTL and GLS Logs

Compare RTL and GLS logs line-by-line to ensure functional equivalence.  
Minor differences in timestamps or formatting may be ignored.

---

## Compare RTL and GLS Waveforms Using GTKWave

Inspect RTL and GLS waveforms side-by-side and verify that:
- SPI transactions match bit-for-bit
- Register read values are identical
- No unexpected X or Z states appear in GLS

---

## Signals to Focus During Waveform Comparison

- Chip Select (CSB)
- SPI Clock (SCK)
- SPI Data In (SDI)
- SPI Data Out (SDO)
- Wishbone bus signals
- Internal register readback paths

---

## Common Problems Faced and Solutions

---

## Problem: Incorrect File or Directory Paths

Symptoms include missing modules and file-not-found errors.

Solution:
- Verify relative paths in Makefiles and testbenches
- Do not modify the repository directory structure

---

## Problem: Incorrect Icarus Verilog Version

Using versions older than 11 causes unstable GLS behavior.

Solution:
- Ensure Icarus Verilog version 11 is installed and active

---

## Problem: Missing caravel_pico Repository

Some simulations require additional management SoC wrapper files.

Solution:
- Clone the caravel_pico repository
- Carefully update include paths

---

## Problem: Insufficient Disk Space

OpenLane and PDK installations require large storage.

Solution:
- Allocate at least 100 GB disk space

---

## Quick Verification Checklist

- SKY130 PDK installed correctly
- PDK root path exported
- Icarus Verilog version 11 verified
- caravel_pico repository available
- RTL simulation passes
- GLS simulation passes
- RTL and GLS logs match
- Waveforms verified

---

## References

- Efabless Caravel Repository
- Volare PDK Manager
- OpenLane Flow
- open_pdks Repository
- SKY130 PDK Documentation

---

This README documents a **robust and industry-aligned RTL and GLS verification workflow** suitable for academic projects, research work, and pre-silicon validation.
