# Raspberry Pi Cluster Setup Guide

---

### Considerations to Consider before starting

**USB cable power throughput**

Lower quality USB cables can restrict power throughput causing undervolt situations when the cluster is underload.

**SD card size**

Starting off with smaller SD cards will ensure that images are limited to a smaller size when saving to an external PC. It will also ensure that the images can be written to larger SD cards once configuration is completed.

---

### Overview of process

1. Install Raspberry Pi Lite OS to 4GB SD card and configure the install:

- EXTERNAL PC
  - Write image of Raspberry Pi Lite OS to 4GB SD card

* HEAD NODE
  - Boot Raspberry Pi with SD card inserted
  - Configure Raspberry Pi
  - Configure Wired Network
  - Update System
  - Create hosts file
  - Create NFS and HPC folder structure
  - Move pi user to new home directory
  - Install MPICH
  - Create Node List
  - Generate SSH key

2. Save an image of the configured base Raspberry Pi OS Lite install:

- EXTERNAL PC
  - Save image of head node

3. Write a copy of the configured image to a new 4GB SD card:

- EXTERNAL PC
  - Write image of generic node (not configured)

4. Configure generic node:

- GENERIC NODE
  - Configure generic node
  - Configure generic ip address
  - Congigure generic hostname
  - Edit hosts file

5. Save an image of the configured generic node:

- EXTERNAL PC
  - Save image of generic node (configured)

6. Install additional head node software:

- HEAD NODE
  - Configure head node for NFS
  - Configure head node for NTP
  - Configure head node for SLURM

7. Save image of completely configured head node:

- EXTERNAL PC
  - Save image of head node (Base configuration, NFS, NTP, SLURM)

8. Install additional compute node software:

- GENERIC NODE
  - Configure compute node for NFS
  - Configure compute node for NTP
  - Configure compute node for SLURM

9. Save image of completely configured generic compute node:

- EXTERNAL PC
  - Save image of generic node (Base configuration, NFS, NTP, SLURM)

10. Deploy remaining nodes using generic compute node image:

- COMPUTE NODE(S)
  - Deploy remaining nodes by configuring generic image to compute node

---

**Head Node**

Hardware:

- Raspberry Pi board x 1
- SD Card 16GB+ x 1
- Ethernet cable x 1
- HDMI cable x 1
- Power cable mini-USB x 1
- 128GB or larger flash drive/external hard drive/external SSD drive

**Compute nodes**

Hardware:

- Raspberry Pi board x 7
- SD Card 16GB+ x 7
- Ethernet cable x 7
- Power cable mini-USB x 7

**Additional Hardware**

- 10 Port USB hub for power
- 16 Port gigabit switch
- 2 4GB SD cards

---

### Setup, Installation, and Testing

**Install operating systems**

Install Raspberry Pi OS Lite on SD card for head unit(s) and each compute node

---

### Configure Raspberry Pi

Setup the locale settings to make sure the correct keyboard, language, timezone, etc are set. This will ensure we are able to enter the correct symbols while working on the command line.

Configure Locale:

Log in with username: **pi** and password **raspberry**

Start the Raspberry Pi configuration tool:

```
sudo raspi-config
```

Setup System Options:

Select **1 System Options**

- Select **S1 Wireless LAN**
  - Select **US United States**
  - Select **Ok**
  - Enter the SSID
  - Enter the passphrase
  - Select **Ok**

Select **1 System Options**

- Select **S4 Hostname**
  - Select **Ok**
  - Enter **node0** for the hostname
  - Press **Enter**

Setup Interfacing Options:

Select **3 Interfacing Options**

- Select **P2 SSH**
  - Select **Yes**
  - Select **Ok**
  - Press **Enter**

Setup Performance Options:

- Select **4 Performance Options**
  - Select **P2 GPU Memory**
    - Enter **16**
    - Press **Enter**

Setup Localisation Options:

- Select **5 Localisation Options**

  - Select Locale **L1 Change Locale**
    - Unselect **en_GB.UTF-8 UTF-8**
    - Select **en_US ISO-8859-1**
    - Press **Enter**
    - Select **en_US**

- Select **5 Localisation Options**

  - Select **L2 Change Timezone**
    - Select **US** (or appropriate country)
    - Select **Central** (or appropriate local timezone)

- Select **5 Localisation Options**

  - Select **L3 Change Keyboard Layout**
    - Use the default selected Keyboard
    - Press **Enter**
    - Select **Other**
    - Select **English (US)**
    - Select **English (US)**
    - Select **The default for the keyboard layout**
    - Select **No compose key**
    - Press **Enter**

Setup Advanced Options:

- Select **6 Advanced Options**
  - Select **A1 Expand Filesystem**
  - Select **Ok**

_Tab_ to **Finish**

