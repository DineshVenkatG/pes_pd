# pes_pd

<details>
  <summary>
    DAY1: Inception of Opensource EDA , Openlane and Sky130 PDK 
  </summary>
  <br>
  To develop an ASIC you need three major components 
  + ``` RTL IP ```

  + ```PDK``` : Process Design Kit ; Collection of files used to model a fabrication .

  + ```EDA Tools```

    ASIC Design flow goes from RTL to GDSII
    The flow is as follows :
    + Synthesis:Converts RTL into a circuit which has elements in the standard libraray 
    + Floor Planning: Partition of CHIP die between different system building blocks and place teh IO pads or define dimension , pin locations
    + Placement: place the cells on the floorplanned rows
    + Clock Tree Synthesis:Create clock distribution networks with minimum skew 
    + Routing: Implementation of interconnects between the different layers.
    + Sign Off: DRC , LVS and STA
   
    For this course we use OpenLane which comes with various other packages such as Yosys and abc .
    Openlane is used to harden the macros and chips , Yosys is used for RTL synthesis , ABC is used for RTL optimizations , abc maps the
    netlist converted by Yosys to a technological library .Further OpenRoad performs placement and routing .

 ## Importing Package:
 Since different software dependencies are needed to run openlane you need to import package , so every time you run the interactive terminal you need to import the package by using the command ```package require openlane 0.9```

    
        cd OpenLane
        sudo make mount
        ./flow.tcl -interactive
    
  Here we are checking for pre defined module picorv32a

  ```
    package require openlane 0.9
    prep -design <filename>
    run_synthesis
```

The task given is to find the ratio of total number of cells to d flip flop 

