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

## Test Instructions
### Repo Setup
1. Clone the repository:
   ```sh
   
   git clone -b iitgn https://github.com/vsdip/vsdRiscvScl180.git --single-branch
   ```
2. Install required dependencies (ensure dc_shell and SCL180 PDK are properly set up).
3. source the synopsys tools
4. go to home
   ```
   csh
   source toolRC_iitgntapeout
   ```


## tool Version 

![version](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task-1/Images/WhatsApp%20Image%202025-12-17%20at%202.28.48%20PM.jpeg)

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

 - Edit Makefile at this path [./dv/hkspi/Makefile](./dv/hkspi/Makefile)
   - Modify and verify `GCC_Path` to point to correct riscv installation
   - Modify and verify `scl_io_PATH` to point to correct io
   - edit the Makefile as follows
   
Here's the complete rewritten Makefile for Synopsys VCS:

```makefile
# SPDX-FileCopyrightText: 2020 Efabless Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

# removing pdk path as everything has been included in one whole directory for this example.
# PDK_PATH = $(PDK_ROOT)/$(PDK)
scl_io_PATH = "/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/verilog/tsl18cio250/zero"
VERILOG_PATH = ../../
RTL_PATH = $(VERILOG_PATH)/rtl
BEHAVIOURAL_MODELS = ../ 
RISCV_TYPE ?= rv32imc

FIRMWARE_PATH = ../
GCC_PATH?=/usr/bin/gcc
GCC_PREFIX?=riscv32-unknown-elf

SIM_DEFINES = +define+FUNCTIONAL +define+SIM

SIM?=RTL

.SUFFIXES:

PATTERN = hkspi

# Path to management SoC wrapper repository
scl_io_wrapper_PATH ?= $(RTL_PATH)/scl180_wrapper

# VCS compilation options
VCS_FLAGS = -sverilog +v2k -full64 -debug_all -lca -timescale=1ns/1ps
VCS_INCDIR = +incdir+$(BEHAVIOURAL_MODELS) \
             +incdir+$(RTL_PATH) \
             +incdir+$(scl_io_wrapper_PATH) \
             +incdir+$(scl_io_PATH)

# Output files
SIMV = simv
COMPILE_LOG = compile.log
SIM_LOG = simulation.log

.SUFFIXES:

all: compile

hex: ${PATTERN:=.hex}

# VCS Compilation target
compile: ${PATTERN}_tb.v ${PATTERN}.hex
	vcs $(VCS_FLAGS) $(SIM_DEFINES) $(VCS_INCDIR) \
	${PATTERN}_tb.v \
	-l $(COMPILE_LOG) \
	-o $(SIMV)

# Run simulation in batch mode
sim: compile
	./$(SIMV) -l $(SIM_LOG)

# Run simulation with GUI (DVE)
gui: compile
	./$(SIMV) -gui -l $(SIM_LOG) &

# Generate VPD waveform
vpd: compile
	./$(SIMV) -l $(SIM_LOG)
	@echo "VPD waveform generated. View with: dve -vpd vcdplus.vpd &"

# Generate FSDB waveform (if Verdi is available)
fsdb: compile
	./$(SIMV) -l $(SIM_LOG)
	@echo "FSDB waveform generated. View with: verdi -ssf <filename>.fsdb &"

#%.elf: %.c $(FIRMWARE_PATH)/sections.lds $(FIRMWARE_PATH)/start.s
#	${GCC_PATH}/${GCC_PREFIX}-gcc -march=$(RISCV_TYPE) -mabi=ilp32 -Wl,-Bstatic,-T,$(FIRMWARE_PATH)/sections.lds,--strip-debug -ffreestanding -nostdlib -o $@ $(FIRMWARE_PATH)/start.s $<
#
#%.hex: %.elf
#	${GCC_PATH}/${GCC_PREFIX}-objcopy -O verilog $< $@ 
#	# to fix flash base address
# #	sed -i 's/@10000000/@00000000/g' $@
#
#%.bin: %.elf
#	${GCC_PATH}/${GCC_PREFIX}-objcopy -O binary $< /dev/stdout | tail -c +1048577 > $@

check-env:
#ifndef PDK_ROOT
#	$(error PDK_ROOT is undefined, please export it before running make)
#endif
#ifeq (,$(wildcard $(PDK_ROOT)/$(PDK)))
#	$(error $(PDK_ROOT)/$(PDK) not found, please install pdk before running make)
#endif
ifeq (,$(wildcard $(GCC_PATH)/$(GCC_PREFIX)-gcc ))
	$(error $(GCC_PATH)/$(GCC_PREFIX)-gcc is not found, please export GCC_PATH and GCC_PREFIX before running make)
endif
# check for efabless style installation
ifeq (,$(wildcard $(PDK_ROOT)/$(PDK)/libs.ref/*/verilog))
#SIM_DEFINES := ${SIM_DEFINES} +define+EF_STYLE
endif

# ---- Clean ----

clean:
	rm -f $(SIMV) *.log *.vpd *.fsdb *.key
	rm -rf simv.daidir csrc DVEfiles verdiLog novas.* *.fsdb+
	rm -rf AN.DB

.PHONY: clean compile sim gui vpd fsdb all check-env
```

