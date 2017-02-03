# Raspberry Pi Cluster Setup Guide

---

## References:

https://www.modmypi.com/blog/how-to-give-your-raspberry-pi-a-static-ip-address-update

https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=44609

http://weworkweplay.com/play/automatically-connect-a-raspberry-pi-to-a-wifi-network/

https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=162096

https://www.raspberrypi.org/forums/viewtopic.php?t=118804&p=808453

https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=37575

http://unix.stackexchange.com/questions/88100/importing-data-from-a-text-file-to-a-bash-script

http://www.tldp.org/LDP/abs/html/arrays.html

http://how-to.wikia.com/wiki/How_to_read_command_line_arguments_in_a_bash_script

http://ryanstutorials.net/bash-scripting-tutorial/bash-variables.php

http://stackoverflow.com/questions/19996089/use-ssh-to-start-a-background-process-on-a-remote-server-and-exit-session

http://stackoverflow.com/questions/29142/getting-ssh-to-execute-a-command-in-the-background-on-target-machine

http://www.igeekstudio.com/blog/setting-up-ganglia-and-hadoop-on-raspberry-pi

http://ccm.net/faq/2540-linux-create-your-own-command

https://github.com/XavierBerger/RPi-Monitor

https://www.raspberrypi.org/blog/visualising-core-load-on-the-pi-2/

https://github.com/davidsblog/rCPU

https://raseshmori.wordpress.com/2012/10/14/install-hadoop-nextgen-yarn-multi-node-cluster/


---

## Head Node Setup

##### Hardware:
*	Raspberry Pi board x 1
*	WiPi USB dongle	x 1
*	SD Card 32GB x 1
*	Ethernet cable x 1
*	HDMI cable x 1
*	Power cable mini-USB x 1

### Compute nodes

##### Hardware:
*	Raspberry Pi board x 7
*	SD Card 16GB+ x 1
*	Ethernet cable x 1
*	Power cable mini-USB x 1

##### Additional Hardware

*	10 Port USB hub
*	16 Port gigabit switch                           

---

### Setup, Installation, and Testing

> ##### Step 1 - Install operating systems

*	Install Raspbian Lite on SD card for head unit(s) and each compute node

