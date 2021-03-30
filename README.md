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

Obviously you will need ethernet cables to connect the Pi's. You can purchase patch cables or build your own (I opted to make my own custom sizes to keep the build clean looking). Patch cables will work well for this if not making your won. 

![pi Cluster](PiClusterImage.jpg)

## Enabling SSH and connecting to the Cluster
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
   
   * I recommend downloading VNC viewer to go in and set up your desktop for each pi and do the initial configuration/ password changes
         
         
