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

another tcl file for placing sram 
```
################################################################################

# SYNOPSYS ICC2 FLOORPLAN SCRIPT

################################################################################



################################################################################

# COMMON SETUP

################################################################################

source -echo ./icc2_common_setup.tcl

source -echo ./icc2_dp_setup.tcl





################################################################################

# OPEN / CREATE LIBRARY

################################################################################

if {![file exists ${WORK_DIR}/${DESIGN_LIBRARY}]} {

   puts "RM-info : Creating library $DESIGN_LIBRARY"

   create_lib ${WORK_DIR}/${DESIGN_LIBRARY} \

      -ref_libs $REFERENCE_LIBRARY \

      -tech $TECH_FILE

} else {

   puts "RM-info : Opening existing library $DESIGN_LIBRARY"

}



open_lib ${WORK_DIR}/${DESIGN_LIBRARY}





################################################################################

# READ NETLIST

################################################################################

puts "RM-info : Reading netlist"



read_verilog \
   -design raven_wrapper/init_dp \
   -top raven_wrapper \
   /home/b_jaynandan/Downloads/icc2_workshop_collaterals-master/raven_wrapper.synth.v






################################################################################

# TECH + TLU+

################################################################################

if {[file exists [which $TCL_TECH_SETUP_FILE]]} {

   source -echo $TCL_TECH_SETUP_FILE

}



if {[file exists [which $TCL_PARASITIC_SETUP_FILE]]} {

   source -echo $TCL_PARASITIC_SETUP_FILE

}





################################################################################

# FLOORPLAN

################################################################################

puts "RM-info : Initializing floorplan"



initialize_floorplan \
   -control_type die \
   -boundary {{0 0} {3588 5188}} \
   -core_offset {300 300 300 300}




save_block -force -label floorplan





################################################################################

# POWER NET CONNECTION (EARLY)

################################################################################

connect_pg_net -automatic -all_blocks

save_block -force -label pre_shape





################################################################################

# IO PAD PLACEMENT

################################################################################

if {[file exists [which $TCL_PAD_CONSTRAINTS_FILE]]} {

   source -echo $TCL_PAD_CONSTRAINTS_FILE

   place_io

}



# Fix IO locations

set_attribute \
   -objects [get_cells -hier -filter "pad_cell==true"] \
   -name status \
   -value fixed







################################################################################

# PAD KEEP-OUTS (HARD)

################################################################################

puts "RM-info : Creating hard keepout around IO pads"



create_keepout_margin \
   -type hard \
   -outer {8 8 8 8} \
   [get_cells -hier -filter "pad_cell==true"]





################################################################################

# HARD PLACEMENT BLOCKAGES AROUND CORE EDGE

################################################################################

puts "RM-info : Creating hard placement blockages around core boundary"



# Core boundary = {{300 300} {3288 4888}}

# Creating 20um hard blockage band inside core edge



create_placement_blockage -type hard \
   -boundary {{300 300} {3288 320}} \
   -name core_hard_blockage_bottom



create_placement_blockage -type hard \
   -boundary {{300 4868} {3288 4888}} \
   -name core_hard_blockage_top



create_placement_blockage -type hard \
   -boundary {{300 320} {320 4868}} \
   -name core_hard_blockage_left



create_placement_blockage -type hard \
   -boundary {{3268 320} {3288 4868}} \
   -name core_hard_blockage_right





################################################################################

# SRAM MACRO PLACEMENT

################################################################################

puts "RM-info : Placing SRAM macro"



set sram [get_cells -quiet sram]



if {[sizeof_collection $sram] > 0} {

   set_attribute $sram origin {365.4500 4544.9250}
   set_attribute $sram orientation MXR90
   set_attribute $sram status placed

}





################################################################################

# MACRO HALOS WITH ASYMMETRIC SPACING

################################################################################

set macros [get_cells -hier -filter "is_hard_macro==true"]



if {[sizeof_collection $macros] > 0} {



   puts "RM-info : Creating asymmetric halos around macros"



   # Create minimum halo (2um) on top, bottom, right

   # No halo on left side (will be blocked separately)

   create_keepout_margin \
      -type hard \
      -outer {0 2 2 2} \
      $macros

}





################################################################################

# HARD BLOCKAGE ON LEFT SIDE OF MACRO TO CORE EDGE

################################################################################

puts "RM-info : Creating hard blockage from macro left side to core edge"



if {[sizeof_collection $sram] > 0} {
   # Create hard blockage with specified coordinates
   create_placement_blockage -type hard \
      -boundary {{320.0000 4522.9250} {594.5300 4802.9150}} \
      -name macro_left_side_blockage
   puts "RM-info : Hard blockage created from (320.0000, 4522.9250) to (594.5300, 4802.9150)"

}





################################################################################

# MCMM CONSTRAINTS

################################################################################

if {[file exists $TCL_MCMM_SETUP_FILE]} {

   source -echo $TCL_MCMM_SETUP_FILE

}





################################################################################

# PLACEMENT CONFIG

################################################################################

set plan.place.auto_generate_blockages true

set_app_options -name place_opt.flow.do_spg -value true

set_app_options -name route.global.timing_driven -value true





################################################################################

# GLOBAL DENSITY CONTROL

################################################################################

set_attribute [current_design] place_global_density 0.65





################################################################################

# FIX MACROS

################################################################################

if {[sizeof_collection $macros] > 0} {

   set_attribute $macros status fixed

}





################################################################################

# PIN PLACEMENT

################################################################################

if {[file exists [which $TCL_PIN_CONSTRAINT_FILE]] && !$PLACEMENT_PIN_CONSTRAINT_AWARE} {

   source -echo $TCL_PIN_CONSTRAINT_FILE

}



set_app_options -as_user_default -list {route.global.timing_driven true}



if {$CHECK_DESIGN} {

   redirect -file ${REPORTS_DIR_PLACE_PINS}/check_design.pre_pin_placement {check_design -ems_database check_design.pre_pin_placement.ems -checks dp_pre_pin_placement}

}



if {$PLACE_PINS_SELF} {

   place_pins -self

}



if {$PLACE_PINS_SELF} {

   # Write top-level port constraint file based on actual port locations

   write_pin_constraints -self \

      -file_name $OUTPUTS_DIR/preferred_port_locations.tcl \

      -physical_pin_constraint {side | offset | layer} \

      -from_existing_pins



   # Verify Top-level Port Placement Results

   check_pin_placement -self -pre_route true -pin_spacing true -sides true -layers true -stacking true



   # Generate Top-level Port Placement Report

   report_pin_placement -self > $REPORTS_DIR_PLACE_PINS/report_port_placement.rpt

}



save_block -hier -force -label ${PLACE_PINS_LABEL_NAME}

save_lib -all





################################################################################

# SAVE SNAPSHOT

################################################################################

save_block -hier -force -label placement_ready

save_lib -all



puts "\n===== FLOORPLAN COMPLETED SUCCESSFULLY =====\n"
```