[Raspbian Install Guides](https://www.raspberrypi.org/documentation/installation/installing-images/)

> ##### Step 2 - Update Raspberry Pi firmware before starting

Install _rpi-update_ package:

``sudo apt-get update rpi-update``

> ##### Step 3 - Configure head node settings

*	Configure Locale
	*	Log in with username: **pi** and password **raspberry**

```
sudo -s

sudo raspi-config
```

Expand the filesystem (_**Option 1**_)

Setup Internationalization Options (_**Option 3**_)


Set Locale (_**Option I1**_)
* Unselect _**en_GB**_
* Select _**en_US ISO-8859-1**_
* Select _**en_US**_


Set TimeZone (_**Option I2**_)
* Select _**America**_
* Select _**Chicago**_


Set Keyboard Layout (_**Option I3**_)
* Use the default selected Keyboard
* Select _**English (US)**_
* Use the default keyboard Layout
* Select _**No compose key**_
* Select _**No**_


Set Wi-Fi country (_**Option I4**_)
* Select _**US United States**_


Set Hostname (_**Option 7**_)
*	Set _Hostname_ (_**Option A2**_)
* Enter _**head**_


Set Memory Split (_**Option 7**_)
* Set _Memory Split_ (_**Option A3**_)
* Enter _**16**_

> ##### Step 4 - Configure head node network

Edit _/etc/dhcpcd.conf_:

``sudo nano /etc/dhcpcd.conf``

Add to the end of the file:

```
interface eth0
static ip_address=192.168.10.5
static domain_name_servers=8.8.8.8
```

Turn on DHCP on _wlan0_.

Edit _/etc/network/interfaces_:

``sudo nano /etc/network/interfaces``

Change:

``iface wlan0 inet manual``

To:

``iface wlan0 inet dhcp``


Add wireless network credentials.

Edit _/etc/wpa_supplicant/wpa_supplicant.conf_:

``sudo nano /etc/wpa_supplicant/wpa_supplicant.conf``

Add to the end of the file:

```
network={
ssid="<network name>"
psk="<network password>"
}
```


###### Turn on SSH service:

``raspi-config``

* Select _**Interfacing Options**_ (_**Option 5**_)
* Select _**SSH**_ (_**Option P2**_)
* Select _**Yes**_
* Select _**Ok**_


Reboot:

``sudo reboot``


> ##### Step 5: Configure compute node.

Log in with username: **pi** and password **raspberry**

```
sudo -s

sudo raspi-config
```

> ##### Step 6 - IP forwarding for nodes to access internet

Enable traffic forwarding and make it permanent:

```
sudo sysctl -w net.ipv4.ip_forward=1

sudo sed -i "s/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g" /etc/sysctl.conf

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

sudo bash -c "iptables-save > /etc/iptables.rules"
```

Add settings to _/etc/network/interfaces_:

``sudo nano /etc/network/interfaces``

Add the following line at the end of the wlan0 section under wpa-conf line to make the changes persistent:

``pre-up iptables-restore < /etc/iptables.rules``

> ##### Step 7 - Update and upgrade

Update all packages:

``sudo apt-get update && sudo apt-get upgrade -y``

Reboot:

``sudo reboot``


> ##### Step 8 - Setup SSH and keys

**_Note:_ MAKE SURE TO EXIT OUT OF ROOT ACCESS IF YOU ARE IN IT BEFORE STARTING THE NEXT PART**

Generate SSH key:

```
cd ~

ssh-keygen -t rsa -C "raspi2@swosubta"
```

_**Enter**_ to select default location
_**Enter**_ to leave passphrase blank
_**Enter**_ to confirm blank passphrase

Copy SSH keys to _authorized_keys_ file:

``cat /home/pi/.ssh/id_rsa.pub >> /home/pi/.ssh/authorized_keys``


---

## Install MPICH3

> ##### Step 1 - Install Fortran

```
sudo apt-get install gfortran
mkdir /home/pi/fortran
```

> ##### Step 2 - Install and Setup MPICH3

Execute from command line:

```
cd ~

mkdir mpich3

cd mpich3
mkdir build install

wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
tar xvfz mpich-3.2.tar.gz

cd build

/home/pi/mpich3/mpich-3.2/configure --prefix=/home/pi/mpich3/install

make
make install

export PATH=$PATH:/home/pi/mpich3/install/bin
```

Add path to environment variables:

``sudo nano /home/pi/.bashrc``

Add the following to the end of the file:

```
# MPI
export PATH="$PATH:/home/pi/mpich3/install/bin"
```

> ##### Step 3 - Create list of nodes for MPI:

Create node list:

```
cd ~
sudo nano nodelist
```

Add the head node ip address to the list:

``192.168.10.5``

_**Note:**_ Anytime you need to add a node to the cluster make sure to add it here as well as _/etc/hosts_ file.

> ##### Step 4 - Test MPI

###### Test 1 - Hostname Test

```
cd ~

mpiexec -f nodelist Hostname
```

Should return:

``head``

###### Test 2 - Calculate Pi

``mpiexec -f nodelist -n 2 ~/mpich3/build/examples/cpi``


Should return similar:

```
Process 0 of 2 is on head
Process 1 of 2 is on head
pi is approximately 3.1415926544231318, Error is 0.0000000008333387
wall clock time = 0.003250
```

---
## Install Slurm

> ##### Step 1 - Install Slurm

``sudo apt-get install slurm-wlm slurmctld slurmd``

> ##### Step 2 - Add configuration file

Delete the current Slurm configuration:

``rm /etc/slurm-llnl/slurm.conf``

Create the new Slurm configuration file _/etc/slurm-llnl/slurm.conf_:

``sudo nano /etc/slurm-llnl/slurm.conf``

Add the following to the file and save:

```
# slurm.conf file generated by configurator easy.html.
# Put this file on all nodes of your cluster.
# See the slurm.conf man page for more information.
#
ControlMachine=head
ControlAddr=192.168.10.5
#
#MailProg=/bin/mail
MpiDefault=none
#MpiParams=ports=#-#
ProctrackType=proctrack/pgid
ReturnToService=2
SlurmctldPidFile=/var/run/slurm-llnl/slurmctld.pid
#SlurmctldPort=6817
SlurmdPidFile=/var/run/slurm-llnl/slurmd.pid
#SlurmdPort=6818
SlurmdSpoolDir=/var/lib/slurm/slurmd
SlurmUser=slurm
#SlurmdUser=root
StateSaveLocation=/var/lib/slurm/slurmctld
SwitchType=switch/none
TaskPlugin=task/none
#
#
# TIMERS
#KillWait=30
#MinJobAge=300
#SlurmctldTimeout=120
#SlurmdTimeout=300
#
#
# SCHEDULING
FastSchedule=1
SchedulerType=sched/backfill
#SchedulerPort=7321
SelectType=select/linear
#
#
# LOGGING AND ACCOUNTING
AccountingStorageType=accounting_storage/none
ClusterName=raspi2
#JobAcctGatherFrequency=30
JobAcctGatherType=jobacct_gather/none
#SlurmctldDebug=3
SlurmctldLogFile=/var/log/slurm/slurmctld.log
#SlurmdDebug=3
SlurmdLogFile=/var/log/slurm/slurmd.log
#
#
# COMPUTE NODES
NodeName=head NodeAddr=192.168.10.5
NodeName=node1 NodeAddr=192.168.1.100
PartitionName=All Default=yes Nodes=head,node1 State=UP
```

_**Note:**_ Any nodes added to the cluster need to be added to the bottom of this file with a _NodeName_ entry.

Check if Slurm controller is running:

``scontrol show daemons``

Should return:

``slurmctld``

> ##### Step 3 - Create Munge key

``sudo /usr/sbin/create-munge-key``

Agree to overwrite.

> ##### Step 4 - Finish installs and start services

```
sudo systemctl enable slurmctld.service
sudo ln -s /var/lib/slurm-llnl /var/lib/slurm-llnl
sudo systemctl start slurmctld.service
```

Verify Slurm controller is running:

``sudo systemctl status slurmctld.service``

Will return feedback to the screen. Verify _Active_ line is _**active (running)**_.

> ##### Step 5 - Add user to Slurm group

``sudo adduser pi slurm``

> ##### Step 6 - Add and take ownership of Slurm log folder

```
sudo mkdir -p /var/log/slurm/accounting

sudo chown -R slurm:slurm /var/log/slurm
```
