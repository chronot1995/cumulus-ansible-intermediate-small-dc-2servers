## cumulus-ansible-intermediate-small-dc-2servers

### DEPRECATED - Please check out Nvidia Air at https://air.nvidia.com for up-to-date examples

### Summary:

  - Cumulus Linux 3.7.11
  - Underlying Topology Converter to 4.7.0
  - Tested against Vagrant 2.1.5 on Mac and Linux. Windows is not supported.
  - Tested against Virtualbox 5.2.32 on Mac 10.14
  - Tested against Libvirt 1.3.1 and Ubuntu 16.04 LTS

### Description:

This is an Ansible demo which configures two Linux servers and four Cumulus VX switches with BGP using J2 / Jinja2 templates

### Network Diagram:

![Network Diagram](https://github.com/chronot1995/cumulus-ansible-intermediate-small-dc-2servers/blob/master/documentation/cumulus-ansible-intermediate-small-dc-2servers.png)

### Install and Setup Virtualbox on Mac

Setup Vagrant for the first time on Mojave, MacOS 10.14.6

1. Install Homebrew 2.1.9 (This will also install Xcode Command Line Tools)

    https://brew.sh

2. Install Virtualbox (Tested with 5.2.32)

    https://www.virtualbox.org

I had to go through the install process twice to load the proper security extensions (System Preferences > Security & Privacy > General Tab > "Allow" on bottom)

3. Install Vagrant (Tested with 2.1.5)

    https://www.vagrantup.com

### Install and Setup Linux / libvirt demo environment:

First, make sure that the following is currently running on your machine:

1. This demo was tested on a Ubuntu 16.04 VM w/ 4 processors and 32Gb of Diagram

2. Following the instructions at the following link:

    https://docs.cumulusnetworks.com/cumulus-vx/Development-Environments/Vagrant-and-Libvirt-with-KVM-or-QEMU/

3. Download the latest Vagrant, 2.1.5, from the following location:

    https://www.vagrantup.com/

### Initializing the demo environment:

1. Copy the Git repo to your local machine:

```
    git clone https://github.com/chronot1995/cumulus-ansible-intermediate-small-dc-2servers/
```

2. Change directories to the following

```
    cumulus-ansible-intermediate-small-dc-2servers
```

3a. Run the following for Virtualbox:

```
    ./start-vagrant-vbox-poc.sh
```

3b. Run the following for Libvirt:

```
    ./start-vagrant-libvirt-poc.sh
```

### Running the Ansible Playbook

1a. SSH into the Virtualbox oob-mgmt-server:

```
    cd vx-vbox-simulation
    vagrant ssh oob-mgmt-server
```

1a. SSH into the Libvirt oob-mgmt-server:

```
    cd vx-libvirt-simulation  
    vagrant ssh oob-mgmt-server
```

2. Copy the Git repo unto the oob-mgmt-server:

```
    git clone https://github.com/chronot1995/cumulus-ansible-intermediate-small-dc-2servers
```

3. Change directories to the following

```
    cumulus-ansible-intermediate-small-dc-2servers/automation
```

4. Run the following:

```
    ./provision.sh
```

This will run the automation script and configure the environment.

### Troubleshooting

Helpful NCLU troubleshooting commands:

- net show route
- net show bgp summary
- net show interface | grep -i UP
- net show lldp

Helpful Linux troubleshooting commands:

- ip route
- ip link show
- ip address <interface>

The BGP Summary command will show if each switch has formed an IPv6 and an l2vpn neighbor relationship:

```
cumulus@leaf01:mgmt-vrf:~$ net show bgp summary

show bgp ipv4 unicast summary
=============================
BGP router identifier 10.1.1.1, local AS number 65111 vrf-id 0
BGP table version 5
RIB entries 9, using 1368 bytes of memory
Peers 2, using 39 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd
spine01(swp1)   4      65333      47      47        0    0    0 00:01:50            3
spine02(swp2)   4      65333      47      48        0    0    0 00:01:50            3

Total number of neighbors 2


show bgp ipv6 unicast summary
=============================

show bgp l2vpn evpn summary
===========================
BGP router identifier 10.1.1.1, local AS number 65111 vrf-id 0
BGP table version 0
RIB entries 3, using 456 bytes of memory
Peers 2, using 39 KiB of memory

Neighbor        V         AS MsgRcvd MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd
spine01(swp1)   4      65333      47      47        0    0    0 00:01:50            2
spine02(swp2)   4      65333      47      48        0    0    0 00:01:50            2

Total number of neighbors 2

```

One should see that the loopback routes are installed with two next hops / ECMP for the other leaf:

```
cumulus@leaf01:mgmt-vrf:~$ net show route bgp
RIB entry for bgp
=================
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR,
       > - selected route, * - FIB route

B>* 10.2.2.2/32 [20/0] via fe80::4638:39ff:fe00:2, swp1, 00:05:59
  *                    via fe80::4638:39ff:fe00:4, swp2, 00:05:59
B>* 10.3.3.3/32 [20/0] via fe80::4638:39ff:fe00:2, swp1, 00:06:00
B>* 10.4.4.4/32 [20/0] via fe80::4638:39ff:fe00:4, swp2, 00:05:59
```

One can also view the MAC addresses of the two switches within the EVPN instance by running the following command:

```
cumulus@leaf01:mgmt-vrf:~$ net show evpn mac vni 11
Number of MACs (local and remote) known for this VNI: 2
MAC               Type   Intf/Remote VTEP      VLAN  Seq #'s
44:38:39:00:00:0b remote 10.2.2.2                    0/0
44:38:39:00:00:09 local  swp10                 11    0/0
```

This will show all of the Type-2 and Type-3 routes, and the encapsulated MAC addresses, from leaf01 and leaf02:

```
cumulus@leaf01:mgmt-vrf:~$ net show bgp evpn route
BGP table version is 2, local router ID is 10.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal
Origin codes: i - IGP, e - EGP, ? - incomplete
EVPN type-2 prefix: [2]:[ESI]:[EthTag]:[MAClen]:[MAC]:[IPlen]:[IP]
EVPN type-3 prefix: [3]:[EthTag]:[IPlen]:[OrigIP]
EVPN type-5 prefix: [5]:[ESI]:[EthTag]:[IPlen]:[IP]

   Network          Next Hop            Metric LocPrf Weight Path
                    Extended Community
Route Distinguisher: 10.1.1.1:2
*> [2]:[0]:[0]:[48]:[44:38:39:00:00:09]
                    10.1.1.1                           32768 i
                    ET:8 RT:65111:11
*> [3]:[0]:[32]:[10.1.1.1]
                    10.1.1.1                           32768 i
                    ET:8 RT:65111:11
Route Distinguisher: 10.2.2.2:2
*  [2]:[0]:[0]:[48]:[44:38:39:00:00:0b]
                    10.2.2.2                               0 65333 65222 i
                    RT:65222:11 ET:8
*> [2]:[0]:[0]:[48]:[44:38:39:00:00:0b]
                    10.2.2.2                               0 65333 65222 i
                    RT:65222:11 ET:8
*  [3]:[0]:[32]:[10.2.2.2]
                    10.2.2.2                               0 65333 65222 i
                    RT:65222:11 ET:8
*> [3]:[0]:[32]:[10.2.2.2]
                    10.2.2.2                               0 65333 65222 i
                    RT:65222:11 ET:8

Displayed 4 prefixes (6 paths)
```

### Errata

1. To shutdown the demo, run the following command from the vx-simulation directory:

```
vagrant destroy -f
```

2. This topology was configured using the Cumulus Topology Converter found at the following URL:

    https://github.com/CumulusNetworks/topology_converter

3. The following command was used to run the Topology Converter within the appropriate vx-sim directory:

```
     ./topology_converter.py cumulus-ansible-intermediate-small-dc-2servers.dot -c --provider=virtualbox
     ./topology_converter.py cumulus-ansible-intermediate-small-dc-2servers.dot -c --provider=libvirt
```

After the above command is executed, the following configuration changes are necessary:

4. Within "<vx-sim>/helper_scripts/auto_mgmt_network/OOB_Server_Config_auto_mgmt.sh"

The following stanza:

echo " ### Creating cumulus user ###"
useradd -m cumulus

Will be replaced with the following:

echo " ### Creating cumulus user ###"
useradd -m cumulus -m -s /bin/bash

The following stanza:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.6.3

Will be replaced with the following:

    #Install Automation Tools
    puppet=0
    ansible=1
    ansible_version=2.9.3

Add the following ```echo``` right before the end of the file.

    echo " ### Adding .bash_profile to auto login as cumulus user"
    echo "sudo su - cumulus" >> /home/vagrant/.bash_profile
    echo "exit" >> /home/vagrant/.bash_profile
    echo "### Adding .ssh_config to avoid HostKeyChecking"
    printf "Host * \n\t StrictHostKeyChecking no\n" >> /home/cumulus/.ssh/config

    echo "############################################"
    echo "      DONE!"
    echo "############################################"
