# Configure and Verify Network Connections

## Testing network connectivity

- check connectivity to google using `ping`

```bash
ping google.com
```

- check `dns` connection using `ping`

```bash
ping 8.8.8.8
```

- check ip address of local computer

```bash
ifconfig
```

```bash
ip addr 
```

- ping your own ipaddress

```bash
ping 172.30.18.188
```

- ping gateway address

```bash
ping 10.10.10.1
```

- check ip routing information

```bash
ip route
```

## Testing DNS

> DNS - Domain Name System

- `ping`, `dig`
- `nslookup`, `host`

### dig

- dig syntax

```bash
dig @server host
```

- dig example

```bash
dig google.com
```

```bash
dig @127.0.0.1 google.com
```

```bash
dig @8.8.8.8 google.com
```

### nslookup

```bash
nslookup host server
```

```bash
nslookup google.com
```

### host

```bash
host host server 
```

```bash
host google.com 
```

## the hosts file `/etc/hosts`

```bash
cat /etc/hosts
```

- make changes / configure ip dns mapping in hosts file and restart local server

```bash
service dnsmasq restart
```

> use these tools to not only specify what host you wanna lookup but what server you want to query when you look it up

## Locating common Network config files

- Everything in linux is configured with text files
- some common network config files are consistent across different linux distros
- there are also going to be specific config files in different distros

```bash
/etc/hosts
/etc/resolv.conf
/etc/nsswitch.conf
```

### /etc/hosts

- `/etc/hosts` is the file acts as a first resort dns lookup
- before system even looks looks things up via dns it looks in here

### /etc/nsswitch.conf

- `/etc/nsswitch.conf` configures a bunch of things on our system like group and password files
- check the `hosts files dns` line, here it configures where to look for dns lookups to find the ipaddresses of hosts. the `files` entry points to `/etc/hosts` and then in order there are other entires like `dns` server.

### /etc/resolv.conf

- `/etc/resolv.conf` tells the computer what nameserver to use (dns cache)

## Identifying Ubuntu and Debian Network Files

- Ubuntu is one of the most common distributions and its built upon Debian.