Select **No** when asked to reboot

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
static ip_address=192.168.10.100/24
static domain_name_servers=1.1.1.1
```

Save and exit

Reboot:

```
sudo reboot
```

At this point you can connect via the network using puTTY or similar SSH utility.

---

### Update the system

```
sudo apt update && sudo apt upgrade -y
```

---

### Create hosts file

Update _/etc/hosts_ file by adding the following to the end of the file:

_**Note:**_ At this point you want to assign and name all of your nodes that **WILL** be in your cluster and enter them in the hosts file. Below is an example of a 6 node cluster including the head node as one of the six. This file will be copied with the image to the compute nodes and will save you a step of developing and deploying the hosts file later.

Edit _/etc/hosts_ file:

```
sudo nano /etc/hosts
```

Modify or add the following lines to the file:

```
127.0.1.1    head

192.168.10.3    nodeX
192.168.10.100    node0
192.168.10.101    node1
192.168.10.102    node2
192.168.10.103    node3
192.168.10.104    node4
192.168.10.105     node5
192.168.10.106     node6
192.168.10.107     node7
```

---

### Create NFS and HPC folder structure

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

Create _/nfs_share_ folder:

```
sudo mkdir /nfs_share
```

Set ownership of _/nfs_share_ folder:

```
sudo chown -R pi:hpc /nfs_share
```

Create folders under _/nfs_share_ folder:

```
cd /nfs_share

mkdir data users
```

Set permissions of _/nfs_share_ folder:

```
sudo chmod 777 -R /nfs_share
```

---

### Move _pi_ user to new home directory

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

Reboot the Raspberry Pi and login as the new _tempuser_ account.

Move the _pi_ user home directory:

```
sudo usermod -m -d /hpc/users/pi pi
```

Close this terminal and login as _pi_ account again.

Remove the _tempuser_ account and directories:

```
sudo userdel -r tempuser
```

---

### Install MPICH

Install prerequisite _Fortran_ which wil be required for compiling MPICH. All other dependencies are already installed.

1. Install Fortran

```
sudo apt install gfortran -y
```

2. Create build and install directory inside mpich3 directory:

```
cd /hpc/lib

mkdir mpich_3.3.2

cd mpich_3.3.2

mkdir build install
```

3. Download mpich3 and unzip:

```
wget http://www.mpich.org/static/downloads/3.3.2/mpich-3.3.2.tar.gz

tar xvfz mpich-3.3.2.tar.gz
```

4. Compile and install mpich3:

```
cd build

/hpc/lib/mpich_3.3.2/mpich-3.3.2/configure --prefix=/hpc/lib/mpich_3.3.2/install

make

make install
```

5. Activate environment variable:

```
export PATH=/hpc/lib/mpich_3.3.2/install/bin:$PATH
```

6. Add path to environment variables for persistance:

```
sudo nano ~/.bashrc
```

Add the following to the end of the file:

```
# MPICH-3.3.2
export PATH="/hpc/lib/mpich_3.3.2/install/bin:$PATH"
```

Save and exit.

7. Create list of nodes for MPI:

This list of nodes will need to be updated as you add nodes later. Initially you will only have the head node.

Create node list:

```
cd ~
nano nodelist
```

Add the head node ip address to the list:

```
192.168.10.100
```

_**Note:**_ Anytime you need to add a node to the cluster make sure to add it here as well as _/etc/hosts_ file.

8. Test MPI

Test 1 - Hostname Test

Enter on command line:

```
cd ~

mpiexec -f nodelist hostname
```

Output:

```
node0
```

Test 2 - Calculate Pi

Enter on command line:

```
mpiexec -f nodelist -n 2 /hpc/lib/mpich_3.3.2/build/examples/cpi
```

Output:

```
Process 0 of 2 is on node0
Process 1 of 2 is on node0
pi is approximately 3.1415926544231318, Error is 0.0000000008333387
wall clock time = 0.003250
```

---

### Generate SSH key

_**Note:**_ _Must be executed from head node as pi user_

Generate SSH key:

```
cd ~