## Key Changes Made

1. **Replaced `iverilog` with `vcs`** compilation
2. **Changed `-I` to `+incdir+`** for include directories
3. **Changed `+define+` syntax** for SIM_DEFINES (VCS standard)
4. **Added VCS-specific flags**: `-sverilog`, `+v2k`, `-full64`, `-debug_all`, `-lca`
5. **Removed all `.vvp` and `.vcd` targets** - replaced with `compile`, `sim`, `gui`, `vpd`, `fsdb`
6. **Updated clean target** to remove VCS-generated files: `simv.daidir`, `csrc`, `DVEfiles`, etc.
7. **Added separate targets** for batch simulation and GUI simulation
8. **Completely removed** any reference to `iverilog`, `vvp`, or `gtkwave`

### Command

```
run make
```

### 📷 Evidence
![RTL Simulation Waveform](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/first%20cmmand.png)
![rtl simulation commnad](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/1st%20command%20end.png)
![log](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/function%20vcs%20log%20sss.png)
![output](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/second%20command%20sim.v%20with%20log%20file.png)
![log file](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/sim%20v%20log%20file.png)
![vdpfileoutput](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/proof%20for%20vcdplus%20vpd.png)


## GTK Wave 

![GTKwave output](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/gtkwave%20output.png)

---
## Errors you might encounter after changing Makefile follow the below solutions to clear those errors 

### Error 1: Variable TMPDIR (tmp) is selecting a non-existent directory.

Create the tmp directory in your current location

```
mkdir -p tmp
```
- rerun the make compile command

#### Why This Happens

VCS needs a temporary directory to store intermediate compilation files. The setup script set TMPDIR=tmp, which is a relative path referring to a tmp subdirectory in your current working director[...]

### Error 2: Error-[IND] Identifier not declared


Now there's an error in dummy_schmittbuf.v where signals are not declared. This happens because default_nettype none is set somewhere in your code, which requires explicit declaration of all sign[...]

#### Fix the dummy_schmittbuf.v file

Step 1: Open the file

```
gedit ../../rtl/dummy_schmittbuf.v
```
Step 2: Change `default_nettype wire to `default_nettype none

```
`default_nettype wire

module dummy_schmittbuf (
    output UDP_OUT,
    input UDP_IN,
    input VPWR,
    input VGND
);
    
    assign UDP_OUT = UDP_IN;
    
endmodule

```
Step 3: In the same file change primitive name which contains a special character $ in dummy__udp_pwrgood_pp$PG module and referencing modules

from :
```
dummy__udp_pwrgood_pp$PG
```

to :
```
dummy__udp_pwrgood_pp_PG
```
Step 4: After all changes re-run the command make


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

as per the task 2 we need to make modules like POR and memory modules such as RAM128 and RAM256 as blackbox and should not synthesize them. To do that we need to change synth.tcl with many change[...]

```synth.tcl
# ========================================================================
# Synopsys DC Synthesis Script for vsdcaravel
# Modified to keep POR and Memory modules as complete RTL blackboxes
# ========================================================================

# ========================================================================
# Load technology libraries
# ========================================================================
read_db "/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/liberty/tsl18cio250_min.db"
read_db "/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/stdcell/fs120/4M1IL/liberty/lib_flow_ff/tsl18fs120_scl_ff.db"

