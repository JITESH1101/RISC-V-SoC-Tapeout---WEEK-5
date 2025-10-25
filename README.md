# RISC-V-SoC-Tapeout---WEEK-5

This week, the focus shifts from individual transistor-level circuits to the automated backend implementation of a full digital design. The OpenROAD Flow Scripts environment is being set up to serve as the foundation for this process.

The primary objectives are to execute the first two critical stages of the physical design flow: floorplanning, where the chip's core area and I/O are defined, and placement, where standard cells are arranged to meet timing and area constraints. This begins the practical conversion of a logical netlist into a physical layout.

# Open Road Tool Installation

- First to install the tool We can clone the github repository , enter the following command

```
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git
cd OpenROAD
```

- Now we need to install the dependencies , so the command for this is

```
sudo ./etc/DependencyInstaller.sh -all
```
<img width="3630" height="2279" alt="installing_dependencies" src="https://github.com/user-attachments/assets/bceeea06-e3c5-4245-b874-4b73437387bb" />

- Now create a directory with name ``build`` and open it

```
mkdir build
cd build
```

- Now use the ``cmake`` command

```
cmake ..
```

<img width="3630" height="2279" alt="cmake" src="https://github.com/user-attachments/assets/766226b9-132a-4e47-8214-cf7b9bc0cdd3" />

- Now enter the ``make`` command

```
make
```

After this command it will take a lot of time to install , around 1 to 2 hours . Unfortunately , i am unable to show the terminal installation pictures as it is too long to take a screenshot.

- After installation is done , enter the following command

```
sudo make install
```

<img width="1672" height="1432" alt="sudo_make_install" src="https://github.com/user-attachments/assets/66b18b71-0f67-4ace-b05d-14f55406b4b7" />

With this the installation of OpenROAD is done and the following picture shows the list of files 


<img width="3752" height="1743" alt="files" src="https://github.com/user-attachments/assets/0c63f8bd-b74d-46c2-a43d-735840afd899" />

# OpenROAD Physical Design: Floorplan & Placement

This repository documents the setup and execution of the initial stages of the VLSI physical design flow using the open-source OpenROAD suite. This project bridges the gap between transistor-level circuit design and full backend implementation, where logical netlists are transformed into physical layouts.

Purpose and Learning Objectives:

The focus of this work is on the practical application of an RTL-to-GDSII flow. By implementing the first steps of physical design, a deeper understanding of how digital circuits are physically realized on silicon is gained.

Key concepts being explored include:

  - How design constraints (like timing and area) are applied before the routing stage.

  - The process by which standard cells are strategically arranged to optimize for competing goals, such as minimizing delay, chip area, and signal congestion.

  - How early physical design choices fundamentally impact the final manufacturability and performance (timing) of the chip.

Implemented Flow Stages:

The flow is initiated by setting up the OpenROAD Flow Scripts. The following key stages are then executed:

   - Floorplanning: This is the initial step of physical layout. The overall chip dimensions are defined, space is allocated for large blocks (macros), and the I/O pins are positioned. A foundational plan for the power grid (PDN) is also established.

   - Placement: Following the floorplan, the standard cells from the synthesized logic are placed into the defined rows. This process is driven by complex algorithms that seek to minimize total wirelength and meet timing constraints, which helps to reduce congestion in the later routing stages.

# Physical Design implementation : Floorplan & Placement Stages 

To initiate the physical design flow in OpenROAD, several key input files are required. The primary input is the gate-level netlist (in Verilog format), which is provided by the synthesis stage and describes the circuit as an interconnection of standard cells. Alongside the netlist, a SDC file is essential; this file specifies the design's timing requirements, such as clock periods, I/O delays, and false paths. Finally, the flow is provided with the technology files: Library Exchange Format (LEF) files supply the physical layout abstracts for standard cells and macros, while Liberty (LIB) files contain their detailed timing and power characterization data.

Our sample design for learning this flow is Nangate45 from the test directory,the following are the library files

<img width="3493" height="354" alt="libarary_files_nandgate45" src="https://github.com/user-attachments/assets/97749ed4-358e-47ec-8436-5dfbe7756eff" />

Now the following file is the tcl script to implement the tool steps