ssh-keygen -t rsa -C "<username>@swosubta" -f ~/.ssh/id_rsa
```

Press 'Enter' for passphrase

Press 'Enter' for same passphrase

Transfer the key to the authorized_keys file:

```
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```

---

### Save image of head node

At this point you will want to save an image of the head node. This will give you a fall back point if you make mistakes moving forward. You will also use this image to begin your node image.

**INSERT INSTRUCTIONS HERE**

---

**REVIEW**

### Write compute node image (not configured)

The overview of this process:

1. Save image of _head node_.
2. On a new SD card write the _head node_ image you just saved.
3. Boot the second SD you just created from the head node and make the following changes for "Creating a Generic Node Image".
4. Save image of newly created _generic compute node_.

At this point you have a copy of both the _head node_ and _generic comput node_ at the MPI stage. This is a checkpoint that you can fall back to if there are errors after this point.

This will be a repeatable process when completed. You will setup an initial _compute node_ image using your saved _head node_ image. You will go in and change specific settings to _generic settings_. Doing this will allow you to always access your _generic compute node_ image at the same IP address and hostname. You will then be able to set up the compute node image to a specific IP address and hostname. Following this process will allow for prompt and efficient deployment of a cluster.

---

**REVIEW**

---

### Configure generic node

This will be a repeatable process when completed. You will setup an initial _compute node_ image using your saved _head node_ image. You will go in and change specific settings to _generic settings_. Doing this will allow you to always access your _generic compute node_ image at the same IP address and hostname. You will then be able to set up the compute node image to a specific IP address and hostname. Following this process will allow for prompt and efficient deployment of a cluster.

_**NOTE:**_ This image will have to be booted from the head node to remove previously configured systems. These systems will be replaced with client systems for the compute nodes.

Boot image and login:

Log in with username: **pi** and password **raspberry**

### Configure generic ip address

Enter a generic ip address:

```
sudo nano /etc/dhcpcd.conf
```

Change the _eth0_ ip address from:

```
static ip_address=192.168.10.100
```

To:

```
static ip_address=192.168.10.3
```

Save and exit

### Configure generic hostname

Enter a generic hostname:

```
sudo nano /etc/hostname
```

Change:

```
node0
```

To:

```
nodeX
```

Save and exit

### Edit hosts file

Edit hosts file:

```
sudo nano /etc/hosts
```

Change:

```
127.0.1.1                node0
```

To:

```
127.0.1.1                nodeX
```

Save and exit

---

### Save image of generic node (configured)

Shutdown and create a new image of the SD:

```
sudo shutdown -h now
```

Now you will go back to WinDiskImager32 and save the image as a node image. This is a generic node image that you can quickly deploy and use to set up your cluster with.

Sample name for SD image:

```
compute_node_mpi_stage_2017_01_03
```

---

### Head node setup

### Mount USB flash drive/External drive for home directories on head node

1. Plug the storage device into a USB port on the Raspberry Pi

2. List all of the disk partitions on the Pi

```
sudo lsblk -o UUID,NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL,MODEL
```

3. Identify the disk properties

Identify and save the following disk properties:

- UUID (unique identifier for the device: 28b98a23-1d12-4fbc-b205-e5ff225bd06a; will be different for your device)
- Name (possibly _sda1_; used in the following steps; may differ on your configuration)
- FSTYPE (ext4)

4. Format the drive:

Replace _sda1_ as needed for your configuration on the following steps.

```
sudo mkfs.ext4 /dev/sda1
```

4. Mount the drive

```
sudo mount /dev/sda1 /nfs_share
```

4. Setup automatic mounting on Raspberry Pi boot

```
sudo nano /etc/fstab
```

Add the following line, being sure to add your UUID from step 4:

```
/dev/sda1 /nfs_share nfs defaults,auto,rw,nofail 0 0
```

---

### Install NFS server on head node

1. Install NFS server package:

```
sudo apt install nfs-kernel-server -y
```

2. Create share folders:

```
cd /nfs_share

