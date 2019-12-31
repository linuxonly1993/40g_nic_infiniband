# Leveraging economical 40G Mellanox Infiniband NIC

## Operating environment
- Ubuntu Bionic 18.04.3
- Vanilla upstream kernel 5.4.3 from kernel.org (no additional patches applied)

## Quick Start
Do the following steps on **BOTH** machines connected by 40Gbit Infiniband cable.

### Package installation
```apt install rdma-core opensm ibutils infiniband-diags```

### Setup interface
Create new file under ```/etc/network/interfaces.d``` - e.g. named ```ib0_4g``` (the filename is not important) containing:
```
allow-hotplug ib0
iface ib0 inet static
    address 10.30.0.1
    netmask 255.255.255.0
    broadcast 10.30.0.255
```
Change the contents of the file to reflect your network (IP address, netmask, broadcast address). Do **NOT** change the interface name (ib0).

### Connect cable between computers
Do this **AFTER** performing above steps on **BOTH** computers.

### Reboot
Your interface (ib0) should be seen, with the right IP address, netmask etc.

If not, use the command ```ifconfig -a``` to see whether interface ```ib0``` was detected at all.

### Troubleshooting steps
- Use ```ibstat``` to see status of interface(s)
- ```cat /sys/class/net/ib0/mode``` - should show ```connected``` or ```datagram```
- Check whether ```opensm``` service is running: ```systemctl status opensm```. Output should resemble the following:

```
● opensm.service - LSB: Start opensm subnet manager.
   Loaded: loaded (/etc/init.d/opensm; generated)
   Active: active (running) since Mon 2019-12-30 00:07:32 PST; 1 day 10h ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3060 ExecStart=/etc/init.d/opensm start (code=exited, status=0/SUCCESS)
    Tasks: 39 (limit: 23347)
   CGroup: /system.slice/opensm.service
           └─3110 /usr/sbin/opensm -g 0x0002c903003f02f2 -f /var/log/opensm.0x0002c903003f02f2.log

Dec 30 00:07:32 fili opensm[3060]: Starting opensm on 0x0002c903003f02f2:
Dec 30 00:07:32 fili systemd[1]: Started LSB: Start opensm subnet manager..
Dec 30 00:07:32 fili OpenSM[3110]: /var/log/opensm.0x0002c903003f02f2.log log file opened
Dec 30 00:07:32 fili OpenSM[3107]: /var/log/opensm.0x0202c9fffe3f02f0.log log file opened
Dec 30 00:07:32 fili OpenSM[3107]: OpenSM 3.3.20
Dec 30 00:07:32 fili OpenSM[3110]: OpenSM 3.3.20
Dec 30 00:07:32 fili OpenSM[3107]: Entering DISCOVERING state
Dec 30 00:07:32 fili OpenSM[3110]: Entering DISCOVERING state
Dec 30 00:07:32 fili OpenSM[3107]: Exiting SM
Dec 30 00:07:32 fili OpenSM[3110]: Entering STANDBY state
```

## Checking performance
- On both machines, A and B connected by 40Gbit Infiniband cable, having IP addresses IP_A and IP_B on interface ```ib0```respectively, install iperf3: ```sudo apt install iperf3```
- On machine A start iperf3 in **server** mode: ```iperf3 -B IP_A -i 3 -s``` - replace IP_A with IP address of interface ```ib0``` on Machine A (sudo / root **not** required)
- On machine B start iperf3 in **client** mode: ```iperf3 -B IP_B -i 3 -t 15 -s IP_A``` - replace IP_A with IP address of interface ```ib0``` on Machine A and replace IP_B with IP address of interface ```ib0``` on machine B (sudo / root **not** required)

Output on machine B should look like:
```
Connecting to host 10.30.0.1, port 5201
[  4] local 10.30.0.2 port 51913 connected to 10.30.0.1 port 5201
[ ID] Interval           Transfer     Bandwidth       Retr  Cwnd
[  4]   0.00-3.00   sec  8.24 GBytes  23.6 Gbits/sec    0   1.81 MBytes       
[  4]   3.00-6.00   sec  8.50 GBytes  24.3 Gbits/sec    0   2.81 MBytes       
[  4]   6.00-9.00   sec  9.58 GBytes  27.4 Gbits/sec    0   2.81 MBytes       
[  4]   9.00-12.00  sec  8.26 GBytes  23.6 Gbits/sec    0   2.81 MBytes       
[  4]  12.00-15.00  sec  8.26 GBytes  23.6 Gbits/sec    0   2.81 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bandwidth       Retr
[  4]   0.00-15.00  sec  42.8 GBytes  24.5 Gbits/sec    0             sender
[  4]   0.00-15.00  sec  42.8 GBytes  24.5 Gbits/sec                  receiver

iperf Done.
```

## Troubleshooting and improving performance
### Identify your card
```lspci | grep Mellanox```

Output will look like
```
81:00.0 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3]
```
- **```81:00.0```** is *domain-bus-device* number
- **MT27500** is Mellanox part number
- **ConnectX-3** indicates card **supports** PCI-Express 3 - actual PCI-Express version also depends on PCI-Express capabilities of your motherboard