![day1](https://github.com/DineshVenkatG/pes_pd/assets/99543009/44dfbd12-5f0f-468f-a325-935e14ee9de7)

From here we can see total number of cells is 9541 and dff (sky130_fd_sc_hd__dfxtp_2)is 1596, thus the ratio is 0.1672

</details>
<details>
  <summary>
    DAY2:Good Floorplan vs Bad Floorplan and introduction to Library cells
  </summary>
  <br>
   Chip floorplanning : 

  + Defining the width and height of core and die : In defining the width and height *Utilization Factor* plays an important role , UTILISATION FACTOR = Area Occupied by the Netlist / Area of the core, Aspect ratio=Height / width
  + Defining location of pre placed cells: Some IPs such as memories , clock gating cells, comparator , mux needs to be instantiated multiple times , such IPs are placed on chip before automated placement and routing .
  + De-coupling of Capacitors :Combinational blocks need to be connected to Vss and Vdd for operation . But if the circuit is large with many resistors, there might be a problem with charging and discharging of capacitors , this can lead to noise margin in the circuits ,for this we use de-coupling capacitors that is placed close to the combinational block , when switching activity takes place it detaches from circuit and the capacitance can be charged fully .
  + Power planning :  When a transition occurs on a net, charge associated with coupling capacitors may be dumped to ground. If there are not enough ground taps charge will accumulate at the tap and the ground line will act like a large resistor, raising the ground voltage and lowering our noise margin. To bypass this problem a robust PDN with many power strap taps are needed to lower the resistance associated with the PDN.
  + Pin Placement: All input pins should be placed on the left and all output pins to he right
  + Logical cell placement Blockage : Block the area occupied by the pins to prevent the PNR tool from placing logical blocks where pins are present

    ### This lab focuses on floorplanning of picorv32a , which we have previously synthesized

    ```
    cd OpenLane
    sudo make mount
    ./flow.tcl -interactive
    package require openlane 0.9
    run_synthesis
    run_floorplan
    ```

    ![day2222](https://github.com/DineshVenkatG/pes_pd/assets/99543009/9ef6e54f-9ea3-4f0c-8ffa-32925a9a52aa)

    ![day2222222](https://github.com/DineshVenkatG/pes_pd/assets/99543009/acadba77-d056-4780-a953-a28f211326d1)
    ![day22222](https://github.com/DineshVenkatG/pes_pd/assets/99543009/d80f1158-4574-48b7-90a5-04aa2a9c0d76)
    ![day222](https://github.com/DineshVenkatG/pes_pd/assets/99543009/7df797cd-479e-444b-b0e4-f32fe254c0a2)

</details>
<details>
  <summary>DAY3 : CMOS Inverter ngspice simulations</summary>

  git clone the following to get all the necessary files
  ```
cd OpenLane
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
cd vsdstdcelldesign
```

make sure your sky130A.tech file is in ```vsdstdcelldesign``` folder , if not you can copy it .

To view the layout use the following command :
```
magic -T sky130A.tech sky130_inv.mag &
```
![day3](https://github.com/DineshVenkatG/pes_pd/assets/99543009/bafa87d7-8d67-4ba4-bf48-eb431d4a131f)

we perform sll the dimulstions on the spice file , to extract the spice file use the following command in tckcon window:

```
pwd
extract all
ext2spice cthresh 0 rthresh 0
ext2spice

```
you need to make some changes in the spice file to get the simulation results :

![image](https://github.com/AdrikaMohanty/pes_pd/assets/84654826/839c3b13-32cc-45d9-b409-44eda5318644)


to see the ngspice simulation:
```ngspice sky130_inv.spice```

![day3333](https://github.com/DineshVenkatG/pes_pd/assets/99543009/8b629d37-1111-44ba-be3e-222b88da4865)

To get the transient analysis of inverter you can do 
```plot y vs time a```

![day3_y_vs_pa](https://github.com/DineshVenkatG/pes_pd/assets/99543009/c1ab2e03-04f4-469c-a50a-f167d93ae1d3)

Use the drc_tests files , I have uploaded can be downloaded from there 

```
cd drc_tests
```

use the following to invoke magic :

```magic -d XR```

![day33](https://github.com/DineshVenkatG/pes_pd/assets/99543009/a7e72d39-4f48-4cb9-a89b-97c5eded51d2)

now open met3.mag using the command
```
magic -T sky130A.tech met3.mag &
```
Typing drc why in the tckcon window gives the drc error associated 

![day3lay](https://github.com/DineshVenkatG/pes_pd/assets/99543009/1f402473-77c6-4d1a-88d5-1249b3a0e348)

Load poly by typing ```load poly ``` in tkcon window 

after this you can see there is incorrect poly 

![day3poly](https://github.com/DineshVenkatG/pes_pd/assets/99543009/32724e19-9cb8-4888-96ee-f72945ae61bf)

To fix this:

Make the following changes in sky130A.tech file :
 Add the following lines after line 5178

 ```
spacing xhrpoly,uhrpoly,xpc allpolynonres 480 touching_illegal \
	"xhrpoly/uhrpoly resistor spacing to diffusion < %d (poly.9)"  
```

Add the following lines after line 4815
```
spacing npres allpolynonres 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```
To fix poly and diff and tap drc, make the following changes to the sky130A.tech file. Substitute the following lines in 4814 and 4815
```
spacing npres alldiff 480 touching_illegal \
	"poly.resistor spacing to N-tap < %d (poly.9)"
```

To fix nwell errors write the foolowings lines after line 4728 in sky130A.tech
```
variants (full)
cifmaxwidth nwell_untapped 0 bend_illegal \
	"Nwell missing tap (nwell.4)"
variants *
```

Add the following afetr line 1239
```
templayer nwell_tapped
bloat -all nsc nwell
 
templayer nwell_untapped nwell
and-not nwell_tapped

```

</details>

<details>
	<summary>DAY 4: Pre layout timing analysis and importance of good clock tree</summary>

 To see the tracks file :

 ![day4](https://github.com/DineshVenkatG/pes_pd/assets/99543009/5989115a-24bc-41b1-b05b-16605b6e0683)

The ports should be in the intersection of vertical and horizontal tracks .
To ensure that :

In magic press ```g``` this activates the grid ,

![day44](https://github.com/DineshVenkatG/pes_pd/assets/99543009/edc319d0-05cc-4a53-b05a-9619bdc31db2)

save the layout with grid by typing ```save sky130_vsdinv.mag```

Open using  ```magic -T sky130A.tch sky130_vsdinv.mag```

in the console type ``` lef write sky130_vsdinv.lef```
This will create the lef file 

![day4444](https://github.com/DineshVenkatG/pes_pd/assets/99543009/44bf8b08-9878-4505-84c7-453ac9602a3d)

## Including custom cell into openlane :

![day44444](https://github.com/DineshVenkatG/pes_pd/assets/99543009/2b4dc305-8ed4-46cb-8018-ebc81606310d)

![day4lef](https://github.com/DineshVenkatG/pes_pd/assets/99543009/a9994a46-ad0a-4c56-a237-5852d4c0af90)

![day4invl](https://github.com/DineshVenkatG/pes_pd/assets/99543009/a6efb260-15ba-4074-88f3-0d9b657104ef)

![day44invlllll](https://github.com/DineshVenkatG/pes_pd/assets/99543009/117dbf52-f0e8-426c-885e-b8e4186df263)
</details>
