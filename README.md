# DFX preparation
- This repo has been copied and modified from this tutorial [kv260 tutorial](https://xilinx.github.io/kria-apps-docs/dfx/build/html/docs/DFX_Landing_Page.html) (it was built for xilinx 2022 environment, i have modified some ip version and tcl configuration to make it runnable on 2023.2 version)

## repo structure
```
repository
    |----2rp_design
            |
            |--- create_new_rm  ## tcl file for reconfigurable module creation
            |--- platform       ## unused I used ubuntu 22.04 kria in this case (I 
            |                   ## think that it is not neccessary)
            |--- rm_tcl         ## tcl to for each example module creation (unused)
            |
         ip_repo ## ip that xilinx given (generated from hls)
            |
            |
            |
        application ## the kv260 application used to comm with reconfig module
```

## environment
### host computer
- ubuntu 22.04 lts
- vivado (2023.2)
- vitis + petalinux(unused at this case)
### client fpga
- Kria 260
- Ubuntu 22.04

## building step
### install ubuntu 22.04 on kria
- I follow this guide [kb260 ubuntu](https://xilinx.github.io/kria-apps-docs/kv260/2022.1/linux_boot/ubuntu_22_04/build/html/docs/intro.html)

### create reconfigurable module

1. ``` cd 2rp_design/create_new_rm ```
2. open vivado ()
3. ``` source ./hw_prj.tcl``` (this step will initialize the ```project_1``` at the ```create_new_rm``` folder)
4. ```source ../rm_tcl/axis_rm_template.tcl```  create RP building block (this will create the block diagram for reconfigurable module)
5. ```cr_bd_axis_rm "" <RMName> <targetRP>``` target ip can be RP_0 or RP_1 depen on which reconfigurable partition that designer to implement
6. ```export_bd_synth -force [get_files ./project_1/project_1.srcs/sources_1/bd/<RMName>_<targetRP>/<RMName>_<targetRP>.bd]``` synthesis the reconfigurable partition.
7. ```source ./generate_rm.tcl``` to retrieve code for the output generation code
8. ```generate_rm <RMName> <targetRP>```
9. repeat step 4 - 8 to build all region that we require
10. the result will be shown in ```create_new_rm/output_files/<RMName>_<target>``` composed of 
- accel.json 
- opendfx_.bit (bitstream file)
- opendfx_.dtsi (device tree source include)

11. copy output_file to ``` /libs/firmware/Xilinx/myProj``` on kria 260
12. at ```myProj/<each RP>``` you must compile the system using this command ```dtc -@ -I dts -O dtb -o custom_overlay.dtbo custom_overlay.dts```
13. load reconfiguration module ```xmutil loadapp <rpName>```. if you don't know what app you want use ```xmutil listapps```
14. copy the application in ```application.cpp``` to somewhere on kria and compile it using this command
```
INC='-I/usr/include/xrt -I/usr/include/dfx-mgr' #Setting up Include files
LNK='-luuid -lxrt_coreutil -lxrt++ -ldfx-mgr' #Setting up Library dependencies
sudo g++ -Wall -g -std=c++1y $INC lib/siha.c src/ADD/main.c $LNK -o testADD #Compile application

```
15. run using this command 
```sudo ./testADD```

## So What is the barier?

- at the step 5, you may have seen the building block but it is only for reconfiguration partition, but the static partition is not published by the xilinx. (Only dcp of the system provided)
![image](report.jpeg)
- If I am required to build custom build reconfiguration for other boards or other feature. I am clealy understand it.
- the example of the fuctionality such as FFT, Add_sub but the source HLS written in C++ are not provided, but I think it would be good idea if I have it as reference.