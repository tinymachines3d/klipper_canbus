# Bigtreetech U2C 1 & 3

## **PI Setup**

` sudo nano /etc/network/interfaces.d/can0 `

```bash
auto can0
iface can0 can static
 bitrate 500000
 up ifconfig $IFACE txqueuelen 256
 pre-up ip link set can0 type can bitrate 500000
 pre-up ip link set can0 txqueuelen 256
 ```

and press <kbd>Ctrl</kbd>+<kbd>X</kbd> then <kbd>Y</kbd> to save.

you can now reboot the pi with ` sudo reboot `



## Test the network

Once the pi has rebooted, run `ip -s link show can0` to check your network status.

You should see a line like below in the results.
The key thing to note is that the network is **UP** for now.

![image](https://user-images.githubusercontent.com/28379694/199144380-fcab121e-6f2c-449f-a3b5-62d6b40992f2.png)