- /etc/network/interfaces
- /etc/netplan/*
- Network Manager (nmtui)

- Check version of linux

```bash
cat /etc/os-release
```

- older version of ubuntu 16.04 use `/etc/network/interfaces` file
- newer version of ubuntu 18.04 use different network config yaml file inside folder `/etc/netplan`
- after making changes in the yaml file run following command to activate new changes on the yaml file

```bash
sudo netplan apply
```

- there's also `nmtui` - Network maanger text user interface to edit network config

## Identifying Network Files on CentOS

- network maanger
- /etc/sysconfig

- centos is really awesome when it comes to network configuration
- centos and red hat put most of their network config inside `/etc/sysconfig` folder, inside look for `network-scripts` folder thats where networking is configured, look for `ifconfig-eth0` which is config file for the interface. we can change this file for example IPV4 address form DHCP to add manual static ip addr and restart network service `sudo service network restart`. Weather we are using GUI or text editing its a single config file being updated.

## Network Bonding Modes

- Network bonding is linux is a way to utilize computers that have more than one network port (most servers now a days have more than one network ports)

Two types of network bonds we will look at

- network bonds that require __switch support__
- __generic__ network bonds that donot require switch to know what is going on

- Linux provides many choices, choices are fairly limited when we actually know what these are. First thing we look at is does it requires swith support, a lot switches  specially layer 3 swithces or smart switches will support link aggregation or lacp or ether channel idea is smart switch will have built in code that will allow them to work togeather.
- If you have smart switch chances are you have the ability to use link aggregation some of these require a switch that supports that

| MODE | Description | Needs Switch Support? |
| ---- | ----------- | -------------------- |
| 0 | balance-rr | sorta |
| 1 | active backup | No |
| 2 | balance-xor | Yes |
| 3 | broadcast | Yes |
| 4 | 802.3ad | Yes |
| 5 | balance-tlb | No |
| 6 | balance-alb | No |

- __balance-rr__ : when you have multiple ports and when we transmit packets across all of our interfaces, if we plug it to a switch it does require a switch support that supports link aggregation but a lot of people use this balakced round robin and they will connect two servers togeather and they will use balance-rr in this case you donot need a switch support as ther is no swtich involved, alot of times this is the way to increase throughput without requiring any special switch support. So if you are directly connecting computers you donot need switch support but if you are connecting to a switch you need switch support. This is Mode 0.
- __active backup__ : This is Mode 1 - Out of many ports on the system only one is going to be active, if active port fails then another one is going to turn active. this is fault tolerant but doesn't speed anything up.
- __balance - xor__ : Requires special switch support, balance - xor works on hash based on your mac address  and the client's mac address, based on this hash it always uses specific port to connect to a specific client. This is just a way to spread out which computers use which port but ports are constant. this is not used often because if you have switch support you will mostly use a better option like `802.3ad` which is the industry standard link aggregation protocol
- __braodcast__ : __broadcast__ is used in very specific cases. it takes all of your ports on the server and spews all of the data out all of the ports at once. its not used very often and definitely requires switch support.
- __802.3ad__ : requires swtich support. Its industry standard link aggregation protocol. This is a smart way of improving throughput, availability, fault tolerance. If your switch supports link aggregation you should use `802.3ad` or mode 4 .
- __balance-tlb__  and __balance-alb__ : doesnot requires switch support. lts say if you have dumb switch which doesnot supports link aggregation. linux is smart enough to be able to utilize a dumb switch and increase bandwitch and throughput and reliability. There's two different ways it does it __balance-tlb__ which is balance of transmit load balance and __balance-tlb__ balance of all load balancing. with __tlb__ data is transmitted with least busy port, hence all the transmit is balanced, however incoming is still going to always go to one active port so that's not as good as __alb__ which is the same concept except it does it for both incoming and outgoing data transmit, basically least busy port gets the traffic. This is brilliant and how it does it by constantly changing the mac address on these ethernet ports (cards). from switch standpoint it doesnot matter how many times you switch the mac address so it generally works just fine. Some people do have issues with this, but for some it works fine.

- for a dumb switch it is highly recommended to use __MODE 6__ i.e. __balance-alb__

- for a swtich which supports link aggregation it is highly recommended to use __MODE 4__ i.e. __802.3ad__ which is industry standard.

- If you are just connecting two servers togeather with multiple cables, __MODE 0__ i.e. __balance-rr__ works really great.

the most difficult part of linux network bonding is figuring out which mdoe to work but really its not that tough of a decision if your switch supports it use __802.3ad__ if your swith doesn't supports it its highly recommened to use __MODE 6__ or __balance-alb__

### Configuring Network Bonds

once you know the type of bond you want to setup for your two ethernet or two or more ethernet connections on the computer configuring them is pretty straight forward although its different based on your linux distro.

#### ubuntu - configuring network bonds

- go to `/etc/netplan`
- you will have a yaml file
- this file has bonding configuration

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
  bonds:
    bond0:
      dhcp4: false
      interfaces:
        - eth0
      addresses: [10.10.10.10/24]
      gateway4: 10.10.10.1
      parameters: 
        mode: active-backup
      nameservers:
              addresses: [8.8.8.8]

```

- we need to use `networkd` as a rederer, we cannot use network manager to configure bonds because it doesn't supports bonding.
- we have to define ethernet cards themselves, ethernet's eth0 we have dhcp4 set to false as we donot want it to assign an address to eth0.
- then we need to setup bonded interface in the section bonds, name of the bond is `bond0` dhcp4 is set to false as we will assign a static ip address, then we give what interfaces (one or more) are going to be part of this bond. in our case we have provide `eth0`. next we have setup the addresses, gateway the nameserver. the only new thing here is `paramters:`  here we have specified the mode by name `active-backup`

- next we save this yaml file and do `netplan apply`

```bash
netplan apply
```

- to check if its working we can do `ip add` and check `bond0` section in the output.
- another way to test is check contents of file `/proc/net/bonding/bond0` this file tells us information about bound0 configured

```bash
cat /proc/net/bonding/bond0
```

#### CentOS - configuring network bonds is little different

- in CentOS to configure ethernet0 port we go to `/etc/sysconfig/network-scripts` where we have `ifconfig` files
- first we will look at the changes we have to make to `ifconfig-eth0`

```bash
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
NAME=eth0
DEVICE=eth0
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

- notice we have not given any ip address info, MASTER=bond0 and SLAVE=yes
- next we will look at `ifconfig-bond0`, here at BONDING_OPTS we tell it what we want it to do, we have set it up to `mode=6` which is balance all loadbalancing 

```bash
DEVICE=bond0
NAME=bond0
BONDING_MASTER=yes
IPADDR=10.10.10.15
PREFIX=24
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="mode=6 miimon=100"
```

- checking to makesure its working, use `ip add`
