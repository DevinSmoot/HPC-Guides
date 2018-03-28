# Ubuntu Server Setup Guide
---
### Definition of repository
This repository and guide is designed to guide the setup of a Ubuntu
supercomputing cluster. This cluster consists of a basic Ubuntu Server install
that is combined with the MPICH3 system. This gives the cluster MPI capability.
By default OpenMP libraries are included with GCC which is also installed in
the process of setting up MPICH3.

---

## Setup Head Node

##### Step 1 - Install Ubuntu Server

Select **Install Ubuntu Server**

Select **English**

Select **United States**

Select **No** for *Detect keyboard layout*

Select **English (US)**

Select **English (US)**

Select **enp0s3** for *Primary network adapter*

Set *hostname* to **head**

Set *New user full name* to last name and first initial

Set *Username* to last name and first initial

Choose a password for your account or use your student id number

Agree to use weak password

Select **No** for *Encrypt your home directory*

Select **Yes** for *Is this time zone correct?*

Select **Guided - use entire disk and set up LMV**

Select the default drive

Select **Yes** to *Write the changes to disks and configure LVM*

Keep the default drive size

Select **Yes** to *Write the changes to the disk*

Leave *HTTP Proxy* empty and Continue

Select **No automatic updates**

Hit *Tab* to move the cursor to **OpenSSH server** and press ``Space`` to select it

**_Note:_** OpenSSH package is selected when a ** \* ** is shown in the box under the cursor

Press ``Enter`` to continue the installation

Select **Yes** to *Install the Grub boot loader to the master boot record*

![Step 7](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step7.png)

##### Step 2 - Set Static IP Address for Secondary Connection

Start the *Head Node* and login using the username and password created during the install process

Edit the network interfaces file:

``sudo nano /etc/network/interfaces``

Add the secondary interface to the file:

```
# Secondary Interface - cluster connection enp0s8
auto enp0s8
iface enp0s8 inet static
address 192.168.10.5
netmask 255.255.255.0
network 192.168.10.0
```
Save and exit


##### Step 3 - Set up IPv4 Traffic Forwarding
Enable traffic forwarding and make it permanent:

``sudo nano /etc/sysctl.conf``

Add the following to the end of the file:

```
# Enable IPv4 forwarding
net.ipv4.ip_forward = 1

# Disable IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Save and exit

Apply those changes:

``sudo sysctl -p``

Apply routing to iptables:

```
sudo iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o eno2 -j MASQUERADE

sudo bash -c "iptables-save > /etc/iptables.rules"
```

Open the network interfaces file:

``sudo nano /etc/network/interfaces``

Add the following line to the end of the file:

``pre-up iptables-restore < /etc/iptables.rules``

Save and exit

Now reboot the system:

``sudo reboot``


##### Step 4 - Update the system packages and kernel

``sudo apt udpate && sudo apt upgrade -y && sudo apt dist-upgrade -y``

Edit */etc/hosts* file:

Add the following to the end of the file:

```
192.168.10.5    head
192.168.10.100  node0
192.168.10.101  node1
192.168.10.102  node2
192.168.10.103  node3
192.168.10.104  node4
192.168.10.105  node5
192.168.10.106  node6
```

##### Step 5 - Set up SSH key

** \*\* VERIFY AT THE COMMAND PROMPT THAT YOU ARE UNDER YOUR USER ACCOUNT AND NOT EXECUTING CODE AS SUPER USER OR ROOT \*\* **

Generate an SSH key:
```
cd ~
ssh-keygen -t rsa -C "cluster@swosu"
```
Press ``Enter`` to select default install location

Press ``Enter`` to leave passphrase blank

Press ``Enter`` to confirm blank passphrase

Copy SSH keys to authorized keys:

``cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys``

---

### MPICH-3.2

##### Step 1 - Create Directories

Install some required compilers and packages:

``sudo apt install make build-essential gfortran``

Create software directory for multiple users:

``sudo mkdir -p /software/lib``

Create hpc user group:

``sudo groupadd hpc``

Add user to hpc user group:

``sudo usermod -aG hpc <username>``

Take ownership of */software*:

``sudo chown -R <username>:hpc /software``

Change to *software* directory and create *mpich-3.2* directory:

```
cd /software/lib

mkdir mpich-3.2
```

Change to the *mpich-3.2* directory and create *build* and *install* directories:

```
cd mpich-3.2

mkdir build install
```


##### Step 2 - Download and install

Download MPICH3 package and install:
http://www.mpich.org/downloads/

```
wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
```

Untar the package:

``tar xvfz mpich-3.2.tar.gz``

Change to *build* directory to begin building the install:

``cd build``

```
/software/lib/mpich-3.2/mpich-3.2/configure  --prefix=/software/lib/mpich-3.2/install

make
make install
```

Add MPI location to system environment variable PATH:

``export PATH=$PATH:/software/lib/mpich-3.2/install/bin``

Make the PATH change permanent by adding it to the profile file:

``sudo nano ~/.bashrc``

Add the following to the end of the file:

``export PATH="$PATH:/software/lib/mpich-3.2/install/bin"``

Save and exit

##### Step 3 - Create Node List

Create a list of nodes for MPI to use:

```
cd ~