# ========================================================================
# Set library variables
# ========================================================================
set target_library "/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/liberty/tsl18cio250_min.db /home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/stdcell/fs120/4M1IL/libert[...]
set link_library " /home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/liberty/tsl18cio250_min.db /home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/stdcell/fs120/4M1IL/libert[...]
set_app_var target_library $target_library
set_app_var link_library $link_library

# ========================================================================
# Define directory paths
# ========================================================================
set root_dir "/home/jkushwaha/vsdRiscvScl180"
set io_lib "/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/verilog/tsl18cio250/zero"
set verilog_files "$root_dir/rtl"
set top_module "vsdcaravel"
set output_file "$root_dir/synthesis/output/vsdcaravel_synthesis.v"
set report_dir "$root_dir/synthesis/report"

# ========================================================================
# Configure Blackbox Handling
# ========================================================================
# Prevent automatic memory inference and template saving
set_app_var hdlin_infer_multibit default_none
set_app_var hdlin_auto_save_templates false
set_app_var compile_ultra_ungroup_dw false

# ========================================================================
# Create Blackbox Stub File for Memory and POR Modules
# ========================================================================
set blackbox_file "$root_dir/synthesis/memory_por_blackbox_stubs.v"
set fp [open $blackbox_file w]
puts $fp "// Blackbox definitions for memory and POR modules"
puts $fp "// Auto-generated by synthesis script"
puts $fp ""

// RAM128 blackbox
puts $fp "(* blackbox *)"
puts $fp "module RAM128(CLK, EN0, VGND, VPWR, A0, Di0, Do0, WE0);"
puts $fp "  input CLK, EN0, VGND, VPWR;"
puts $fp "  input [6:0] A0;"
puts $fp "  input [31:0] Di0;"
puts $fp "  input [3:0] WE0;"
puts $fp "  output [31:0] Do0;"
puts $fp "endmodule"
puts $fp ""

// RAM256 blackbox
puts $fp "(* blackbox *)"
puts $fp "module RAM256(VPWR, VGND, CLK, WE0, EN0, A0, Di0, Do0);"
puts $fp "  input CLK, EN0;"
puts $fp "  inout VPWR, VGND;"
puts $fp "  input [7:0] A0;"
puts $fp "  input [31:0] Di0;"
puts $fp "  input [3:0] WE0;"
puts $fp "  output [31:0] Do0;"
puts $fp "endmodule"
puts $fp ""

// dummy_por blackbox
puts $fp "(* blackbox *)"
puts $fp "module dummy_por(vdd3v3, vdd1v8, vss3v3, vss1v8, porb_h, porb_l, por_l);"
puts $fp "  inout vdd3v3, vdd1v8, vss3v3, vss1v8;"
puts $fp "  output porb_h, porb_l, por_l;"
puts $fp "endmodule"
puts $fp ""

close $fp
puts "INFO: Created blackbox stub file: $blackbox_file"

# ========================================================================
# Read RTL Files
# ========================================================================
# Read defines first
read_file $verilog_files/defines.v

# Read blackbox stubs FIRST (before actual RTL)
puts "INFO: Reading memory and POR blackbox stubs..."
read_file $blackbox_file -format verilog

# ========================================================================
# Read RTL files excluding memory and POR modules
# ========================================================================
puts "INFO: Building RTL file list (excluding RAM128.v, RAM256.v, and dummy_por.v)..."

