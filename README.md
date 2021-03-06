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
  * [Library files](#library-files)
  * [Hierarchial and Flat Synthesis](#hierarchial-and-flat-synthesis)
  * [Flipflop Coding and Synthesis](#flipflop-coding-and-synthesis)
* [Day 3](#day-3)
  * [Combinational Logic Optimisations](#combinational-logic-optimisations)
  * [Sequential Logic Optimisations](#sequential-logic-optimisations)
* [Day 4](#day-4)
  * [GLS and Synthesis Simulation Mismatch](#gls-and-synthesis-simulation-mismatch)
  * [Blocking and Non Blocking Statements](#blocking-and-non-blocking-statements) 
* [Day 5](#day-5)
  * [IF and CASE Statements](#if-and-case-statements)
  * [FOR loop and FOR GENERATE](#for-loop-and-for-generate)
* [Acknowledgement](#acknowledgement)


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

![sky130 lib](https://user-images.githubusercontent.com/78468534/120100304-bc49bb00-c15d-11eb-8d26-593b74b3120a.jpeg)
  
_SKY130 library file_
  
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
  
### Flipflop Coding and Synthesis
When it comes to flipflops, the set/reset logic is very important. They can be either synchronous or asynchronous. For flops with synchronous set/reset, the set/reset is dependant on clock and therefore this logic gets realised on the input (D- input).  
In the HDL code the asynchronous and  synchronous reset can be distinguished by the sensistivity list. A flipflop with asynchronous reset should have both clock and reset on the sensitiivity list. A flipflop with synchronous reset on the other hand would have only clock on the sensitivity list as the reset is also dependant on the clock.  

![dff_syncres_waveform](https://user-images.githubusercontent.com/78468534/120097697-ab924880-c14f-11eb-91f5-96a38ad9ea57.jpeg)
D flipflop with synchronous reset  
![asyncres_wave](https://user-images.githubusercontent.com/78468534/120100865-ad183c80-c160-11eb-9be7-a200e64fea33.jpeg)
D flipflop with asynchronous reset. 

Now during synthesis of sequential circuits, an additional step is required to point the tools towards library containing the flipflops. This is done by the command:

              yosys> synth -top file_name
              yosys> dfflibmap -liberty ../path_to_library
              yosys> abc -liberty ../path_to_library
              yosys> show
              
The circuits obtained by designing flipflops with asynchronous and synchronous reset are given below
![asyncres ckt](https://user-images.githubusercontent.com/78468534/120100923-f8324f80-c160-11eb-89cf-18652f8c6287.jpeg)
D flipflop with asynchronous reset.  

![syncres ckt](https://user-images.githubusercontent.com/78468534/120100877-be614900-c160-11eb-9bf1-e62b4c2b48ec.jpeg)
D flipflop with synchronous reset.  

The netlist clearly shows that the synchronous reset is applied at D input as expected.  
---------
 ## Day 3
 The synthesis tool comes with many features. One of such features which has a huge impact on design is _optimisation_. The tool does optimisation on the logic (RTL design). Usually these optimisation are done to obtain least hardware and omit unwanted components.  
 ### Combinational Logic Optimisations
 In designs containing complex combinational logic, most of the time certain components are not reflected in the output and hence are considered _useless_. Such parts are removed by the tool. Sometimes sub-modules within a top module which does not affect the output may also be removed.    
 Consider the example:  
 ![combi_logic](https://user-images.githubusercontent.com/78468534/120101359-5d874000-c163-11eb-8b49-157afd599d3b.jpeg)
  
From the boolean logic for this RTL, it is clear that "b" input is not reflected on the output. Hence the synthesised netlist should optimisations. This becomes clear from the netlist given below.  
![combi_ckt](https://user-images.githubusercontent.com/78468534/120101495-fae27400-c163-11eb-875a-bc249eadb565.jpeg)  
  
Now consider a combinational circuit written as sub-modules.  
![mulmodopt logic](https://user-images.githubusercontent.com/78468534/120101567-42690000-c164-11eb-8b61-f9b4b2a1d80e.jpeg)
  
The optimised netlist obtained is:  
![mulmod opt ckt](https://user-images.githubusercontent.com/78468534/120101580-557bd000-c164-11eb-8872-62997595bc81.jpeg) 

### Sequential Logic Optimisations
The most common optimisations in sequential circuits are optimisations for constant and optimising unused outputs.
  
For optimisation of constants in sequential circuits, consider below example.  
![seq logic](https://user-images.githubusercontent.com/78468534/120102197-75f95980-c167-11eb-9137-9b57da2b3af4.jpeg)
  
After simulating with the help og testbench the waveform obtained is:  
![seq wave](https://user-images.githubusercontent.com/78468534/120102221-91fcfb00-c167-11eb-94e8-25df46601e67.jpeg)
  
From the waveform it is clear the output remains constant for all cases and hence can be optimised. The synthesis report obtained is given below.  
![seq synth report](https://user-images.githubusercontent.com/78468534/120102254-bf49a900-c167-11eb-8c79-aee5e70d2bb0.jpeg)
  
Finally the optimised netlist given by the tools is:  
![seq netlist](https://user-images.githubusercontent.com/78468534/120102284-e1dbc200-c167-11eb-8b27-055b8dc806e2.jpeg)
  
Now consider the next example for optimisation of unused outputs in sequential circuits.  
![count_opt logic](https://user-images.githubusercontent.com/78468534/120102437-a1307880-c168-11eb-9daa-0c278fbec260.jpeg)
  
![countopt wave](https://user-images.githubusercontent.com/78468534/120102452-b2798500-c168-11eb-9b44-a7a583a3f607.jpeg)  
_Waveform_  
  
  
From the waveform it is clear that output is only dependant single bit of the counter. Hence the other flops can be optimised.  
![coutopt rprt](https://user-images.githubusercontent.com/78468534/120102517-0b491d80-c169-11eb-9a9f-6e3a65572fa7.jpeg)  
![countopt ckt](https://user-images.githubusercontent.com/78468534/120102528-1439ef00-c169-11eb-8560-326d1c94b493.jpeg)  
The synthesised output contains only one flipflop as other unused flops are optimised off.

---------
## Day 4
### GLS and Synthesis Simulation Mismatch
While using an HDL to write the RTL logic, if not careful, synthesis simulation mismatch may occur.
Gate-level simulation(GLS) is the simulation of gate level netlist of a design unlike the normal functional simulation where the behavioural RTL is simulated. When the RTL simulation is different from GLS the circuit is said to exhibit synthesis simulation mismatch.  
This can occur due to a variety of reasons including:
* missing sensitivity list
* wrong use of blocking/non-blocking statements
* non-standard verilog coding  

Lets see an example of such sythesis simulation mismatch
![badmx logic](https://user-images.githubusercontent.com/78468534/120114674-ea4eef80-c19d-11eb-82c8-a14058378278.jpeg)  
This is the RTL code for a mux, but the sensitivity list shows only "sel". This means the output will only change when there is a change in "sel" signal. This is obviously not the desired logic and will cause synthesis simulation mismatch due to the missing elements in sensitivity list.  
The waveform obtained from RTL simulation is :
![bdmx wave](https://user-images.githubusercontent.com/78468534/120114836-a01a3e00-c19e-11eb-8462-a5da3a576bad.jpeg)  
compared to the waveform from GLS
![badmx gls](https://user-images.githubusercontent.com/78468534/120114887-e079bc00-c19e-11eb-9cca-113090b823c1.jpeg)  
  
_Note: The verilog models of standard cells must also be called on the iverilog for GLS._
### Blocking and Non Blocking Statements
In verilog HDL, there are two types of assignment operators:
* <= this assignment creates a non-blocking statement
* =  this assignment creates a blocking statement
  
For non-blocking statements all the RHS are first found out before it is assigned. Since all such statements are assigned at the same time the order of statements are irrelevant. In the case of blocking statements, values are assigned instantly and therefore the order of statements are extremely important.  
While using blocking statements if care is not taken it could result in synthesis simulation mismatch. For example consider:
![block logc](https://user-images.githubusercontent.com/78468534/120115260-82e66f00-c1a0-11eb-992a-7c61bab6db34.jpeg)  
_RTL simulation waveform_
![blckng wave](https://user-images.githubusercontent.com/78468534/120115319-d3f66300-c1a0-11eb-94a1-545790d023a3.jpeg)  
_GLS waveform_
![blckng gls](https://user-images.githubusercontent.com/78468534/120115337-e1abe880-c1a0-11eb-92bc-b8c735d712c7.jpeg)  


## Day 5
### IF and CASE statements
The _IF_ and _CASE_ statements are the conditional statements present in verilog HDL.  
If statements-    
              
              if (condition1)
                  ......
              else if (condition2)
                  ......
              else
                  ......
Case statements-

             case(sel)
                condition1 : ......
                condition2 : ......
                default    : ......
                
The main difference between  case and if statements is the priority level. If statements has priority level and case statements do not. Although these statements are required in almost all high level design it also brings complexities. An incomplete if or case statements may create additional problems. For example consider:

![inco_if](https://user-images.githubusercontent.com/78468534/120116005-aa8b0680-c1a3-11eb-823d-00933e44d842.jpeg)  
The incomplete if statement in this example with infer unwanted latches. This is evident in simulation and synthesis results given below.  
_Simulation waveform_
![incomp_if wave](https://user-images.githubusercontent.com/78468534/120116051-e3c37680-c1a3-11eb-84d1-8028ce0dcabb.jpeg)  
_Synthesis report_
![incomp_if rprt](https://user-images.githubusercontent.com/78468534/120116070-fd64be00-c1a3-11eb-9360-dfbc121d186b.jpeg)  
_Synthesised circuit_
![incomp_if ckt](https://user-images.githubusercontent.com/78468534/120116091-0eadca80-c1a4-11eb-8ca5-44eb35488b11.jpeg)  
Similarly unwanted latches are inferred for incomplete case statements as well.


### FOR loop and FOR GENERATE
For loops are always written inside "always" statements. The syntax for "for" loop is similar to that in C. For loops are used where multiple evaluating statements need to be run.  
For example:  
_Demux using for loop_
![demux_gen logic](https://user-images.githubusercontent.com/78468534/120116279-17eb6700-c1a5-11eb-9c1a-0b65321bd293.jpeg)

  
For generate statements are written outside "always" statement. It is used for instantiating a module multiple times within an RTL.  
For example:  
_Ripple carry adder using for generate_
![rca logic](https://user-images.githubusercontent.com/78468534/120116313-39e4e980-c1a5-11eb-945e-9cd88ea0a5ed.jpeg)

  
  
## Acknowledgement
* [Kunal Ghosh](https://github.com/kunalg123)
* [Shon Taware](https://github.com/ShonTaware)
* Mukesh Nadar