sudo mkdir data users
```

3. Take ownership of /nfs_share folder:

```
sudo chown -R pi:hpc /nfs_share
```

4. Mount and bind users directory:

```
sudo mount --bind /hpc/users /nfs_share/users
```

5. Add reboot persistance:

Open _/etc/fstab_ file:

```
sudo nano /etc/fstab
```

Add file information to the end of the file:

```
/hpc/users     /nfs_share/users   none      bind 0    0
/hpc/data     /nfs_share/data     none      bind 0    0
```

6. Add the share folders to _/etc/exports_ file:

Open _/etc/exports_ file:

```
sudo nano /etc/exports
```

Add share information to the end of the file:

```
/nfs_share              192.168.10.0/24(rw,fsid=0,no_subtree_check,sync)
/nfs_share/users        192.168.10.0/24(rw,nohide,insecure,no_subtree_check,sync)
/nfs_share/data         192.168.10.0/24(rw,nohide,insecure,no_subtree_check,sync)
```

Enable new shares from NFS server:

```
sudo exportfs -ra
```

7. Restart the service:

```
sudo service nfs-kernel-server restart
```

8. Check that share folders are correct and shared:

```
showmount -e
```

Output:

```
Export list for head:
/nfs_share/data  192.168.10.0/24
/nfs_share/users 192.168.10.0/24
/nfs_share       192.168.10.0/24
```

References:

https://help.ubuntu.com/community/NFSv4Howto

https://linuxize.com/post/how-to-mount-and-unmount-file-systems-in-linux/

---

### Install NTP on head node

NTP is used to keep the cluster time close together using outside NTP servers to sync with the head node. All computer nodes will sync with the head node.

Reference: <http://raspberrypi.tomasgreno.cz/ntp-client-and-server.html> <http://www.pool.ntp.org/zone/north-america>

Install NTP:

```
sudo apt install ntp -y
```

Edit the _/etc/ntp.conf_:

```
sudo nano /etc/ntp.conf
```

Change:

```
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst
```

To:

```
server 0.north-america.pool.ntp.org
server 1.north-america.pool.ntp.org
server 2.north-america.pool.ntp.org
server 3.north-america.pool.ntp.org
```

Restart NTP:

```
sudo /etc/init.d/ntp restart
```

---

### Install Slurm on head node (node0)

1. Install Slurm

```
sudo apt install slurm-wlm -y
```

2. Add configuration file

Create the new Slurm configuration file _/etc/slurm-llnl/slurm.conf_:

```
sudo nano /etc/slurm-llnl/slurm.conf
```

Add the following to the file and save:

```
# Cluster configuration
ControlMachine=node0
AuthType=auth/munge
CacheGroups=0
CryptoType=crypto/munge
JobCheckpointDir=/var/lib/slurm-llnl/checkpoint
MpiDefault=none
ProctrackType=proctrack/pgid
ReturnToService=1
SlurmctldPidFile=/var/run/slurm-llnl/slurmctld.pid
SlurmctldPort=6817
SlurmdPidFile=/var/run/slurm-llnl/slurmd.pid
SlurmdPort=6818
SlurmdSpoolDir=/var/lib/slurm-llnl/slurmd
SlurmUser=slurm
StateSaveLocation=/var/lib/slurm-llnl/slurmctld
SwitchType=switch/none
TaskPlugin=task/none
InactiveLimit=0
KillWait=30
MinJobAge=300
SlurmctldTimeout=300
SlurmdTimeout=300
Waittime=0
FastSchedule=1
SchedulerType=sched/backfill
SchedulerPort=7321
SelectType=select/linear
AccountingStorageType=accounting_storage/none
ClusterName=picluster
JobCompType=jobcomp/none
JobAcctGatherFrequency=30
JobAcctGatherType=jobacct_gather/none
SlurmctldDebug=3
SlurmctldLogFile=/var/log/slurm-llnl/slurmctld.log
SlurmdDebug=3
SlurmdLogFile=/var/log/slurm-llnl/slurmd.log
NodeName=node0 Procs=4 State=UNKNOWN
PartitionName=picluster Nodes=node0 Default=YES MaxTime=INFINITE State=UP
```

Take ownership of configuration folder:

```
sudo chown -R slurm:slurm /etc/slurm-llnl/
```

3. Enable Munge service:

```
systemctl enable munge

munge -n
```

4. Test Munge:

```
sudo munge -n | unmunge
```

Output:

```
STATUS:           Success (0)
ENCODE_HOST:      node0 (127.0.1.1)
ENCODE_TIME:      2020-12-21 10:28:35 -0600 (1608568115)
DECODE_TIME:      2020-12-21 10:28:35 -0600 (1608568115)
TTL:              300
CIPHER:           aes128 (4)
MAC:              sha256 (5)
ZIP:              none (0)
UID:              root (0)
GID:              root (0)
LENGTH:           0
```

5. Correct Slurm configuration issues:

Create _/var/run/slurm-llnl_ folder and take ownership:

```
sudo mkdir /var/run/slurm-llnl

sudo chown -R slurm:slurm /var/run/slurm-llnl
```

Edit systemd file for slurmd:

```
sudo nano /usr/lib/systemd/system/slurmd.service
```

Change the following line:

```
PIDFile=/run/slurmd.pid
```

to

```
PIDFile=/var/run/slurm-llnl/slurmd.pid
```

Edit systemd file for slurmctld:

```
sudo nano /usr/lib/systemd/system/slurmctld.service
```

Change the following line:

```
PIDFile=/run/slurmctld.pid
```

to

```
PIDFile=/var/run/slurm-llnl/slurmctld.pid
```

Reload daemon units:

```
sudo systemctl daemon-reload
```

Start Slurm controller service:

```
sudo service slurmctld start
```

Check if Slurm controller is running:

```
scontrol show daemons
```

Output:

```
slurmctld slurmd
```

5. Create folder _/var/run/slurm-llnl_ and assign permissions:

```
sudo mkdir /var/run/slurm-llnl

sudo chown -R slurm:slurm /var/run/slurm-llnl
```

6. Start Slurm controller service:

```
sudo service slurmctld start

