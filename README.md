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



## Steps for the Lab:
#### Task for the Day 1 Lab: Find the flipflop ratio percentage for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells  

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
 


## Steps for the Lab:
#### Task for the Day 2 Lab: Find the area of the die  

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


The first layer is local-interconnect layer or local-i then metal 1 to 5. [Here is the process stack diagram](https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html) of sky130nm PDK. Metal 1 is for Power and Ground lines. `Nsubstratecontact` connects the N-well to locali. `licon` connects the locali to metal1.Locali is for local connections of cells. 

The layer hierarchy for NMOS is: Psubstrate -> Psubstrate Diffusion (psd) -> Psubstrate Contact (psc) -> Local-interconnect (li) -> Mcon -> Metal1. For poly: Poly -> Polycontact -> Locali. P-substrate diffusion an N-substrate diffusion is also referred to as P-tap and N-tap. 

The output of the layout is the LEF file. [LEF (Library Exchange Format)](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) is used by the router tool in PnR design to get the location of standard cells pins to route them properly. So it is basically the abstract form of layout of a standard cell. `picorv32a/runs/[DATE]/tmp` contains the merged lef files (cell LEF and tech LEF). Notice how metal layer directon (horizontal or vertical) is alternating. Also, metal layer width and thickness is increasing. 

