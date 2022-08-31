# OpenLANE-Sky130-Physical-Design-Workshop

This is the compilation of my notes for the 5 Day Workshop: [Advanced Physical Design using OpenLANE/Sky130](https://www.vlsisystemdesign.com/advanced-physical-design-using-openlane-sky130/). The goal is to cover the complete RTL2GDS flow using the open-source flow named [OpenLANE](https://github.com/The-OpenROAD-Project/OpenLane).

![image](https://user-images.githubusercontent.com/87559347/183438001-f1bee6d2-6e8c-47a7-a3cb-b63082727adc.png)



# Table of Contents
 - [DAY 1: Inception of open-source EDA, OpenLANE and Sky130 PDK](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#day-1-inception-of-open-source-eda-openlane-and-sky130-pdk)
   - [Simplified RTL-to-GSDII Flow](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#simplified-rtl-to-gdsii-flow)
   - [Directories inside OpenLANE Working Directory](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#notable-directories)
   - [Steps for the Lab](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#steps-for-the-lab)
 - [DAY 2: Good floorplan vs bad floorplan and introduction to library cells
](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#day-2-good-floorplan-vs-bad-floorplan-and-introduction-to-library-cells)
   - [Steps for Placement](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#steps-for-placement)
   - [Steps for the Lab](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#steps-for-the-lab-1)
   - [Library Characterization](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#library-characterization)
 - [DAY 3: Design library cell using Magic Layout and ngspice characterization
](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#day-3-design-library-cell-using-magic-layout-and-ngspice-characterization)
   - [Steps for the Lab](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#steps-for-the-lab-2)
 - [DAY 4: Pre-layout timing analysis and importance of good clock tree](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#day-4-pre-layout-timing-analysis-and-importance-of-good-clock-tree)
   - [Steps for Plugging in the Customized Cell](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#steps-for-plugging-in-the-customized-cell-to-openlane)
   - [About Delay Table](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#about-delay-table)
 - [DAY 5: Final steps for RTL2GDS using tritonRoute and openSTA](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#day-5-final-steps-for-rtl2gds-using-tritonroute-and-opensta)
 


# DAY 1: Inception of open-source EDA, OpenLANE and Sky130 PDK:

The core of the chip will contain two types of blocks:
 - **Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - **Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 
 
 ![image](https://user-images.githubusercontent.com/87559347/182751377-2810d388-21b0-4df1-b1d4-c72176d80d28.png)

Open Source Digital ASIC Design requires three open-source components:  
- **RTL Designs** = github.com, librecores.org, opencores.org
- **EDA Tools** = OpenROAD, OpenLANE,QFlow  
- **PDK** = Google + Skywater 130nm Production PDK

**PDK (Process Design Kit)** = A set of data files and documents which serves as the interface between the designer and the fab. This includes cell libraries, IO libraries, design rules (DRC, LVS, etc.)

### Simplified RTL to GDSII Flow:
- **Sythesis** = The RTL is converted into a gate level netlist made up of components of standard cell libary. 
- **Floor Planning/ Power Planning** = Plan silicon area and create robust power distribution network. The power network usually uses the upper metal layer which are thicker than lower layer and thus lower resistance. This lowers the IR drop problem
 - **Placement** = There are two steps, first is global placement which is the general optimal positons for cells and might not be legal. Next is detailed placement which is the actual legal placements of the cells.
 - **Clock tree synthesis** = clock distribution to all flip flops and is usually a tree (H-tree, X-tree ... )
 - **Routing** = Use horizontal and vertical wires to connect cells together. The router uses PDK information (thickness, pitch, width,vias) for each metal layer to do the routing. The Sky130 defines 6 routing layers. It doe global routing and detailed routing.
 - **Verification before sign-off** = Involves physical verification like DRC and LVS and timing verification. Design Rule Checking or DRC ensures final layout honors all design rules and Layout versus Schematic or LVS ensures final layout matches the gate level netlist from synthesis phase. Timing verification ensures timing constraints are met.  

 The final layout is in [GDSII file format](https://www.wikiwand.com/en/GDSII).
 
 [OpenLANE](https://github.com/The-OpenROAD-Project/OpenLane) = An open-source ASIC developement flow reference. It consists of multiple open-source tools needed for the whole RTL to GDSII flow. This is tuned epecially for Sky130 PDK. It also works for OSU 130nm. It is recommended to read the OpenLANE documentation before going forward.
 ![image](https://user-images.githubusercontent.com/87559347/182759711-6b9352ec-7652-4589-af31-53a409eb2830.png)

### Notable details are:  
- The input for the whole flow are the rtl files, sdc file, and PDK files. The output is GDSII/LEF file.

- Yosys is used to convert the HDL to gate level netlist using generic components. The [ABC script](http://people.eecs.berkeley.edu/~alanmi/abc/) is then used to map the generic components to the standard cell library of the PDK. These ABC scripts is used to make various synthesis strategies (using the Synthesis Exploration) which can optimize the design either with least area or best timing.  

- The Logic Equivalency Cheking (LEC) is used to compare the resulting netlist after optimization of place and route to the gate level netlist from synthesis phase

- Antenna Rules Violation = long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges  

### Notable Directories:

``` 
├── openlane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the commerical PDK (which is normally just compatible to commercial tools) to also be compatible with the open-source EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for open-source tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgen,magic...) 
```

Inside a specfic design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl in `Openlane/designs/[design]/`
2. config.tcl in `Openlane/designs/[design]/`
3. Default values in `Openlane/configuration/`

#### Task for the Day 1 Lab: Find the flipflop ratio percentage for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells

## Steps for the Lab:
**1. Run OpenLANE:**
 - `$ make mount` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in step by step `-interactive` mode
 - `% package require openlane 0.9` == retrives all dependencies for running v0.9 of OpenLANE  
 
 ![image](https://user-images.githubusercontent.com/87559347/182833010-c5b32449-bfa1-42d0-8433-9edfdefbf1f6.png)

 
**2. Design Setup Stage:**
 - `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This merged the [cell LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.lef` and [technology LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.tlef` generating `merged.nom.lef` inside `run/tmp/`
 

![image](https://user-images.githubusercontent.com/87559347/182833339-f117de40-af1f-4607-9c47-81b3ae5f7b7e.png)

**3. Run synthesis:**
 - `% run_synthesis` = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  
 
![image](https://user-images.githubusercontent.com/87559347/182847715-b0da0f41-b444-4036-9ca3-c5335bfd42cf.png)

##### The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%

After running synthesis, inside the `runs/[date]/results/synthesis` is `picorv32a_synthesis.v` which is the mapping of the netlist to standard cell library using ABC. The `runs/[date]/reports/synthesis` will contain synthesis statistic reports and static timing analysis reports. The `runs/[date]/synthesis/logs` contains log files for the terminal output dumps for running yosys and OpenSTA.

![image](https://user-images.githubusercontent.com/87559347/182874085-12a7d8b8-d095-4a18-b9ab-101050473046.png)



# DAY 2: Good floorplan vs bad floorplan and introduction to library cells

### Floor Plan General Steps:

#### 1. Find height and width of core and die.   
Core is where the logic blocks are placed and this seats at the center of the die. The width and height depends on dimensions of each standard cells on the netlist. Utilization factor is (area occupied by netlist)/(total area of the core). In practical scenario, utilization factor is 0.5 to 0.6. This is space occupied by netlist only, the remaining space is for routing and more additional cells. Aspect ratio is (height)/(width) of core, so only aspect ratio of 1 will produce a square core shape.

#### 2. Define location of Preplaced Cell.   
These are reusable complex logicblocks or modules or IPs or macros that is already implemented (memory, clock-gating cell, mux, comparator...) . The placement on the core is user-defined and must be done before placement and routing (thus preplaced cells). The automated place and route tools will not be able to touch and move these preplaced cells so this must be very well defined

#### 3. Surround preplaced cells with decoupling capacitors. 
The complex preplaced logicblock requires a high amount of current from the powersource for current switching. But since there is a distance between the main powersource and the logicblock, there will be voltage drop due to the resistance and inductance of the wire. This might cause the voltage at the logicblock to be not within the [noise margin](https://www.electronics-tutorial.net/digital-logic-families/noise-margin/) range anymore (logic is unstable). The solution is to use decoupling capacitors near the logic block, this capacitor will send enough current needed by the logicblock to switch within the noise margin range.

![image](https://user-images.githubusercontent.com/87559347/183011079-5f08eb01-28cf-4617-bcbc-f8f26eacc83d.png)

#### 4. Power Planning
Decoupling capactor for sourcing logic blocks with enough current is not feasible to be applied all over the chip but only on the critical elements (preplaced complex logicblocks). Large number of elements switching to logic 0 might cause ground bounce due to large amount of current that needs to be sink at the same time, and switcing to logic 1 might cause voltage droop due to not enough current from the powersource to source needed current of all elements. Ground bounce and voltage droop might cause the voltage to not be within the noise margin range. The solution is to have multiple powersource taps (power mesh) where elements can source current from the nearest VDD and sink current to the nearest VSS tap. This is the reason why most chips have multiple powersource pins.

#### 5. Pin Placement
The input and output ports are placed on the space between the core and the die. The placements of the ports depens on where the cells connected to those ports are placed on the core. The clock port is thicker(least resistance path) than data ports since this clock must be capable to drive the whole chip.

#### 6. Logical Cell Placement Blockage
This makes sure that the automated placement and routing tool does not place any cell on the pin locations of the die.

Below are all 6  steps for floor planning:  

![image](https://user-images.githubusercontent.com/87559347/183446309-a0714ec5-0619-4327-bdfe-890c19cc97e0.png)


### Steps for Placement
1. Bind the netlist to a physical cell with real dimensions. The physical cell will come from a library that can provide multiple options for shapes, dimensions, and delay for same cells. 
2. Next is placement of those physical cells to the floorplan. The flip flops must be placed as near as possible to the input and output pins to reduce timing delay. 
3. Optimize placement to maintain signal integrity. This is where we estimate wirelength and capacitance (C=EA/d) and based on that insert repeaters/buffers. The wirelength will form a resistanace which will cause unnecessary voltage drop and a capacitance which will cause a slew rate that might not be permissible for fast current switching of logic gates. The solution to reduce resistance and capacitance is to insert buffers for long routes that will act as intermediary and separate a single long wire to multilple ones. Sometime we also do abutment where logic cells are placed very close to each other (almost zero delay) if it has to run at high frequency (2GHz). Crisscrossing of routes is a normal condition for PnR since we can use separate metal layer (using vias) for crisscrossed path.
4. After placement optimization, We will setup timing analysis using idle clock (zero delay for wires and has no clock buffer related delays) considering we have not yet done CTS.   

The goal of placement is not yet on timing but on congestion. Also, standard cells are not placed on floorplan stage, it is done on Placement stage. Macros or preplaced cells are the ones placed on floorplan stage.Macros or preplaced cells are placed on floorplan stage.

![image](https://user-images.githubusercontent.com/87559347/183224947-67a29c54-9a18-45a4-bbd1-9132bcebc304.png)  

Placement is done on two stages:
 - Global Placement = placement with no legalizations and goal is to reduce wirelength. It uses Half Perimeter Wirelength (HPWL) reduction model. 
 - Detailed Placement = placement with legalization where the standard cells are placed on stadard rows, abutted, and must have no overlaps    
 
#### Task for the Day 2 Lab: Find the area of the die

## Steps for the Lab:
**1. Set configuration variables.** Before running floorplan stage, the configuration variables or switches must be configured first. The configuration variables are on `openlane/configuration`:  

```
.
├── README.md      
├── checkers.tcl
├── cts.tcl
├── floorplan.tcl  
├── general.tcl
├── lvs.tcl
├── placement.tcl
├── routing.tcl
└── synthesis.tcl 

```  

The  `README.md` describes all configuration variables for every stage and the tcl files contain the default OpenLANE settings. All configurations accepted by the current run is on `openlane/designs/picorv32a/runs/config.tcl`. This may come either from (with priority order):
 - PDK specific configuration inside the design folder
 - `config.tcl` inside the design folder
 - System default settings inside `openlane/configurations`

**2. Run floorplan on Openlane:** `% run floor_plan`

 
**3. Check the results.** The output of this stage is `runs/[date]/results/floorplan/picorv32a.floorplan.def` which is a [design exchange format](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html), containing the die area and positions. 
```
...........
DESIGN picorv32a ;
UNITS DISTANCE MICRONS 1000 ;
DIEAREA ( 0 0 ) ( 660685 671405 ) ;
............
```
The die area here is in database units and 1 micron is equivalent to 1000 database units. **Thus area of the die is (660685/1000)microns\*(671405/1000)microns = 443587 microns squared.** 

**4. View the layout on magic**. Open def file using `magic`:  

```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def
```  

![image](https://user-images.githubusercontent.com/87559347/183226953-8bf8b067-5a70-43a3-9b92-f39caaf02a4a.png)


To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in `tkcon` to display information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.  

![image](https://user-images.githubusercontent.com/87559347/183100900-b3527702-5375-4a4e-ad87-194fce382128.png)


#### Note on using Magic:
 - To select a particular cell instance (e.g. cell \_08555_ which can be searched in the def file): `% select cell _08555_`
 - To list all cells in the layout: `% cellname allcells`
 - To check if a cell exists: `% cellname exists sky130_fd_sc_hd__xor3_4`

**5 Run placement:** `% run_placement`. This commmand is a wrapper which does global placement (performed by RePlace tool), Optimization (by Resier tool), and detailed placement (by OpenDP tool). It displays hundreds of iterations displaying HPWL and OVFL. The algorithm is said to be converging if the overflow is decreasing. It also checks the legality. 

**6. View the output of this stage**. The output of this stage is `runs/[date]/results/placement/picorv32a.placement.def.` To see actual layout after placement, open def file using `magic`:  

```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```  

![image](https://user-images.githubusercontent.com/87559347/183227636-cc5b24b3-5b05-469f-9af1-de89b3c7ed1e.png)  

Power Distibution Network (PDN) is normally created on floorplan stage but on OpenLANE, PDN is done on post-CTS (before routing).  


### Library Characterization
Of all RTL-to-GDSII stages, one common thing that the EDA tool always need is data from the library of gates which keeps all standards cells (and, or, buffer gates,...), macros, IPs, decaps, etc. Same cells might have different flavors inside the library (different sizes, delays, threshold voltage). Bigger cell sizes means bigger drive strength to drive longer and thicker wires. Bigger threshold voltage (due to bigger size) will take more time to switch(slower clock) than those with smaller threshold voltage.  

A single cell needs to go through cell design flow. The inputs to make a single cell comes from the foundry Process Design Kits:
 - DRC & LVS Rules = tech files and poly subtrate paramters (CUSTOME LAYOUT COURSE)
 - SPICE Models  = Threshold, linear regions, saturation region equations with added foundry parameters. Including NMOS and PMOS parameteres (Ciruit Deisgn and Spice simulation Course)
 - User defined Spec = Cell height (separation between power and ground rail), Cell width (depends on drive strength), supply voltage, metal layer requirement (which metal layer the cell needs to work)

The library cell developer must adhere to the rules given on the inputs so that when the cell is used on a real design, there will be no errors. Next is design the library cell:
1. Design the circuit function (Output: circuit design language (CDL))
2. Model the pmos and nmos that meets input library requirement
3. Layout the design using Euler's path and sticky diagram to produce best area. This can be done on `magic` layout tool.The outputs are:
   - GDSII (layout file)
   - LEF (defines the width and height of cell)
   - extract spice netlist .cir (parasitics of each element of cell: resistance, capacitance)
 Afte design is characterization using GUNA software, where the outputs are timing, noise, and power characterization.
 .
 ### Timing Characterization (timing .lib)
 
Below are the timing variables for slew. This is two inverters in series, red is output of first inverter and blue is output of second inverter:  

![image](https://user-images.githubusercontent.com/87559347/183231913-a9b3826b-5139-4bdc-b12b-3495b87cd8b9.png)

Below are the timing variables for propagation delay. The red is input waveform and blue is output waveform of the buffer. The left side is rise delay and right side is fall delay.

![image](https://user-images.githubusercontent.com/87559347/183232515-fe3cef76-8a2f-475d-9a64-392fc2fda111.png)

Negative propagation delay is unexpected. That means the output comes before the input so designer needs to choose correct threshold point to produce positive delay. Delay threshold is usually 50% and slew rate threshold is usually 20%-80%.

# DAY 3: Design library cell using Magic Layout and ngspice Characterization

Configurations on OpenLANE can be changed on the flight. For example, to change IO_mode to be not equidistant, use `% set ::env(FP_IO_MODE) 2;` on OpenLANE. The IO pins will not be equidistant on mode 2 (default of 1). Run floorplan again via `% run_floorplan` and view the def layout on magic. However, changing the configuration on the fly will not change the `runs/config.tcl`, the configuration will only be available on the current session. To echo current value of variable: `echo $::env(FP_IO_MODE)`


### Steps to design a CMOS inverter cell
1. SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.
2. SPICE deck values = value for W/L (0.375u/0.25u means width is 375nm and lengthis 250nm). PMOS should be wider in width(2x or 3x) than NMOS. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)  
3. Add nodes to surround each component and name it. This will be used in SPICE to identify a component.    

#### Notes:
 - Width is the length of source and drain. Length is the distance between source and drain
 - PMOS' hole carrier is slower than NMOS' electron carrier mobility, so to match the rise and fall time PMOS must be thicker (less resistance thus higher mobility) than NMOS  
 - A good refresher on MOSFETS and CMOS [is this video](https://www.youtube.com/watch?v=oSrUsM0hoPs) and [this site.](http://courseware.ee.calpoly.edu/~dbraun/courses/ee307/F02/02_Shelley/Section2_BasilShelley.htm)

### Spice deck netlist description  

![image](https://user-images.githubusercontent.com/87559347/183240195-608727e5-2d04-4e44-ab4a-2df545cd13ea.png)

#### Notes:
 - Syntax for the PMOS and NMOS descriptiom:
     - `[component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]`
 - All components are described based on nodes and its values
 - `.op` is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
 - `tsmc_025um_model.mod` is the model file containing the technological parameters for the 0.25um NMOS and PMOS
The steps to simulate in SPICE:
```
source [filename].cir
run
setplot 
dc1 
plot out vs in 
```  

### SPICE Analysis for Switching Threshold and Propagation Delay
CMOS robustness depends on:  

**1. Switching threshold** = Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current is high. If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.

**2. Propagation delay** = rise or fall delay

DC transfer analysis is used for finding switching threshold. SPICE DC analysis below uses DC input of 2.5V. Simulation operation is DC sweep from 0V to 2.5V by 0.05V steps:
```
Vin in 0 2.5
*** Simulation Command ***
.op
.dc Vin 0 2.5 0.05
```  
Below is the result of SPICE simulation for DC analysis, the line intersection is the switching threshold:  

![image](https://user-images.githubusercontent.com/87559347/187056328-d6f6d5f5-4ce1-4454-9a5a-26be83a84734.png)




Meanwhile, transient analysis is used for finding propagation delay. SPICE transient analysis uses pulse input: 
1. starts at 0V
2. ends at 2.5V
3. starts at time 0
4. rise time of 10ps
5. fall time of 10ps
6. pulse-width of 1ns
7. period of 2ns  

![image](https://user-images.githubusercontent.com/87559347/187055752-dd66feae-f1e7-4b5b-a037-d1a148b01833.png)  

The simulation operation has 10ps step and ends at 4ns:  

```
Vin in 0 0 pulse 0 2.5 0 10p 10p 1n 2n 
*** Simulation Command ***
.op
.tran 10p 4n
```  
Below is the result of SPICE simulation for transient analysis:

![image](https://user-images.githubusercontent.com/87559347/187056370-18949899-a158-4307-96d9-d5c06bbeed66.png)
 
 ### CMOS Fabrication Process (16-Mask CMOS Process)
 **1. Selecting a substrate** = Layer where the IC is fabricated. Most commonly used is P-type substrate  
 **2. Creating active region for transistor** = Separate the transistor regions using SiO2 as isolation
  - Mask 1 = Covers the photoresist layer that must not be etched away (protects the two transistor active regions)
  - Photoresist layer = Can be etched away via UV light  
  - Si3N4 layer = Protection layer to prevent SiO2 layer to grow during oxidation (oxidation furnace)  
  - SiO2 layer = Grows during oxidation (LOCOS = Local Oxidation of Silicon) and will act as isolation regions between transistors or active regions  
  
![image](https://user-images.githubusercontent.com/87559347/187062659-9e18e9a5-eff4-4d01-804d-cc1e10597486.png)  

 **3. N-Well and P-Well Fabrication** = Fabricate the substrate needed by PMOS (N-Well) and NMOS (P-Well)  
  - Phosporus (5 valence electron) is used to form N-well  
  - Boron (3 valence electron) is used to form P-Well.  
  - Mask 2 protects the N-Well (PMOS side) while P-Well (NMOS side) is being fabricated then Mask 3 while N-Well (PMOS side) is being fabricated
   
![image](https://user-images.githubusercontent.com/87559347/187099587-4a837f08-b6d3-4cb9-afe6-75ee8d88cfff.png) 

 **4. Formation of Gate** = Gate fabrication affects threshold voltage. Factors affecting threshold voltage includes:    
 
![image](https://user-images.githubusercontent.com/87559347/187111068-874f408a-d41b-4b16-a5f0-49edfced8926.png)

Main parameters are:
  - Doping Concentration = Controlled by ion implantation (Mask 4 for Boron implantation in NMOS P-Well and Mask 5 for Arsenic implantation in PMOS N-Well)
  - Oxide capacitance = Controlled by oxide thickness  (SiO2 layer is removed then rebuilt to the desire thickness)  
  
 Mask 6 is for gate formation using polysilicon layer.
 
![image](https://user-images.githubusercontent.com/87559347/187116601-0ac34212-3622-4719-9309-fca887ad995a.png)
**5. Lightly Doped Drain formation** = Before forming the source and drain layer, lightly doped impurity is added: 
 - Mask 7 for N- implantation (lightly doped N-type) for NMOS 
 - Mask 8 for P- implantation (lightly doped P-type) for PMOS.  
Heavily doped impurity (N+ for NMOS and P+ for PMOS) is for the actual source and drain but the lightly doped impurity will help maintain spacing between the source and drain and prevent hot electron effect and short channel effect. 

![image](https://user-images.githubusercontent.com/87559347/187121868-94dfade0-2c63-4c9c-afef-942ef9662d5a.png)
**6. Source and Drain Formation** = Mask 9 is for N+ implantation and Mask 10 for P+ implantation  
 - Channeling is when implantations dig too deep into substrate so add screen oxide before implantation
 - The side-wall spacers maintains the N-/P- while implanting the N+/P+    
 
![image](https://user-images.githubusercontent.com/87559347/187128442-76d48790-53a0-4ad2-9856-924f3efd33eb.png)

**7. Form Contacts and Interconnects** =  TiN is for local interconnections and also for bringing contacts to the top. TiS2 is for the contact to the actual Drain-Gate-Source. Mask 11 is for etching off the TiN interconnect for the first layer contact. 

![image](https://user-images.githubusercontent.com/87559347/187141267-b043152d-0a76-4101-90ec-82c9adcc64e2.png)

**8. Higher Level Metal Formation** = We need to planarize first the layer via CMP before adding a metal interconnect. Aluminum contact is used to connect the lower contact to higher metal layer. Process is repeated until the contact reached the outermost layer.
 - Mask 12 is for first contact hole
 - Mask 13 is for first Aluminum contact layer
 - Mask 14 is for second contact hole
 - Mask 15 is for second Aluminum contact layer. Mask 16 is for making contact to topmost layer. 
 
![image](https://user-images.githubusercontent.com/87559347/187158161-4d230654-5102-4225-8e58-d6d8ed950990.png)

### Layout and Metal Layers

When polysilicon crosses N-diffusion/P-diffusion (diffusion is also called implantation), then an NMOS/PMOS is created. [Explained here](https://electronics.stackexchange.com/questions/223973/why-diffusions-in-cmos-cad-tool-magic-is-continuous) is the reason why the diffusion layer of source and drain "seems" to be connected under the polysilicon (diffusion layer for source and drain supposedly be separated).

[Chip design stick diagram](http://www.southampton.ac.uk/~bim/notes/cad/guides/sticks.html)  

The first layer is local-interconnect layer or local-i then metal 1 to 5. [Here is the process stack diagram](https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html) of sky130nm PDK. Metal 1 is for Power and Ground lines. `Nsubstratecontact` connects the N-well to locali. `licon` connects the locali to metal1.Locali is for local connections of cells

[Great guide on layout using Magic.](https://www.youtube.com/watch?v=RPppaGdjbj0)


[LEF (Library Exchange Format)](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) = A LEF file is used by the router tool in PnR design to get the location of standard cells pins to route them properly. So it is basically the abstract form of layout of a standard cell. `picorv32a/runs/[DATE]/tmp` contains the merged lef files (cell LEF and tech LEF). Notice how metal layer directon (horizontal or vertical) is alternating. Also, metal layer width and thickness is increasing. 

## Magic Commands  
- Left click = lower-left corner of box  
- Right click = upper-right corner of box  
- :box = display parameters of selected box  
- :grid 0.5um 0.5um = turn on/off and set grid   
- :snap user = snap based on current grid  
- :help snap = display help for command  
- :drc style drc(full)
- "z" = zoom in, "Z" = zoom out, "ctrl + z" = zoom into the box 
- :paint poly = paint "poly" to current box
- :drc why = show drc violation inside selected area (white dots are DRC violations )
- :erase poly = delete poly inside the box
- middle click on empty area will turn the box into empty (similar to erasing it)
- :select area = select all geometries inside the box
- :copy n 30 = copy selected geometries to North by 30 grid steps
`drc why` will show DRC violation and also the DRC name which can be referenced from [Sky130 PDK Periphery Rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root).

![image](https://user-images.githubusercontent.com/87559347/187588800-f083e5a5-2f22-4670-8a69-93d222794d27.png)

## (ON FORWARD ARE LABS ONLY STARTING FROM DAY 3 SK1 L4)  
#### Task for the Day 3 Lab: Modify a sample cell (inverter) and insert it to OpenLANE flow  

## Steps for the Lab:

1. Clone [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign). Copy the techfile `sky130A.tech` from `pdks/sky130A/libs.tech/magic/` to directory of the cloned repo. Below are the contents of `vsdstdcelldesign/libs/`:
![image](https://user-images.githubusercontent.com/87559347/187333491-7fd10850-9f8a-486d-9d4c-a93f4002fdea.png)


2. View the mag file using magic `magic -T sky130A.tech sky130_inv.mag &`:  

![image](https://user-images.githubusercontent.com/87559347/183270193-c3e58fcd-951a-4d29-8856-921de11e7903.png)

3. Make an extract file `.ext` by typing `extract all` in the tkon terminal. 
4. Extract the `.spice` file from this ext file by typing `ext2spice cthresh 0 rthresh 0` then `ext2spice` in the tcon terminal.  

![image](https://user-images.githubusercontent.com/87559347/183270366-f5a70661-b0d3-4b4e-b72d-5831b6ff6da3.png)

We then modify the spice file to be able to plot a transient response:

```
** SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/nshort.lib
.include ./libs/pshort.lib

//.subckt sky130_inv A Y VPWR VGND

M0 Y A VGND VGND nshort_model.0 ad=0 pd=0 as=0 ps=0 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=0 pd=0 as=0 ps=0 w=37 l=23
VDD VPWR 0 3.3V
VSS VGND 0 0V
Va A VGND PULSE(0V 3.3V 0 0.1n 0.1n 2ns 4ns)
C0 Y A 0.05fF
C1 VPWR A 0.07fF
C2 Y VPWR 0.11fF
C3 Y VGND 2fF
C4 VPWR VGND 0.59fF
//.ends
.tran 1n 20ns
.control
run
.endc
.end
```  

Open the spice file by typing `ngspice sky130A_inv.spice`. Generate a graph using `plot y vs time a` :  

![image](https://user-images.githubusercontent.com/87559347/183271057-ef99f8f2-5c76-49ac-a4a4-d425a41f6cf5.png)


# DAY 4: Pre-layout timing analysis and importance of good clock tree

To run previous flow, add tag to prep design:
```
prep -design picorv32a -tag [date]
```


Now that we have the layout of the cell, we need to make sure that: 
 - The input and output ports lies on the intersection of the horizontal and vertical tracks (ensure the routes can reach that ports). 
 - The width of the standard cell must be odd multiple of the tracks horizontal pitch and height must be odd multiples of tracks vertical pitch (this is a PnR requirement).  
 To check these requirements, we need to change the grid of the magic to match the real tracks. The `pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info` contains those information.   

Use `grid` command inside the tkon terminal to match the above needed settings:  

![image](https://user-images.githubusercontent.com/87559347/183272925-3e0fb501-33e0-4cb8-9863-6c55f965b1d5.png)\

Below, we can see that the requirement is satisfied:  

![image](https://user-images.githubusercontent.com/87559347/183273195-485b64e0-fbb4-4c2b-85bf-6e578f7cc5df.png)

Next, we will extract the lef file. But before that I saved first the magfile with a new file name `save sky130_inv_new.mag`. Then type `lef write` on the tcon terminal. It will generate a lef file with same name as the magfile: `sky130_inv_new.lef`. Looking inside the lef file is:  

![image](https://user-images.githubusercontent.com/87559347/183273517-743f4b0a-cbe7-47f0-a473-74d105d878b2.png)

## Steps for plugging in the customized cell to OpenLANE

1. Copy the new lef file `sky130_inv_new.lef` and the libraries `sky130*.lib` inside the `/openlane/vsdstdcelldesign/libs` to the src directory of picorv32a.  

![image](https://user-images.githubusercontent.com/87559347/183274773-9a56636f-f24f-4596-a831-3c2cd477a9f7.png)

2. Add the folowing to `config.tcl` inside the picorv32a:    
```  
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

The whole `config.tcl` is:  

![image](https://user-images.githubusercontent.com/87559347/183275069-8400ccfa-5b1f-471f-993d-d58c47601fe3.png)


2. Run docker and prepare the design. Then before running synthesis, plug first the new lef file to the OpenLANE flow via:  

```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```  

Below is the statistics report, and as we can see that `sky130_vsdinv` is really used on the design!  

![image](https://user-images.githubusercontent.com/87559347/183275187-20f03e48-1790-4d37-b463-f58726dacc9d.png)

However, notice that we are failing timing significantly,  

![image](https://user-images.githubusercontent.com/87559347/183279413-1f9f4512-cb41-4e90-a611-19bea86ff6f0.png)



### About Delay Table
Delay tables are used to find delay of each cell. In order to avoid large skew (signal arrives at different point in time), buffers on the same level must have same capacitive load to ensure same timing delay or latency on the same level. Same level buffers must also be the same size (since different buffer sizes pertains to different RC constant thus different delay and delay table).

Let us change some variables to minimize the negative slack. Use `echo $::env(SYNTH_STRATEGY) to view the current variables. The following are changed:



```
echo $::env(SYNTH_STRATEGY)
AREA 0
set ::env(SYNTH_STRATEGY) "DELAY 0"
echo $::env(SYNTH_BUFFERING)
1
echo $::env(SYNTH_SIZING)
0
set ::env(SYNTH_SIZING) 1
```  

Below is the area and also the negative slack. The area becomes bigger but slack is reduced to zero!  

![image](https://user-images.githubusercontent.com/87559347/183282096-63749738-c6e8-453e-8e1e-3858db8fd567.png)

Next, we do `run_floorplan` then check on magic the output layout BUT:  

![image](https://user-images.githubusercontent.com/87559347/183292764-6684d384-2974-4069-b647-93121db3e496.png)

Wth the help from the course tutors, the solution is to follow a manual approach to floorplan instead of using the wrapper `run_floorplan`. Below is the sequence of commands. :
```
init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement
gen_pdn
run_routing
```

# DAY 5: Final steps for RTL2GDS using tritonRoute and openSTA

Wth the help from the course tutors, the error yesterday is found to be due to the newer version of OpenLANE. For an error-free flow, use the below sequence of commands. :
```
init_floorplan
place_io
global_placement_or
detailed_placement
tap_decap_or
detailed_placement
gen_pdn
run_routing
```
Today we will do generation of power distribution network. After running `gen_pdn`, we can verify that the def file for PDN is generated by echoing `CURRENT_DEF`:

![image](https://user-images.githubusercontent.com/87559347/183293313-adc46bad-0e2e-4bcf-a2fb-2e2e5b6a43b7.png)

Next is routing. One simple algorithm for routing is Maze Routing or Lee's routing. The shortest path is one that follows a steady increment of one (1-to-12 on the example below). There might be multiple path like this but the best path that the tool will choose is one with less bends. 

![image](https://user-images.githubusercontent.com/87559347/183368014-e7f0e3d1-c968-42ea-8e4c-f8e965c6b748.png)

OpenLANE routing stage consists of two stages:

 - Global Routing - Makes the general guide that will route all pins to its destination. The tool used is FastRoute
 - Detailed Routing - Uses the global routing's general guide to actually connect the pins with least amount of wire. The tool used is TritonRoute.

DRC checking is due to the constrains of the photolitographic machine for chip fabrication:. Some DRC are:
1. The minimum wirewidth
2. Minimum wire pitch
3. Minimum wire spacing


Now, we will finally do the routing, simply run `run_routing`. After approximately 15 minutes, the output is:
![image](https://user-images.githubusercontent.com/87559347/183294065-92c9541d-e300-4e83-ae4e-bdd3ce252af4.png)

Now if we go to`openlane/designs/picorv32a/runs/07-08_08-07/results/routing`, we will see the following files:  

![image](https://user-images.githubusercontent.com/87559347/183294232-ec494cb7-d05d-4476-90e2-d88968574d3f.png)

Notice how the def file is named simply as `picrov32a`. This is because this is the final def file in the whole RTL2GDSII flow. Let us open that def read at magic using:
```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def
```
![image](https://user-images.githubusercontent.com/87559347/183294391-13c987c5-33fa-4761-ba82-396c12749aab.png)

Zooming in:  

![image](https://user-images.githubusercontent.com/87559347/183294992-8d197684-99e7-42d7-b10f-9105cb5b1761.png)
 

### DONE! CONGRATULATIONS!

# Acknowledgements
 - [Nickson Jose - Workshop Instructor](https://www.udemy.com/user/nickson-jose/)
 - [Kunal Ghosh - Co-founder of VSD](https://www.udemy.com/user/anagha/)
 
 
# Inquiries  
Connect with me at my linkedin: https://www.linkedin.com/in/angelo-jacobo/
