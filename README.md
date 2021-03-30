# Data-Extraction-and-Analysis-System
## Hardware Required

For cost effectiveness I purchased three Canakits which come with everything you need to set up the Raspberry Pi cluster:

[Canakit](https://www.amazon.com/gp/product/B07V5JTMV9/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

   * pi version 4, 4GB
   * Micro Sd memory card
   * power supply
   * heatsinks

You can use the included case but I opted for a cluster case by Geek Pi:

[GeekPi cluster case](https://www.amazon.com/gp/product/B07MW3GM1T/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

You can set up the Pi headless using WiFi, but for stability I opted to hardwire the cluster. You will need a network switch if your cluster is not going to be near a router with sufficient ports. If you plan on scaling this project up in the future to more than four Pis a larger switch could be a justified purchase. An unmanged switch is fine for this purpose and I opted for a 5 port switch:

[Netgear 5 Port Switch](https://www.amazon.com/gp/product/B07S98YLHM/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1)

You will need ethernet cables to connect the Pi's to the switch. You can purchase patch cables or build your own (I opted to make my own custom sizes to keep the build clean looking). Patch cables will work well for this if not making your won. 

![pi Cluster](PiClusterImage.jpg)

## Enabling SSH, Connecting to the Cluster, Initial Setup
For this project there will be three Raspberry Pi version 4 micro computers set up in a cluster. One name node and two data nodes. The following is the process for setting up the Pis headlessly (no need for periphereals, i.e. monitor or keyboard).

**1.** Navigate to [Raspberry Pi OS Installer](https://www.raspberrypi.org/software/).

  * Download the Raspberry Pi OS installer for your desired OS.
         
  * Insert the micro sd card that will be used in your pi into your PC.
         
  * From the program menu select the Raspberry Pi OS, then the intended drive
  
**2.** After flashing the OS image to the micro sd card:

   * Remove the card from the PC and reinsert. If you do not do this the drive will show as Fat32 and you wont be able to see the root directory.
         
   * Now create a blank text file in the **_root_** directory of the sd card and name it **_ssh_**.
   
**3.** SSH into the Pis:

   * To connect fromm windows, open a command prompt window and type ssh pi@raspberrypi, your initial password is raspberry. 
   
   * I recommend downloading VNC viewer to go in and set up your desktop for each pi and do the initial configuration/ password changes. 
   * Once connected to the pi at the terminal type:
  ```console
        sudo raspi-config
  ```
  * This should bring up the following interfact where you will select option 1 System Options.
  
  ![config](raspi-config1.PNG)
  
  Then go to option 54 Hostname
  
  ![config](raspi-config2.PNG)
   
   * You can rename your pi to whatever you want, personally I named them for their function and started with the bottom pi in my rack as NameNode, then DataNode1, DataNode2, ...... That way if a node goes out you can easily swap in a new node and know its location. 
   
   
  ![config](raspi-config3.PNG)
  
  * While in the menu if you plan on using VNC viewer for the desktop, I had to change the resolution in this configuration menu to get it to display. 

**4.** Creating a New Group and User

* Type the following commands:
```console
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
```
* Set the password to whatever you like, but write it down.
* No need to fill out the information of hduser, you can press enter and leave all fields blank.
* everything Hadoop is going to happen via this newly created user so change to it by typing:
  ```console
      su hduser
  ```
