## Raspberry Pi Cluster Guide

Parts of this guide were developed using an article by Garrett Mills titled [Building a Raspberry Pi Cluster](https://glmdev.medium.com/building-a-raspberry-pi-cluster-784f0df9afbd) on Medium.com. Please see this article for further detailed information. The below guide gets right to the deployment instructions. This guide does deviate greatly at points to follow a more linear path for faster deployment and less back and forth between nodes.

---

### A note on microSD card sizes

I will generally used 4GB microSD cards to build my images. Once I am satisfied with my final head node and compute node image I will put them on a larger microSD card such as a 16GB, 32GB, or 64GB card and expand the filesystem using `raspi-config`. This allows for a much smaller image files on your hard drive plus faster read and write times when working with imaging software.

---

### Parts List

- 8 x Raspberry Pi 3 Model B (7 for compute nodes and 1 for head node)
- 8 x MicroSD cards
- 8 x Micro-USB power cables
- 1 x 8-port 10/100/1000 network switch
- 1 x 10-port USB power-supply
- 1 x 128GB USB flash drive

---

### Before starting:

Download and install Raspberry Pi OS Lite. The easiest way to do this is by using the new Raspberry Pi Imager tool provided [here](https://www.raspberrypi.org/software/).

Use this tool to install the Raspberry Pi OS Lite image directly to your microSD card for your head node. Instructions are provided on the above linked page.

---

### Initial configuration

Using a keyboard and monitor you can complete the initial configuration.

Default username: `pi`
Default password: `raspberry`

Setup the locale settings to make sure the correct keyboard, language, timezone, etc are set. This will ensure we are able to enter the correct symbols while working on the command line.

Configure Locale:

Log in with username: **pi** and password **raspberry**

Start the Raspberry Pi configuration tool:

```
sudo raspi-config
```

Setup System Options:

Select `1 System Options`

- Select `S1 Wireless LAN`
  - Select `US United States`
  - Select `Ok`
  - Enter the SSID
  - Enter the passphrase
  - Select `Ok`

Select `1 System Options`

- Select `S4 Hostname`
  - Select `Ok`
  - Enter `nodeX` for the hostname
  - Press `Enter`

Setup Interfacing Options:

Select `3 Interfacing Options`

- Select `P2 SSH`
  - Select `Yes`
  - Select `Ok`
  - Press `Enter`

Setup Performance Options:

- Select `4 Performance Options`
  - Select `P2 GPU Memory`
    - Enter `16`
    - Press `Enter`

Setup Localisation Options:

- Select `5 Localisation Options`

  - Select Locale `L1 Change Locale`
    - Unselect `en_GB.UTF-8 UTF-8`
    - Select `en_US ISO-8859-1`
    - Press `Enter`
    - Select `en_US`

- Select `5 Localisation Options`

  - Select `L2 Change Timezone`
    - Select `US` (or appropriate country)
    - Select `Central` (or appropriate local timezone)

- Select `5 Localisation Options`

  - Select `L3 Change Keyboard Layout`
    - Use the default selected Keyboard
    - Press `Enter`
    - Select `Other`
    - Select `English (US)`
    - Select `English (US)`
    - Select `The default for the keyboard layout`
    - Select `No compose key`
    - Press `Enter`

Setup Advanced Options:

- Select `6 Advanced Options`
  - Select `A1 Expand Filesystem`
  - Select `Ok`

_Tab_ to `Finish`

Select `No` when asked to reboot

---

### Configure Wired Network

Setup _eth0_ static ip address:

Edit _/etc/dhcpcd.conf_:

```
sudo nano /etc/dhcpcd.conf
```

Add to the end of the file:

```
interface eth0
static ip_address=192.168.10.3/24
static domain_name_servers=1.1.1.1
```

Save and exit

---

### Install NTP

This will ensure that the system time is synced and for the SLURM scheduler and Munge authentication.

```
sudo apt install ntpdate -y
```

---

### Update the system

```
sudo apt update && sudo apt upgrade -y
```

---

### Create hosts file

Update _/etc/hosts_ file by adding the following to the end of the file:

_**Note:**_ At this point you want to assign and name all of your nodes that **WILL** be in your cluster and enter them in the hosts file. Below is an example of a 8 node cluster including the head node as one of the six. This file will be copied with the image to the compute nodes and will save you a step of developing and deploying the hosts file later. Notice that the last octet of the ip address correlates to the node name. E.g 192.168.10.**100** on node**0**, 192.168.10.**101** on node**1**, etc.

Edit _/etc/hosts_ file:

```
sudo nano /etc/hosts
```

Modify or add the following lines to the file:

```
127.0.1.1          nodeX

192.168.10.3       nodeX
192.168.10.100     node0
192.168.10.101     node1
192.168.10.102     node2
192.168.10.103     node3
192.168.10.104     node4
192.168.10.105     node5
192.168.10.106     node6
192.168.10.107     node7
```

---

### Reboot

```
sudo reboot
```

---

### Shutdown Raspberry Pi and create generic node image

Shutdown the Raspberry Pi node:

```
sudo shutdown -h now
```

Using [Win32DiskImager](https://sourceforge.net/projects/win32diskimager/) read the node image to your hard drive. Insert the SD card adapter with the SD card from the Raspberry Pi node into a USB port on your computer.

Click the blue folder icon in the `Image File` groupbox.

Select a location and type a name in the filename field (i.e. generic_node_image_2021-01-12.img).

Click the `Open` button and you should see the path and filename in the `Image File` textbox to the left of the blue folder.

Select the device from the `Device` dropdown box.

Click `Read`. Win32DiskImager will now read the microSD card and write the an image to the location and filename you entered in the steps above.

---

### Create another microSD card using the generic node image

This will be the head or master node.

Using the Raspberry Pi Imager software click `Choose OS`. Scroll down the list to the bottom option `Use custom`.

Navigate to the image you created and click on it. Then click `Open`.

Click `Choose SD Card` and select your SD card.

Click `Write` to write the image to the microSD card.

---

### Login to the head node

Using puTTY login to the head node using the ip address `192.168.10.3`.

---

### Configure head node

##### Change the hostname

```
sudo nano /etc/hostname
```

Change `nodeX` to `node0`.

##### Change the ip address

```
sudo nano /etc/dhcpcd.conf
```

Change `192.168.10.3/24` to `192.168.10.100/24`.

##### Change the hosts file

```
sudo nano /etc/hosts
```

Change `127.0.1.1 nodeX` to `127.0.0.1 node0`.

---

### Connect and Mount Flash Drive

##### Find the drive identifier

Plug the flash drive into on of the USB ports on the head node. To figure out its device location use the `lsblk` command.

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    1 115.7G  0 disk
`-sda1        8:1    1 115.7G  0 part
mmcblk0     179:0    0   3.8G  0 disk
|-mmcblk0p1 179:1    0   256M  0 part /boot
`-mmcblk0p2 179:2    0   3.5G  0 part /
```

In this case, the main partition of the flash drive is at `/dev/sda1`.

##### Format the drive

We're first going to format the flash drive to use the `ext4` filesystem:

```
sudo mkfs.ext4 /dev/sda1
```

---

### Setup automatic mounting

To mount our flash drive on boot, we need to find the UUID. To do this, run `blkid` and make note of the UUID from `/dev/sda1` like so:

```
UUID="23b86a23-3c56-47bd-b3b9-f5bdd2ad3931"
```

Now, edit `fstab` to mount the drive on boot:

```
sudo nano /etc/fstab
```

Add the following line to the end of the file:

```
UUID=23b86a23-3c56-47bd-b3b9-f5bdd2ad3931 /hpc ext4 defaults 0 2
```

Finally, mount the drive:

```
sudo mount -a
```

---

### Export the NFS share

Now we need to export the mounted drive as a network file system share so the other nodes can access it.

##### Install the NFS server

```
sudo apt install nfs-kernel-server -y
```

##### Export the NFS share

Edit `/etc/exports` and add the following line:

```
/hpc 192.168.10.0/24(rw,sync,no_root_squash,no_subtree_check)
```

Run the following command to update the NFS kernel server:

```
sudo exportfs -a
```

---

### Create NFS and HPC folder structure

This will create a folder structure that will be shared across the nodes using NFS. This will allow all data files, compiled software libraries, and user files to be shared across the cluster from the head node.

Create hpc group:

```
sudo groupadd hpc
```

Add pi user to hpc group:

```
sudo usermod -aG hpc pi
```

Create hpc directory in root:

```
sudo mkdir /hpc
```

Take ownership of _/hpc_:

```
sudo chown -R pi:hpc /hpc
```

Create hpc subdirectories:

```
cd /hpc

mkdir users data lib
```

Set permissions of _/hpc_ folder:

```
sudo chmod 777 -R /hpc
```

---

### Move _pi_ user to the new home directory

This account is used to move the home directory of the pi user since that users home directory can't be moved while signed in. This is account creation is temporary and will be removed at the end. Additional accounts can be added once configuration is completed to include moving of home directories to the new _/hpc/users/_ folder.

Create a new user account:

```
sudo adduser --home /hpc/users/tempuser tempuser
```

Set the password for the user and save it. No other settings required.

Give need group permissions:

```
sudo usermod -aG hpc tempuser
sudo usermod -aG sudo tempuser
```

Logout of the _pi_ user:

```
exit
```

Login as the new _**tempuser**_ account using puTTY.

Move the _pi_ user home directory:

```
sudo usermod -m -d /hpc/users/pi pi
```

Close this terminal and login as _**pi**_ account again.

Remove the _tempuser_ account and directories:

```
sudo userdel -r tempuser
```

---

### Install SLURM Controller Packages

```
sudo apt install slurm-wlm -y
```

### Slurm Configuration

```
cd /etc/slurm-llnl
sudo cp /usr/share/doc/slurm-client/examples/slurm.conf.simple.gz .
sudo gzip -d slurm.conf.simple.gz
sudo mv slurm.conf.simple slurm.conf
```

##### Set the control machine info

```
sudo nano /etc/slurm-llnl/slurm.conf
```

Set the _SlurmctldHost_ line to the ip address of the head node:

```
SlurmctldHost=node0
```

##### Set the cluster name

Set the _ClusterName_ field:

```
ClusterName=swosucluster
```

##### Add the nodes

Add the following to the end of the file:

```
NodeName=node0 NodeAddr=192.168.10.100 CPUs=4 State=UNKNOWN
NodeName=node1 NodeAddr=192.168.10.101 CPUs=4 State=UNKNOWN
NodeName=node2 NodeAddr=192.168.10.102 CPUs=4 State=UNKNOWN
NodeName=node3 NodeAddr=192.168.10.103 CPUs=4 State=UNKNOWN
NodeName=node4 NodeAddr=192.168.10.104 CPUs=4 State=UNKNOWN
NodeName=node5 NodeAddr=192.168.10.105 CPUs=4 State=UNKNOWN
NodeName=node6 NodeAddr=192.168.10.106 CPUs=4 State=UNKNOWN
NodeName=node7 NodeAddr=192.168.10.107 CPUs=4 State=UNKNOWN
```

##### Create a partition

```
PartitionName=swosucluster Nodes=node[0-7] Default=YES MaxTime=INFINITE State=UP
```

##### Configure cgroups support

Create _/etc/slurm-llnl/cgroup.conf_ file and add the following lines:

```
CgroupMountpoint="/sys/fs/cgroup"
CgroupAutomount=yes
CgroupReleaseAgentDir="/etc/slurm-llnl/cgroup"
AllowedDevicesFile="/etc/slurm-llnl/cgroup_allowed_devices_file.conf"
ConstrainCores=no
TaskAffinity=no
ConstrainRAMSpace=yes
ConstrainSwapSpace=no
ConstrainDevices=no
AllowedRamSpace=100
AllowedSwapSpace=0
MaxRAMPercent=100
MaxSwapPercent=100
MinRAMSpace=30
```

Whitelist system devices by creating the file _/etc/slurm-llnl/cgroup_allowed_devices_file.conf_:

```
/dev/null
/dev/urandom
/dev/zero
/dev/sda*
/dev/cpu/*/*
/dev/pts/*
/hpc*
```

### Copy the Configuration Files to Shared Storage

```
sudo cp slurm.conf cgroup.conf cgroup_allowed_devices_file.conf /hpc
sudo cp /etc/munge/munge.key /hpc
```

### Enable and Start SLURM Control Service

Munge:

```
sudo systemctl enable munge
sudo systemctl start munge
```

The SLURM daemon:

```
sudo systemctl enable slurmd
sudo systemctl start slurmd
```

The control daemon:

```
sudo systemctl enable slurmctld
sudo systemctl start slurmctld
```

---
