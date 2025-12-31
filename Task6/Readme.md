# Backend Flow Bring-Up: 100 MHz Physical Design Implementation

## Overview

This repository presents a structured backend implementation flow aimed at validating digital physical design methodologies and tool interoperability at a target operating frequency of **100 MHz**. The emphasis of this work is not on achieving signoff-quality closure, but on demonstrating a **complete and correct end-to-end backend flow**, with clean and consistent handoffs between each stage of the design automation toolchain.

The project covers the full backend lifecycle, beginning with a synthesized gate-level netlist and proceeding through floorplanning, placement, clock tree synthesis (CTS), routing, parasitic extraction, and static timing analysis (STA). Industry-standard **Synopsys tools** are used throughout the flow: **IC Compiler II** for physical implementation, **Star-RC** for parasitic extraction, and **PrimeTime** for comprehensive timing analysis under realistic constraints.

The implemented design is the **Raven wrapper**, a moderately complex digital block consisting of more than **45,000 standard cells** and an embedded **32×1024 SRAM macro**. The design is synthesized using the **Nangate Open Cell Library** targeting the **FreePDK45 (45 nm)** technology node. This configuration enables realistic backend experimentation while maintaining a controlled and reproducible environment.

---

### End-to-End Flow Diagram 

The backend flow follows a strictly sequential pipeline where outputs from one stage become inputs to the subsequent stage:

```
┌──────────────────────┐
│ Synthesis Outputs    │
│ - Verilog Netlist    │
│ - Timing Libraries   │
└──────────┬───────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 1: Floorplanning (ICC2)        │
│ - Define die/core boundaries         │
│ - Place IO pads and SRAM macro       │
│ - Create blockages                   │
│ Output: Floorplan DEF                │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 2: Power Planning (ICC2)       │
│ - Define power/ground patterns       │
│ - Synthesize power grids             │
│ - Create power rings                 │
│ Output: Power-planned DEF            │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 3: Placement (ICC2)            │
│ - Place 45K+ standard cells          │
│ - Optimize placement                 │
│ Output: Placed DEF + Netlist         │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 4: Clock Tree (ICC2)           │
│ - Synthesize CTS trees               │
│ - Minimize skew                      │
│ Output: CTS Netlist                  │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 5: Routing (ICC2)              │
│ - Global and detailed routing        │
│ - Fix DRC violations                 │
│ Output: Routed DEF + Netlist         │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 6: Extraction (Star-RC)        │
│ - Calculate RC parasitics            │
│ - Generate SPEF                      │
│ Output: design.spef                  │
└──────────┬───────────────────────────┘
           │
           v
┌──────────────────────────────────────┐
│ PHASE 7: STA Analysis (PrimeTime)    │
│ - Apply timing constraints           │
│ - Analyze setup/hold                 │
│ Output: Timing Reports               │
└──────────────────────────────────────┘
```

## Design Specifications

### Design Identity

The target of this backend implementation is a large-scale hierarchical wrapper integrating multiple functional subsystems. The design is representative of a realistic SoC-style block, suitable for validating physical design flows and tool handoffs.

- **Design Name:** `raven_wrapper`
- **Total Cell Count:** 45,000+ standard cells
- **Embedded Memory:** 1 × SRAM (32 × 1024 bits), FreePDK45
- **Technology Node:** FreePDK45 (45 nm)
- **Standard Cell Library:** Nangate Open Cell Library
- **Die Size:** 3588 µm × 5188 µm
- **Core Size:** 2988 µm × 4588 µm  
  *(300 µm core offset from all die edges)*
- **Target Core Utilization:** 65%

---

### Timing and Frequency Specifications

The design is constrained to operate at a uniform nominal frequency across multiple clock domains. Each clock domain is independently defined but shares identical frequency and duty-cycle characteristics to simplify cross-domain validation.

| Clock Domain | Target Frequency | Period | Duty Cycle |
|--------------|------------------|--------|------------|
| `ext_clk`    | 100 MHz          | 10.0 ns | 50% (0–5 ns) |
| `pll_clk`    | 100 MHz          | 10.0 ns | 50% (0–5 ns) |
| `spi_sck`    | 100 MHz          | 10.0 ns | 50% (0–5 ns) |

