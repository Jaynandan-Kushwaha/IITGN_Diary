# RISC-V SoC Tapeout Program Documentation

---

# End-to-End Design Verification of a VSD-Caravel SoC with Technology Migration from SKY130 to SCL-180 and Physical Implementation of Raven SOC 

---
##  Task-wise Repository Structure

| Task       | Description                                     |
| ---------- | ----------------------------------------------- |
| **Task-1** | RTL & GLS of Caravel IC using SKY130            |
| **Task-2** | RTL & GLS of Caravel IC using SCL180            |
| **Task-3** | RTL & GLS using SCL180 with Synopsys-style flow |
| **Task-4** | Caravel without dummy POR – final clean GLS     |
| **Task-5** | Individual module testing of dvlike storage, Gpio, IRQ, mprj_ctrl  |
| **Task-6** | Floorplanning of Raven SoC                      |
| **Task-7** | Complete Physical Design of Raven SoC           |

## 📘 Project Overview

This repository documents my **end-to-end System-on-Chip (SoC) design workflow** carried out during **Phase-2 of the RISC-V SoC Tapeout Program**, covering both **frontend verification** and **backend physical design** using industry-style methodologies.

The work is organized into **two distinct but complementary design tracks**:

1. **RTL and Gate-Level Simulation (GLS) verification of the HKSPI module within the VSDCaravel SoC**, including **technology migration from SKY130 to SCL-180 (180 nm)** and validation of functional equivalence across technology nodes.
2. **Complete Physical Design (PD) implementation of the Raven SoC**, focusing exclusively on backend stages such as floorplanning, power planning, placement, and routing, without performing technology migration.

The objective of this repository is to demonstrate a **tapeout-ready SoC design mindset**, where RTL correctness, synthesis consistency, and physical implementation are treated as a **single continuous flow**, rather than isolated frontend and backend tasks. The project emphasizes **technology awareness, verification robustness, and physical feasibility**, which are essential for real-world silicon implementation.

## Design Scope Overview

| Aspect                  | Implementation Details                                             |
| ----------------------- | ------------------------------------------------------------------ |
| **Target SoCs**         | VSDCaravel (verification & technology migration) & Raven SoC (PD)  |
| **Verification Scope**  | RTL functional simulation, Gate-Level Simulation, reset analysis, module validation |
| **Physical Design Scope** | Core floorplanning, power grid creation, placement, and routing    |
| **Technology Nodes**    | Migration from SKY130 to SCL180 (180 nm)                            |
| **Tools & Methodology** | Combination of open-source and industry-grade EDA flows            |
| **Primary Focus Areas** | Technology migration, backend correctness, timing and power optimization |

## 📊 Overview of Technical Workflow

### 🔹 Stage 1: Functional Verification & Gate-Level Validation of HKSPI in VSDCaravel

In this stage, the focus was on ensuring **correct RTL behavior, synthesis integrity, and robustness of the verification flow** for the **HKSPI module** within the **VSDCaravel SoC**.  

The workflow emphasizes **reliable RTL and gate-level verification** across multiple technology nodes and toolchains, guaranteeing that functional behavior remains consistent after synthesis.

---

### 🎯 Goals

* Confirm RTL functionality through simulation
* Ensure **RTL ↔ Gate-Level Netlist equivalence**
* Validate flows using the **SCL180 technology node**
* Transition verification methodology from open-source tools to industry-standard EDA flows

---

### 🛠️ Verification Highlights

| Area of Focus                        | Tools & Methodology      | Results & Insights                                                       |
| ------------------------------------ | ----------------------- | ------------------------------------------------------------------------ |
| HKSPI Module Functional Validation   | Icarus Verilog, Yosys   | RTL and GLS waveforms aligned, interface behavior fully verified          |
| SCL180-Based Synthesis & Gate-Level  | Synopsys DC Shell       | Successful synthesis with no functional mismatches                        |
| Migration to Professional Tools      | Synopsys VCS, DC_TOPO   | Faster compile times, enhanced debug, industry-grade verification process |


## 🔹 Stage 2: RTL Architecture Investigation & Optimization

In this stage, we performed a **deep RTL-level audit** to ensure **synthesizability, functional correctness, and backend readiness** for the VSDCaravel SoC. The focus was on **identifying fragile constructs, improving reset behavior, and eliminating functional inconsistencies** across multiple PDKs.

---

### 🎯 Objectives

* Detect and remove **non-synthesizable or timing-sensitive constructs**
* Ensure **deterministic and robust reset behavior**
* Investigate and resolve **functional mismatches** at module and interface level
* Prepare the design for **seamless GLS and PD handoff**

---

### 🛠️ Key Debug Findings & Design Enhancements