---

## Running the Floorplanning Flow

The ICC2 shell is invoked using the floorplanning script:

```
icc2_shell -f floorplan.tcl | tee floorplan.log
```

After successful execution, the design library is created, the floorplan is initialized.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-23%2017-29-40.png)

---

## GUI Inspection

The ICC2 graphical interface can be launched using:

```
start_gui
```

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-23%2018-05-09.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-30%2021-09-30.png)

Within the GUI, the floorplan initialization section shows the defined die and core dimensions. This confirms that the floorplan geometry matches the intended specification.

The die area is defined as 3588 × 5188 microns, and the core area is offset uniformly by 200 microns on all sides, ensuring sufficient spacing between core logic and IO pads.

---

## Pin Placement

IO pins can be placed automatically for visualization and early validation purposes. This is done directly from the ICC2 GUI console:

```
place_pins -self
```

This command distributes the top-level ports along the periphery without enforcing ordering or side constraints. While not final pin placement, this helps verify port visibility, orientation, and connectivity at the floorplan stage.

![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-23%2018-23-25.png)
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task5%20Floorplan/Images/Screenshot%20from%202025-12-30%2021-09-26.png)

---

## Note

Do the following changes in pad_placement_constraint.tcl to place the IO pads properly:

create_io_guide -side right -pad_cells {analog_out_sel_buf comp_ena_buf comp_in_buf comp_ninputsrc_buf0 comp_ninputsrc_buf1 comp_pinputsrc_buf0 comp_pinputsrc_buf1 ext_clk_buf ext_clk_sel_buf ext_reset_buf flash_clk_buf flash_csb_buf} -line {{3588 5188} 5188}


create_io_guide -side left -pad_cells {flash_io_buf_0 flash_io_buf_1 flash_io_buf_2 flash_io_buf_3 gpio0 gpio1 gpio10 gpio11 gpio12 gpio13 gpio14} -line {{0 0} 5188}
create_io_guide -side top -pad_cells {gpio15 gpio2 gpio3 gpio4 gpio5 gpio6 gpio7 gpio8 gpio9 irq_pin_buf opamp_bias_ena_buf} -line {{0 5188} 3588}
create_io_guide -side bottom -pad_cells {opamp_ena_buf overtemp_buf overtemp_ena_buf pll_clk_buf rcosc_ena_buf rcosc_in_buf reset_buf ser_rx_buf ser_tx_buf spi_sck_buf trap_buf xtal_in_buf} -line {{3588 0} 3588}


## Summary

This task successfully establishes a clean SoC floorplan in ICC2 using the synthesized netlist from DC. The die size, core offset, and IO keep-out regions are explicitly controlled through TCL commands, ensuring reproducibility and correctness. The generated DEF and reports serve as a solid handoff point for subsequent placement, CTS, and routing stages.
