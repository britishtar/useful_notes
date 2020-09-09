# How to set up Alfa range extender on Ubuntu
References:
+ https://tacticalware.com/install-alfa-awus036ach-drivers-on-ubuntu-18-04/
+ https://dalewifisec.wordpress.com/2014/03/03/changing-your-mac-address-using-macchanger/
+ https://unix.stackexchange.com/questions/111256/how-to-permanently-disable-a-network-interface
+ https://askubuntu.com/questions/767786/changing-network-interfaces-name-ubuntu-16-04
+ https://www.linuxuprising.com/2018/05/how-to-permanently-change-mac-address.html
+ https://superuser.com/questions/1322777/in-systemd-service-file-how-do-i-say-after-usb-is-ready
***
## Install drivers and supporting software
```shell
$ sudo apt-get install wireless-tools
$ sudo apt-get install rtl8812au-dkms
$ sudo apt-get install macchanger
```

## Plug in the Alfa

Plug the range extender into a free USB port.

## Disable the Native Interface

+ Get the names and MAC addresses for my native wireless card and the Alfa:

```shell
$ ifconfig
<native wireless interface>: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.4.85  netmask 255.255.255.0  broadcast 10.0.4.255
        inet6 fe80::2324:7bb0:5854:224d  prefixlen 64  scopeid 0x20<link>
        ether <native NIC MAC>  txqueuelen 1000  (Ethernet)
        RX packets 2601921  bytes 1369425955 (1.3 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1754790  bytes 304610576 (304.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

<alfa interface>: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether <alfa MAC>  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
+ Disable current wireless interface:

```shell
$ sudo nmcli d disconnect <native wireless interface>
$ sudo ifconfig <native wireless interface> down
```
+ Permanently [disable NetworkManager management](https://unix.stackexchange.com/questions/111256/how-to-permanently-disable-a-network-interface) of native wireless interface:

```shell
$ sudo echo "iface <native wireless interface> inet manual" >>/etc/network/interfaces
$ sudo /etc/init.d/network-manager restart
```

## Change the Alfa MAC
I want to keep the same IP address on my local network, which is currently using **dhcp**, and I don't want to go through the hassle of manually configuring my router.  I will just spoof the MAC address of my machine's native wireless interface instead.

+ Change the MAC address for the Alfa interface:

```shell
$ sudo ifconfig <alfa interface> down
$ sudo macchanger --mac=<native NIC MAC> <alfa interface>
$ sudo ifconfig <alfa interface> up
```

## Create systemd service to change MAC on boot
I don't want to have to manually configure the MAC address after every boot.

+ Create a file called `/etc/systemd/system/mac-changer.service`, and paste in the following contents:

```
[Unit]
Description=changes mac for <interface>
Wants=network.target
Before=network.target
BindsTo=sys-sybsystem-net-devices-<interface>.device
After=sys-subsystem-net-devices-<interface>.device

[Service]
Type=oneshot
ExecStart=/usr/bin/macchanger --mac=<native NIC MAC> <interface>
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
+ Enable the new systemd service:
```shell
sudo systemctl enable mac-changer.service
```

## Check Original and New MAC Addresses

Use `macchanger` to verify that the MAC address for the desired interface has been properly changed:
```shell
macchanger -s <interface>
```

## TODO: Change the Alfa interface name
+ https://askubuntu.com/questions/767786/changing-network-interfaces-name-ubuntu-16-04