sudo service slurmd start
```

Verify Slurm controller is running:

```
sinfo
```

Check Slurm status:

```
sinfo
```

Output:

```
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
picluster*    up   infinite      1   idle node0
```

References:

https://stackoverflow.com/questions/56553665/how-to-fix-slurmd-service-cant-open-pid-file-error-in-slurm

[Troubleshooting node issues](https://www.eidos.ic.i.u-tokyo.ac.jp/~tau/lecture/parallel_distributed/2016/html/fix_broken_slurm.html)

---

### Save image of head node (node0)

**INSERT INSTRUCTIONS HERE**

_**NOTE:**_ Insert as appendix at end of file

---

### Generic node setup

### Configure NFS client on generic node

SSH into generic node from head node:

```
ssh pi@nodeX
```

Install NFS software:

```
sudo apt install nfs-common
```

Remove file to resolve issues:

```
sudo rm /usr/lib/systemd/system/nfs-common.service
```

Reload the daemon:

```
sudo systemctl daemon-reload
```

Start _nfs-common_ service:

```
sudo systemctl start nfs-common
```

Confirm _nfs-common_ is running:

```
sudo systemctl status nfs-common
```

Output:

```
* nfs-common.service - LSB: NFS support files common to client and server
   Loaded: loaded (/etc/init.d/nfs-common; generated)
   Active: active (running) since Mon 2020-12-21 17:16:40 CST; 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 818 ExecStart=/etc/init.d/nfs-common start (code=exited, status=0/SUCCESS)
    Tasks: 1 (limit: 2182)
   CGroup: /system.slice/nfs-common.service
           `-839 /usr/sbin/rpc.idmapd
```

Mount NFS shares from head node:

```
sudo mount -t nfs4 -o proto=tcp,port=2049 192.168.10.100:/nfs_share/users /hpc/users
sudo mount -t nfs4 -o proto=tcp,port=2049 192.168.10.100:/nfs_share/data /hpc/data
```

Configure mounts in _etc/fstab_ file:

```
192.168.10.100:/nfs_share/users   /hpc/users     nfs4 _netdev,auto 0 0
192.168.10.100:/nfs_share/data    /hpc/data      nfs4 _netdev,auto 0 0
```

### Configure NTP client on generic node

Install NTP on compute node:

```
sudo apt install ntp
```

Edit _/etc/ntp.conf_:

```
sudo nano /etc/ntp.conf
```

Under _restrict ::1_ add:

```
restrict 192.168.10.0 mask 255.255.255.0
```

Change:

```
#broadcast 192.168.123.255
```

To:

```
broadcast 192.168.10.255
```

Save and exit

Restart NTP service:

```
sudo /etc/init.d/ntp restart
```

### Configure Slurm on generic node

Install slurm client software:

```
sudo apt install slurmd slurm-client -y
```

Copy Slurm configuration from head not to generic node:

```
exit

rsync --rsync-path="sudo rsync" /etc/slurm-llnl/slurm.conf nodeX:/etc/slurm-llnl/slurm.conf
```

Enable munge on generic node:

```
ssh pi@nodeX

sudo systemctl enable munge
```

Prepare folder permissions for on generic node for _munge.key_ file transfer:

```
sudo chmod -R 0777 /etc/munge/
```

Add _pi_ user to _munge_ group temporarily:

```
sudo usermod -aG munge pi

exit
```

Prepare folder permissions on head node for _munge.key_ file transfer:

```
sudo chmod -R 0777 /etc/munge/
```

Add _pi_ user to _munge_ group temporarily:

```
sudo usermod -aG munge pi
```

Copy munge key to generic node:

```
rsync --rsync-path="sudo rsync" /etc/munge/munge.key nodeX:/etc/munge/munge.key
```

Restore permissions for head node _munge_ folder and files:

```
sudo chmod 400 /etc/munge/munge.key

sudo chmod 700 /etc/munge
```

Restore permissions for generic node _munge_ folder and files:

```
ssh pi@nodeX

sudo chmod 400 /etc/munge/munge.key

sudo chmod 700 /etc/munge

exit
```

Remote test munge:

```

munge -n | ssh nodeX unmunge

```

Output:

```

```

SSH to generic node:

```

ssh pi@nodeX

```

Start Slurm on generic node:

```

sudo service slurmd start

```

Check node status:

```

sinfo

```

Output:

```

```

Give rsync proper permission to run:

```

sudo visudo

```

Add the following to the end of the file:

```

<username> ALL=NOPASSWD: /usr/bin/rsync \*

```

Exit back to head node:

```

exit

```

2. Install Slurm daemon

**Execute on _node0_:**

SSH in to _node0_:

```

ssh pi@node0

```

Install Slurm daemon and Slurm client:

```

sudo apt install slurmd slurm-client
sudo ln -s /var/lib/slurm-llnl /var/lib/slurm

```

Create log folders and take ownership:

