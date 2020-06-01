# How To Compile Your Own OpenWrt Firmware 

## Contents
- [Prerequisites](#prerequisites)
  - [Downloads](#downloads)
  - [Virtual PC Settings](#virtual-pc-settings)
- [Devices](#devices)
  - [Linksys](#linksys)
    - [EA6350v3](#linksys-ea6350v3)
      - [Switch Partition to Stock Firmware Linksys EA6350v3](#switch-partition-to-stock-firmware-linksys-ea6350v3)
      - [Compile Linksys EA6350v3 Firmware](#compile-linksys-ea6350v3-firmware)
    - [Linksys WRT1900ACSv2](#linksys-wrt1900acsv2)
      - [Switch Partition to Stock Firmware Linksys WRT1900ACSv2](#switch-partition-to-stock-firmware-linksys-wrt1900acsv2)    
      - [Compile Linksys WRT1900ACSv2 Firmware](#compile-linksys-wrt1900acsv2-firmware)
- [How To Use the diff file](#how-to-use-the-diff-file)

## Prerequisites
If you're running a Windows OS you will need access to a Linux PC to compile the final firmware. I personally use Ubuntu 20.04 x64 Desktop in a virtual enviroment on top of my Windows 10 Pro x64 OS. If you don't have access to a physcial Linux PC don't worry as you could potenitally run a virtual machine in your Windows OS using VMWare Workstation player or Oracle VirtualBox like I do, so long as you have enough hardware resources.

### Downloads
* https://www.virtualbox.org/wiki/Downloads
* https://vmware.com/go/downloadplayer
* https://ubuntu.com/download/desktop

### Virtual PC Settings
On the hypervisor PC I create a virtual machine with 4 CPU cores, 50% of my physical RAM which is 8096MB and ensure the firmware type is set from BIOS to UEFI. I won't be detailing how to setup the rest of the virtual machine in this guide as it would make the guide much longer than it needs to be, so I'm just focussing on explaing how to compile the OpenWrt firmware. Once Ubuntu has been installed we are ready for compiling OpenWrt.

## Devices
### Linksys
#### Linksys EA6350v3
##### Switch Partition to Stock Firmware Linksys EA6350v3
To switch to the other partition on this router it will require turning the router on four times and off three.

Instructions coming soon...

##### Compile Linksys EA6350v3 Firmware
Run the following steps in order:-

1. ```sudo apt install subversion g++ zlib1g-dev build-essential git python rsync man-db libncurses5-dev gawk gettext unzip file libssl-dev wget zip time```

2. ```mkdir -p ~/OpenWrt/{"Pre-compiled OpenWrt Firmware",v19.07.3/EA6350v3}```

3. ```cd ~/OpenWrt```

4. ```git clone https://git.openwrt.org/openwrt/openwrt.git v19.07.3/EA6350v3```

5. ```git clone https://github.com/TheSurgeNetwork/Pre-compiled-OpenWrt-Firmware.git "Pre-compiled OpenWrt Firmware"```

6. ```cd v19.07.3/EA6350v3 && git fetch --tags && git checkout openwrt-v19.07```

7. ```./scripts/feeds update -a && ./scripts/feeds install -a```

8. ```cp ~/OpenWrt/"Pre-compiled OpenWrt Firmware"/"Compile Your Own Firmware"/EA6350v3/02_network ~/OpenWrt/v19.07.3/EA6350v3/target/linux/ipq40xx/base-files/etc/board.d```

9. ```cp ~/OpenWrt/"Pre-compiled OpenWrt Firmware"/"Compile Your Own Firmware"/EA6350v3/715-net-essedma-disable-default-vlan.patch ~/OpenWrt/v19.07.3/EA6350v3/target/linux/ipq40xx/patches-4.14```

10. Now you will need to decide if you would like to compile have an exact copy of the official OpenWrt firmware with the exception of the VLAN fixes applied, or compile from a blank slate. If you would like to copy the offical OpenWrt firmware go to step **11**, otherwsie if you would like to compile your firmware from a clean slate go to step **12**.

11. 
    - ```cp ~/OpenWrt/"Pre-compiled OpenWrt Firmware"/"Compile Your Own Firmware"/EA6350v3/config.buildinfo ~/OpenWrt/v19.07.3/EA6350v3/.config```
  
    - ```make defconfig```
  
    - ```make menuconfig```

    - Continue to step **13**.

12. 
    - ```
      cat > .config << EOF
      CONFIG_TARGET_ipq40xx=y
      CONFIG_TARGET_ipq40xx_generic=y
      CONFIG_TARGET_ipq40xx_generic_DEVICE_linksys_ea6350v3=y
      EOF
      ```
  
    - `make menuconfig`
  
    - As you have selected the clean slate method you will need to head to `LuCI > Collections >` and select your preferred LuCI collection so that you have a web-GUI included in the firmware. The default is `luci` but 19.07.3 brings improved SSL performance, so whilst you have the option to change the default LuCI collection, so you would be safer selecting `luci-ssl-openssl` as this enables a HTTPS connection to your router web-GUI and will prevent man-in-the-middle attacks.
  
    - Continue to step **13**

13. Whilst using `make menuconfig` to select packages and features there are three options that can be chosen with the `y`, `n` and `m` keys:-
    - `y` = `<*>` The package/feature will be compiled to later user with OPKG and also included in the final firmware

    - `n` = `< >` The package/feature will be not compiled at all

    - `m` = `<M>` The package/feature will be compiled to later user with OPKG but not included in the final firmware

All three options can also be toggled through by pressing the spacebar. I find it easier to start at `LuCI > Application` as they automatically select other packages/features for dependencies. I then head back to the top of the main menu at `Base system` and work my way down saving after leaving each menu. To leave a menu and return to the previous menu press the `ESC` key

14. Once you're happy with the configuration go back to main menu using the `ESC` key and use the arrows on your keyboard to move to the `< Save >` option, hit enter/return key and save it as `.config`

15. 
    - If you would like to re-use your `.config` file for future builds we can reduce the contents of file by scrubbing away ununsed and excluded packages/features, and only showing the chnages compared to the default configuration. This can be done running `./scripts/diffconfig.sh > EA6350v3`

    - To re-use the exported configuration please see [How To Use the diff file](#how-to-use-the-diff-file) instructions

16. Once the `.config` file is saved exit out of `make menuconfig` by hitting enter/return on `< Exit >` and this will return you back to CLI.

17. To now compile the firmware it is as simple as running `make -j* clean download world`. Replace `*` with the number of cores your Linux PC has plus one. In my case I had 4 cores so I will use `-j5`. If you run into errors please run `make -j1 V=sc` so that you can see the errors, address the issue(s) and then re-run `make -j* clean download world`. If you get really stuck you should open a post at https://forum.openwrt.org

18. After some time your firmware will be finished compiling and without any errors. You will find the final firmware images under `/bin/targets/ipq40xx/generic`. Copy these to a safe place as you will need it next.

19. Ensure your router is booted into the stock/Linksys firmware. If you have already done so please see [Switch Partition to Stock Firmware Linksys EA6350v3](#switch-partition-to-stock-firmware-linksys-ea6350v3)

I also highly advise you to connect to the router via ethernet cable to eliminate a corrupt flash as WiFi can be prone to dropping packets. Don't connect the router to any other device other than the one that contains the firmware. 

20. Login into your Linksys stock fimrware at IP address `https://192.168.1.1` using the username `admin` and the password `admin`. If you are struggling to connect to the router it may be because the last configuration was setup as a DHCP client to another router or with a manual IP addresss for example. All you need to do is hold a pin in the reset button on the back to do a factory reset. After a couple of minutes the router will be back up and running and you can re-connect to `https://192.168.1.1`.

21. Once logged in, click `Connectivity` under `Router Settings` and under the `Basic` tab you will see a title labelled `Firmware Update`. Under the `Manual:` section click the `Choose File` button and browse to the firmware file called `...squashfs-factory.bin`. Follow the on-screen prompts and then let the router finish flashing the firmware. 

22. You will find that the Linksys stock firmware animation or proggress bar appear stuck, but if you give the router five minutes and the LED lights appear to be up, you can now safely access your OpenWrt router at `http://192.168.1.1`. Congratulations you have compiled your own OpenWrt firmware.





#### Linksys WRT1900ACSv2
##### Switch Partition to Stock Firmware Linksys WRT1900ACSv2
To switch to the other partition on this router it will require turning the router on four times and off three.

Instructions coming soon...

##### Compile Linksys WRT1900ACSv2 Firmware
Run the following steps in order:-

1. ```sudo apt install subversion g++ zlib1g-dev build-essential git python rsync man-db libncurses5-dev gawk gettext unzip file libssl-dev wget zip time```

2. ```mkdir -p ~/OpenWrt/{"Pre-compiled OpenWrt Firmware",v19.07.3/WRT1900ACSv2}```

3. ```cd ~/OpenWrt```

4. ```git clone https://git.openwrt.org/openwrt/openwrt.git v19.07.3/WRT1900ACSv2```

5. ```git clone https://github.com/TheSurgeNetwork/Pre-compiled-OpenWrt-Firmware.git "Pre-compiled OpenWrt Firmware"```

6. ```cd v19.07.3/EA6350v3 && git fetch --tags && git checkout openwrt-v19.07```

7. ```./scripts/feeds update -a && ./scripts/feeds install -a```

8. Now you will need to decide if you would like to compile have an exact copy of the official OpenWrt firmware, or compile from a blank slate. If you would like to copy the offical OpenWrt firmware go to step **9**, otherwsie if you would like to compile your firmware from a clean slate go to step **10**.

9. 
    - ```cp ~/OpenWrt/"Pre-compiled OpenWrt Firmware"/"Compile Your Own Firmware"/WRT1900ACSv2/config.buildinfo ~/OpenWrt/v19.07.3/WRT1900ACSv2/.config```
  
    - ```make defconfig```
  
    - ```make menuconfig```

    - Continue to step **11**.

10. 
    - ```
      cat > .config << EOF
      CONFIG_TARGET_mvebu=y
      CONFIG_TARGET_mvebu_cortexa9=y
      CONFIG_TARGET_mvebu_cortexa9_DEVICE_linksys_wrt1900acs=y
      EOF
      ```
  
    - `make menuconfig`
  
    - As you have selected the clean slate method you will need to head to `LuCI > Collections >` and select your preferred LuCI collection so that you have a web-GUI included in the firmware. The default is `luci` but 19.07.3 brings improved SSL performance, so whilst you have the option to change the default LuCI collection, so you would be safer selecting `luci-ssl-openssl` as this enables a HTTPS connection to your router web-GUI and will prevent man-in-the-middle attacks.
  
    - Continue to step **11**

11. Whilst using `make menuconfig` to select packages and features there are three options that can be chosen with the `y`, `n` and `m` keys:-
    - `y` = `<*>` The package/feature will be compiled to later user with OPKG and also included in the final firmware

    - `n` = `< >` The package/feature will be not compiled at all

    - `m` = `<M>` The package/feature will be compiled to later user with OPKG but not included in the final firmware

All three options can also be toggled through by pressing the spacebar. I find it easier to start at `LuCI > Application` as they automatically select other packages/features for dependencies. I then head back to the top of the main menu at `Base system` and work my way down saving after leaving each menu. To leave a menu and return to the previous menu press the `ESC` key

12. Once you're happy with the configuration go back to main menu using the `ESC` key and use the arrows on your keyboard to move to the `< Save >` option, hit enter/return key and save it as `.config`

13. 
    - If you would like to re-use your `.config` file for future builds we can reduce the contents of file by scrubbing away ununsed and excluded packages/features, and only showing the chnages compared to the default configuration. This can be done running `./scripts/diffconfig.sh > WRT1900ACSv2`

    - To re-use the exported configuration please see [How To Use the diff file](#how-to-use-the-diff-file) instructions

14. Once the `.config` file is saved exit out of `make menuconfig` by hitting enter/return on `< Exit >` and this will return you back to CLI.

15. To now compile the firmware it is as simple as running `make -j* clean download world`. Replace `*` with the number of cores your Linux PC has plus one. In my case I had 4 cores so I will use `-j5`. If you run into errors please run `make -j1 V=sc` so that you can see the errors, address the issue(s) and then re-run `make -j* clean download world`. If you get really stuck you should open a post at https://forum.openwrt.org

16. After some time your firmware will be finished compiling and without any errors. You will find the final firmware images under `/bin/targets/mvebu/cortexa9`. Copy these to a safe place as you will need it next.

17. Ensure your router is booted into the stock/Linksys firmware. If you have already done so please see [Switch Partition to Stock Firmware Linksys WRT1900ACSv2](#switch-partition-to-stock-firmware-linksys-wrt1900acsv2)

I also highly advise you to connect to the router via ethernet cable to eliminate a corrupt flash as WiFi can be prone to dropping packets. Don't connect the router to any other device other than the one that contains the firmware. 

18. Login into your Linksys stock fimrware at IP address `https://192.168.1.1` using the username `admin` and the password `admin`. If you are struggling to connect to the router it may be because the last configuration was setup as a DHCP client to another router or with a manual IP addresss for example. All you need to do is hold a pin in the reset button on the back to do a factory reset. After a couple of minutes the router will be back up and running and you can re-connect to `https://192.168.1.1`.

19. Once logged in, click `Connectivity` under `Router Settings` and under the `Basic` tab you will see a title labelled `Firmware Update`. Under the `Manual:` section click the `Choose File` button and browse to the firmware file called `...squashfs-factory.bin`. Follow the on-screen prompts and then let the router finish flashing the firmware. 

20. You will find that the Linksys stock firmware animation or proggress bar appear stuck, but if you give the router five minutes and the LED lights appear to be up, you can now safely access your OpenWrt router at `http://192.168.1.1`. Congratulations you have compiled your own OpenWrt firmware.

## How To Use the diff file
You can the diff file to do a couple of things:-
* Re-compile firmware for the same device
* Compile firmware for different device using the same configuration file

Instructions coming soon...