All clocks are treated as asynchronous with respect to one another, enabling verification of clock-domain isolation and timing robustness.

To model realistic chip-level IO behavior, conservative input constraints are applied:

- **Input transition time (min):** 0.1 ns  
- **Input transition time (max):** 0.5 ns  
- **Input delay w.r.t. `ext_clk` (min):** 0.2 ns  
- **Input delay w.r.t. `ext_clk` (max):** 0.6 ns  

---

### Metal Stack Configuration

The technology files provides a ten-layer metal stack with alternating preferred routing directions. The stack is organized to support efficient signal routing and robust power delivery.

| Metal Layer | Preferred Direction | Primary Usage |
|-------------|---------------------|---------------|
| M1  | Horizontal | Standard cell power rails, local routing |
| M2  | Vertical   | Local signal routing |
| M3  | Horizontal | Macro pin access and connections |
| M4  | Vertical   | Signal routing |
| M5  | Horizontal | Signal routing |
| M6  | Vertical   | Signal routing |
| M7  | Horizontal | Signal routing |
| M8  | Vertical   | Signal routing |
| M9  | Vertical   | Power grid (vertical stripes) |
| M10 | Horizontal | Power grid (horizontal straps) |

### Design Data and Models

The physical design relies on the following design data inputs:

**Synthesized Netlist:**
- `raven_wrapper.synth.v` - Generated from Design Compiler synthesis with ~45,000 standard cell instances and 1 SRAM macro instance

**Technology Definition:**
- `nangate.tf` - Technology file defining process layers (M1-M10), site definitions, and technology rules

**Physical Libraries:**
- `nangate_stdcell.lef` - NangateOpenCellLibrary cell definitions
- `sram_32_1024_freepdk45.lef` - SRAM macro physical model

**Timing Libraries:**
- `nangate_typical.db` - Standard cell timing (TT corner)
- `sram_32_1024_freepdk45_TT_1p0V_25C_lib.db` - SRAM timing

**Parasitic Models:**
- TLU+ (Technology Library Unit Plus) files for RC extraction corner definitions

## Prerequisites

### Required Files
1. **Verilog Netlist:** `raven_wrapper.synth.v`
2. **Technology File:** `nangate.tf`
3. **LEF Files:** Standard cell and SRAM LEF files
4. **Timing Libraries:** `.db` files for standard cells and SRAM
5. **TLU+ Files:** Parasitic extraction models
6. **Constraint Files:** MCMM setup, timing constraints

### Directory Setup
```bash
# clone the repository
git clone https://github.com/kunalg123/icc2_workshop_collaterals
```

## Design Setup

> According to End-to-End Flow Diagram

