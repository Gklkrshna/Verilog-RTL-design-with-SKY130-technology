# Verilog-RTL-design-with-SKY130-technology

![VSD workshop](https://user-images.githubusercontent.com/78468534/120083126-45241080-c0e4-11eb-9c1e-c970c2737ca1.jpeg)
VSD workshop - RTL design using Verilog with SKY130 Technology

This repo is the report for 5-day workshop on RTL design using Verilog with SKY130 Technology by VLSI sytem design.

## *Contents*
------------
* [Introduction to tools](#introduction-to-tools)
* [Day 1](#day-1)
  * [Simulation Flow](#simulation-flow)
  * [Synthesis Flow](#synthesis-flow)
* [Day 2](#day-2)
* [Day 3](#day-3)
* [Day 4](#day-4)
* [Day 5](#day-5)

---------
## Introduction to tools
* **_SKY130_** - Sky130 process node and pdk(process design kit) are an open-source packages provided by Google and skywater for 130nm technology.
* **_iverilog_** - Iverilog stands for Icarus verilog, is an open source verilog simulator.
* **_GTKWave_** - GTKWave is an open-source vcd(value change dump) waveform viewer.
* **_Yosys_** - Yosys is an open-source synthesis tool.
These are the _open-source_ tools used in the labs for the workshop.
---------
## Day 1
First step is to import all files including sky130 libraries into the system. We import the files with the help of git clone. Next, to work with the tools, the tool flow must be clear.
The design flow consists of various types of files. These include:
* RTL design - This is the HDL(hardware description language) code for the logic which is to be implemented. Verilog is the most popular HDL used in RTL design and ends with extension ".v".
* Testbench - The testbench is code also written in HDL. The testbench instantiated the RTL design and observes its outputs for different input values to check the logic functionality of code.
* Gate level netlist - The gate level netlist contains the design in terms of individual gate connections as opposed to RTL design which is the behavioural code for the logic implemented.
### Simulation Flow
The iverilog simulator takes the verilog(RTL or netlist) file and test bench as input and produces vcd (Value Change Dump) file. This vcd file can be viewed using the GTKWave.  
*Note: The iverilog simulation flow is similar for both RTL simulation and Gate-level simulation(GLS).*
##### Commands for simulation
Use the below commands for simulation and view the waveform with iverilog and GTKWave respectively.

        iverilog file_name.v testbench_name.v           //creates an executable file
        ./a.out                                         //generates vcd file
        gtkwave testbench_name.vcd                      //view vcd file in gtkwave viewer
Snippet below shows the terminal for simulation flow of a synchronous reset d-flipflop.
![Simulation Flow CLI](https://user-images.githubusercontent.com/78468534/120084681-dd73c280-c0ef-11eb-9831-792c15cb5a2d.jpeg)
  
Waveform shown by the GTKwave for the same example is:
![dff_syncres_waveform](https://user-images.githubusercontent.com/78468534/120097697-ab924880-c14f-11eb-91f5-96a38ad9ea57.jpeg)
  
### Synthesis Flow
The synthesis tool being used is Yosys. The inputs for Yosys are the RTL design written in HDL and the libraries required. Here the libraries by sky130 is used. Libraries exist as ".lib" files. These libraries contain different flavours of the most common logical blocks(like AND, NOT gate, MUX, Flip-flops, etc). The circuit is synthesised with these logical blocks.  
##### Why do we need different flavours?
The different flavours of same logic blocks(standard cells) allows these to be used in different applications. These flavours may work on different speeds. The faster the cell the more area and power required. The cell used depends on which parameter(s) is to be optimised.  
Additionally different cells are required to meet the timing requirements. More about that is discussed in [Day 2](day-2).  

First of all, Yosys tool is invoked in the terminal.

                $ yosys
                
![Yosys](https://user-images.githubusercontent.com/78468534/120098641-b8656b00-c154-11eb-8ade-6dea00d7616c.jpeg)

Now inside the yosys, type the following commands for synthesis

                yosys> read_liberty -lib ../path_to_library             //reads library file
                yosys> read_verilog file_name.v                         //reads RTL file to be synthesized
                yosys> synth -top file_name
                yosys> abc -liberty ../path_to_library                  //actual synthesis
                yosys> write_verilog -noattr netlist_name.v             //write created netlist as verilog file
                yosys> show                                             //shows netlist as circuit diagram with cells

Finally use "exit" command when you want to exit from yosys.  
*Note: The present working directory should contain RTL design before invoking yosys.
---------

## Day 2
### Library files
The library file used is *_sky130_fd_sc_hd_tt_025C_1v80.lib_*. The nomenclature for library is not random and shows important parameters most importantly the *Process*,*Temperature* and *Voltage*. These parameters are show in the end of the name "tt_025C_1v80".  
* tt means the process is "typical"
* 025C shows the temperature - 25 degree celsius
* 1v80 shows the voltage - 1.8V
  
##### Flavours of standard cells
As mentioned before, the library contains different flavours of same logic gates. This is done mainly to meet timing constraints.  
* *Faster cells* increases the clock frequency.
  T(clk) > T(cq_launch_flop) + T(combi) +T(setup_capture_flop)
* *Slower cells* are required to prevent hold violations.
  T(hold_capture_flop) < T(cq_launch_flop) + T(combi)
  
These different flavours are named different within the library. For example, the different flavours of and gate present are:
- based on delay:      _sky130_fd_sc_hd_and2_0 > sky130_fd_sc_hd_and2_2 > sky130_fd_sc_hd_and2_4_
- based on power/area: _sky130_fd_sc_hd_and2_0 < sky130_fd_sc_hd_and2_2 < sky130_fd_sc_hd_and2_4_

### Hierarchial and Flat synthesis
Many times in complex systems, different parts of system are designed seperately as RTL blocks(sub-modules) which can later be combined to form the whole system(top module). In such cases the synthesis can be done in two ways:
* *Hierarchial synthesis* - Here the top module is synthesised in terms of sub-modules. Only the higher level abstraction is required.
* *Flat Synthesis* - Here all sub-modules are also expanded and the top module is sysnthesised in terms of standard cells. It has lower abstraction compared to hierarchial synthesis.
Consider the example
![multiple_modules](https://user-images.githubusercontent.com/78468534/120099836-52c8ad00-c15b-11eb-837a-cb3ac34914a2.jpeg)
  
If this logic is synthesised noramlly it will do hierarchial synthesis. The netlist made following hierarchial synthesis is. 

![multiplemodules_hier](https://user-images.githubusercontent.com/78468534/120099897-97544880-c15b-11eb-8940-ef1309bf5517.jpeg)
  
For flat synthesis an additional command needs to be added to the normal synthesis flow.

              yosys> synth -top file_name
              yosys> flatten
              yosys> abc -liberty ../path_to_library
              yosys> write_verilog flat_netlist_name

This will give us a flattened netlist like shown below.  

![multiplemodules_flat](https://user-images.githubusercontent.com/78468534/120100002-10ec3680-c15c-11eb-8ca5-7a7dc335edf0.jpeg)


