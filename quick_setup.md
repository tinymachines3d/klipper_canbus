## USB Adapter with passthrough example wiring

![image](https://user-images.githubusercontent.com/28379694/199144803-06d91adc-1ae4-48f1-87eb-2e5ef4a12f47.png)
# Bigtreetech U2C


### *add 10a auto fuse inline from PSU to U2C 24V*


## **PI Setup**
Connect to the printer you are flashing

` sudo nano /etc/network/interfaces.d/can0 `

```bash
auto can0
iface can0 can static
 bitrate 500000
 up ifconfig $IFACE txqueuelen 1024
 pre-up ip link set can0 type can bitrate 500000
 pre-up ip link set can0 txqueuelen 1024
 ```
 
   Press <kbd>Ctrl</kbd>+<kbd>X</kbd> to save.



You can now reboot the pi with ` sudo reboot `


## Test the network


Once the pi has rebooted, run `ip -s link show can0` to check your network status.

You should see a line like below in the results.
The key thing to note is that the network is **UP** for now.

![image](https://user-images.githubusercontent.com/28379694/199144380-fcab121e-6f2c-449f-a3b5-62d6b40992f2.png)


# Flashing the canboot firmware via DFU on EBB36/42 (ST32G0B1)


## **Generate the CANboot firmware file**

1. clone the CanBoot repository to your pi  
    >```bash
    >cd ~/
    >cd CanBoot
    >```


If no dir exists, go to step 2. If dir exists, go to step 3.

2. clone the CanBoot repository to your pi  
    >```bash
    >cd ~/
    >git clone https://github.com/Arksine/CanBoot
    >```

3. run the following

    >```bash
    >cd CanBoot
    >make menuconfig
    >```

4. Configure your makefile for the **EBB 36 / 42 v1.1/v1.2 with STM32G0B1**
   
![image](https://user-images.githubusercontent.com/28379694/199145678-1e9bfa05-c0ee-4715-b83d-38a19700cd2b.png)

    
   Exit using <kbd>ESC</kbd> or <kbd>Q</kbd>, confirm with yes(<kbd>Y</kbd>)

5. Build the firmware
    >```bash
    >make clean
    >
    >make
    >```

![image](https://user-images.githubusercontent.com/28379694/199145709-6e06d311-9c28-41f5-9e44-468078cfd86a.png)



## **Hook up the Board for flashing**


1. Add the 5v jumper to the pins highlighted below in green


   ![image](https://user-images.githubusercontent.com/28379694/199145766-7a8ffc1d-fe24-481d-9b28-7bc7ceeda131.png)
   
2. Connect your device to your PI via USB 

3. Hold the RESET button and BOOT shown above
    - Release Reset
    - Release Boot

4. Verify the device is in bootloader moad by using `lsusb`
   - you should see something like 
   >```bash 
   >    Bus 001 Device 005: ID 0483:df11 STMicroelectronics STM Device in DFU Mode
   >```

5. Flash the canboot bootloader to the board **DeviceID (0483:df11) may be different CHECK IT!** 

6. ERASE AND FLASH THE CANBOOT FIRMWARE
   
   >```bash 
   >sudo dfu-util -a 0 -D ~/CanBoot/out/canboot.bin --dfuse-address 0x08000000:force:mass-erase:leave -d 0483:df11
   >```

![image](https://user-images.githubusercontent.com/28379694/199145487-b1502c5b-d1ed-428d-b4ba-02cc83deedf3.png)
7.  Power off the Board, and insert the CANBUS cable

-----


8. You can now power up your printer with the toolhead board attached via the appropriate wiring scheme using the H L 24v and Gnd wires.  

9. Wait for the device to boot and ensure your CAN0 network is up and you can see the device 
    
    >```bash
    >~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
    >```

    or

    >```bash
    >~/CanBoot/scripts/flash_can.py -i can0 -q
    >```

    You should see something like 

    >```bash
    >"Found canbus_uuid=XXXXXXXXXX, Application: CanBoot"
    >```


10. Assuming the above gave you a UUID you can now flash Klipper to your board via CanBoot... (if not see the troubleshooting section [here](../troubleshooting.md))

    >```bash 
    >cd ~/klipper
    >make menuconfig
    >```

    >Enable low level configuratation
    >set the following.

    !![image](https://user-images.githubusercontent.com/28379694/199145452-1794f32f-c47a-4cdf-ada4-fa7c3dc3157b.png)

    Hit <kbd>Q</kbd> to exit and select <kbd>Y</kbd> to save changes.

    >```bash
    >make clean
    >make
    >```

    **You can now flash the board**

    >```
    >python3 ~/CanBoot/scripts/flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u MYUUID
    >```

    ![image](https://user-images.githubusercontent.com/28379694/199145395-ea3565e4-12d2-4ce8-8413-8ad766936b28.png)


    If everything worked properly, klipper is on the toolboard!

    To verify this you can query the canbus uuid with 

    >```bash
    >~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
    >```

    You should see something like 

    >```bash
    >"Found canbus_uuid=XXXXXXXXXX, Application: Klipper"
    >```
## Add canbus_uuid to printer.cfg
![image](https://user-images.githubusercontent.com/28379694/199154237-917828db-cb2f-49ad-8114-876f55bbdcb5.png)

## **Update G-Code Shell Command**

Open Kiauh  
    >```bash
    >cd ~/
    >./kiauh/kiauh.sh
    >```

![image](https://user-images.githubusercontent.com/28379694/199354454-f9f59389-f880-4d13-b907-51511cdea5a7.png) 

![image](https://user-images.githubusercontent.com/28379694/199354495-adffdd7e-2b04-406b-b139-e004e5590ca3.png) 

![image](https://user-images.githubusercontent.com/28379694/199354543-16d85b0f-9b28-46e6-8219-9d2087e77047.png)




