# OpenLANE-Sky130-Physical-Design-Workshop

# MY NOTES
## Day 1:

The core of the chip will contain two type of blocks:
 - Foundry IP Blocks (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - Macro blocks (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 
 
 ![image](https://user-images.githubusercontent.com/87559347/182751377-2810d388-21b0-4df1-b1d4-c72176d80d28.png)

Open Source Digital ASIC Design requires three opensourced components:  
- RTL Designs = github.com, librecores.org, opencores.org
- EDA Tools = OpenROAD, OpenLANE,QFlow  
- PDK = Google + Skywater 130nm Production PDK

PDK (Process Design Kit) = A set of data files and documents which serves as the interface between the designer and the fab. Cell libraries, IO libraries, design rules (DRC, LVS, etc.)

### Simplified RTl to GDSII Flow:
- Sythesis = The RTL is converted into a gate level netlist made up of components of standard cell libary. 
- Floor Planning/ Power Planning = Plan silicon area and create robust power distribution network. The power network usually uses the upper metal layer which are thicker than lower layer and thus lower resistance. This lowers the IR drop problem
 - Placement = There are two steps, first is global placement which is the general optimal positons for cells and might not be legal. Next is detailed placement which is the actual legal placements of the cells.
 - Clock tree synthesis = clock distribution to all flip flops and is usually a tree (H-tree, X-tree ... )
 - Routing = Use horizontal and vertical wires to connect cells together. The router uses PDK information (thickness, pitch, width,vias) for each metal layer to do the routing. The Sky130 defines 6 routing layers. It doe global routing and detailed routing.
 - Verification before sign-off = Involves physical verification like DRC and LVS and timing verification. Design Rule Checking or DRC ensures final layout honors all design rules and Layout versus Schematic or LVS ensures final layout matches the gate level netlist from synthesis phase. Timing verification ensures timing constraints are met.  
 The final layout is in GDSII file format.
 
 [OpenLANE](https://github.com/The-OpenROAD-Project/OpenLane) = Opensourced ASIC developemnt flow rerference. It consists of multiple opensourced tools needed for the whole RTL to GDSII flow. This is tuned epecially for Sky130 PDK. It also works for OSU 130nm. It is recommended to read the OpenLANE documentation before going forward.
 ![image](https://user-images.githubusercontent.com/87559347/182759711-6b9352ec-7652-4589-af31-53a409eb2830.png)

### Notable details are:  
- Yosys is used to convert the HDL to gate level netlist using generic components. The [ABC script](http://people.eecs.berkeley.edu/~alanmi/abc/) is then used to map the generic components to the standard cell library of the PDK. These ABC scripts is used to make various synthesis strategies (using the Synthesis Exploration) which can optimize the design either with least area or best timing.  

- The OpenROAD application automates Place and Route  

- The Logic Equivalency CHceking (LEC) is used to compare the resulting netlist after optimization of place and route to the gate level netlist from synthesis phase
Antenna Rules Violation = long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to eithre use bridging or antenna diode insertion to leak away the charges  

### Notable Directories:

``` 
├── openlane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the PDK (which is normally just compatible to commercial tools) to work with the opensource EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for opensource tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgenmagic...) 
```

Inside a specfic design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority level for the OpenLANE settings:
1. Default values
2. config.tcl
3. sky130_xxxxx_config.tcl

### Task for the Day 1 Lab: Find the flipflop ratio percentage for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells

## Steps:
1. Run OpenLANE. Below are the commands used in sequence:
 - `$ docker` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in `-interactive` mode
 - `% package require openlane 0.9` == retrives all dependecies for running v0.9 of OpenLANE  
 
 ![image](https://user-images.githubusercontent.com/87559347/182833010-c5b32449-bfa1-42d0-8433-9edfdefbf1f6.png)

 
2. Design Setup Stage.Command used is:
 - `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This merges the cell lef files `.lef` and technology lef files `.tlef` producing `merged.lef`. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage.
 

![image](https://user-images.githubusercontent.com/87559347/182833339-f117de40-af1f-4607-9c47-81b3ae5f7b7e.png)

3. Run synthesis. Command used is:
 - `% run_synthesis` = Run yosys synthesis, ABC scripts, and OpenSTA.  
 
![image](https://user-images.githubusercontent.com/87559347/182847715-b0da0f41-b444-4036-9ca3-c5335bfd42cf.png)

The flipflop ratio percentage (number of flip flops)/(total number of cells) is 1613/14876 = 10.843%

After running synthesis, inside the `runs/[date]/results/synthesis` is `picorv32a_synthesis.v` which is the mapping of the netlist to standard cell library using ABC. The `runs/[date]/reports/synthesis` will contain synthesis statistic reports and static timing analysis reports. The `runs/[date]/synthesis/logs` contains log files for the terminal output dumps for running yosys and OpenSTA.

![image](https://user-images.githubusercontent.com/87559347/182874085-12a7d8b8-d095-4a18-b9ab-101050473046.png)