### 1. Floorplaning and Io placement
TCL file used for floorplaning and IO placemnet 
```
puts "RM-info : Running script [info script]\n"

##########################################################################################
# Tool: IC Compiler II
# Script: icc2_common_setup.tcl
# Version: P-2019.03-SP4
# Copyright (C) 2014-2019 Synopsys, Inc. All rights reserved.
##########################################################################################

##########################################################################################
## Required variables
## These variables must be correctly filled in for the flow to run properly
##########################################################################################
set DESIGN_NAME 		"raven_wrapper" ;# Required; name of the design to be worked on; also used as the block name when scripts save or copy a block
set LIBRARY_SUFFIX		"Nangate" ;# Suffix for the design library name ; default is unspecified   
set DESIGN_LIBRARY 		"${DESIGN_NAME}${LIBRARY_SUFFIX}" ;# Name of the design library; default is ${DESIGN_NAME}${LIBRARY_SUFFIX}
set REFERENCE_LIBRARY 		[list /home/prakhan/task_6/icc2_workshop_collaterals/nangate_stdcell.lef /home/prakhan/task_6/icc2_workshop_collaterals/sram/sram_32_1024_freepdk45.lef]	;# Required; a list of reference libraries for the design library.
					;#	for library configuration flow (LIBRARY_CONFIGURATION_FLOW set to true below): 
					;#		- specify the list of physical source files to be used for library configuration during create_lib
				       	;# 	for hierarchical designs using bottom-up flows: include subblock design libraries in the list;
					;# 	for hierarchical designs using ETMs: include the ETM library in the list.
					;# 		- If unpack_rm_dirs.pl is used to create dir structures for hierarchical designs, 
					;#		  in order to transition between hierarchical DP and hierarchical PNR flows properly, 
					;#		  absolute paths are a requirement.
set COMPRESS_LIBS               "false" ;# Save libs as compressed NDM; only used in DP.
#set VERILOG_NETLIST_FILES      "/home/kunal/workshop/icc2_workshop_collaterals/pnrScripts/spi_slave.synth.v"
set VERILOG_NETLIST_FILES	"/home/prakhan/task_6/icc2_workshop_collaterals/raven_wrapper.synth.v"	;# Verilog netlist files;
					;# 	for DP: required
					;# 	for PNR: required if INIT_DESIGN_INPUT is ASCII in icc2_pnr_setup.tcl; not required for DC_ASCII or DP_RM_NDM
set UPF_FILE 			""	;# A UPF file
					;# 	for DP: required
					;# 	for PNR: required if INIT_DESIGN_INPUT is ASCII in icc2_pnr_setup.tcl; not required for DC_ASCII or DP_RM_NDM
                                        ;#          for hierarchical designs using ETMs, load the block upf file
                                        ;#          for each sub-block linked to ETM, include the following line in the UPF_FILE 
                			;#              load_upf block.upf -scope block_instance_name
set UPF_SUPPLEMENTAL_FILE	""      ;# The supplemental UPF file. Only needed if you are running golden UPF flow, in which case, you need both UPF_FILE and this.
					;# 	for DP: required
					;# 	for PNR: required if INIT_DESIGN_INPUT is ASCII in icc2_pnr_setup.tcl; not required for DC_ASCII or DP_RM_NDM
					;#	    If UPF_SUPPLEMENTAL_FILE is specified, scripts assume golden UPF flow. load_upf and save_upf commands will be different.	

set TCL_PARASITIC_SETUP_FILE	"./init_design.read_parasitic_tech_example.tcl"	;# Specify a Tcl script to read in your TLU+ files by using the read_parasitic_tech command;
					;# refer to the example in templates/init_design.read_parasitic_tech_example.tcl 

#set TCL_MCMM_SETUP_FILE         ""
set TCL_MCMM_SETUP_FILE		"./init_design.mcmm_example.auto_expanded.tcl"	;# Specify a Tcl script to create your corners, modes, scenarios and load respective constraints;
					;# two examples are provided in templates/: 
					;# init_design.mcmm_example.explicit.tcl: provide mode, corner, and scenario constraints; create modes, corners, 
					;# and scenarios; source mode, corner, and scenario constraints, respectively 
					;# init_design.mcmm_example.auto_expanded.tcl: provide constraints for the scenarios; create modes, corners, 
					;# and scenarios; source scenario constraints which are then expanded to associated modes and corners
					;# 	for DP: required
					;# 	for PNR: required if INIT_DESIGN_INPUT is ASCII in icc2_pnr_setup.tcl; not required for DC_ASCII or DP_RM_NDM

set TECH_FILE 			"/home/prakhan/task_6/icc2_workshop_collaterals/nangate.tf" 	;# A technology file; TECH_FILE and TECH_LIB are mutually exclusive ways to specify technology information; 
					;# TECH_FILE is recommended, although TECH_LIB is also supported in ICC2 RM. 
set TECH_LIB			""	;# Specify the reference library to be used as a dedicated technology library;
                        		;# as a best practice, please list it as the first library in the REFERENCE_LIBRARY list 
set TECH_LIB_INCLUDES_TECH_SETUP_INFO true 
					;# Indicate whether TECH_LIB contains technology setup information such as routing layer direction, offset, 
					;# site default, and site symmetry, etc. TECH_LIB may contain this information if loaded during library prep.
					;# true|false; this variable is associated with TECH_LIB. 
set TCL_TECH_SETUP_FILE		"./init_design.tech_setup.tcl"
					;# Specify a TCL script for setting routing layer direction, offset, site default, and site symmetry list, etc.
					;# init_design.tech_setup.tcl is the default. Use it as a template or provide your own script.
					;# This script will only get sourced if the following conditions are met: 
					;# (1) TECH_FILE is specified (2) TECH_LIB is specified && TECH_LIB_INCLUDES_TECH_SETUP_INFO is false 
set ROUTING_LAYER_DIRECTION_OFFSET_LIST "{metal1 horizontal} {metal2 vertical} {metal3 horizontal} {metal4 vertical} {metal5 horizontal} {metal6 vertical} {metal7 horizontal} {metal8 vertical} {metal9 horizontal} {metal10 vertical}" 
					;# Specify the routing layers as well as their direction and offset in a list of space delimited pairs;
					;# This variable should be defined for all metal routing layers in technology file;
					;# Syntax is "{metal_layer_1 direction offset} {metal_layer_2 direction offset} ...";
					;# It is required to at least specify metal layers and directions. Offsets are optional. 
					;# Example1 is with offsets specified: "{M1 vertical 0.2} {M2 horizontal 0.0} {M3 vertical 0.2}"
					;# Example2 is without offsets specified: "{M1 vertical} {M2 horizontal} {M3 vertical}"
##########################################################################################
## Optional variables
## Specify these variables if the corresponding functions are desired 
##########################################################################################
set DESIGN_LIBRARY_SCALE_FACTOR	""	;# Specify the length precision for the library. Length precision for the design
					;# library and its ref libraries must be identical. Tool default is 10000, which
					;# implies one unit is one Angstrom or 0.1nm.

set UPF_UPDATE_SUPPLY_SET_FILE	""	;# A UPF file to resolve UPF supply sets

#set DEF_FLOORPLAN_FILES		"/home/kunal/design/scripts/pnr/ICC2-RM_P-2019.03-SP4/write_data_dir/picorv32/picorv32.icc.floorplan/floorplan.def.gz"	;# DEF files which contain the floorplan information;
set DEF_FLOORPLAN_FILES                ""  ;# DEF files which contain the floorplan information;
					;# 	for DP: not required
					;# 	for PNR: required if INIT_DESIGN_INPUT = ASCII in icc2_pnr_setup.tcl and neither TCL_FLOORPLAN_FILE or 
					;#		 initialize_floorplan is used; DEF_FLOORPLAN_FILES and TCL_FLOORPLAN_FILE are mutually exclusive;
					;# 	         not required if INIT_DESIGN_INPUT = DC_ASCII or DP_RM_NDM

set DEF_SCAN_FILE		""	;# A scan DEF file for scan chain information;
					;# 	for PNR: not required if INIT_DESIGN_INPUT = DC_ASCII or DP_RM_NDM, as SCANDEF is expected to be loaded already

set DEF_SITE_NAME_PAIRS		{}	;# A list of site name pairs for read_def -convert; 
					;# specify site name pairs with from_site first followed by to_site;
					;# Example: set DEF_SITE_NAME_PAIRS {{from_site_1 to_site_1} {from_site_2 to_site_2}} 	
set SITE_DEFAULT		""	;# Specify the default site name if there are multiple site defs in the technology file;
					;# this is to be used by initialize_floorplan command; example: set SITE_DEFAULT "unit";
					;# this is applied in the init_design.tech_setup.tcl script 
set SITE_SYMMETRY_LIST	""		;# Specify a list of site def and its symmetry value;
					;# this is to be used by read_def or initialize_floorplan command to control the site symmetry;
					;# example: set SITE_SYMMETRY_LIST "{unit Y} {unit1 Y}"; this is applied in the init_design.tech_setup.tcl script 

set MIN_ROUTING_LAYER		"metal1"	;# Min routing layer name; normally should be specified; otherwise tool can use all metal layers
set MAX_ROUTING_LAYER		"metal10"	;# Max routing layer name; normally should be specified; otherwise tool can use all metal layers

set LIBRARY_CONFIGURATION_FLOW	false	;# Set it to true enables library configuration flow which calls the library manager under the hood to generate .nlibs, 
					;# save them to disk, and automatically link them to the design.
					;# Requires LINK_LIBRARY to be specified with .db files and REFERENCE_LIBRARY to be specified with physical
					;# source files for the library configuration flow. Also search_path (in icc2_pnr_setup.tcl) should include paths 
					;# to these .db and physical source files.

set LINK_LIBRARY		[list /home/prakhan/task_6/icc2_workshop_collaterals/nangate_typical.db /home/prakhan/task_6/icc2_workshop_collaterals/sram_32_1024_freepdk45_TT_1p0V_25C_lib.db]	;# Specify .db files;
					;# 	for running VC-LP (vc_lp.tcl) and Formality (fm.tcl): required
					;# 	for ICC-II without LIBRARY_CONFIGURATION_FLOW enabled: not required
					;#	for ICC-II with LIBRARY_CONFIGURATION_FLOW enabled: required; 
					;#      	- the .db files specified will be used for the library configuration under the hood during create_lib 

##########################################################################################
## Variables related to flow controls of flat PNR, hierarchical PNR and transition with DP
##########################################################################################
set DESIGN_STYLE		"hier"	;# Specify the design style; flat|hier; default is flat; 
					;# specify flat for a totally flat flow (flat PNR for short) and 
					;# specify hier for a hierarchical flow (hier PNR for short);
					;# 	for hier PNR: required and auto set if unpack_rm_dirs.pl is used; (see README.unpack_rm_dirs.txt for details)
					;# 	for flat PNR: this should set to flat (default)
					;#	for DP: not used 

set PHYSICAL_HIERARCHY_LEVEL	"" 	;# Specify the current level of hierarchy for the hierarchical PNR flow; top|intermediate|bottom;
					;# 	for hier PNR: required and auto set if unpack_rm_dirs.pl is used; (see README.unpack_rm_dirs.txt for details)
					;# 	for flat PNR and for DP: not used.
set RELEASE_DIR_DP		"write_data_dir_hier" 	;# Specify the release directory of DP RM; 
					;# this is where init_design.tcl of PNR flow gets DP RM released libraries;
					;# 	for hier PNR: required and auto set if unpack_rm_dirs.pl is used; (see README.unpack_rm_dirs.txt for details)
					;# 	for flat PNR: required if INIT_DESIGN_INPUT = DP_RM_NDM, as init_design.tcl needs to know where DP RM libraries are
					;#	for DP: not used 
set RELEASE_LABEL_NAME_DP 	"rave_wrapperNangate"	
					;# Specify the label name of the block in the DP RM released library;
					;# this is the label name which init_design.tcl of PNR flow will open. 
set RELEASE_DIR_PNR		"" 	;# Specify the release directory of PNR RM; 
					;# this is where the init_design.tcl of hierarchical PNR flow gets the sub-block libraries;	
					;# 	for hier PNR: required and auto set if unpack_rm_dirs.pl is used; (see README.unpack_rm_dirs.txt for details)
					;# 	for flat PNR and for DP: not used.
##########################################################################################
## Variables related to REDHAWK ANALYSIS FUSION
##########################################################################################
set REDHAWK_SEARCH_PATH		"" 	;# Required. Search path to the NDM, reference libraries, and etc.

puts "RM-info : Completed script [info script]\n"

```