```

sudo mkdir -p /var/log/slurm

sudo chown -R slurm:slurm /var/log/slurm

```

Take ownership of Slurm run folder:

```

sudo chown -R slurm:slurm /var/run/slurm-llnl

```

Exit to head node:

```

exit

```

**On _head node_:**

Copy Munge and Slurm configuration files from head node to compute node:

```

rsync -a --rsync-path="sudo rsync" /etc/munge/munge.key pi@node0:/etc/munge/munge.key

rsync -a --rsync-path="sudo rsync" /etc/slurm-llnl/slurm.conf pi@node0:/etc/slurm-llnl/slurm.conf

```

**On _compute node_:**

SSH in to _node0_:

```

ssh pi@node0

```

Take ownership of _munge.key_ file:

```

sudo chown munge:munge /etc/munge/munge.key

```

Finish install and start Slurm and Munge:

```

sudo systemctl enable slurmd.service
sudo systemctl restart slurmd.service
sudo systemctl enable munge.service
sudo systemctl restart munge.service

```

Verify Slurm daemon is running:

```

sudo systemctl status slurmd.service

```

Will return feedback to the screen. Verify _Active_ line states: _**active (running)**_.

Verify Munge is running:

```

sudo systemctl status munge.service

```

Will return feedback to the screen. Verify _Active_ line states: _**active (running)**_.

3. Add user to Slurm group

```

sudo adduser pi slurm

```

Execute on _head node_:

```

sudo scontrol reconfigure

```

Check node status:

```

sinfo

```

This should show all nodes in an idle state.

**SLURM references:**

https://moc-documents.readthedocs.io/en/latest/hpc/Slurm.html#slurm-installation

http://www.feacluster.com/pi_slurm_cluster.php

---

### Configure generic node to compute node

1. Copy generic node image created earlier to an SD card

2. Boot and login to your system

Log in with username: **pi** and password **raspberry**

SSH into the new node:

```

ssh pi@nodeX

```

Enter **yes** to accept the key

Verify you are logged in:

Command prompt should read `pi@nodeX:~ $`

3. Adjust _/etc/hostname_ file

```

sudo nano /etc/hostname

```

Change:

```

nodeX

```

To:

```

node0

```

Save and exit

_**Note:**_ This number will increment by one each time you add a node and must be unique on your cluster.

4. Adjust _/etc/dhcpcd.conf_

```

sudo nano /etc/dhcpcd.conf

```

Change the _eth0_ ip address from:

```

static ip_address=192.168.10.3

```

To:

```

static ip_address=192.168.10.101

```

Save and exit

5. Edit hosts file

```

sudo nano /etc/hosts

```

Change:

```

127.0.1.1 nodeX

```

To:

```

127.0.1.1 node1

```

Save and exit

6. Configure NFS

Install nfs-common:

```

sudo apt install nfs-common

```

Mount the NFS share to local folder

```

sudo mount -t nfs -o proto=tcp,port=2049 192.168.10.100:/nfs_share /hpc/users

```

7. Expand Filesystem

Open configuration tool:

```

sudo raspi-config

```

Select **7 Advanced Options**

Select **A1 Expand Filesystem**

Select **Ok**

_Tab_ to Select **Finish**

Select **Yes**

All settings should take effect on reboot

---

### Deploy Head Node SSH Key

Issue the following command from the head node for each node in the cluster:

Only run this command once the node is restarted with a node number.

```

rsync -a --rsync-path="sudo rsync" ~/.ssh/authorized_keys pi@node1:~/.ssh/authorized_keys

```

_**Note:**_ At this point you will just do this once to develop a compute node image with Slurm installed. After that is complete you will create a new generic image of the compute node. Once that is complete you can use that image to finish deploying your compute nodes for the rest of your cluster.

SSH in to the new node:

```

ssh pi@nodeX

```

Reboot the node:

```

sudo reboot

```

---

> ##### Compute Node

SSH in to compute node:

```

ssh pi@node0

```

---

---

---

## Deploying the Rest of the Cluster

By now you have developed a head node image that contains both MPI and Slurm. You have also developed a compute node image that contains both MPI and Slurm as well. Now you should go back to the instructions for "Create Node Image" to save both images and then use the compute node image to finish deploying your cluster. Saving these images at each stage gives you different configurations that you can easily deploy in the future and also allows you to have a checkpoint in case something goes wrong. You can write the saved node image to your SD and start from that point rather then starting from the beginning.

---

## Add an ethernet adapter

> #### Add _eth0_:

> Edit _/etc/network/interfaces_ file:

```

sudo nano /etc/network/interfaces

```

Add below eth0 section:

```

iface eth1 inet manual

```

Add to the end of the file:

```

pre-up iptables-restore < /etc/iptables_wired.rules

```

