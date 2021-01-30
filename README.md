# VSD OpenLANE Workshop

<br />
<p align="center">


![advanced_physical_design](https://user-images.githubusercontent.com/53702961/105613023-e40ec800-5de5-11eb-9f19-9598a9099e25.png)

<h3 align="center">Advanced Physical Design - OpenLANE Workshop</h3>
</p>


## Day 1:- Introduction to OpenLANE,various open-source EDA tools and Sky130 PDK

### Skywater PDK Files

The PDK files that were used for the workshop was kept inside PDK directory. The directory had 3 sub-directories:-

1. `Skywater-pdk`: Contains all the foundry provided PDK related files
2. `Open_pdks`: Contains scripts that are used to bridge the gap between closed-source and open-source PDK to EDA tool compatibility
3. `Sky130A`: The open-source compatible PDK files

Under `Sky130A/libs.ref` directory the process specific files were kept. The one which was used in our design was sky130_fd_sc_hd, where:- 

- sky130A is the Process name(SkyWater Technology Foundry's 130nm node)
- fd stands for foundry
- sc stands for standard cell
- hd stands for standard cell

`Sky130A/libs.tech` directory contains all the tools specific files (klayout, magic, netgen, ngspice, openflow, qflow).

### Invoking OpenLane

To invoke OpenLANE, the command `./flow.tcl` was used inside the OpenLANE directory. In order to run OpenLANE in an interactive mode the command `./flow.tcl -interactive` was used.

![Openlane_invoking](https://user-images.githubusercontent.com/53702961/106334815-71a05b00-62b1-11eb-9fc6-74c2ccde989e.png)

### Package Importing & Prepare design

- The command `package require openlane 0.9` was used to import all the packages to run OpenLANE.
- `prep -design <design_name>` is used to make file structure for our design.
   Besides,this command merges the cell LEF and technology LEF information as "merged.lef".
    - Cell LEF: contains information of each standard cell.
    - Tech LEF: contains layer definitons and a set of restricted design rules.
   
![Package](https://user-images.githubusercontent.com/53702961/106335574-1bccb280-62b3-11eb-8363-a4265db96746.png)

### Design

The design that was used in this workshop was "picorv32a" (Picorv32 is a RISC-V based Processor).
The full command used to prepare the design was `prep -design picorv32a -tag trial_run1` (tag flag creates a directory with a custom name in the `runs` directory)

![design](https://user-images.githubusercontent.com/53702961/106337753-060dbc00-62b8-11eb-8ee9-c2411b0f8eda.png)

Inside `openlane/designs/picorv32a` directory there are two distinct components:-

- `src` directory: contains verilog(`.v`) files and `.sdc` constraint files.
- `config.tcl` files: design specific configuration switches used by OpenLANE.

To make changes in the `config.tcl` file from OpenLANE, the `-overwrite` flag is used(shown below). However, this usually cleans everything preceding it, which is why there is emphasis laid on using the set keyword for making changes.

![overwrite](https://user-images.githubusercontent.com/53702961/106359379-7fe58a00-6338-11eb-8fb6-59acbed5e4f1.png)

### Synthesis

To run synthesis and generate the netlist we used the command `run_synthesis`.
The initial report of the synthesized netlist are as follows(can be obtained from `yosys_2.stat.rpt` file inside `reports/synthesis` directory)

![synth_results1](https://user-images.githubusercontent.com/53702961/106339280-1e7fd580-62bc-11eb-9545-0c5855b49067.png)
![synth_results2](https://user-images.githubusercontent.com/53702961/106339284-22abf300-62bc-11eb-9daf-3bd0b501360b.png)

> Note: Inside the timing report the WNS(worst negative slack) must not be a negative number as it indicates a timing violation(example given below).

> ![wns_neg](https://user-images.githubusercontent.com/53702961/106339463-a82fa300-62bc-11eb-9727-81ae7906bd65.png)

## Day 2:- Floorplanning & Placement

The following things are taken care of during the Floorplanning stage:-
- Defining the Core & Die Area.
- Defining the Utilization Factor and the Aspect Ratio.
- Placement of Macros & IO pins.
- Power Distribution Network (Normally done in Floorplan stage but done later in the OpenLANE flow)
 	
#### Utilization Factor and Aspect Ratio
The amount of core area occupied by the netlist (Standard cells) is known as Utilization Factor. Generally, it is kept in the range of 0.5-0.7 or 50-70%. This allows for optimization of placement and realizable routing of a system. Aspect ratio is the ratio of the Height and the Width of the core area. An aspect ratio of 1 discribes the chip as a square.

#### Preplaced Cells
Preplaced cells or MACROs or IPs are used in our designs(eg: Memories,PLL,etc.).The locations and blockages for preplaced cells are defined in the Floorplanning stage itself to ensure no standard cells are mapped where the those cells are located.

#### Decoupling Capacitors
Decoupling capacitors are placed local to preplaced cells during Floorplanning. Voltage drops associated with interconnect wires can heavily affect our noise margin or put it into an indeterminate state. Decoupling capacitor is a big capacitor located next to the macros to fix this problem. The capacitor will charge up to the power supply voltage over time and it will work as a charge reservoir when a transition is needed by the circuit instead of the charge coming from the power supply. Therefore it “decouples” the circuit from the main supply. The capacitor acts like the power supply.

#### Power Planning
Power planning during the Floorplanning phase is essential to lower noise in digital circuits attributed to voltage droop and ground bounce. Coupling capacitance is formed between interconnect wires and the substrate which needs to be charged or discharged to represent either logic 1 or logic 0. When a transition occurs on a net, charge associated with coupling capacitors may be dumped to ground. If there are not enough ground taps charge will accumulate at the tap and the ground line will act like a large resistor, raising the ground voltage and lowering our noise margin. To bypass this problem a robust PDN with many power strap taps are needed to lower the resistance associated with the PDN.

#### Pin Placement
Pin placement is an essential part of floorplanning to minimize buffering and improve power consumption and timing delays. The goal of pin placement is to use the connectivity information of the HDL netlist to determine where along the I/O ring a specific pin should be placed. In many cases, optimal pin placement will be accompanied with less buffering and therefore less power consumption. After pin placement is formed we need to place logical cell blockages along the I/O ring to discriminate between the core area and I/O area.

> Notes:- 
> - To check clock period, the command used was: `echo ::$env(CLOCK_PERIOD)`.
> - To change clock period to 12ns(say), the command used was: `set ::env(CLOCK_PERIOD) 12` .
> - Precedence order: `sky130A_sky130_fd_sc_hd.cofig.tcl` > `config.tcl` > `floorplan.tcl`.

### Floorplan

To execute Floorplan in OpenLANE, the command `run_floorplan` was used.

![fplan_done](https://user-images.githubusercontent.com/53702961/106359702-50d01800-633a-11eb-9da8-c9c45653e3c4.png)

- Output of Floorplan Stage: A DEF (Design Exchange Format) file `picorv32a.floorplan.def`, containing information about core area, placement of Standard Cell SITES,etc.

### Viewing Floorplan in Magic

For viewing Floorplan on Magic Layout Tool, 3 files are needed as inputs:-

1. Magic technology file (`sky130A.tech`)
2. DEF file of floorplan (`picorv32a.floorplan.def`)
3. Merged LEF file (`merged.lef`)

Inside the `results/floorplan` directory (containing `picorv32a.floorplan.def`), run the following command: 

![fplan_magic](https://user-images.githubusercontent.com/53702961/106360095-a1e10b80-633c-11eb-8ac3-f327719c81a9.png)

![fplan_layout](https://user-images.githubusercontent.com/53702961/106360233-5b3fe100-633d-11eb-87fe-535bb519280f.png)

An area was selected by making a box with left & right mouse clicks and pressed `z` to Zoom-IN that area to locate the pins(placed equidistantly from each other) of the die. The DeCAPs and Tap Cells(small rectangles right side of the picture) were also spotted.

![fplan_pins](https://user-images.githubusercontent.com/53702961/106360421-59c2e880-633e-11eb-95d5-b0fce17717d4.png)

### Placement

With both the Floorplan and Synthesized netlist being ready, the next step in ASIC flow is Standard cell placement.
OpenLANE does placement in two stages:

1. Global Placement: Optimized but not legal placement. Optimization works to reduce wirelength by reducing half parameter wirelength (HPWL). It is not a legal placement as there might be instance of standard cells getting overlapped.
2. Detailed Placement: Legalizes placement of cells into standard cell rows while adhering to Global Placement.

Command used for Placement is: `run_placement`

The Evaluation and Legality checks are reported at the end.

![Placement_reports](https://user-images.githubusercontent.com/53702961/106361021-a956e380-6341-11eb-8c30-e9c5ced761d2.png)



> Note: Placement is an iterative process, the `OVFL`(overflow) must converge to 0 in the end in order for placement to be successful.

- Output of Placement Stage: A DEF (Design Exchange Format) file `picorv32a.placement.def`.

![Placement_def](https://user-images.githubusercontent.com/53702961/106361155-66494000-6342-11eb-9679-fff3b1fae467.png)

### Viewing Placement in Magic

For viewing Placement on Magic Layout Tool, 3 files are needed as inputs:-

1. Magic technology file (`sky130A.tech`)
2. DEF file of Placement (`picorv32a.placement.def`)
3. Merged LEF file (`merged.lef`)

Inside the `results/placement` directory (containing `picorv32a.placement.def`), run the following command:

![Placement_magic](https://user-images.githubusercontent.com/53702961/106361800-86c6c980-6345-11eb-9d01-1664dbdbc635.png)

The post-Placement Layout:
![Placement_layout](https://user-images.githubusercontent.com/53702961/106361971-41ef6280-6346-11eb-9d0b-0f471e854148.png)

### Standard Cell Characterization

Standard Cell Libraries consist of cells with different functionality/drive strengths. These cells need to be characterized by liberty files to be used by synthesis tools to determine optimal circuit arrangement. The open-source software GUNA is used for characterization.

#### Cell Design Flow

The three stages of Standard cell design flow are:-
- Inputs: PDKs, DRC & LVS rules, SPICE models, Library and User-defined specifications.
- Design: Circuit Design, Layout Design, Standard cell Characterization (performed by GUNA). Types of characterization includes Timing characterization, Power characterization and Noise characterization.
- Outputs: CDL (Circuit Description Language), GDSII, LEF, extracted Spice netlist (.cir).