#### Command to run

```
icc2_shell -f floorplan_io.tcl| tee floorplan_io.log

gui_start
```

#### Input Files
these files should be present in folder while running tcl file

- `raven_wrapper.synth.v` - Synthesized netlist with hierarchy flattened
- `nangate.tf` - Technology definitions including site and layer information
- `nangate_stdcell.lef` - Cell dimensions and pin locations
- `sram_32_1024_freepdk45.lef` - SRAM macro boundaries and ports
- `nangate_typical.db` - Timing information for constraints

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-30%2023-54-15.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-30%2023-54-28.png)

###  2: Power Planning

#### Objectives

Power planning defines the physical power delivery infrastructure for the design. The goal of this phase is to provide reliable VDD and VSS distribution to all standard cells and macros while controlling voltage drop, electromigration risk, and overall power integrity.

**Key Objectives:**
1. **Robust Current Delivery:** Provide low-resistance paths from power entry points to all logic and memory elements  
2. **Voltage Integrity:** Limit IR drop across the core (target < 5% of nominal supply)  
3. **Core Encapsulation:** Create continuous VDD/VSS rings around the core region  
4. **Scalable Distribution:** Implement power stripes on upper metal layers for uniform coverage  
5. **Vertical Connectivity:** Ensure sufficient via insertion between power layers and standard cell rails  