# Get all verilog files
set all_rtl_files [glob -nocomplain ${verilog_files}/*.v]

# Define files to exclude
set exclude_files [list \
    "${verilog_files}/RAM128.v" \
    "${verilog_files}/RAM256.v" \
    "${verilog_files}/dummy_por.v" \
]

# Build list of files to read
set rtl_to_read [list]
foreach file $all_rtl_files {
    set excluded 0
    foreach excl_file $exclude_files {
        if {[string equal $file $excl_file]} {
            set excluded 1
            puts "INFO: Excluding $file (using blackbox instead)"
            break
        }
    }
    if {!$excluded} {
        lappend rtl_to_read $file
    }
}

puts "INFO: Reading [llength $rtl_to_read] RTL files..."

# Read all RTL files EXCEPT RAM128.v, RAM256.v, and dummy_por.v
read_file $rtl_to_read -define USE_POWER_PINS -format verilog

# ========================================================================
# Elaborate Design
# ========================================================================
puts "INFO: Elaborating design..."
elaborate $top_module

# ========================================================================
# Set Blackbox Attributes for Memory Modules
# ========================================================================
puts "INFO: Setting Blackbox Attributes for Memory Modules..."

# Mark RAM128 as blackbox
if {[sizeof_collection [get_designs -quiet RAM128]] > 0} {
    set_attribute [get_designs RAM128] is_black_box true -quiet
    set_dont_touch [get_designs RAM128]
    puts "INFO: RAM128 marked as blackbox"
}

# Mark RAM256 as blackbox
if {[sizeof_collection [get_designs -quiet RAM256]] > 0} {
    set_attribute [get_designs RAM256] is_black_box true -quiet
    set_dont_touch [get_designs RAM256]
    puts "INFO: RAM256 marked as blackbox"
}

# ========================================================================
# Set POR (Power-On-Reset) Module as Blackbox
# ========================================================================
puts "INFO: Setting POR module as blackbox..."

# Mark dummy_por as blackbox
if {[sizeof_collection [get_designs -quiet dummy_por]] > 0} {
    set_attribute [get_designs dummy_por] is_black_box true -quiet
    set_dont_touch [get_designs dummy_por]
    puts "INFO: dummy_por marked as blackbox"
}

# Handle any other POR-related modules (case insensitive)
foreach_in_collection por_design [get_designs -quiet "*por*"] {
    set design_name [get_object_name $por_design]
    if {![string equal $design_name "dummy_por"]} {
        set_dont_touch $por_design
        set_attribute $por_design is_black_box true -quiet
        puts "INFO: $design_name set as blackbox"
    }
}

# ========================================================================
# Protect blackbox instances from optimization
# ========================================================================
puts "INFO: Protecting blackbox instances from optimization..."

# Protect all instances of RAM128, RAM256, and dummy_por
foreach blackbox_ref {"RAM128" "RAM256" "dummy_por"} {
    set instances [get_cells -quiet -hierarchical -filter "ref_name == $blackbox_ref"]
    if {[sizeof_collection $instances] > 0} {
        set_dont_touch $instances
        set inst_count [sizeof_collection $instances]
        puts "INFO: Protected $inst_count instance(s) of $blackbox_ref"
    }
}

# ========================================================================
# Link Design
# ========================================================================
puts "INFO: Linking design..."
link

# ========================================================================
# Uniquify Design
# ========================================================================
puts "INFO: Uniquifying design..."
uniquify

# ========================================================================
# Read SDC constraints (if exists)
# ========================================================================
if {[file exists "$root_dir/synthesis/vsdcaravel.sdc"]} {
    puts "INFO: Reading timing constraints..."
    read_sdc "$root_dir/synthesis/vsdcaravel.sdc"
}

# ========================================================================
# Compile Design (Basic synthesis)
# ========================================================================
puts "INFO: Starting compilation..."
compile_ultra -incremental

# ========================================================================
# Write Outputs
# ========================================================================
puts "INFO: Writing output files..."

# Write Verilog netlist
write -format verilog -hierarchy -output $output_file
puts "INFO: Netlist written to: $output_file"

# Write DDC format for place-and-route
write -format ddc -hierarchy -output "$root_dir/synthesis/output/vsdcaravel_synthesis.ddc"
puts "INFO: DDC written to: $root_dir/synthesis/output/vsdcaravel_synthesis.ddc"

# Write SDC with actual timing constraints
write_sdc "$root_dir/synthesis/output/vsdcaravel_synthesis.sdc"
puts "INFO: SDC written to: $root_dir/synthesis/output/vsdcaravel_synthesis.sdc"

# ========================================================================
# Generate Reports
# ========================================================================
puts "INFO: Generating reports..."

report_area > "$report_dir/area.rpt"
report_power > "$report_dir/power.rpt"
report_timing -max_paths 10 > "$report_dir/timing.rpt"
report_constraint -all_violators > "$report_dir/constraints.rpt"
report_qor > "$report_dir/qor.rpt"

# Report on blackbox modules
puts "INFO: Generating blackbox module report..."
set bb_report [open "$report_dir/blackbox_modules.rpt" w]
puts $bb_report "========================================"
puts $bb_report "Blackbox Modules Report"
puts $bb_report "========================================"
puts $bb_report ""

foreach bb_module {"RAM128" "RAM256" "dummy_por"} {
    puts $bb_report "Module: $bb_module"
    set instances [get_cells -quiet -hierarchical -filter "ref_name == $bb_module"]
    if {[sizeof_collection $instances] > 0} {
        puts $bb_report "  Status: PRESENT"
        puts $bb_report "  Instances: [sizeof_collection $instances]"
        foreach_in_collection inst $instances {
            puts $bb_report "    - [get_object_name $inst]"
        }
    } else {
        puts $bb_report "  Status: NOT FOUND"
    }
    puts $bb_report ""
}
close $bb_report
puts "INFO: Blackbox report written to: $report_dir/blackbox_modules.rpt"

# ========================================================================
# Summary
# ========================================================================
puts ""
puts "INFO: ========================================"
puts "INFO: Synthesis Complete!"
puts "INFO: ========================================"
puts "INFO: Output netlist: $output_file"
puts "INFO: DDC file: $root_dir/synthesis/output/vsdcaravel_synthesis.ddc"
puts "INFO: SDC file: $root_dir/synthesis/output/vsdcaravel_synthesis.sdc"
puts "INFO: Reports directory: $report_dir"
puts "INFO: Blackbox stub file: $blackbox_file"
puts "INFO: "
puts "INFO: NOTE: The following modules are preserved as blackboxes:"
puts "INFO:   - RAM128 (Memory macro)"
puts "INFO:   - RAM256 (Memory macro)"
puts "INFO:   - dummy_por (Power-On-Reset circuit)"
puts "INFO: These modules will need to be replaced with actual macros during P&R"
puts "INFO: ========================================"

# Exit dc_shell
# dc_shell> exit
```
## Synthesis tcl Flow Summary

The synthesis flow consists of the following major stages:

1. Technology library loading  
2. RTL and blackbox handling  
3. Design elaboration and linking  
4. Timing constraint application  
5. Topographical synthesis  
6. Netlist and report generation  

---

### 1. Technology Library Loading

The synthesis script begins by loading **compiled Liberty database (.db) files** corresponding to the SCL180 process.

**Purpose:**
- Provide timing, power, and area models for standard cells
- Enable accurate timing analysis during synthesis

**Libraries Used:**
- I/O pad library for chip periphery
- Standard cell library for core logic
- Fast-fast and minimum timing corners for safe analysis

This ensures that all synthesized logic is mapped correctly to the target technology.

---

### 2. Target and Link Library Configuration

The script explicitly sets:

- **Target Library** → Cells used for logic mapping  
- **Link Library** → Libraries used to resolve all references

**Why both are required:**
- Target library defines *where logic is mapped*
- Link library ensures *all referenced cells are found*

This prevents unresolved references and synthesis mismatches.

---

### 3. Project Path and Variable Setup

All major paths are defined using TCL variables:

- Project root directory  
- RTL source location  
- Top module name (`vsdcaravel`)  
- Output netlist location  
- Report directory  

**Advantage:**
- Easy maintenance
- Portable script
- Clean structure without hard-coded paths

---

### 4. HDL and Memory Inference Control

Several DC application variables are configured to **disable unwanted memory inference**.

**Key Intent:**
- Prevent RAM blocks from being synthesized into flip-flops
- Avoid automatic template creation
- Preserve hierarchy for critical blocks

This is essential for SoC designs where memories are treated as macros.

---

### 5. Blackbox Stub Generation

A Verilog file containing **blackbox module declarations** is generated dynamically.

**Modules declared as blackboxes:**
- `RAM128`
- `RAM256`
- `dummy_por` (Power-On Reset)

These modules include **only port definitions**, with no internal logic.

**Why this is important:**
- Allows synthesis to proceed without memory internals
- Ensures correct interface visibility
- Prevents accidental optimization or removal

---

### 6. RTL Read Order Management

The RTL files are read in a **controlled order**:

1. `defines.v` (macros and compile-time definitions)
2. Blackbox stub file
3. Remaining RTL files (excluding actual RAM/POR implementations)

**Reason:**
- Ensures blackbox definitions take priority
- Avoids duplicate or conflicting module definitions

---

### 7. RTL File Filtering

All RTL files are scanned, and **memory/POR implementation files are excluded** from synthesis input.

This guarantees that:
- Only blackbox versions of RAM and POR are used
- DC never attempts to synthesize memory internals

---

### 8. Design Elaboration

The top-level module (`vsdcaravel`) is elaborated.

**During elaboration:**
- Module hierarchy is constructed
- Signal connectivity is resolved
- Registers and combinational logic are inferred
- Blackbox modules are instantiated using stub definitions

This step converts RTL into an internal design database.

---

### 9. Blackbox Protection and Dont-Touch Constraints

Multiple protection layers are applied:

- Mark RAM and POR modules as **blackboxes**
- Apply **dont_touch** on:
  - Module definitions
  - All hierarchical instances
- Wildcard protection for any POR-related modules

**Result:**
- No logic optimization across blackbox boundaries
- No boundary merging or removal
- Guaranteed preservation of memory interfaces

---

### 10. Design Linking and Uniquification

- **Linking** resolves all references to library cells and modules
- **Uniquify** creates separate copies of multiply instantiated modules

This enables independent optimization while maintaining functional correctness.

---

### 11. Timing Constraint Application

If an SDC file is available, it is read into the design.

**Constraints include:**
- Clock definitions
- Input/output delays
- Timing exceptions
- Path constraints

This step ensures synthesis is timing-driven rather than purely area-driven.

---

### 12. Topographical Synthesis

The design is synthesized using **DC_TOPO** with high effort settings.

**What happens here:**
- Boolean logic optimization
- Technology mapping to SCL180 cells
- Timing and area optimization
- Hierarchy-aware synthesis

Blackbox modules remain untouched throughout this stage.

---

### 13. Output Generation

The following outputs are generated:

- **Gate-level Verilog netlist** (hierarchical)
- **DDC file** (Synopsys internal format)
- **SDC file** with mapped netlist names

These files are suitable for:
- Gate-level simulation
- Physical design tools
- Timing sign-off

---

### 14. Report Generation

The synthesis generates detailed reports:

- Area utilization
- Power consumption
- Worst-case timing paths
- Constraint violations
- Overall Quality of Results (QoR)

Additionally, a **custom blackbox report** is generated to verify:
- Presence of RAM and POR modules
- Instance count
- Hierarchical paths

---

### Issues Faced

1. **Memory inferred as flip-flops**
   - Root cause: Missing blackbox control
   - Fix: Disabled memory inference and added stub modules

2. **Unlinked module errors**
   - Root cause: Incorrect library load order
   - Fix: Proper target and link library setup

3. **Blackbox optimization**
   - Root cause: Missing dont_touch on instances
   - Fix: Applied dont_touch at both module and instance level

4. **Timing violations during early runs**
   - Root cause: Missing or incomplete SDC
   - Fix: Added proper clock and I/O constraints

---

## Command to run 
```
dc_shell -f run_dc_topo.tcl |tee dc_shell.log
```

## 📷 Evidence
![DC_TOPO Synthesis Report](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/synthesis%20dc_shell.png)
![Dc SYnthesis report](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/synthesis%20end%20paert.png)
![dc log image](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/log%20synhesis.png)

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

- Now go the gls folder and give the command 
```  
vcs -full64 -sverilog -timescale=1ns/1ps -debug_access+all +define+FUNCTIONAL+SIM+GL +notimingchecks hkspi_tb.v +incdir+../synthesis/output +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/iopad/cio250/4M1L/verilog/tsl18cio250/zero +incdir+/home/Synopsys/pdk/SCL_PDK_3/SCLPDK_V3.0_KIT/scl180/stdcell/fs120/4M1IL/verilog/vcs_sim_model -o simv

./simv

```

### 📷 Evidence

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/gls%20simv%20command%20simulation%20passed%20with%20command.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/simulation%20make%20simv%20log.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task2/Images/gtkwave%20after%20gls.png)

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

