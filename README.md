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
   Besides,this command merges the cell LEF and technology LEF information as "merge.lef".
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

### Synthesis

To run synthesis and generate the netlist we used the command `run_synthesis`.
The initial report of the synthesized netlist are as follows(can be obtained from `yosys_2.stat.rpt` file inside `reports/synthesis` directory)

![synth_results1](https://user-images.githubusercontent.com/53702961/106339280-1e7fd580-62bc-11eb-9545-0c5855b49067.png)
![synth_results2](https://user-images.githubusercontent.com/53702961/106339284-22abf300-62bc-11eb-9daf-3bd0b501360b.png)

> Note: Inside the timing report the WNS(worst negative slack) must not be a negative number as it indicates a timing violation(example given below).

> ![wns_neg](https://user-images.githubusercontent.com/53702961/106339463-a82fa300-62bc-11eb-9727-81ae7906bd65.png)