---

#### Power Planning Strategy

##### Power Grid Architecture

The power distribution network is implemented as a hierarchical, multi-layer grid:

- **M1:** Local power rails embedded within standard cell rows  
- **M2:** Vertical tap connections between standard cell rails and upper layers  
- **M9–M10:** Top-level global power mesh providing primary current delivery  

This layered approach balances routing efficiency with current capacity and minimizes localized voltage drop.

---

##### Power Ring Structure

A dedicated power ring surrounds the core region to provide a stable interface between external power sources and the internal grid.

- **Ring Signals:** VDD and VSS  
- **Ring Width:** 4.0–6.0 µm per rail  
- **Ring Offset:** 10 µm inside the core boundary  
- **Metal Layer:** Selected lower metal layer suitable for high-current conduction  

The power ring acts as the entry point for the internal power grid and ensures uniform current injection around the core perimeter.

---

##### Power Stripe Configuration

To distribute power uniformly across the core area, stripes are deployed on higher metal layers:

- **M9:** Vertical power stripes (alternating VDD and VSS)  
- **M10:** Horizontal power stripes (alternating VDD and VSS)  
- **Stripe Pitch:** ~50 µm  
- **Stripe Width:** 2.0 µm  

The orthogonal M9/M10 stripe arrangement forms a mesh structure that reduces resistance, improves redundancy, and enhances overall power integrity.

