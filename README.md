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

## Day 3:- Library Cell Design using Magic & Ngspice

### Cloning the GitHub Repository for obtaining the design file

The work done in Day-3 mainly revolved around a simple CMOS inverter which was predesigned in MAGIC Layout tool. The `.mag` file of the concerned Inverter was obtained from @nickson-jose's GitHub repository titled "vsdstdcelldesign" (Link: https://github.com/nickson-jose/vsdstdcelldesign).

So, the entire repository was cloned on our system in the `openlane_working_dir/openLANE_flow/designs` directory using the command: `git clone https://github.com/nickson-jose/vsdstdcelldesign`.

To see the Layout of this Inverter in Magic Tool, we need two files:
- `sky130A_inv.mag` (obtained from cloning the Git Repo.)
- `sky130A.tech` (copied from `openlane_working_dir/pdks/sky130A/libs.tech/magic` to `designs/vsdstdcelldesign` directory)

To view the layout in Magic Tool, the following command is given as shown:-
![stdcell_magic](https://user-images.githubusercontent.com/53702961/106368108-424f2400-636d-11eb-8003-e2aead081e21.png)

The Layout of the Inverter:
![stdcell_layout](https://user-images.githubusercontent.com/53702961/106368113-47ac6e80-636d-11eb-8d84-922e85497b01.png)

> Note: The right side panel in Magic contains different colour palettes indicating different layers that are used in the layout of a cell.
> Example(Layers and Components of the Inverter):-

![initial_metal2](https://user-images.githubusercontent.com/53702961/106368359-00bf7880-636f-11eb-8758-284a16c6eda9.jpg)

### Design Rule Check (DRC)

In Magic's GUI, the DRC errors can be viewed on the top panel. If DRC=0(indicated by GREEN), the design has no violations but if the value is greater than zero(indicated by RED), the design contains errors and must be fixed.

![drc](https://user-images.githubusercontent.com/53702961/106368730-e63ace80-6371-11eb-9350-0bd476991501.png)

![drc_2](https://user-images.githubusercontent.com/53702961/106368739-fbaff880-6371-11eb-8768-0fa26b46cd5b.png)

> Note: Instead of iteratively doing DRC Checks after the layout process, the designer can make use of Magic's dynamic and continuous DRC checks to make sure the design is DRC free during creation of the design/netlist.

### Parasitic Extraction in Magic

Extraction of Parasitic `.spice` file is done in the `tkcon` window of Magic by the use of the following commands:-
- `extract all` command to create the `.ext` (extraction) file.
- `ext2spice` command to create the `.spice` file from the `.ext` file.

![pex_tkcon](https://user-images.githubusercontent.com/53702961/106369056-564a5400-6374-11eb-90d7-9f57283f6ce8.png)

The `.ext` and `.spice files` were being created in the `designs/vsdstdcelldesign` directory:-
![pex_files](https://user-images.githubusercontent.com/53702961/106369116-d07ad880-6374-11eb-9d5e-318535d210be.png)

### Spice wrapper for Simulation

The `.spice` file as obtained after the extraction was:-
![pex_spice](https://user-images.githubusercontent.com/53702961/106369324-7c70f380-6376-11eb-8e37-b7dde8ed55c8.png)

Some changes were made in this file (`sky130A_inv.spice`) to prepare it for further stages of the OpenLANE flow. They are as follows:

- `.include ./libs/phsort.lib & .include./libs/nshort.lib`: includes the pshort and nshort libraries 
- pshort & nshort are replaced with `pshort_model.0` and `nshort_model.0` respectively.
- Some ports were in the design: VDD, VSS, Va. For example, `Va A GND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)`: which implies a source Va, between A and GND, whose waveform is defined as a PULSE function with min value = 0V and max value = 3.3V.
- `.tran 1n 20n` - transient sweep from 1ns to 20 ns.

The `.spice` was opened in Ngspice:-
![ng_spice](https://user-images.githubusercontent.com/53702961/106369519-47fe3700-6378-11eb-82f7-9b19dfc2809c.png)

Next, we view the plot in Ngspice by the command : `% plot y vs time a`
![ng_waveform](https://user-images.githubusercontent.com/53702961/106369489-edfd7180-6377-11eb-90f3-ff6e3ba717b5.png)

### Characterization of the Cell

The plot was used to compute the 4 parameters which intricately define the inverter designed in question, this is called characterization of the cell. They are:-

- Rise Time: The time taken for the signal to go from 20% of its max value to 80% of its max value.
- Fall Time: The time taken for the signal to go from 80% of its max value to 20% of its max value.
- Propagation Delay(Rising): The time difference between the points where the input and output are at 50% of their magnitude when the signal rises from 0V to max value.
- Propagation Delay(Falling): The time difference between the points where the input and output are at 50% of their magnitude when the signal falls from max value to 0V.

All the parameters are calculated respectively from the results obtained below (where x0 indicates time(in seconds) and y0 indicates signal value(in V)):- 
![ng_charact](https://user-images.githubusercontent.com/53702961/106369773-29993b00-637a-11eb-8f4e-52fcacb7fad9.png)

## Day 4:- Pre-Layout Timing Analysis & Clock Tree Synthesis

### LEF files

- Cell LEF: contains abstract information of Standard Cells.
- Technology LEF: contains information on metal layers,IO, power & ground pins, via information and DRC rules of a particular technology(used by PnR tools).

> Note: LEFs does not contains information about the logical functionality of the circuit. So, in a way LEFs protect the IPs.

### PnR Guidelines for making Standard Cells

1. Input and Output ports must lie on Vertical and Horizontal tracks. 
2. Width of Standard cell should be Odd multiples of Track Horizontal Pitch (x pitch) & Height should be Odd multiples of Track Vertical Pitch (y pitch).

#### Track Info

![track_info](https://user-images.githubusercontent.com/53702961/106370268-5ea78c80-637e-11eb-9b5f-09b128e5d22a.png)

The information in the file says that for the li1 layer the x or horizontal track is at an offset of 0.23 and a pitch of 0.46. The offset is the distance from the origin to the routing track in either the x or y direction. It is half the pitch so that means the tracks are centered around the origin.

### Verifying the PnR Guidelines

#### Guideline #1 :-

- Open Layout in Magic
- make `grid on`
- Converging the Grid definition with Track definition: To ensure a cell is aligned with routing grids in Magic we can display a grid on top of the layout in accordance with the Track definition.

![grid_track](https://user-images.githubusercontent.com/53702961/106370393-9d8a1200-637f-11eb-8037-992a25b98a2c.png)

![grid_track2](https://user-images.githubusercontent.com/53702961/106370450-1d17e100-6380-11eb-8ff1-fad59c0d67c0.png)

#### Guideline #2 :-

- Count the number of boxes insides the PR boundary both vertically and horizontally (should be in odd multiples).


### Creating Port Definitions

The required port was selected (`Y` in this example) & then went to `Edit -> Text`.
![port_define](https://user-images.githubusercontent.com/53702961/106370580-5270fe80-6381-11eb-8364-4a9a67722839.png)

#### Setting port class and port use attributes

The ports are defined on the terminal as follows :-
```
% port class inout 
% port use power // for VPWR

% port class input
% port use signal // for A

% port class output
% port use signal // for Y

% port class inout
% port use ground // for VGND
```

### LEF Generation in Magic

The Magic Tool allows users to generate cell LEF information directly from the Magic terminal.
This was done by using the command `lef write` in the terminal. 

A LEF file by the name `sky130_vsdinv.lef` got generated inside the `designs/vsdstdcelldesign directory`.

### Including Custom Cells in OpenLANE

In order to integrate our Custom cell into the design `picorv32a`, firstly we need to make some changes in the `picorv32a/src/config.tcl` file:- 
- Adding a few libraries with their respective directories.
- Adding a line to integrate the `.lef` files that have been generated for the custom cell.

![modified_cnfdtcl](https://user-images.githubusercontent.com/53702961/106370782-6f0e3600-6383-11eb-9e7c-0836f3f75931.png)

 
Next, in the OpenLANE flow after preparing the design, two commands were executed to merge the `sky130_vsdinv.lef` with the `merged.lef`.

```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```

As a result, our MACRO `sky130_vsdinv` got included in the `merged.lef` file as shown:-
![vsdinv_in_merged](https://user-images.githubusercontent.com/53702961/106370947-222b5f00-6385-11eb-8c28-b58c0b704a3b.png)

#### Opening the Layout after Inclusion of the Custom cell

After running synthesis, floorplan and placement, Magic tool was opened again to make sure whether the Custom cell `sky130_vsdinv` was present or not.
> Note: We will use the currently obtained (after placement stage) DEF file from `results/placement` directory to open the Magic Tool.

![vsdinv_in_layout](https://user-images.githubusercontent.com/53702961/106371056-273cde00-6386-11eb-847b-60080b8bf42b.png)

As observed, our Custom cell `sky130_vsdinv` was present inside the picorv32 design. 

Expanded view (by typing the command `expand` in tkcon window :-

![expanded](https://user-images.githubusercontent.com/53702961/106371128-cd88e380-6386-11eb-9b6f-de2a02408c33.png)

### Fixing Slack Violations

After running synthesis, the initial timing reports showed negative Setup slack (wns was negative) which indicates a timing violation in our design.

![tim_1](https://user-images.githubusercontent.com/53702961/106371258-666c2e80-6388-11eb-852a-65f201b7d2c2.png)

Next, we tried to fix the violations in OpenSTA tool by running the command `sta pre_sta.conf` outside OpenLANE.

For that, we created a file called `pre_sta.conf`
![presta_config](https://user-images.githubusercontent.com/53702961/106371311-f8743700-6388-11eb-878b-6fb03615d10b.png)

Running `sta pre_sta.conf` gives a table of data that communicates the necessary information regarding the delays and skews respectively for various pins. It also provides information about the fanout at each terminal, and upon executing `report_net -connections <net_number>` one can get a clearer idea about what other connections that net has.

![sta1](https://user-images.githubusercontent.com/53702961/106371376-9831c500-6389-11eb-87d1-77a49dfa67af.png)

#### Strategy 1: Altering Synthesizing Strategies

![synth_strat](https://user-images.githubusercontent.com/53702961/106371478-d1b70000-638a-11eb-85c1-d0e1444f451e.png)

We changed the value of `SYNTH_STRATEGY` from 0 to 1.

#### Strategy 2: Upsizing the Buffers

Inside OpenSTA, we observed that certain nets (_11343_ in this case) are driven by sky130_fd_sc_hc__buf1 and are connected to 4 different pins. These nets are known as HFN (High Fanout Nets). The large fanout of these nets leads to an increased value of capacitance, coupled with the fact that many of these nets ave been assigned buffers that are not powerful enough to cater to the needs of all the components it fans out to. This is leading to both HOLD and SETUP slack violations.

![sta3](https://user-images.githubusercontent.com/53702961/106371567-f19af380-638b-11eb-9889-82daac5d8755.png)