| Area of Focus                   | Observation / Issue                                      | Resolution / Outcome                                                             |
| --------------------------------| -------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Reset & POR Logic               | `dummy_por` induced unpredictable delays for synthesis   | Replaced with a **single deterministic active-low reset (`reset_n`)**           |
| GPIO Interface & CSR Mapping    | Misaligned register addresses, pad control conflicts     | Fixed CSR/MMIO mapping; ensured pads are correctly driven                       |
| Module Integration              | Cross-module net inconsistencies during GLS              | Verified RTL ↔ GLS equivalence; adjusted hierarchical connections               |
| Clock & Reset Synchronization   | Metastability risks in asynchronous reset domains       | Added proper clock domain synchronization for reset propagation                 |

---

### 🔍 Key Activities

* Executed **RTL simulations** to validate baseline behavior  
* Conducted **Gate-Level Simulations (GLS)** with synthesized netlists  
* Verified **RTL ↔ GLS equivalence** across:

  * SKY130 PDK  
  * SCL180 PDK  

* Tested **critical scenarios**, including:

  * Reset sequences and POR propagation  
  * Clock domain transitions and timing edges  
  * Module-level interface transactions  

---
### Key Debug & Design Actions

* Removed **dummy POR logic**, replaced with **deterministic active-low reset (`reset_n`)**.  
* Corrected **GPIO and MPRJ pad mapping** to fix MMIO/CSR incompatibilities.  
* Verified **all module interfaces** under different reset and clock transitions.
  
  ---

### 🔌 Reset Distribution & Module Connectivity Diagram

Below is a **reimagined visual diagram** of the reset and module connectivity architecture. The layout, signals, and hierarchy are uniquely represented to highlight **active-low reset propagation** and module interactions.

```text
─────────────────────── RESET LOGIC TRANSFORMATION ──────────────────────
┌───────────────────────────── Before ────────────────────────────┐
│  External Reset Pad: resetb -> rstb_h (unused)                   │
│                                                                 │
│    ┌─────────────── dummy_por module ──────────────┐             │
│    │  Generates POR outputs:                       │             │
│    │  porb_h, porb_l, por_l                        │             │
│    └───────────────┬─────────────────────────────┘             │
│                    │                                           │
│                    ▼                                           │
│       ┌───────────────┐  ┌───────────────┐  ┌───────────────┐ │
│       │ caravel_core  │  │ housekeeping  │  │ chip_io       │ │
│       │ .reset(porb_h)│ │ .reset(porb_l) │ │ .reset(por_l) │ │
│       └───────────────┘  └───────────────┘  └───────────────┘ │
│          │                   │                   │             │
│          ▼                   ▼                   ▼             │
│       CPU/RAM            SPI/Flash            HV Pads & IO      │
└───────────────────────────────────────────────────────────────┘

───────────────────── After ─────────────────────
┌───────────────────────────── After ──────────────────────────────┐
│  External Reset Pad: resetb -> rstb_h (unused)                     │
│                                                                     │
│  ┌───────────── Removed dummy POR ─────────────┐                     │
│  │  dummy_por module completely removed       │                     │
│  └─────────────┬─────────────────────────────┘                     │
│                │                                                 │
│                ▼                                                 │
│       ┌───────────────────────────┐                                │
│       │ New deterministic reset_n │                                │
│       │ (active-low, synthesis    │                                │
│       │ friendly)                 │                                │
│       └─────────────┬────────────┘                                │
│                       │                                          │
│        ┌──────────────┼────────────────────┐                       │
│        ▼              ▼                    ▼                       │
│    porb_h           porb_l                por_l                     │
│    │                │                     │                          │
│    │  <--- assigned from reset_n --->     │                          │
│    │    (assign porb_h = reset_n;         │                          │
│    │     assign porb_l = reset_n;         │                          │
│    │     assign por_l   = reset_n;)       │                          │
│                                                                     │
│       ┌───────────────┐  ┌───────────────┐  ┌───────────────┐       │
│       │ caravel_core  │  │ housekeeping  │  │ chip_io       │       │
│       │ .reset(porb_h)│ │ .reset(porb_l) │ │ .reset(por_l) │       │
│       └───────────────┘  └───────────────┘  └───────────────┘       │
│          │                   │                   │                 │
│          ▼                   ▼                   ▼                 │
│       CPU/RAM            SPI/Flash            HV Pads & IO          │
└───────────────────────────────────────────────────────────────────┘
```
![](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task6/Image/WhatsApp%20Image%202025-12-31%20at%201.31.38%20PM.jpeg)
### Transition from Dummy POR to Deterministic reset_n