---

##### Via Strategy

Vertical connectivity between power layers is ensured through systematic via insertion:

- **M1–M2:** Standard cell rail connections  
- **M2–M3:** Local vertical power taps  
- **M8–M9:** Interface between signal layers and global power mesh  
- **M9–M10:** Inter-layer power mesh connectivity  
- **Via Spacing:** Typically 5–10 µm  

Adequate via density is maintained to prevent current bottlenecks and reduce via-level electromigration risk.

---

#### Power Planning Implementation

Power planning is implemented using scripted commands to ensure repeatability and consistency across runs.

- **Script File:** `power_planning.tcl`
- **Key Tasks:**
  - Core power ring generation  
  - M9/M10 stripe insertion with defined pitch and width  
  - Automated via insertion across power layers  
  - Power grid integrity checks  

This scripted approach enables rapid iteration, easy parameter tuning, and clean handoff to subsequent placement and routing phases.


```tcl
################################################################################
# POWER PLAN TCL – CONSOLIDATED & CLEAN
# Compatible with Synopsys ICC2
################################################################################

puts "RM-info : Starting Power Planning Flow"

remove_pg_strategies -all
remove_pg_patterns -all
remove_pg_regions -all
remove_pg_via_master_rules -all
remove_pg_strategy_via_rules -all
remove_routes -net_types {power ground} -ring -stripe -macro_pin_connect -lib_cell_pin_connect
########################################
# 0. Global PG Nets
########################################
set PG_NETS {VDD VSS}

########################################
# 1. Connect PG nets automatically
########################################
puts "RM-info : Connecting PG nets automatically"
connect_pg_net -automatic -all_blocks

########################################
# 2. CORE POWER RING
########################################
puts "RM-info : Creating Core PG Ring"

create_pg_ring_pattern ring_pattern -horizontal_layer metal10 \
    -horizontal_width {5} -horizontal_spacing {2} \
    -vertical_layer metal9 -vertical_width {5} \
    -vertical_spacing {2} -corner_bridge false
set_pg_strategy core_ring -core -pattern \
    {{pattern: ring_pattern}{nets: {VDD VSS}}{offset: {3 3}}} \
    -extension {{stop: innermost_ring}}

########################################
# 3. MACRO POWER RINGS
########################################
puts "RM-info : Creating Macro PG Rings"

create_pg_ring_pattern macro_ring_pattern -horizontal_layer metal10 \
    -horizontal_width {5} -horizontal_spacing {2} \
    -vertical_layer metal9 -vertical_width {5} \
    -vertical_spacing {2} -corner_bridge false
set_pg_strategy macro_core_ring -macros [get_cells -hierarchical -filter "is_hard_macro==true"] -pattern \
    {{pattern: macro_ring_pattern}{nets: {VDD VSS}}{offset: {10 10}}} 

########################################
# 4. PG MESH (CORE ONLY)
########################################
puts "RM-info : Creating PG Mesh"

create_pg_region pg_mesh_region -core -expand -2 -exclude_macros sram -macro_offset 20
create_pg_mesh_pattern pg_mesh1 \
   -parameters {w1 p1 w2 p2 f t} \
   -layers {{{vertical_layer: metal9} {width: @w1} {spacing: interleaving} \
        {pitch: @p1} {offset: @f} {trim: @t}} \
 	     {{horizontal_layer: metal10} {width: @w2} {spacing: interleaving} \
        {pitch: @p2} {offset: @f} {trim: @t}}}


set_pg_strategy s_mesh1 \
   -pattern {{pattern: pg_mesh1} {nets: {VDD VSS VSS VDD} } \
{offset_start: 10 20} {parameters: 4 80 6 120 3.344 false}} \
   -pg_region pg_mesh_region -extension {{stop: innermost_ring}} 

########################################
# 5. MACRO PG PIN CONNECTIONS
########################################
puts "RM-info : Connecting Macro PG Pins"

create_pg_macro_conn_pattern hm_pattern -pin_conn_type scattered_pin -layer {metal3 metal3}
set toplevel_hms [filter_collection [get_cells * -physical_context] "is_hard_macro == true"]
set_pg_strategy macro_con -macros $toplevel_hms -pattern {{name: hm_pattern} {nets: {VDD VSS}} }

########################################
# 6. STANDARD CELL RAILS
########################################
puts "RM-info : Creating Standard Cell PG Rails"

create_pg_std_cell_conn_pattern \
    std_cell_rail  \
    -layers {metal1} \
    -rail_width 0.06

set_pg_strategy rail_strat  -pg_region pg_mesh_region \
    -pattern {{name: std_cell_rail} {nets: VDD VSS} }

########################################
# 7. Compile PG
########################################
puts "RM-info : Compiling PG strategies"

compile_pg 

########################################
# 8. PG CHECKS
########################################
puts "RM-info : Running PG Checks"

check_pg_missing_vias
check_pg_drc -ignore_std_cells
check_pg_connectivity -check_std_cell_pins none

########################################
# 9. Save Block
########################################
puts "RM-info : Saving block after power planning"
save_block -hier -force -label CREATE_POWER
save_lib -all

puts "RM-info : Power Planning Completed Successfully"

estimate_timing
redirect -file $REPORTS_DIR_TIMING_ESTIMATION/${DESIGN_NAME}.post_estimated_timing.rpt     {report_timing -corner estimated_corner -mode [all_modes]}
redirect -file $REPORTS_DIR_TIMING_ESTIMATION/${DESIGN_NAME}.post_estimated_timing.qor     {report_qor    -corner estimated_corner}
redirect -file $REPORTS_DIR_TIMING_ESTIMATION/${DESIGN_NAME}.post_estimated_timing.qor.sum {report_qor    -summary}

save_block -hier -force   -label ${TIMING_ESTIMATION_LABEL_NAME}
save_lib -all


set path_dir [file normalize ${WORK_DIR_WRITE_DATA}]
set write_block_data_script ./write_block_data.tcl
source ${write_block_data_script}

```

