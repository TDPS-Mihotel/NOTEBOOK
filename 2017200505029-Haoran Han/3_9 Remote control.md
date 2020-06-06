## Subject: **Remote control from Windows to RaspberryPi**

### Date:  March 17   Author: Haoran Han



- ### introduction

This experiment contains how to remote control RaspberryPi from Window through 3 different method: putty (command line), VNS_Viewer and remote control function embedded in Windows.



- ### location

Home

- ### preparation

Hardware: laptop, router,netting twine, RaspberryPi,

Software: DIsk_Genius, Win32_DiskImager, Advanced_IP_SCanner, Putty, VNC_Viewer



## Experiment Procedure

1. I firstly download the image of OS, **Raspbian** , through the [official website](https://www.raspberrypi.org/downloads/).

2. Then, I use **Win32_DiskImager** to copy this operating system in to the SD card.

    :bulb: In order to connect the laptop, we need to create a file name `SSH` **without any suffix** in the SD card.

3. Next, put the SD card into the RaspberryPi and charge the electricity.

4. I connect the PraspberryPi with the router of my home and thus it could connect to the Internet.

5. Then, use **Advanced_IP_Scanner** to check the IP in the default region (`192.168.1.0-192.168.1.255`) and we could find the IP address for our raspberry. In my experiment, it is initially `192.168.1.104`.

6. then, past this IP into the **Putty**, and we could connect the Raspberry through SHH method and control it through the command line.

next, following this procedure:

- input default user name and password: "pi" and "raspberry" respectively.

- type `sudo apt-get install tightvncserver xrdp`. When the installation is finished, we could use remote control function embedded in the Windows to connect the RaspberryPi.

- type `sudo raspi-config`, go to the "5 Interfacing Option", go to the "P3 VNC", enable it. After this operation, we could use VNC viewer to connect it.

  :bulb:Do not reverse the sequence of the upper two steps, or the VNC Viewer will cannot be used.

- type `sudo raspi-config`, go to the "7 Advanced Option", go to the "A5 Resolution", choose an  option apart from the default one. Or, your connection will show nothing but a line of "the desktop cannot be shown currently".

- Type the IP address into the VNC Viewer, connect to the RaspberryPi, and we could operate the GUI from now on.

- When we looking into the upright corner, we could find a "wifi" symbol, connect it to the router in your home, and let the mouse stay on that symbol, you could find another IP address. In my experiment, the IP address is `192.168.1.105`. Notice that, for the same hardware, there are two different IP address, one is for the connection by the netting twine, one is for the wireless wifi.

- Next, you could disconnect the  netting twine, and use the `192.168.1.105` to connect the RaspibarryPi.

- Then, RaspberryPi could also connect to your phone hotspot and this connection could be automatic when you start your Phone. However, the IP address allocated by my iPhone is `172.20.10.101`. So if you want to use IP_scanner to find it, add another section of IP address.

All the work has been done.

## Tips

- Every time when you want to connect the RaspberryPi, the IP address mybe different, that because the IP address is allocated by the routor dynamically.

- However, don't try to set the static IP address. Even though the IP address could be detected by my IP Scanner, but I could not connect the raspberry through the static IP address among any methods.

- I strongly suggest use VNC viewer, it has a better performance than the Windows remote control.

- When you choose a Country, choose China but not "Chile", or the RaspberryPi cannot detect any Wifi.

## Failure

I cannot use SVC to control the RaspberryPi remotely. this work will be continued by Song.