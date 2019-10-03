## Overview

Setting up a Single Board Computer to work as a router for WiFi VLANs.

I have 3 distinct WiFi network needs

- Normal access for phones, tablets, laptops, etc - home network
- Guest or work network - internet access only no crosstalk to home
- IoT devices - things I really do not trust at all

There is a subset of IoT devices which I am aware can be compromised to leak 
the WiFi password so they really need to be on a completely seperate network.

The IoT devices, in some cases, need to communicate with those on the home WiFi
and there may even be a need for the management network to talk to them also.

## Network

### Subnets

- 192.168.123.0/24 - Management network, all AP, switches, etc sit on this
- 192.168.127.0/24 - Guest WiFi network
- 192.168.133.0/24 - Wireless network - since several IoT devices have phone/tablet apps that need to communicate with each other ALL WiFi devices are in the same subnet


### VLANs

- 6 - guest/work wireless network
- 7 - home wireless network
- 8 - IoT network
- 9 - IoT network for insecure devices
- 17 - External traffic 

The VLANs for 7, 8, 9, and 17 are all on the 192.168.133.0/24 subnet


### Hardware

Using an [APU D2](https://pcengines.ch/apu2d2.htm) device.

This device has three gigabit network connections.

```
    +--------------------------+          +--------------------------+ 
    |                          |          |                          |
    |   APU2 - Gingy Router    |          |  APU 2 - Harold gateway  |
    | enp1s0  enp2s0   enp3s0  |          | enp1s0  enp2s0   enp3s0  |
    +--------------------------+          +--------------------------+
        |        |        |                   |       |         | 
        |        |        |                   |       |         |__  Other
        |        |        |                   |       |____________ Connections
        |        |        |                   |
        |        |        |                   |
        |        |        |                   |
 +-------------------------------------------------------------------------+
 |   VLAN 1    VLAN 7   VLAN 17             VLAN 1                         |
 |             VLAN 8                       VLAN 6                         |
 |             VLAN 9                       VLAN 17                        |
 |                                                                         |
 |                      Gigabit Managed Layer2 Switch                      |
 |                                                                         |
 |   VLAN 1                  VLAN 1                                        |
 |   VLAN 6                  VLAN 6                                        |
 |   VLAN 7                  VLAN 7                                        | 
 |   VLAN 8                  VLAN 8                                        |
 |   VLAN 9                  VLAN 9                                        |
 +-------------------------------------------------------------------------+
        |                       | 
        |                       |
  +--------------+        +--------------+
  | Access Point |        | Access Point |
  +--------------+        +--------------+
```


## PC Engines APU2 Setup

Setting up an APU2 D2 device

* Downloading Ubuntu 18.04-Server
* Create boot USB key using this command on MacOS
 
```
dd bs=1m if=ubuntu-18.04.2-server-amd64.iso of=/dev/rdisk2
```

* Booted APU2
* Used the following command to connect to the serial console (in Linux VM)
* (I couldn't get my USB serial lead to work on my Mac)
 
```
screen /dev/ttyUSB0 115200
```

* Once the APU2 started to boot I hit F10
* Selected USB as boot device
* Boot halted with an error
* Typed help at the boot prompt
* Then tried this
 
```
boot: install append console=ttyS0,115200n8
```

* It whinged about vga mode but I told it to ignore that
* Seems to continue and actually starts installing
* Set up a user with a reasonable password
* Let it configure the network interface with DHCP for now
* The first device is nearest the serial port
* All seemed to work and now have an installed system


## References

### Man pages

* [bridge](http://man7.org/linux/man-pages/man8/bridge.8.html)
* [ebtables](https://linux.die.net/man/8/ebtables)