### Power Planning TCL Script – Explanation

This section explains the intent and functionality of the **power planning TCL flow** used in Synopsys IC Compiler II. The explanation is organized by logical phases of the script rather than listing commands line-by-line, to clearly describe *what is being done* and *why it is required*.

---

#### 1. Flow Initialization and Cleanup

The script begins by resetting the power planning environment. All previously defined power-grid strategies, patterns, regions, via rules, and any existing routed power structures are removed. This guarantees that the flow is **fully repeatable** and prevents legacy power objects from interfering with the new implementation.

---

#### 2. Global Power Net Definition

Global power and ground nets are defined for the design. These nets act as the backbone of the power distribution network and are referenced consistently throughout the power planning process to avoid ambiguity in connectivity.

---

#### 3. Automatic Power Connectivity

Before constructing explicit power structures, the script automatically connects the global power and ground nets to all hierarchical blocks in the design. This ensures that every macro and standard cell instance is logically associated with the correct power nets prior to physical grid generation.

---

#### 4. Core Power Ring Creation

A dedicated **core power ring** is constructed around the core boundary using higher metal layers. The ring serves as the primary interface between the external power sources and the internal power grid. Proper ring width, spacing, and offset are chosen to support high current delivery while maintaining routing safety margins inside the core.

---

#### 5. Macro Power Rings

