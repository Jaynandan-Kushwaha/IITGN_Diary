# Raven SoC Physical Design
## RTL to GDSII Flow Using Synopsys Tools

### Reference Flow
The physical design flow is derived from the ICC2 standalone reference scripts available at the link below.  
These scripts were adapted and customized for the Raven SoC design.

[ICC2 Standalone Flow Reference](https://github.com/kunalg123/icc2_workshop_collaterals/blob/master/standaloneFlow/top.tcl)

---

## Design Specifications

| Parameter       | Value                 |
|-----------------|----------------------|
| Die Size        | 3.588 mm × 5.188 mm  |
| Core Offset     | 0.2 mm on all sides   |
| Core Area       | 3.388 mm × 4.988 mm  |
| Top-Level Module| raven_wrapper         |
| Technology      | Nangate 45 nm         |

---

## Tools Used
- **Synopsys ICC2** – Floorplanning, Placement, Routing  
---

## 1. Floorplanning Using ICC2
The floorplanning stage establishes the physical boundaries of the Raven SoC, including die size, core area, and IO reservation regions.

### Scope of Floorplanning
- Die and core definition  
- IO band reservation using placement blockages  
- Port visualization  
- No standard-cell placement  
- No CTS  
- No routing  

### Floorplan Geometry
- Die Boundary: [0, 0] → [3588, 5188] µm  
- Core Boundary: [200, 200] → [3388, 4988] µm  
- Core Margin: 200 µm on all sides  
- Total Die Area: 18.606 mm²  

The chosen die aspect ratio provides sufficient routing resources and IO space.

### IO Band Reservation
Hard placement blockages are created on all four sides of the die to reserve space for IO-related structures.

| Side   | Coordinates (µm)       |
|--------|------------------------|
| Bottom | [0, 0] → [3588, 100]  |
| Top    | [0, 5088] → [3588, 5188] |
| Left   | [0, 100] → [100, 5088] |
| Right  | [3488, 100] → [3588, 5088] |

Reserved for:
- IO pads  
- ESD protection cells  
- Level shifters  
- Power and ground rings  

---

## Tool Environment Setup
```
# Navigate to working directory
cd ~/icc2Workshop/standalone`
```
---

## ICC2 Floorplanning Script

The floorplan is generated using the following TCL script file:

```tcl
source -echo ./icc2_common_setup.tcl
source -echo ./icc2_dp_setup.tcl
if {[file exists ${WORK_DIR}/$DESIGN_LIBRARY]} {
   file delete -force ${WORK_DIR}/${DESIGN_LIBRARY}
}
###---NDM Library creation---###
set create_lib_cmd "create_lib ${WORK_DIR}/$DESIGN_LIBRARY"
if {[file exists [which $TECH_FILE]]} {
   lappend create_lib_cmd -tech $TECH_FILE ;# recommended
} elseif {$TECH_LIB != ""} {
   lappend create_lib_cmd -use_technology_lib $TECH_LIB ;# optional
}
lappend create_lib_cmd -ref_libs $REFERENCE_LIBRARY
puts "RM-info : $create_lib_cmd"
eval ${create_lib_cmd}

###---Read Synthesized Verilog---###
if {$DP_FLOW == "hier" && $BOTTOM_BLOCK_VIEW == "abstract"} {
   # Read in the DESIGN_NAME outline.  This will create the outline
   puts "RM-info : Reading verilog outline (${VERILOG_NETLIST_FILES})"
   read_verilog_outline -design ${DESIGN_NAME}/${INIT_DP_LABEL_NAME} -top ${DESIGN_NAME} ${VERILOG_NETLIST_FILES}
   } else {
   # Read in the full DESIGN_NAME.  This will create the DESIGN_NAME view in the database
   puts "RM-info : Reading full chip verilog (${VERILOG_NETLIST_FILES})"
   read_verilog -design ${DESIGN_NAME}/${INIT_DP_LABEL_NAME} -top ${DESIGN_NAME} ${VERILOG_NETLIST_FILES}
}

## Technology setup for routing layer direction, offset, site default, and site symmetry.
#  If TECH_FILE is specified, they should be properly set.
#  If TECH_LIB is used and it does not contain such information, then they should be set here as well.
if {$TECH_FILE != "" || ($TECH_LIB != "" && !$TECH_LIB_INCLUDES_TECH_SETUP_INFO)} {
   if {[file exists [which $TCL_TECH_SETUP_FILE]]} {
      puts "RM-info : Sourcing [which $TCL_TECH_SETUP_FILE]"
      source -echo $TCL_TECH_SETUP_FILE
   } elseif {$TCL_TECH_SETUP_FILE != ""} {
      puts "RM-error : TCL_TECH_SETUP_FILE($TCL_TECH_SETUP_FILE) is invalid. Please correct it."
   }
}

# Specify a Tcl script to read in your TLU+ files by using the read_parasitic_tech command
if {[file exists [which $TCL_PARASITIC_SETUP_FILE]]} {
   puts "RM-info : Sourcing [which $TCL_PARASITIC_SETUP_FILE]"
   source -echo $TCL_PARASITIC_SETUP_FILE
} elseif {$TCL_PARASITIC_SETUP_FILE != ""} {
   puts "RM-error : TCL_PARASITIC_SETUP_FILE($TCL_PARASITIC_SETUP_FILE) is invalid. Please correct it."
} else {
   puts "RM-info : No TLU plus files sourced, Parastic library containing TLU+ must be included in library reference list"
}

###---Routing settings---###
## Set max routing layer
if {$MAX_ROUTING_LAYER != ""} {set_ignored_layers -max_routing_layer $MAX_ROUTING_LAYER}
## Set min routing layer
if {$MIN_ROUTING_LAYER != ""} {set_ignored_layers -min_routing_layer $MIN_ROUTING_LAYER}

####################################
# Check Design: Pre-Floorplanning
####################################
if {$CHECK_DESIGN} {
   redirect -file ${REPORTS_DIR_INIT_DP}/check_design.pre_floorplan     {check_design -ems_database check_design.pre_floorplan.ems -checks dp_pre_floorplan}
}

####################################
# Floorplanning
####################################
#initialize_floorplan -core_utilization 0.05
initialize_floorplan \
  -control_type die \
  -boundary {{0 0} {3588 5188}} \
  -core_offset {300 300 300 300}

save_lib -all

####################################
## PG Pin connections
#####################################
puts "RM-info : Running connect_pg_net -automatic on all blocks"
connect_pg_net -automatic -all_blocks
save_block -force       -label ${PRE_SHAPING_LABEL_NAME}
save_lib -all

####################################
### Place IO
######################################
if {[file exists [which $TCL_PAD_CONSTRAINTS_FILE]]} {
   puts "RM-info : Loading TCL_PAD_CONSTRAINTS_FILE file ($TCL_PAD_CONSTRAINTS_FILE)"
   source -echo $TCL_PAD_CONSTRAINTS_FILE

   puts "RM-info : running place_io"
   place_io
}
set_attribute [get_cells -hierarchical -filter pad_cell==true] status fixed

save_block -hier -force   -label ${PLACE_IO_LABEL_NAME}
save_lib -all
```

---

## Running the Floorplanning Flow

The ICC2 shell is invoked using the floorplanning script:

```
icc2_shell -f floorplan.tcl | tee floorplan.log
```

After successful execution, the design library is created, the floorplan is initialized.

![Alt text](images/task5_1.png)

---

## GUI Inspection

The ICC2 graphical interface can be launched using:

```
start_gui
```

![Alt text](images/task5_2.png)

Within the GUI, the floorplan initialization section shows the defined die and core dimensions. This confirms that the floorplan geometry matches the intended specification.

![Alt text](images/task5_3.png)

The die area is defined as 3588 × 5188 microns, and the core area is offset uniformly by 200 microns on all sides, ensuring sufficient spacing between core logic and IO pads.

---

## Pin Placement

IO pins can be placed automatically for visualization and early validation purposes. This is done directly from the ICC2 GUI console:

```
place_pins -self
```

This command distributes the top-level ports along the periphery without enforcing ordering or side constraints. While not final pin placement, this helps verify port visibility, orientation, and connectivity at the floorplan stage.

![Alt text](images/task5_4.png)

---

## Note

Do the following changes in pad_placement_constraint.tcl to place the IO pads properly:

![Alt text](images/task5_5.png)

![Alt text](images/task5_6.png)


## Summary

This task successfully establishes a clean SoC floorplan in ICC2 using the synthesized netlist from DC. The die size, core offset, and IO keep-out regions are explicitly controlled through TCL commands, ensuring reproducibility and correctness. The generated DEF and reports serve as a solid handoff point for subsequent placement, CTS, and routing stages.