### Get additional details on your card
Use *domain-bus-device* number obtained above

```lspci -v -s 81:00.0```

Output will look like:
```
81:00.0 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3]
	Subsystem: Hewlett-Packard Company InfiniBand FDR/EN 10/40Gb Dual Port 544QSFP Adapter
	Physical Slot: 5
	Flags: bus master, fast devsel, latency 0, IRQ 47, NUMA node 1
	Memory at ec100000 (64-bit, non-prefetchable) [size=1M]
	Memory at 3800fe000000 (64-bit, prefetchable) [size=32M]
	Expansion ROM at ec000000 [disabled] [size=1M]
	Capabilities: <access denied>
	Kernel driver in use: mlx4_core
	Kernel modules: mlx4_core
```
***Subsystem: Hewlett-Packard Company InfiniBand FDR/EN 10/40Gb Dual Port 544QSFP Adapter*** gives further OEM details

### Get current PCI-Express version and width used
Use *domain-bus-device* number obtained above

```sudo lspci -vv -s 81:00.0 | grep LnkSta:```

Output will look like:

```		LnkSta:	Speed 8GT/s, Width x8, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-```
- **Speed 8GT/s** : 8 GT/s indicates PCI-Express version 3.x is being currently used for that slot
- **Width x8** : indicates logical width is x8 (8 lanes)


## PCI-Express speeds and maximum possible bandwidth for network link
PCI-E version | Per lane GT/sec | Per lane MBytes/sec
------------------- | ------------- | -------------------
1.x | 2.5 GT/sec | 250 MBytes/sec
2.x | 5 GT/sec | 500 MBytes/sec
3.x | 8 GT/sec | 985 MBytes/sec
4.x | 16 GT/sec | 1.97 GBytes/sec

PCI-E Version | Per lane GT/sec | Physical | Logical | Bandwidth MBytes/sec
------------- | --------------- | -------- | ------- | --------------------
2.x | 5 GT/sec | x8 | x1 | 250 MBytes/sec
2.x | 5 GT/sec | x8 | x4 | 1 GBytes/sec
2.x | 5 GT/sec | x8 | x8 | 2 GBytes/sec
2.x | 5 GT/sec | x16 | x1 | 250 MBytes/sec
2.x | 5 GT/sec | x16 | x4 | 1 GBytes/sec
2.x | 5 GT/sec | x16 | x8 | 2 GBytes/sec
3.x | 8 GT/sec | x8 | x1 | 985 MBytes/sec
3.x | 8 GT/sec | x8 | x4 | 3.94 GBytes/sec
3.x | 8 GT/sec | x8 | x8 | 7.88 GBytes/sec
3.x | 8 GT/sec | x16 | x1 | 985 MBytes/sec
3.x | 8 GT/sec | x16 | x4 | 3.94 GBytes/sec
3.x | 8 GT/sec | x16 | x8 | 7.88 GBytes/sec

Notes:
- Physical width will never be **smaller** than physical width of PCI-Express device (x8 in this case)
- Logical width will never be **larger** than physical width
- Logical width will never be **larger** than actual width of PCI-Express device lane width (x8 in this case)

### Limiting factors for maximum bandwidth of network link
- PCI-Express version
- Logical slot width - may depend on configurable settings in the BIOS for your motherboard
- Maximum bandwidth for network link will be **LESSER** of maximum bandwidth for each of the connected machines as explored above

### sysctl settings for TCP/IP stack
Put ```etc/sysctl.d/60-infiniband.conf``` under ```/etc/sysctl.d``` and reboot

Contents of ```etc/sysctl.d/60-infiniband.conf```

```
# For Mellanox MT27500 ConnextX-3 (HP InfiniBand FDR/EN 10/40Gb Dual Port 544QSFP)
# Settings from https://furneaux.ca/wiki/IPoIB#Kernel_Tuning
# Originally settings from Mellanox:
#   https://community.mellanox.com/s/article/linux-sysctl-tuning
net.ipv4.tcp_timestamps=0
net.ipv4.tcp_sack=1
net.core.netdev_max_backlog=250000
net.core.rmem_max=4194304
net.core.wmem_max=4194304
net.core.rmem_default=4194304
net.core.wmem_default=4194304
net.core.optmem_max=4194304
net.ipv4.tcp_low_latency=1
net.ipv4.tcp_adv_win_scale=1
net.ipv4.tcp_rmem=4096 87380 4194304
net.ipv4.tcp_wmem=4096 65536 4194304
```

### interface connected state and MTU
- Put ```etc/systemd/system/setup_ib0.service``` under ```etc/systemd/system/```
- Run ```systemctl enable setup_ib0.service``` and reboot

Contents of ```etc/systemd/system/setup_ib0.service```

```
[Unit]
Description=Setup ib0
After=sys-subsystem-net-devices-ib0.device

[Service]
Type=oneshot
ExecStart=/root/40g/setup_ib0.sh
```

### IP over Infiniband limitations
- Number and power of CPUs
- Available RAM


# Background

< Fill in here >

# Links
1. ABC
1. DEF

# Buying Links
1. ABC
1. DEF
