# Raspberry Pi Cluster Setup Guide
## Using Raspbian Jessie Lite 2016-11-25

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

## Considerations to Consider before starting

* SD card size
	* If your SD card size will vary you will want to build the head node using the smallest size of SD card. This will ensure that the image for that SD card will ALWAYS be able to be written to a similar sized SD or larger. If you start with a 64GB SD card you will not be able to write the image to a 16GB SD card.

---

## Head Node

##### Hardware:
*	Raspberry Pi board x 1
*	WiPi USB dongle	x 1
*	SD Card 32GB x 1
*	Ethernet cable x 1
*	HDMI cable x 1
*	Power cable mini-USB x 1

## Compute nodes

##### Hardware:
*	Raspberry Pi board x 7
*	SD Card 16GB+ x 1
*	Ethernet cable x 1
*	Power cable mini-USB x 1

## Additional Hardware

*	10 Port USB hub
*	16 Port gigabit switch                           

---

## Setup, Installation, and Testing

> ##### Step 1 - Install operating systems

Install Raspbian Lite on SD card for head unit(s) and each compute node

[Raspbian Install Guides](https://www.raspberrypi.org/documentation/installation/installing-images/)

> ##### Step 2 - Configure head node settings

Setup the locale settings to make sure the correct keyboard, language, timezone, etc are set. This will ensure we are able to enter the correct symbols while working on the command line.

Configure Locale:

Log in with username: **pi** and password **raspberry**

```
sudo raspi-config
```

Expand the filesystem (_**Option 1**_)
* Select _**Yes**_

Setup Internationalization Options (_**Option 3**_)

* Set Locale (_**Option I1**_)
	* Unselect _**en_GB**_
	* Select _**en_US ISO-8859-1**_
	* Select _**en_US**_


* Set TimeZone (_**Option I2**_)
	* Select _**America**_
	* Select _**Chicago**_


* Set Keyboard Layout (_**Option I3**_)
	* Use the default selected Keyboard
	* Select _**English (US)**_
	* Use the default keyboard Layout
	* Select _**No compose key**_
	* Select _**No**_


* Set Wi-Fi country (_**Option I4**_)
	* Select _**US United States**_


* Set Hostname (_**Option 7**_)
	*	Set _Hostname_ (_**Option A2**_)
	* Enter _**head**_


* Set Memory Split (_**Option 7**_)
	* Set _Memory Split_ (_**Option A3**_)
	* Enter _**16**_
* ##### Setup SSH service:

* Select _**Interfacing Options**_ (_**Option 7**_)
	* Select _**SSH**_ (_**Option P2**_)
	* Select _**Yes**_
	* Select _**Ok**_

Select _Finish_ and _Yes_ to reboot

> ##### Step 3 - Configure head node network

Set a static address for the cluster facing network interface connection _etho0_. Turn on wireless and setup wireless connection on network interface connection _wlan0_. Turn on SSH service and then reboot the head node.

##### Setup _eth0_:

Edit _/etc/dhcpcd.conf_:

``sudo nano /etc/dhcpcd.conf``

Add to the end of the file:

```
interface eth0
static ip_address=192.168.10.5
static domain_name_servers=8.8.8.8
```

##### Setup _wlan0_:

Add wireless network credentials:

Edit _/etc/wpa_supplicant/wpa_supplicant.conf_:

``sudo nano /etc/wpa_supplicant/wpa_supplicant.conf``

For connecting to a secure network add the following to the end of the file:

```
network={
ssid="<network name>"
psk="<network password>"
}
```
For connecting to an unsecure network add the following to the end of the file:

```
network={
ssid="<network name>"
key_mgmt=NONE
}
```

Reboot:

``sudo reboot``

> ##### Step 4 - Update the system

``sudo apt-get update && sudo apt-get upgrade -y``

Reboot:

``sudo reboot``

> ##### Step 5 - IP forwarding for nodes to access internet

Setup IP forwarding so that all compute nodes will have access to the internet for package installation and to download any needed materials on later use.

Log in with username: **pi** and password **raspberry**

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

Save and exit

Disable IPv6:

``sudo nano /etc/sysctl.conf``

Add the following lines to the end of the file (this includes the IP forwarding rule from above):

```
# Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Save and exit

Update the configuration files:

``sudo sysctl -p``

Update _etc/hosts_ file:

Add the following to the end of the file:

_**Note:**_ At this point you want to assign and name all of your nodes that **WILL** be in your cluster and enter them in the hosts file. Below is an example of a 6 node cluster including the head node as one of the six. This file will be copied with the image to the compute nodes and will save you a step of developing and deploying the hosts file later.

```
192.168.10.3		nodeX
192.168.10.5		head
192.168.10.100	node0
192.168.10.101	node1
192.168.10.102	node2
192.168.10.103	node3
192.168.10.104	node4
```

Reboot:

``sudo reboot``

---

## Install MPICH3

Install prerequisite _Fortran_ which wil be required for compiling MPICH. All other dependencies are already installed.

> ##### Step 1 - Install Fortran

```
sudo apt-get install gfortran
mkdir /home/pi/fortran
```

> ##### Step 2 - Install and Setup MPICH3

Execute from command line:

Create folder structure for _MPICH3_.

```
cd ~

mkdir mpich3

cd mpich3
mkdir build install
```

Download MPICH3 and untar:

```
wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
tar xvfz mpich-3.2.tar.gz
```

Compile and install _MPICH3_

```
cd build

/home/pi/mpich3/mpich-3.2/configure --prefix=/home/pi/mpich3/install

make
make install
```

Activate environment variable:

```
export PATH=$PATH:/home/pi/mpich3/install/bin
```

Add path to environment variables for persistance:

``sudo nano /home/pi/.bashrc``

Add the following to the end of the file:

```
# MPI
export PATH="$PATH:/home/pi/mpich3/install/bin"
```

> ##### Step 3 - Create list of nodes for MPI:

This list of nodes will need to be updated as you add nodes later. Initially you will only have the head node.

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

mpiexec -f nodelist hostname
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

## Save SD Image

At this point you will want to save an image of the head node. This will give you a fall back point if you make mistakes moving forward. You will also use this image to begin your node image.

Using the same guide as described in the beginning you will want to reverse the process of writing an image to the SD and _read_ an image from the SD and save that image to your PC. Now you have saved your SD like a checkpoint.

## Create Node image

The overview of this process:

1. Save image of _head node_.
2. On a new SD card write the _head node_ image you just saved.
3. Boot the second SD you just created from the head node and make the following changes for "Creating a Generic Node Image".
4. Save image of newly created _generic compute node_.

At this point you have a copy of both the _head node_ and _generic comput node_ at the MPI stage. This is a checkpoint that you can fall back to if there are errors after this point.

This will be a repeatable process when completed. You will setup an initial _Compute Node_ image using your saved _Head Node_ image. You will go in and change specific settings to _generic settings_. Doing this will allow you to always access your _generic Compute Node_ image at the same IP address and hostname. You will then be able to set up the compute node image to a specific IP address and hostname. Following this process will allow for prompt and efficient deployment of a cluster.

[Raspbian Install Guides](https://www.raspberrypi.org/documentation/installation/installing-images/)

---

## Create Generic Node image

This will be a repeatable process when completed. You will setup an initial _Compute Node_ image using your saved _Head Node_ image. You will go in and change specific settings to _generic settings_. Doing this will allow you to always access your _generic Compute Node_ image at the same IP address and hostname. You will then be able to set up the compute node image to a specific IP address and hostname. Following this process will allow for prompt and efficient deployment of a cluster.

> ##### Step 1 - Boot image and login

Log in with username: **pi** and password **raspberry**

> ##### Step 2 - Enter a generic ip address

``sudo nano /etc/dhcpcd.conf``

Change the _eth0_ ip address from:

``static ip_address=192.168.10.5``

To:

``static ip_address=192.168.10.3``

Also add to the end of the file:

``static routers=192.168.10.5``

Save and exit

> ##### Step 3 - Enter a generic hostname

``sudo nano /etc/hostname``

Change:

``head``

To:

``nodeX``

Save and exit

> ##### Step 4 - Shutdown and create a new image of the SD

``sudo shutdown -h now``

Now you will go back to WinDiskImager32 and save the image as a node image. This is a generic node image that you can quickly deploy and use to set up your cluster with.

Sample name for SD image: ``head_node_mpi_stage_2017_01_03``

---

## Setup Generic Node image

[Raspbian Install Guides](https://www.raspberrypi.org/documentation/installation/installing-images/)

> ##### Step 1 - Copy generic node image created earlier to an SD card using WinDiskImager32.

> ##### Step 2 - Boot and login to your system

Log in with username: **pi** and password **raspberry**

> ##### Step 3 - Adjust _/etc/hostname_ file

``sudo nano /etc/hostname``

Change:

``nodeX``

To:

``node0``

Save and exit

_**Note:**_ This number will increment by one each time you add a node and must be unique on your cluster.

> ##### Step 4 - Adjust _/etc/dhcpcd.conf_

``sudo nano /etc/dhcpcd.conf``

Change the _eth0_ ip address from:

``static ip_address=192.168.10.3``

To:

``static ip_address=192.168.10.100``

Save and exit

_**Note:**_ At this point you will just do this once to develop a compute node image with Slurm installed. After that is complete you will create a new generic image of the compute node. Once that is complete you can use that image to finish deploying your compute nodes for the rest of your cluster.

---
## Install Slurm on Head Node

> ##### Step 1 - Install Slurm

``sudo apt-get install slurm-wlm slurmctld slurmd``

> ##### Step 2 - Add configuration file

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
NodeName=node0 NodeAddr=192.168.10.100
NodeName=node1 NodeAddr=192.168.10.101
NodeName=node2 NodeAddr=192.168.10.102
NodeName=node3 NodeAddr=192.168.10.103
NodeName=node4 NodeAddr=192.168.10.104
PartitionName=raspi2 Default=yes Nodes=head,node0,node1,node2,node3,node4 State=UP
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
sudo ln -s /var/lib/slurm-llnl /var/lib/slurm
sudo systemctl start slurmctld.service
sudo systemctl enable munge.service
```

Verify Slurm controller is running:

``sudo systemctl status slurmctld.service``

Will return feedback to the screen. Verify _Active_ line is _**active (running)**_.

Verify Munge is running:

``sudo systemctl status munge.service``

Will return feedback to the screen. Verify _Active_ line is _**active (running)**_.

> ##### Step 5 - Add user to Slurm group

``sudo adduser pi slurm``

> ##### Step 6 - Add and take ownership of Slurm log folder

```
sudo mkdir -p /var/log/slurm/accounting

sudo chown -R slurm:slurm /var/log/slurm
```
sinfo
## Install Slurm on Compute Node

> ##### Step 1 - Copy Slurm configuration and Munge files from _Head Node_

**On _head node_**

``sudo cat /etc/munge/munge.key | ssh pi@node0 "cat >> ~/munge.key"``

``sudo cat /etc/slurm-llnl/slurm.conf | ssh pi@node0 "cat >> ~/slurm.conf"``

> ##### Step 2 - Install Slurm daemon

**On _node0 node_**

SSH into _node0_

```
sudo apt-get install slurmd slurm-client
sudo ln -s /var/lib/slurm-llnl /var/lib/slurm
```

Copy Slurm configuration file and Munge key file to proper location:

``sudo cp ~/slurm.conf /etc/slurm-llnl/``

``sudo cp ~/munge.key /etc/munge/``

Finish install and start Slurm and Munge:

```
sudo systemctl enable slurmd.service
sudo systemctl restart slurmd.service
sudo systemctl enable munge.service
sudo systemctl restart munge.service
sudo systemctl status slurmd.service
```

Verify Slurm daemon is running:

``sudo systemctl status slurmd.service``

Will return feedback to the screen. Verify _Active_ line is _**active (running)**_.

Verify Munge is running:

``sudo systemctl status munge.service``

Will return feedback to the screen. Verify _Active_ line is _**active (running)**_.

> ##### Step 3 - Add user to Slurm group

``sudo adduser pi slurm``

> ##### Step 4 - Add and take ownership of Slurm log folder

```
sudo mkdir -p /var/log/slurm/accounting

sudo chown -R slurm:slurm /var/log/slurm
```

> ##### Step 5 - Setup SSH and keys

**On each compute node**

Generate SSH key:

```
cd ~

ssh-keygen -t rsa -C "raspi2@swosubta"
```

_**Enter**_ to select default location
_**Enter**_ to leave passphrase blank
_**Enter**_ to confirm blank passphrase

**On head node**

Setup an SSH key that will be used by the entire cluster to eliminate the need to sign in to each node individually while setting up and using the cluster.

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

``cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys``

Copy _authorized_keys_ file to all nodes:

``sudo cat ~/.ssh/authorized_keys | ssh pi@node0 "cat >> ~/.ssh/authorized_keys"``

---

## Deploying the Rest of the Cluster

By now you have developed a head node image that contains both MPI and Slurm. You have also developed a compute node image that contains both MPI and Slurm as well. Now you should go back to the instructions for "Create Node Image" to save both images and then use the compute node image to finish deploying your cluster. Saving these images at each stage gives you different configurations that you can easily deploy in the future and also allows you to have a checkpoint in case something goes wrong. You can write the saved node image to your SD and start from that point rather then starting from the beginning.