Flush old iptables rules:

```

sudo iptables -F

```

Create new iptables rules:

```

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

sudo bash -c "iptables-save > /etc/iptables.wired"

```

Refresh _eth1_ connection:

```

sudo ifdown eth1

sudo ifup eth1

```

> #### Disable _wlan0_:

Edit _/etc/wpa_supplicant/wpa_supplicant.conf_ file:

```

sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

```

Comment out the `network={ connection information }` section (all lines)

Disable eth1 adapter:

```

sudo ifconfig eth1 down

```

Reboot:

```

sudo reboot

```

Now all traffic for the cluster is routed through eth0 and out eth1 to the internet. Any returning traffic or downloads come in via eth1 and through eth0 to the cluster unless its meant for the head node.

---

## Scripts

Create a _/software/scripts_ folder:

```

mkdir -p /software/scripts
mkdir -p /software/code
mkdir -p /software/data

```

Edit _.bashrc_ file:

```

nano ~/.bashrc

```

Add to the end of the file:

```

# SCRIPTS

export PATH="/software/scripts:\$PATH"

```

Add scripts to the _/software/scripts_ folder to use as commands system wide.

### Deploy a single file to all compute nodes

This script will deploy files to all nodes to a folder defined by the user.

Create _/software/scripts/deploy_file_ file:

```

sudo nano /software/scripts/deploy_file

```

Add the following:

```

#!/bin/bash

if [ "$1" == "-help" ] || [ "$1" == "" ]; then
echo -e "Command \tExample"
echo "----------------------------------------------------------"
echo "deploy_file deploy_file <filename> <option> <destination folder>"
echo -e "\t\tNote:Default destination folder is '/software/files'"
echo "Help deploy -help"
exit
fi

case "$1"
if [ "$1" != "" ]; then
if [ "$2" == "" ]; then
filelocation=/software/data
else
filelocation=$2
  fi
    echo "Transferring file: $1 to node0:$filelocation"
    rsync -ar $1 pi@node0:$filelocation
    echo "Transferring file: $1 to node1:$filelocation"
    rsync -ar $1 pi@node1:$filelocation
    echo "Transferring file: $1 to node2:$filelocation"
    rsync -ar $1 pi@node2:$filelocation
    echo "Transferring file: $1 to node3:$filelocation"
    rsync -ar $1 pi@node3:$filelocation
    echo "Transferring file: $1 to node4:$filelocation"
    rsync -ar $1 pi@node4:$filelocation
    echo "Transferring file: $1 to node5:$filelocation"
    rsync -ar $1 pi@node5:$filelocation
    echo "Transferring file: $1 to node6:$filelocation"
    rsync -ar $1 pi@node6:\$filelocation
else
echo "All nodes are already defined in the script"
echo "Please enter a filename and destination folder: ie. deploy_file <filename> <destination folder>"
fi

```

### Push the entire /software folder to all compute nodes

---

## Troubleshooting Section

> #### Received SIGHUP or SIGTERM from Nano

> Enter the command:

```

bash

```

> #### NETWORK UNREACHABLE:

> When experiencing network connectivity problems with compute nodes:

1. Flush the iptables in Memory

```

sudo iptables --flush

```

1. Delete the rules file

```

sudo rm -rf /etc/iptables.rules

```

1. Rebuild the rules and file Repeat the IP tables section of the guide, starting with the commands:

```

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE

```

1. Save the iptables.rules file:

```

sudo bash -c "iptables-save > /etc/iptables.rules"

```

1. Check for the iptable rules in the _/etc/network/interfaces file_:

Make sure that the line below is present and not commented out:

```

pre-up iptables-restore < /etc/iptables.rules

```

If it is missing then add it to the end of the file. Save and exit.

> ### RSYNC ISSUES:

If having trouble with using rsync commands:

> #### Setup Rsync:

**On _Both nodes_**

Edit the /etc/sudoers file:

```

sudo visudo

```

Add this line to the end of the file:

```

<username> ALL=NOPASSWD: /usr/bin/rsync \*

```

> #### MPI ISSUES

If mpiexec command fails to execute, stalls, or displays an error message about an unreadable path file:

- Mpich3 could be in the wrong directory
- Make sure the export path correlates to the actual install path for MPICH3
- Reinstalling MPICH3 and setting up the proper environment variables can fix many problems, re-evaluate the MPICH3 install instructions and verify all settings before attempting a reinstall.

> #### SSH ISSUES

If the Pi is displaying SSH errors when running the mpiexec command: Check the problematic node's authorized_keys file, and compare it with the head node's authorized_keys file.

Check the file by going to the SSH directory:

```

cd ~/.ssh

```

Now check the file information for _authorized_keys_ file:

```

ls -ls

```

The filesize is listed after the owner and group names.