* **Before:** The `dummy_por` module generated three output wires:
  - `porb_h` → connected to caravel_core
  - `porb_l` → connected to housekeeping
  - `por_l`   → connected to chip_io
* **Problem:** The `dummy_por` module was non-synthesizable and caused backend issues.
* **After:** We completely **removed the `dummy_por` module**.
* Introduced a clean **`reset_n` signal** (active-low, deterministic).
* **Assignment to old POR wires:**
```verilog
assign porb_h = reset_n;
assign porb_l = reset_n;
assign por_l   = reset_n;
```
### 🔹 Significance of GLS Verification

Gate-Level Simulation (GLS) plays a **critical role in bridging RTL design with physical implementation**. This process confirms that the **synthesized netlist behaves exactly as intended by the RTL**, ensuring a reliable transition from logical design to backend flows.

Key benefits of GLS verification include:

* **Functional Fidelity:** Confirms that all RTL logic is accurately preserved post-synthesis.  
* **Regression Prevention:** Detects potential functional mismatches or errors introduced during synthesis.  
* **Backend Readiness:** Guarantees that the design is stable and safe to hand off to physical design flows.  
* **Module Integrity:** Validates interactions across all sub-modules and interfaces, including HV pads, GPIO, and SPI blocks.

---

### 🔹 Preparing for Physical Implementation

After GLS verification, the design reached a stage where it was **logically robust and backend-compatible**. During this process, several issues were uncovered in:

* **GPIO Block:** Misaligned register mappings and control signal inconsistencies.  
* **MPRJ Block:** Improper pad connections and initialization sequencing.
* **STorage Block**
* **IRQ Block**
These issues were **carefully documented and resolved** before moving forward.  

To streamline the **physical design process**, we transitioned to the **Raven SoC**, leveraging its **Synopsys-based PD flow**. This approach allowed us to:

* Reuse verified module-level designs from VSDCaravel  
* Apply a robust PD flow in Raven SoC without reintroducing known issues  
* Explore both **frontend verification and backend physical design workflows** simultaneously  