sudo nano nodelist
```

Add the *head node* ip address to the file:

``192.168.10.5``

##### Step 4 - Test MPI

**Test 1**

``mpiexec -f nodelist hostname``

Should return **head** on the next line

**Test 2**

``mpiexec -f nodelist -n 2 /software/lib/mpich-3.2/build/examples/cpi``

Should give an output similar to the following:
[image]

![Step 4](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part2step4.png)

Shutdown head node:

``sudo shutdown -h now``


##### Step 5 - SSH into Compute Node 0 to Acquire Authentication key

In VirtualBox select *Head Node* in the left column

With *Head Node* selected click **Start** in the toolbar

Login to *Head Node*

On *Head Node* enter:

``ssh <username>@node0``

Type ``yes`` and press ``Enter`` when asked *Are you sure you want to continue connection (yes/no)?*

Type ``exit`` and press ``Enter`` to return to *Head Node*

Verify *Head Node* by checking the command prompt for ``<username>@head:~$``


##### Step 6 - Add Compute Node 1 to the nodelist File on Head Node

On the *Head Node* edit the nodelist:

```
cd ~

sudo nano nodelist
```

Add ``192.168.10.100`` to the second line

Save and exit


---

## Set up Cluster Compute Node

Begin by setting up the network interface connection:

``sudo nano /etc/network/interfaces``

Under the line ``auto enp0s3`` change or add the following:

```
iface enp0s3 inet static
address 192.168.10.100
netmask 255.255.255.0
gateway 192.168.10.5
dns-nameservers 8.8.8.8
```

Now reboot *Compute Node 0*

``sudo reboot``

Wait for *Compute Node 0* to reboot before continuing


##### Step 7 - Test MPI

On the *Head Node* enter:

```
mpiexec -f nodelist -n 6 /software/lib/mpich-3.2/build/examples/cpi
```

You should get an output similar to the following:
[image]

**_Note:_** Each process shows which node it was executed on. You should see both head and node1 displayed. This shows that MPI is sending and executing the script on both nodes in the cluster.

Congratulations! This cluster is ready to execute MPI code.

==CHANGES
Add ssh key generation StateSaveLocation
==END CHANGES

---

## Slurm on Head Node

##### Step 1 - Install needed packages

Execute:

``sudo apt-get install slurm-wlm slurmctld slurmd``

##### Step 2 - Develop configuration file

Edit */etc/slurm-llnl/slurm.conf* and add or edit to match the following:

```
#Virtual cluster slurm.conf
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
SlurmdSpoolDir=/var/lib/slurm
SlurmUser=slurm
#SlurmdUser=root
StateSaveLocation=/var/lib/slurm
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
NodeName=head State=UNKNOWN
NodeName=node1 Procs=1 State=UNKNOWN
PartitionName=TEST Default=YES Nodes=head,node1 State=UP
```

Create and take ownership of required folders for Slurm:

```
sudo mkdir /var/lib/slurm

sudo chown -R slurm:slurm /var/lib/slurm

sudo mkdir /var/log/slurm

sudo chown -R slurm:slurm /var/log/slurm
```


##### Step 4 - Create Munge authentication keyboard

``sudo /usr/sbin/create-munge-key``

##### Step 5 - Fix Munge issue so it will boot

``sudo systemctl edit --system --full munge``

Change this line:

``ExecStart=/usr/sbin/munged``

To:

``ExecStart=/usr/sbin/munged --syslog``

Save and exit.

```
sudo systemctl start munge
```

_**Note:**_ The _systemctl enable munge_ may show a failed notification but its fine. Just move to the next command.

##### Step 6 - Enable Slurm Controller

``sudo systemctl enable slurmctld``

Reboot:

``sudo reboot``

## Slurm on Compute Node

**On _head node_**

##### Step 1 - Copy Slurm configuration file and Munge key to _node1_ <username's> home directory:

``sudo cat /etc/munge/munge.key | ssh <username>@node1 "cat > ~/munge.key"``

``sudo cat /etc/slurm-llnl/slurm.conf | ssh <username>@node1 "cat > ~/slurm.conf"``

**On _compute node_**

##### Step 2 - Install Slurm

``sudo apt-get install slurmd slurm-client``

##### Step 3 - Copy the configuration files to proper locations

```
sudo cp ~/munge.key /etc/munge/
sudo cp ~/slurm.conf /etc/slurm-llnl/
```

##### Step 4 - Fix Munge issue so it will boot

```
sudo systemctl edit --system --full munge
```

Change this line:

``ExecStart=/usr/sbin/munged``

To:

``ExecStart=/usr/sbin/munged --syslog``

Save and exit.

```
sudo systemctl enable munge

sudo systemctl start munge
```

_**Note:**_ The _systemctl enable munge_ may show a failed notification but its fine. Just move to the next command.

##### Step 5 - Enable Slurm daemon

``sudo systemctl enable slurmd``

##### Step 6 - Set Slurm folder permissions

``
sudo mkdir /var/lib/slurm

sudo chown -R slurm:slurm /var/lib/slurm

sudo mkdir /var/log/slurm

sudo chown -R slurm:slurm /var/log/slurm
``

##### Step 7 - Reboot both nodes

Execute on both nodes:

``sudo reboot``


---


## Troubleshooting

#### Host Verification Key Error

In case of *host verification key* error when executing MPI follow the steps for deleting and regenerating SSH keys.

On *Head Node* as user delete previous SSH keys:

```
rm -rf ~/.ssh
mkdir ~/.ssh
```

Generate new SSH keys:

```
cd ~
ssh-keygen -t rsa -C "cluster@swosu"
```

``Enter`` to select default install location

``Enter`` to leave passphrase blank

``Enter`` to confirm blank passphrase

Copy new SSH keys to local system and nodes:

```
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub | ssh <username>@node0 "cat > .ssh/authorized_keys"
```