These file should be identical in length, if not redistribute the head node's authorized_keys file to the compute node using the following command:

```

rsync -a --rsync-path="sudo rsync" ~/.ssh/authorized_keys pi@nodeX:~/.ssh/authorized_keys

```

> #### COMMANDS TO CHECK SERVICE STATUSES

These commands do the same thing, just with a different syntax:

```

sudo systemctl [start,stop,restart,status] <service name>

sudo service <service name> [start,stop,restart,status]

sudo /etc/init.d/<service name> [start,stop,restart,status]

```

> #### ENABLING/DISABLING NETWORK INTERFACE CONNECTIONS

This is a quick way to bring down and bring back up network interfaces without restarting.

Disable the specified connection

```

sudo ifdown <connection name>

```

Enable the specified connection

```

sudo ifup <connection name>

```

> #### SLURM ISSUES

Make sure the slurm.conf file is identical across all nodes.

Use `sudo scontrol reconfigure` to distribute the slurm.conf from the head node to all compute nodes responding.

When running the service status command, read the error messages that are displayed: _**these messages are vital in order to troubleshoot current problems**_.

> #### PROBLEMATIC NODES

On many occasions, certain nodes fail to work because of a software/hardware malfunction. This can be fixed by removing and reinstalling the software. Hardware problems can be fixed by reformatting the node's SD card, and rewriting it with a functional node image. Also check each Ethernet cable for weaknesses, and verify that each node in the cluster is properly connected.

-For Pi 3 Clusters: The head node is connected via Wi-Fi, and each compute node uses the head node's wireless connection to download files.

-For Pi 2 Clusters: A Wi-Pi adapter is a tested solution for establishing a wireless connection with a Raspberry Pi model 2\. Using other wireless adapters could result in incompatible drivers or other various issues. The head node can also be connected to the Internet via an Ethernet cable.

---

## Network Diagrams

<center><strong>Base Equipment Layer (Pictured Below)</strong>

<img src="images\Raspberry_Pi_Cluster_Network_Configuration_-_Base_Equipment_Layer.png">

<strong>Physical Layer (Pictured Below)</strong>

<img src="images\Raspberry_Pi_Cluster_Network_Configuration_-_Physical_Layer.png">

<strong>Logical Layer (Pictured Below)</strong>

<img src="images\Raspberry_Pi_Cluster_Network_Configuration_-_Logical_Layer.png">

<strong>Physical and Logical Layers (Pictured Below)</strong>

<img src="images\Raspberry_Pi_Cluster_Network_Configuration_-_Physical_and_Logical_Layers.png"></center>

---

## References

<https://www.thegeekdiary.com/how-to-backup-linux-os-using-dd-command/>

<https://sysadmins.co.za/setup-a-nfs-server-and-client-on-the-raspberry-pi/>

<https://pimylifeup.com/raspberry-pi-nfs/>

<https://www.raspberrypi.org/documentation/configuration/external-storage.md>

<https://www.howtoforge.com/tutorial/hpl-high-performance-linpack-benchmark-raspberry-pi/>

<https://www.netlib.org/benchmark/hpl/>

<https://www.modmypi.com/blog/how-to-give-your-raspberry-pi-a-static-ip-address-update>

<https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=44609>

<http://weworkweplay.com/play/automatically-connect-a-raspberry-pi-to-a-wifi-network/>

<https://www.raspberrypi.org/forums/viewtopic.php?f=36&t=162096>

<https://www.raspberrypi.org/forums/viewtopic.php?t=118804&p=808453>

<https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=37575>

<http://unix.stackexchange.com/questions/88100/importing-data-from-a-text-file-to-a-bash-script>

<http://www.tldp.org/LDP/abs/html/arrays.html>

<http://how-to.wikia.com/wiki/How_to_read_command_line_arguments_in_a_bash_script>

<http://ryanstutorials.net/bash-scripting-tutorial/bash-variables.php>

<http://stackoverflow.com/questions/19996089/use-ssh-to-start-a-background-process-on-a-remote-server-and-exit-session>

<http://stackoverflow.com/questions/29142/getting-ssh-to-execute-a-command-in-the-background-on-target-machine>

<http://www.igeekstudio.com/blog/setting-up-ganglia-and-hadoop-on-raspberry-pi>

<http://ccm.net/faq/2540-linux-create-your-own-command>

<https://github.com/XavierBerger/RPi-Monitor>

<https://www.raspberrypi.org/blog/visualising-core-load-on-the-pi-2/>

<https://github.com/davidsblog/rCPU>

<https://raseshmori.wordpress.com/2012/10/14/install-hadoop-nextgen-yarn-multi-node-cluster/>

<https://www.packtpub.com/hardware-and-creative/raspberry-pi-super-cluster>

```

```