<img width="2019" height="1198" alt="tcl_scriptfile" src="https://github.com/user-attachments/assets/dd7e6e0d-364c-4110-bcbe-cfdf6c6b4f19" />

and the next picture is also a tcl script which has the script of the entire physical design process, sourcing this file into openroad will finish entire physical design. 


<img width="3967" height="2464" alt="flow_tcl" src="https://github.com/user-attachments/assets/e2d88721-5715-4296-b369-5b556edf5ebd" />

Since we want only floor planning and placement , we have updated the tcl script shown firstly and also make 4 new tcl script which are sourced into this . The 4 new tcl scripts are uploaded  above for reference. The core and die area dimension are also part of this tcl file

<img width="2017" height="1621" alt="updated_tcl_scriptfile" src="https://github.com/user-attachments/assets/4a602b8c-2a9a-40c7-a6e8-019bf58607c5" />

The reason for doing so is to do step by step implementation. The 4 steps are 

- Floorplan
  
- Power distribution network
  
- global placement

- full detailed placement(more optimised placement without overlap of standard cells)

For first stage we include only Floorplan tcl file and comment other three files , we do the same for each step .

To launch the tool using this tcl file , enter the following command

```
openroad -gui -log gcd_logfile.log gcd_nangate45_copy.tcl
```

We see that the floor plan is done with the core area and die area being shown

<img width="3972" height="2505" alt="floor_plan_1" src="https://github.com/user-attachments/assets/c232d82c-22f7-4c20-acc0-000af49b1a28" />

By zooming into the core boundary we see the tapcells 


<img width="3972" height="2505" alt="tapcell" src="https://github.com/user-attachments/assets/f89d451c-6c91-4fbe-9c9c-6be335820d0c" />

Also we see the standard cell rows


<img width="3972" height="2505" alt="standard_Cell_rows" src="https://github.com/user-attachments/assets/ff8a7f28-2480-4d66-a160-dc8490ed2b81" />

If we zoom into the standard cell rows ,we see that each of these unit cells is called a site

<img width="3972" height="2505" alt="site" src="https://github.com/user-attachments/assets/3e62ad95-24cc-4b2f-b691-b22a1652c8fa" />

Now exit the gui terminal and go back to the tcl file and comment the ``include gcd_floorplan.tcl`` and uncomment the ``include gcd_pdn.tcl`` and run the command again

```
openroad -gui -log gcd_logfile.log gcd_nangate45_copy.tcl
```

We see the power lines like VDD and GND and power lines were implemented.

<img width="3972" height="2505" alt="power_distribution_network" src="https://github.com/user-attachments/assets/532cbfdf-dab6-4ec3-98ac-835295be5607" />

We repeat the same steps for the global placement and full detailed placement stages too and we get

For global placement,

<img width="3972" height="2505" alt="global_placement" src="https://github.com/user-attachments/assets/15564015-e8cd-4c01-86ad-6f514a68dcec" />

and if we zoom in we see that the standard cells are overlapping 

<img width="3972" height="2505" alt="standardcells" src="https://github.com/user-attachments/assets/e85312f0-2479-49c0-a235-926c1439c034" />

To optimize this , we implement the full detail placement stage , we get

<img width="3972" height="2505" alt="detailed_placement" src="https://github.com/user-attachments/assets/c9875619-4de6-43ec-a8ca-116eb3e0ef57" />

Here the placement of standard cells is optimized and they do not overlap each other.

- For each of the step we implemented till now , a log file has been generated and after finishing placement we wrote a verilog file (basically netlist) which are also uploaded above.

# Challenges faced

The main challenge i have faced in doing so i had a lot of time spent to install OpenRoad Flow Scripts on my system by cloning the github repo provided

``
https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
``

Which seems to be not working  well in Ubuntu Linux 24.04 version . I realized that it works well on 22.0 version but i can't change the OS version as it is a lengthy process for me to take backup for current data i have on my system. So i have followed the above mentioned process that i have shown to install OpenROAD.  


# Conclusion

This week's task is successfully concluded with the completion of the floorplan and placement stages. Through this process, the synthesized gate-level netlist has been transformed from a purely logical description into a spatially-aware physical layout. The standard cells are now arranged within the core boundaries, optimized for timing and congestion based on the provided constraints. Week 6 is coming soon.






