This methodology ensured that **time and resources were optimized**, while maintaining **high design fidelity** from RTL through to full-chip placement and routing.
![Alt text](https://github.com/Jaynandan-Kushwaha/IITGN_Diary/blob/main/Task_NoPOR_Final_GLS/lab%20work/gls%20simulation.png)
## 🔄 Technology Node Transition: SKY130 → SCL180 (180 nm)

One of the core contributions of this project is the **seamless adaptation of the VSDCaravel SoC design from the SKY130 PDK to SCL180**, ensuring functional integrity while aligning with the design rules of a new fabrication process.

### Challenges Overcome During Migration

| Migration Aspect        | SKY130 Characteristics         | SCL180 Adaptation                        |
| ---------------------- | ------------------------------ | ---------------------------------------- |
| Feature Size            | 130 nm                        | 180 nm                                   |
| Standard Cell Libraries | SkyWater open-source cells    | SCL proprietary standard-cell libraries |
| Design Rule Complexity  | Tighter constraints           | Slightly relaxed but structurally different |
| Metal Layer Stack       | SKY130 metal stack            | SCL180 metal routing hierarchy           |
| Timing Profile          | Faster switching times        | Slightly slower but more predictable     |

### Actions Taken for Successful Migration

* Rewrote and optimized **synthesis scripts** to suit SCL180 libraries  
* Adjusted **floorplan and placement strategies** for larger feature size and modified routing layers  
* Updated **timing constraints and path definitions** to maintain functional equivalence  
* Verified compliance with SCL180 **design rules and DRC checks**  
* Ensured that the **RTL-to-GDS flow remained stable** without introducing logic regressions  

### Key Takeaways

* Demonstrates **portability of SoC designs across technology nodes**  
* Highlights the importance of **PD-aware RTL verification**  
* Balances **functional correctness with process-specific physical constraints**  
* Prepares the design for **realistic manufacturing environments**  

This migration reflects a **hands-on, tapeout-oriented mindset**, where RTL, synthesis, and backend flows are harmonized for **new process technologies**.

## 🔹 Stage 3: Backend Layout Preparation for Raven SoC

This stage represents the **critical bridge between verified RTL and manufacturable silicon**, focusing on setting up the **Raven SoC** for accurate placement, routing, and power planning. The goal is to ensure **physical correctness and layout readiness** before full-chip implementation.

### Main Goals

* Create a **structured physical entry point** for the chip
* Prepare the netlist and hierarchy for **placement and routing**
* Validate **design-rule compliance and physical constraints**
* Enable smooth downstream **timing closure and signal integrity analysis**

### Backend Preparation Activities

| Design Focus               | Approach / Tools           | Deliverables & Insights                                        |
| -------------------------- | ------------------------- | --------------------------------------------------------------- |
| **Core Floor Layout**       | Physical Design Tools      | Core area estimation, die outline, macro placement, I/O planning |
| **Power Network Setup**     | Automated TCL Scripts      | VDD/VSS grid, ring structures, power stripes, EM/IR safety analysis |
| **Standard-Cell Placement** | Placement Engines         | Timing-driven cell distribution, congestion-aware placement, routing channels prepped |
| **Pre-Routing Evaluation**  | Physical Verification Tools | Area & congestion maps, preliminary timing analysis, DRC check results |

### Key Advantages of this Phase

* Ensures **backend-friendly chip organization**  
* Minimizes potential **routing congestion and timing violations**  
* Prepares for **high-fidelity routing, signal integrity, and power analysis**  
* Provides a **solid foundation for manufacturable silicon**  

By completing this phase, the Raven SoC is fully **aligned for detailed placement, routing, and final PD verification**, reducing downstream iterations and improving overall design quality.

## 🔹 Part 2: Raven SoC Physical Implementation (Core Contribution)

This section highlights the **end-to-end Physical Design (PD) flow of the Raven SoC**, carried out with a **fabrication-ready mindset**.  

Unlike simulation-focused tasks, this phase addresses **real-world silicon constraints**, including layout, routing, power distribution, and signal integrity, ensuring the design is **ready for manufacturing**.

---

## 🏗️ Comprehensive Physical Design Workflow

### 1️⃣ Floorplanning

Floorplanning establishes the **structural blueprint** for the chip.

**Key Activities:**

* Estimating die size and core area
* Optimizing core utilization to balance performance and routability
* Mapping logical hierarchy to physical layout
* Strategically placing I/Os and macros for minimal congestion

**Importance:**

* Prevents routing bottlenecks
* Facilitates timing closure
* Provides a foundation for power distribution and placement

---

### 2️⃣ Power Network Design

Power planning ensures **stable and reliable voltage delivery** across the chip.

**Strategies Implemented:**

* Constructing robust power rings around the core
* Deploying power stripes throughout standard-cell regions
* Separating VDD and VSS networks to minimize IR drop
* Incorporating electromigration-aware design for long-term reliability

**Objective:**

* Maintain uniform voltage across all blocks
* Avoid hotspots and voltage droops
* Enable dense and complex routing without power issues

---

### 3️⃣ Standard Cell Placement

Placement translates the **logical netlist into a physically realizable layout**.

**Focus Areas:**

* Congestion-aware distribution of cells
* Timing-conscious placement to minimize delays
* Reducing long interconnects to improve performance
* Preparing optimal channels for routing

**Impact:**

* Directly influences timing closure and overall chip performance
* Lays groundwork for efficient routing

---

### 4️⃣ Routing (Global & Detailed)

Routing finalizes the **physical connectivity** of all signals.

**Key Considerations:**

* Ensuring signal integrity and minimal crosstalk
* Efficient layer assignment for metal usage
* Avoiding congested regions and routing conflicts
* Guaranteeing design rule compliance

**Outcome:**

* A manufacturable layout
* Reliable connectivity across all modules
* Ready for downstream tapeout and fabrication

---

This workflow ensures that the Raven SoC design moves from **logical correctness** to **silicon-ready physical implementation**, addressing both **backend constraints** and **manufacturability concerns**.
## ✅ Conclusion

This project captures a **complete journey of SoC design and physical implementation**, bridging the gap between **frontend verification and backend tapeout readiness**.  

Key highlights include:  

* **RTL & GLS Verification** of the HKSPI module within VSDCaravel, ensuring robust functional correctness and synthesis equivalence.  
* **Technology Migration** from SKY130 to SCL180, demonstrating the design’s adaptability across different fabrication nodes.  
* **Physical Design of Raven SoC**, covering floorplanning, power planning, placement, and routing while addressing real-world fabrication constraints.  
* Establishing a **holistic tapeout-oriented workflow**, where RTL, synthesis, and physical implementation are seamlessly integrated.  

Overall, this work reflects the **end-to-end mindset required for modern SoC development**, emphasizing both **functional integrity** and **physical manufacturability**.

---

## 🙏 Acknowledgments

I extend my sincere gratitude to [**Kunal Ghosh**](https://github.com/kunalg123) and the entire **[VLSI System Design (VSD)](https://vsdiat.vlsisystemdesign.com/)** team for providing the opportunity to actively participate in the **RISC-V SoC Tapeout Program**.  

I also acknowledge the support and guidance of **RISC-V International**, [**Efabless**](https://github.com/efabless), and **IIT Gandhinagar**, whose resources and mentorship made this project possible and enriched the learning experience.

<div align="center">