Each hard macro in the design is surrounded by its own local power ring. This strategy isolates macro power delivery from the standard cell logic and significantly reduces localized IR drop around high-current macro blocks such as SRAMs. A larger offset is applied to prevent overlap with macro keep-out regions.

---

#### 6. Core Power Mesh Formation

A structured **power mesh** is generated across the core region using orthogonal metal layers. Vertical and horizontal stripes alternate between VDD and VSS, forming a uniform grid that distributes current evenly across the design.

Macros are explicitly excluded from the mesh region, with a defined clearance margin to avoid routing conflicts. Parameterized stripe widths, pitches, and offsets allow flexible tuning of grid density and power capacity.

---

#### 7. Macro Power Pin Connectivity

Macro power pins are connected to the global power grid using a scattered pin connection approach on an intermediate metal layer. This provides reliable vertical power access for macros and ensures proper integration with the surrounding mesh and rings.

---

#### 8. Standard Cell Power Rails

Standard cell power rails are generated along the cell rows on the lowest routing layer. These rails directly supply VDD and VSS to the logic cells and connect upward through vias to the global power mesh, completing the hierarchical power delivery path.

---

#### 9. Power Grid Compilation

All defined power strategies—rings, meshes, rails, and via connections—are compiled into actual routed geometry. At this stage, the tool generates the physical metal shapes and vias that form the complete power distribution network.

---

#### 10. Power Grid Verification

Post-implementation checks are performed to validate the power grid. These include verification for missing vias, design rule compliance, and overall connectivity integrity. The checks ensure that the grid is electrically continuous and physically legal before proceeding to placement and routing.

---

#### 11. Design Save and Timing Estimation

The design is saved at a dedicated checkpoint after power planning. A preliminary timing estimation using estimated parasitics is then executed to confirm that the power grid does not introduce major timing degradation. Timing and quality-of-results reports are generated for reference.

---

#### 12. Final Save and Data Export

Finally, the design state is saved again with timing information, and auxiliary block data is written out for downstream implementation steps. This marks the successful completion of the power planning phase.

---

#### Summary

This power planning flow establishes a **hierarchical, multi-layer power distribution network** consisting of:

- Core-level power rings  
- Macro-level isolation rings  
- A global M9/M10 power mesh  
- Standard cell M1 power rails  
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-30%2021-41-50.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-30%2021-47-02.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-30%2021-47-15.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-30%2021-47-49.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/WhatsApp%20Image%202025-12-31%20at%2012.42.14%20PM.jpeg)


> Here i have one floating I/O pads.

### 3. Place, route, cts

```tcl
<details>
  <summary>place_cts_route</summary>
####################################
# Place, CTS, Route
####################################
eval create_placement $CMD_OPTIONS
report_placement    -physical_hierarchy_violations all    -wirelength all -hard_macro_overlap    -verbose high > $REPORTS_DIR_PLACEMENT/report_placement.rpt
set_host_options -max_cores 8
remove_corners [get_corners estimated_corner]
set_app_options -name place.coarse.continue_on_missing_scandef -value true
place_opt
clock_opt
route_auto -max_detail_route_iterations 5
set FILLER_CELLS [get_object_name [sort_collection -descending [get_lib_cells NangateOpenCellLibrary/FILL*] area]]
create_stdcell_fillers -lib_cells $FILLER_CELLS

save_block -hier -force   -label post_route
save_lib -all
```

![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-38-27.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-38-50.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-40-02.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-40-29.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-41-52.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/Screenshot%20from%202025-12-31%2002-44-29.png)

> I will try to resolve that floating I/O pad issue and drc errors Due to exams i completed till here 



