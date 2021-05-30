# Verilog-RTL-design-with-SKY130-technology

![VSD workshop](https://user-images.githubusercontent.com/78468534/120083126-45241080-c0e4-11eb-9c1e-c970c2737ca1.jpeg)
VSD workshop - RTL design using Verilog with SKY130 Technology

This repo is the report for 5-day workshop on RTL design using Verilog with SKY130 Technology by VLSI sytem design.

## *Contents*
------------
* [Introduction to tools](introduction-to-tools)
* [Day 1](day-1)
* [Day 2](day-2)
* [Day 3](day-3)
* [Day 4](day-4)
* [Day 5](day-5)

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
                
Now inside the yosys, type the following commands for synthesis

                yosys> read_liberty -lib ../path_to_library             //reads library file
                yosys> read_verilog file_name.v                         //reads RTL file to be synthesized
                yosys> synth -top file_name
                yosys> abc -liberty ../path_to_library                  //actual synthesis
                yosys> write_verilog -noattr netlist_name.v             //write created netlist as verilog file
                yosys> show                                             //shows netlist as circuit diagram with cells


