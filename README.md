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

## Improving performance
### Checking performance
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

# Background

< Fill in here >

# Links
1. ABC
1. DEF

# Buying Links
1. ABC
1. DEF