## Magic Commands  
[Here is a great video guide](https://www.youtube.com/watch?v=RPppaGdjbj0) on layout using Magic. And [here is the Magic website](http://opencircuitdesign.com/magic/) with turorials.
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
- :move n 1 = move selected geometries to North by 1 step ("." to move more, "u" to undo)
- "s" three times will select all geometries electrically connected to each other    

`drc why` will show DRC violation and also the DRC name which can be referenced from [Sky130 PDK Periphery Rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root).

![image](https://user-images.githubusercontent.com/87559347/187588800-f083e5a5-2f22-4670-8a69-93d222794d27.png)

## (ON FORWARD ARE LABS ONLY STARTING FROM DAY 3 SK1 L4)  

## Steps for the Lab Part 1:
#### Task for the Day 3 Lab Part 1: Characterize a sample inverter cell by its slew rate and propagation delay.  

1. Clone [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign). Copy the techfile `sky130A.tech` from `pdks/sky130A/libs.tech/magic/` to directory of the cloned repo. Below are the contents of `vsdstdcelldesign/libs/`:
![image](https://user-images.githubusercontent.com/87559347/187333491-7fd10850-9f8a-486d-9d4c-a93f4002fdea.png)


2. View the mag file using magic `magic -T sky130A.tech sky130_inv.mag &`:  

![image](https://user-images.githubusercontent.com/87559347/183270193-c3e58fcd-951a-4d29-8856-921de11e7903.png)

3. Make an extract file `.ext` by typing `extract all` in the tkon terminal. 
4. Extract the `.spice` file from this ext file by typing `ext2spice cthresh 0 rthresh 0` then `ext2spice` in the tcon terminal.  

![image](https://user-images.githubusercontent.com/87559347/188252410-0f71a30b-b05e-4ddb-9d95-2fd40712825d.png)

We then modify the spice file to be able to plot a transient response:

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

* .subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
C0 A VPWR 0.08fF
C1 Y VPWR 0.08fF
C2 A Y 0.02fF
C3 Y VGND 0.18fF
C4 VPWR VGND 0.74fF
* .ends

* Power supply 
VDD VPWR 0 3.3V 
VSS VGND 0 0V 

* Input Signal
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

* Simulation Control
.tran 1n 20n
.control
run
.endc
.end
```  

Open the spice file by typing `ngspice sky130A_inv.spice`. Generate a graph using `plot y vs time a` :  

![image](https://user-images.githubusercontent.com/87559347/183271057-ef99f8f2-5c76-49ac-a4a4-d425a41f6cf5.png)

Using this transient response, we will now characterize the cell's slew rate and propagation delay:  
- Rise Transition [output transition time from 20%(0.66V) to 80%(2.64V)]:
    - **Tr_r = 2.19981ns - 2.15739ns = 0.04242 ns**  
![image](https://user-images.githubusercontent.com/87559347/188260029-84633ed7-446e-4d1b-b723-12397dcfc71a.png)  


- Fall Transition [output transition time from 80%(2.64V) to 20%(0.66V)]:
   - **Tr_f = 4.0672ns - 4.04007ns = 0.02713ns**   
![image](https://user-images.githubusercontent.com/87559347/188260236-4cc5d4c7-654a-4600-a277-9f6c1df63b11.png)


- Rise Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_r = 2.18197ns - 2.15003ns = 0.03194ns**   
![image](https://user-images.githubusercontent.com/87559347/188261194-395c7cfd-caea-4efa-a670-310cb30ff6a2.png)


- Fall Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_f = 4.05364ns - 4.05001ns =0.00363ns**  
![image](https://user-images.githubusercontent.com/87559347/188261518-792d3e99-6a5a-423d-9309-62287c608ec0.png)


## Steps for the Lab Part 2:
#### Task for the Day 3 Lab Part 2: Use Magic to fix incorrect DRCs of the tech file    

Read through [this site about tech file](http://opencircuitdesign.com/magic/techref/maint2.html). All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types, design rules, rules for mask generation, and rules for extracting netlists for circuit simulation. 
Read through also [this site on the DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)

1. Download the [lab contents from this site](opencircuitdesign.com/open_pdks/archive/drc_tests.tgz). Extract the tarball. Inside the `drc_tests/` are the `.mag` layout files and the `sky130A.tech`.  

2. Open magic with `poly.mag` as input: `magic poly.mag`. Focus on `Incorrect poly.9` layout. As described on the poly.9 [design rule of SKY130 PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly), the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using `:box`, we can see that the distance is 0.250um YET there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.  
![image](https://user-images.githubusercontent.com/87559347/188370620-7e802ce0-cd15-4385-9b73-d8f5ee5fe8ae.png)

3. Open `sky130A.tech`. The included rules for poly.9 are only for the spacing between the n-poly resistor with n-diffusion and the spacing between the p-poly resistor with diffusion. We will now add new rules for the spacing between the **poly resistor with poly non-resistor**, highlighted green below are the two added rules. On the left is the rule for spacing between n-poly resistor with poly non-resistor and on the right is the rule for the spacing between the p-poly resistor with poly non-resistor. The `allpolynonres` is a macro under `alias` section of techfile. 
![image](https://user-images.githubusercontent.com/87559347/188374444-4999b439-40ab-42ae-91cd-91017c217f3e.png)

4. Run `tech load sky130A.tech` then `drc check` in tkcon to reload the tech file. The new DRC rules will now take effect.Notice the white dots on the poly indicating the design rule violations. Command `drc find` to iterate in each violations.  
![image](https://user-images.githubusercontent.com/87559347/188373919-e9d1bd08-7c50-400a-9a17-65fa4296c82e.png)

5. Next, notice below that there are violations between N-substrate diffusion with the polyresistors (from left: npolyres, ppolyres, xpolyres) which is good. But between npolyres with P-substrate diffusion, there is no violation shown. 
![image](https://user-images.githubusercontent.com/87559347/188421029-0f94f6c8-8fc7-4aab-b895-05de5de40f7c.png)

6. To fix that, just modify the tech file to include not only the spacing between npolyres with N-substrate diffusion in poly.9 but between **npolyres and all types of diffusion**. `alldif` is also a macro under `alias` section. Load the tech file again, the new DRC will now take effect.  
![image](https://user-images.githubusercontent.com/87559347/188384339-225f2a84-8aca-44c6-b742-272448051fc9.png)  
![image](https://user-images.githubusercontent.com/87559347/188421488-3d84c048-06b3-46ac-9816-513dd7c721f2.png)


# DAY 4: Pre-layout timing analysis and importance of good clock tree

To run previous flow, add tag to prep design:
```
prep -design picorv32a -tag [date]
```
## Extracting the LEF File
PnR tool does not need all informations from the `.mag` file like the logic part but only PnR boundaries, power/ground ports, and input/output ports. This is what a [LEF file](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) actually contains. So the next step is to extract the LEF file from Magic. But first, we need to follow guidelines of the PnR tool for the standard cells:
 - The input and output ports lies on the intersection of the horizontal and vertical tracks (ensure the routes can reach that ports). 
 - The width of the standard cell must be odd multiple of the tracks horizontal pitch and height must be odd multiples of tracks vertical pitch   
 
 To check these guidelines, we need to change the grid of Magic to match the actual metal tracks. The `pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd/tracks.info` contains those metal informations.   

1. Use `grid` command inside the tkon terminal to match the tracks informations:  

![image](https://user-images.githubusercontent.com/87559347/188419121-ce050fc7-6984-4266-9b24-47002934fc83.png)

The grids show where the routing for the local-interconnet layer can only happen, the distance of the grid lines are the required pitch of the wire. Below, we can see that the guidelines are satisfied:  

![image](https://user-images.githubusercontent.com/87559347/183273195-485b64e0-fbb4-4c2b-85bf-6e578f7cc5df.png)

2. Next, we will extract the LEF file. The LEF file contains the cell size, port definitions, and properties which aid the placer and router tool. With that, the ports definition, port class, and port use must be set first. The instructions to set these definitions via Magic are on the [vsdstdcelldesign repo](https://github.com/nickson-jose/vsdstdcelldesign#create-port-definition). 

3. Next, save the mag file with a new filename `save sky130_myinverter.mag`. Then type `lef write` on the tcon terminal. It will generate a LEF file with same name as the magfile `sky130_myinverter.lef`. Inside that LEF file is:  

![image](https://user-images.githubusercontent.com/87559347/188555080-03e4d472-9dcd-4c46-b0f0-7a37c952e5c3.png)

## Plug-in the Customized Inverter Cell to OpenLANE

Inside `pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/` are the [liberty timing files](https://teamvlsi.com/2020/05/lib-and-lef-file-in-asic-design.html) for SKY130 PDK which contains the timing and power parameters for each cell needed in STA. It can either be slow, typical, fast with different different supply voltages (1v80, 1v65, 1v95, etc.). These are the so called [PVT corners](https://chipedge.com/what-are-pvt-corners-in-vlsi/). The library name `sky130_fd_sc_hd__ss_025C_1v80` describes the PVT corner as slow-slow (delay is maximum), 25° Celsius temperature, at 1.8V power supply. Timing and power parameter of a cell is obtained by simulating the cell in a variety of operating conditions (different corners) and these data are represented in the liberty file. 

The liberty file characterizes all cells and is used by the ABC script during synthesis stage which maps the generic cells to the actual standard cells available in the liberty file.  Same cell functionality but different sizes are available inside the library. As shown below, both are inverter cells but one with size-1 and another with size-4. Notice how size-4 inverter is simply four instances of size-1 inverter:

![image](https://user-images.githubusercontent.com/87559347/188778191-1ca86454-98eb-4f95-8e1d-e5276a4edc40.png)


Provided inside the cloned `vsdstdcelldesign` are the liberty files containing the customized inverter cell.

1. Copy the extracted lef file `sky130_myinverter.lef` and the liberty files `sky130*.lib` from `/openlane/vsdstdcelldesign/libs` to the src directory of picorv32a. Open each liberty files then change the cell name `sky130_vsdinv` to `sky130_myinverter` to match the new LEF file cell name.

![image](https://user-images.githubusercontent.com/87559347/188646711-c1715a14-7b55-40d5-90d0-cc1344577d3f.png)

2. Add the folowing to `config.tcl` inside the picorv32a:    
```  
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

This sets the liberty file that will be used for ABC mapping of synthesis (`LIB_SYNTH`) and for STA (`_FASTEST`,`_SLOWEST`,`_TYPICAL`) and also the extra LEF files (`EXTRA_LEFS`) for the customized inverter cell. The whole `config.tcl` then is:  

![image](https://user-images.githubusercontent.com/87559347/188798219-0b11e661-97f9-4960-b3a7-39aa43c19281.png)


3. Run docker and prepare the design picorv32a. Plug the new lef file to the OpenLANE flow via:  

```
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
```  

4. Next `run_synthesis`. Below is the synthesis statistics report `runs/[date]/reports/synthesis/1-synthesis.AREA_0.stat.rpt` after the run, and as we can see `sky130_myinverter` cell is successfully included in the design!  

![image](https://user-images.githubusercontent.com/87559347/188657588-5686cf61-4978-4842-bbf6-b0c01b111c12.png)

HOWEVER, looking  at the STA log `runs/[date]/logs/synthesis/sta.log`, we are not meeting timing. Our neext goal is to solve this negative slack:

![image](https://user-images.githubusercontent.com/87559347/188801207-1972f34d-80cd-4178-ae3c-e8e49bc0d841.png)



### About Delay Table  

In order to avoid large skew between endpoints of a clock tree (signal arrives at different point in time):
 - Buffers on the same level must have same capacitive load to ensure same timing delay or latency on the same level. 
 - Buffers on the same level must also be the same size (different buffer sizes -> different W/L ratio -> different resistance -> different RC constant -> different delay).    
 
 ![image](https://user-images.githubusercontent.com/87559347/188773408-e503023f-0288-4993-a68a-5f20bccb886c.png)


Buffers on different level will have different capacitive load and buffer size but as long as they are the same load and size on the same level, the total delay for each clock tree path will be the same thus skew will remain zero. **This means different levels will have varying input transition and output capacitive load and thus varying delay.** 

Delay tables are used to capture the timing model of each cell and is included inside the liberty file. The main factor in delay is the output slew. The output slew in turn depends on **capacitive load** and **input slew**. The input slew is a function of previous buffer's output cap load and input slew and it also has its own transition delay table.

![image](https://user-images.githubusercontent.com/87559347/188783693-423bd170-dd0b-4f2f-9652-8fae9418df31.png)

Notice how skew is zero since delay for both clock path is x9'+y15.

### Fix Negative Slack

Let us change some variables to minimize the negative slack. We will now change the variables "on the flight". Use `echo $::env(SYNTH_STRATEGY)` to view the current value of the variables before changing it:

```
% echo $::env(SYNTH_STRATEGY)
AREA 0
% set ::env(SYNTH_STRATEGY) "DELAY 0"
% echo $::env(SYNTH_BUFFERING)
1
% echo $::env(SYNTH_SIZING)
0
% set ::env(SYNTH_SIZING) 1
% echo $::env(SYNTH_DRIVING_CELL)
sky130_fd_sc_hd__inv_2
```  
With `SYNTH_STRATEGY` of `Delay 0`, the tool will focus more on optimizing/minimizing the delay, index can be 0 to 3 where 3 is the most optimized for timing (sacrificing more area). `SYNTH_BUFFERING` of 1 ensures cell buffer will be used on high fanout cells to reduce delay due to high capacitance load. `SYNTH_SIZING` of 1 will enable cell sizing where cell will be upsize or downsized as needed to meet timing. `SYNTH_DRIVING_CELL` is the cell used to drive the input ports and is vital for cells with a lot of fan-outs since it needs higher drive strength (larger driving cell needed).

Below is the log report for slack and area. The area becomes bigger (from 98492 to 103364) but no negative slack anymore (from -1.2ns to +0.35ns)!  

![image](https://user-images.githubusercontent.com/87559347/189464181-d8649d12-e4ef-4cb6-afab-8a305787dd72.png)

Next, we do `run_floorplan` then BUT:  

![image](https://user-images.githubusercontent.com/87559347/189466107-b3c13af9-d01b-4033-8d9c-f83518c69ab8.png)  

The solution for this error is found on [this issue thread](https://github.com/The-OpenROAD-Project/OpenLane/issues/1307). `basic_macro_placement` command is failing since `EXTRA_LEFS` variable inside `config.tcl` is assumed as a macro which is not. The temporary solution is to comment call on `basic_macro_placement` inside the `OpenLane/scripts/tcl_commands/floorplan.tcl` (this is okay since we are not adding any macro to the design). 

After that `run_placement`, another error will occur relating to `remove_buffers`, the solution is to comment the call to `remove_buffers_from_nets` in `OpenLane/scripts/tcl_commands/placement.tcl`. After successfully running placement, `runs/[date]/results/placement/picorv32.def` will be created. Search for instance of cell `sky130_myinverter` by grep `cat picorv32.def | grep sky130_myinverter`:  
![image](https://user-images.githubusercontent.com/87559347/189475352-f74731e2-6ef8-4620-a3a9-16c31d326c82.png)


Open the def file via magic: `magic -T /home/angelo/Desktop/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read picorv32.def &`. Select a single sky130_myinverter cell from listed above by commanding on tkon `% select cell _07237_` then ctrl+z to zoom into that cell. As shown below, our customized inverter cell sky130_myinverter is sucessfully placed on the core. Use `expand` on tkon to show the footprint of the cell and notice how the power and ground of sky130_myinverter overlaps the power and ground pins of its adjacent cells.  

![image](https://user-images.githubusercontent.com/87559347/189507538-6d0c1b46-dc45-4768-b00c-01e8336ab518.png)



### Timing Analysis (Pre-Layout STA using Ideal Clocks)
Pre-layout STA will not yet include effects of clock buffers and net-delay due to RC parasitics (wire delay will be derived from PDK library wire model).    
![image](https://user-images.githubusercontent.com/87559347/189510818-050c6b22-a319-4969-a23e-c82c57ebd4ff.png)  

Setup timing analysis equation is:  
```
Θ < T - S - SU
```  

- Θ =  Combinational delay which includes clk to Q delay of launch flop and internal propagation delay of all gates between launch and capture flop  
- T = Time period, also called the required time
- S = Setup time. As demonstrated below, signal must settle on the middle (input of Mux 2) before clock tansists to 1 so the delay due to Mux 1 must be considered, this delay is the setup time. 
![image](https://user-images.githubusercontent.com/87559347/189511212-8e1ea86f-b2d6-4a68-9948-7d9999087886.png)
- SU = Setup uncertainty due to jitter which is temporary variation of clock period. This is due to non-idealities of PLL/clock source.

### Pre-Layout STA with OpenSTA
STA can either be **single corner** which only uses the `LIB_TYPICAL` library which is the one used in pre-layout(pos-synthesis) STA or **multicorner** which uses `LIB_SLOWEST`(setup analysis, high temp low voltage),`LIB_FASTEST`(hold analysis, low temp high voltage), and `LIB_TYPICAL` libraries. 

Run STA engine using openroad, run openroad first then source `/openlane/scripts/openroad/sta.tcl` which contains the commands for single corner STA. This file also contains the path to the [SDC file](https://teamvlsi.com/2020/05/sdc-synopsys-design-constraint-file-in.html) which specifies the timing constraints of the design. 
![image](https://user-images.githubusercontent.com/87559347/189568030-f442a238-21e8-4fc1-b5d0-22de00b11af9.png)

The result of running STA in openroad will be exactly the same as the log result of STA after running `run_synthesis`. Observe the delay:
![image](https://user-images.githubusercontent.com/87559347/189686801-46a9fb96-9be6-40c7-b62a-da3160489cb0.png)

To reduce negative slack, focus on large delays. Notice how net `_02682_` has big fanout of 5. Use `report_net -connections _02682_` to display connections. First thing we can do is to go back to openlane and reduce fanouts by `set ::env(SYNTH_MAX_FANOUT) 4` then `run_synthesis` again. As shown below, wns is reduced from -1.35ns to -0.82ns.  
![image](https://user-images.githubusercontent.com/87559347/189788023-9f6d85a9-a769-4b54-b156-2fa7b8980178.png)

To further reduce the negative slack, we can also try changing the sizes of cell with high fanout so bigger driver will be used. As shown below, cell `_41882_` has a high cap load of 0.04nF and this causes a large delay due to `buf_1` not having enough drive strength to drive that high cap load. We can try upsizing the `buf_1` to `buf_4` (listed on the used liberty files are all cells which you can choose) inside OpenSTA: `replace_cell _41882_ sky130_fd_sc_hd__buf_4` 
![image](https://user-images.githubusercontent.com/87559347/189793281-6acff965-b4d1-48a8-a6c3-17d312f901a2.png)
This can be done iteratively until desired slack is reached, this is called timing ECO (Engineering Change Order).  

#### Summary of OpenSTA Commands  
```
report_net -connections _02682_
report_checks -fields {cap slew nets} -digits 4
report_checks -from _18671_ -to _18739_ -fields {cap slew nets} -digits 4
report_wns
report_tns
```

### SDC File Parameters

- [create_clock](http://ebook.pldworld.com/_Semiconductors/Actel/Libero_v70_fusion_webhelp/create_clock_sdc_constraint.htm)
```
create_clock clk  -name sys_clk  -period 10
```
This creates a clock named `sys_clk` to the port `clk` with period of 10ns. However, it is recommended to add `get_ports` when referencing a port to not confuse the object type (is it a clk, net, or port?):
```
create_clock [get_ports clk]  -name sys_clk  -period 10
```

 - [set_input_delay](http://ebook.pldworld.com/_Semiconductors/Actel/Libero_v70_fusion_webhelp/set_input_delay_%28sdc_input_delay_constraint%29.htm)/[set_output_delay](https://www.intel.com/content/www/us/en/docs/programmable/683432/21-4/tcl_pkg_sdc_ver_1-5_cmd_set_output_delay.html) = Defines the arrival/exit time of an input/output signal relative to the input clock. This is the delay of the signal coming from an external block and internal delay of the signal to be propagated to external ports.
 
 ```
 set_input_delay 1 -clock [get_clock clk] [all_input]
 set_output_delay 0.5 -clock [get_clock clk] [all_output]
 ```
 This adds a delay of 1ns relative to `clk` to all signals going to input ports, and delay of 0.5ns relative to `clk` to all signals going to output ports.


 - [set_max_fanout](https://hdvacademy.blogspot.com/2014/07/design-constraints.html) = below constraint specifies a max fanout of 10 for all output ports in the design
 ```
 set_max_fanout 10 [current_design]
 ```
- [set_driving_cell](https://www.micro-ip.com/tw/STA/dictionary_516_17/set_driving_cell.html) = Models an external driver at the input port of the current design 

```
set_driving_cell -lib_cell sky130_fd_sc_hd__inv_2 -pin Y $all_inputs_wo_clk_rst
set_driving_cell -lib_cell sky130_fd_sc_hd__inv_8 -pin Y clk
```
The first constraint sets `sky130_fd_sc_hd__inv_2` (specifically pin `Y` of the cell) to drive all input ports except `clk`. The second consraint sets `sky130_fd_sc_hd__inv_8` (specifically pin 'Y' of this cell) for the driving the clk input port.


- set_load = Below constraint sets a 10nF capacitive load to all output ports. set_load can also be used on internal net.
```
set_load 10 [all_outputs]
```

- set_clock_uncertainty = Below constraint incorporates 0.25ns skew to `clk`
```
set_clock_uncertainty 0.25 [get_clock clk]
```

- [set_clock_transition](https://www.micro-ip.com/tw/Synopsys(DC)/dictionary_37_8/set_clock_transition.html) = Sets both rise and fall transition times to 0.15ns on clock pins of all sequential elements clocked by `clk`:
```
set_clock_transition 0.15 [get_clocks clk]

```
[Here](https://hdvacademy.blogspot.com/2014/07/design-constraints.html) and [here](https://www.micro-ip.com/tw/drchip.php?mode=2&cid=8) is a great reference for some common SDC constraints. As a side note, [as said here](https://electronics.stackexchange.com/questions/339401/get-ports-vs-get-pins-vs-get-nets-vs-get-registers) I/Os of the top-level block are called port while I/Os of the subblocks are called pin.

STA  
- no wire delay yet: clk-to-Q delay -> gates propagation delays -> D-input
jitter due to non-idealities of the PLL

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



Tech file `.tech` contains the metal layer, connectivity between layers, DRC rules, and other definitions needed by Magic layout tool to view a single cell.

LEF file is divided to tech lef which contains metal layer geometries and cell lef which contains geometries for all cells in the standard cell library. This lef file does not contain the logic part of cells, only the footprint that is needed by the PnR tool. 

DEF file is derived from LEF file and is used to transfer the design data from one EDA tool to another EDA tool and contains connectivty of cells of the design and is just a footprint (does not contains the logic part of cells) that the PnR needs. Each EDA tool to run will need to read first the LEF file `runs/[date]/tmp/merged.nom.lef` and the DEF file output of the previous stage's EDA tool.


As seen in delay table, delay depends on input slew and ouput load capatiance. To reduce delay, focus on large slew and cap. As seen below, higher fanouts means larger load caps thus larger slew
Load cap will not change but we can change cell size to better drive that large cap load for less delay

