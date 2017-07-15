# Ubuntu Supercomputing Virtual Cluster Setup Guide
---
### Definition of repository
This repository and guide is designed to guide the setup of a Ubuntu
supercomputing cluster. This cluster consists of a basic Ubuntu Server install
that is combined with the MPICH3 system. This gives the cluster MPI capability.
By default OpenMP libraries are included with GCC which is also installed in
the process of setting up MPICH3.

---
### References

---
### Notes

##### Legend for this document:

Proper named objects will appear as *Proper Named Object* text

Action items and examples will appear as **Action item** text

Code, plain text to be entered, or text from the command line will appear as ``code`` text



##### Linux commands that help throughout this guide:

``cd ``
change directory followed by the name of the directory to change to **Example** ``cd home``

``cd .. ``
go back up one directory

``dir ``
list all folders and files in current directory

``nano ``
command to open Nano text editor

* All commands listed at the bottom of the Nano editor are executed using ``Ctrl``+``<key listed>``
* ``Ctrl``+``x`` - exit, ``y`` to confirm, and ``Enter`` to verify filename
* ``Ctrl``+``w`` - search function; enter string to search for;``Ctrl``+``w`` again to search for previous string
* ``Ctrl``+``c`` - cursor location; used to find exact line when bug tracing

``sudo`` used to execute commands using the Super User; if you receive a warning about permissions when editing or executing commands try running that same command prefixed by sudo **Example** ``sudo nano /etc/hostname``

``sudo service <name of service> start stop`` or ``restart`` used to start, stop, or restart system services **Example** ``sudo service networking restart``

``shutdown -h now`` immediate system halt and shutdown

``reboot`` initiate a system reboot

``apt-get update`` pulls all update information from the internet; does not perform an update

``apt-get upgrade`` performs an update based on information received from ``apt-get update`` command. Using the ``-y`` option performs the operation without querying the user to proceed with the install.

``apt-get dist-upgrade`` performs an update of the system kernel based on information received from ``apt-get update`` command. Using the ``-y`` option performs the operation without querying the user to proceed with the install.

``ls`` lists all directories and files in the current directory

``ls -l `` lists all files and directories in the current directory with owner information, group information, and permissions

``ls -ld `` lists information about the current directory including owner information, group information, and permissions

`` ~`` indicates the home directory for the current user

``sudo -s`` Super User mode; be careful using this mode as any commands executed under Super User will reflect the **root** user and not a user account

`` /`` denotes the root directory of the system

`` ./`` denotes the running of an executable file **Example** ``./install.sh``

``ssh`` allows ssh-ing into other systems

##### Information provided by the command prompt:

There is a lot of information that you can gather just by looking at the command prompt. Using ``user@somecomputer:~$`` as an example we will examine the command prompt. To begin with we can tell which user account is currently logged in. In this case its ``user``. Next we can tell which system we are logged in to. In this example it is ``somecomputer``. Next is `` ~`` which tells us which folder we are in on the system. As stated above `` ~`` denotes the ``user`` home directory. Next we can tell if we are executing commands as a user or Super User by looking at the last character. In this case it is ``$`` which tells us we are executing with whatever permissions ``user`` has.

For another example we will use ``root@somecomputer:/usr/local/#``. This example shows we are signed in as ``root`` user on ``somecomputer`` and we are in the directory ``/usr/local/``. The first ``/`` always denotes the root directory. Next we can tell we are executing commands as a Super User (root always executes as Super User) by the ``#``.

---

## Set up Cluster Head Node


##### Step 1 - Install VirtualBox
Download and install Oracle VirtualBox
https://www.virtualbox.org/wiki/Downloads


##### Step 2 - Create Virtual Machine
Create a virtual machine in virtualbox by starting VirtualBox and clicking the **New** button

![Step 2](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step2.png)

##### Step 3 - Create Virtual Machine Continued
Set *Name:* to **Head Node**

Set *Type:* to **Linux**

Set *Version:* to **Ubuntu (64-bit)**

Set *Memory size* to 1/4 of total host system memory or **1024**

Set *Hard disk* to **Create a virtual hard disk now**

Click **Create**

![Step 3](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step3.png)

##### Step 4 - Create Virtual Hard Disk

Set *File location* to **Head Node**

**_Note:_** File location can be changed by clicking the icon to the right of the *File location* input box

Set *File size* to **20.00**

Set *Hard disk file type* to **VDI (VirtualBox Disk Image)**

Set *Storage on physical hard disk* to **Dynamically allocated**

Click **Create**

![Step 4](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step4.png)

##### Step 5 - Set Processors and Network Adapters

Right click the VM for *Head Node* in the left column of VirtualBox and click on **Settings**

Click **System** and select the **Processor** tab

Change *Processor(s)* to **2**

![Step 5a](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step5a.png)

Click **Network** and select the **Adapter 2** tab

Check **Enable Network Adapter**

Set *Attached to:* to **Internal Network**

Set *Name:* to **cluster**

![Step 5b](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step5b.PNG)

##### Step 6 - Start-up the Head Node VM
Download and install Ubuntu Server 64-bit ISO
https://www.ubuntu.com/download/server

Select the Head Node VM from the left column and click **Start**

Click the icon to the right of the drop-down box and navigate to the downloaded *Ubuntu Server 64-bit ISO*

Click **Start**

![Step 6](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part1step6.png)

##### Step 7 - Install Ubuntu Server

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

##### Step 8 - Set Static IP Address for Secondary Connection

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

Edit the hosts file:

``sudo nano /etc/hosts``

Add to the end of the file:

```
192.168.10.5    head
192.168.10.100  node1
```

Save and exit

##### Step 9 - Set up IPv4 Traffic Forwarding
Enable traffic forwarding and make it permanent:

``sudo sysctl -w net.ipv4.ip_forward=1``

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

```
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
sudo bash -c "iptables-save > /etc/iptables.rules"
```

Open the network interfaces file:

``sudo nano /etc/network/interfaces``

Add the following line to the end of the file:

``pre-up iptables-restore < /etc/iptables.rules``

Save and exit.

Now reboot the system:

``sudo reboot``


##### Step 10 - Update the system packages and kernel

``sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y``


##### Step 11 - Set up SSH keys

**VERIFY AT THE COMMAND PROMPT THAT YOU ARE UNDER YOUR USER ACCOUNT AND NOT EXECUTING CODE AS SUPER USER OR ROOT**

Generate an SSH key:
```
cd ~
ssh-keygen -t rsa -C "vmcluster@swosu"
```
Press ``Enter`` to select default install location

Press ``Enter`` to leave passphrase blank

Press ``Enter`` to confirm blank passphrase

Copy SSH keys to authorized keys:

``cat /home/<username>/.ssh/id_rsa.pub >> /home/<username>/.ssh/authorized_keys``

---

### Java

##### Step 1 - Install Java 8

Add the PPA key:

``sudo apt-key adv --recv-key --keyserver keyserver.ubuntu.com EEA14886``

Add sources to */etc/apt/sources.list*:

``sudo nano /etc/apt/sources.list``

Add the source links to the end of the file:

```
deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
```

Save and exit.

Update sources and install Java 8:

```
sudo apt-get update

sudo apt-get install oracle-java8-installer
```

##### Step 2 - Verify Java install

Execute:

``java -version``

Should return the most current version of Java 8
At the time of this guide it is *1.8.0_121*

Execute:

``javac -version``

Should return the most current version of the Java 8 compilers
At the time of this guide it is *1.8.0_121*

##### Step 3 - Set Java environment variables

Verify settings by checking */etc/profile.d/jdk.sh* using the following command:

``cat /etc/profile.d/jdk.sh``

It should display 5 export commands

Execute a shell script to make environment variables take effect:

``source /etc/profile``

Verify:

``echo $JAVA_HOME``

Should show:

``/usr/lib/jvm/java-8-oracle``

---

### MPI

##### Step 1 - Create Directories

Install some required compilers and packages:

``sudo apt-get install make build-essential``

Change to *home* directory and create *mpich3* directory:

```
cd ~
mkdir mpich3
```

Change to the *mpich3* directory and create *build* and *install* directories:

```
cd mpich3
mkdir build install
```

##### Step 2 - Download and install

Install Fortran which is requred by MPICH3:

``sudo apt-get install gfortran``

Make a directory for Fortran code:

``mkdir /home/<username>/fortran``

Download MPICH3 package and install:
http://www.mpich.org/downloads/

```
wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
```

Untar the package:

``tar xvfz mpich-3.2.tar.gz``

Change to *build* directory to begin building the install:

``cd build``

Configure the install:

```
/home/<username>/mpich3/mpich-3.2/configure  --prefix=/home/<username>/mpich3/install
```

Compile the install:

```
make
make install
```

Add MPI location to system environment variable PATH:

``export PATH=$PATH:/home/<username>/mpich3/install/bin``

Make the PATH change permanent by adding it to the profile file:

``sudo nano /etc/profile``

Add the following to the end of the file:

```
export PATH="$PATH:/home/<username>/mpich3/install/bin"
```

Save and exit

##### Step 3 - Create Node List

Create a list of nodes for MPI to use:

```
cd ~

sudo nano nodelist
```

Save and exit.

Add the *head node* ip address to the file:

``192.168.10.5``

##### Step 4 - Test MPI

``cd ~``

**Test 1**

``mpiexec -f nodelist hostname``

Should return **head** on the next line

**Test 2**

``mpiexec -f nodelist -n 2 ~/mpich3/build/examples/cpi``

Should give an output similar to the following:
[image]

![Step 4](https://github.com/swosu/MAPSS/blob/dev/WinterCamp/Ubuntu%20Cluster%20Guide/images/part2step4.png)

Shutdown and get ready to clone:

``sudo shutdown -h now``

---

## Set up Cluster Compute Node

##### Step 1 - Clone the Virtual Machine

In VirtualBox right click the *Head Node* in the left column and select **Clone**

Set *Name* to **Compute Node 1**

Click **Next**

Select **Full clone**

Click **Clone**

##### Step 2 - Set Static IP Address

In VirtualBox select *Compute Node 1* in the left column

With *Compute Node 1* selected click **Start** in the toolbar

Login to *Compute Node 1*

At the terminal enter:

``sudo nano /etc/network/interfaces``

Remove all of the following lines:

```
# Secondary Interface - cluster connection enp0s8
auto enp0s8
iface enp0s8 inet static
address 192.168.10.5
netmask 255.255.255.0
network 192.168.10.0
```

Under the line ``auto enp0s3`` change or add the following:

```
iface enp0s3 inet static
address 192.168.10.100
netmask 255.255.255.0
gateway 192.168.10.5
dns-nameservers 8.8.8.8
```

Save and exit

Shutdown the *Compute Node 1*:

``sudo shutdown -h now``


##### Step 3 - Change Compute Node 1 Network Adapters

In VirtualBox right click *Compute Node 1* in the left column

Select **Settings**

Click **Network** and select **Adapter 2**

Uncheck the **Enable Network Adapter** box

Next, select **Adapter 1** tab

Set *Attached to:* to **Internal Network**

Set *Name:* to **cluster**


##### Step 4 - Set hostname

In VirtualBox select *Compute Node 1* in the left column

With *Compute Node 1* selected click **Start** in the toolbar

Login to *Compute Node 1*

Edit the hostname file:

``sudo nano /etc/hostname``

Change ``head`` to ``node1``

Save and exit

Edit the hosts file:

``sudo nano /etc/hosts``

Change ``head`` to ``node1``

Save and exit

Now reboot *Compute Node 1*

``sudo reboot``

Wait for *Compute Node 1* to reboot before continuing


##### Step 5 - SSH into Compute Node 1 to Acquire Authentication key

In VirtualBox select *Head Node* in the left column

With *Head Node* selected click **Start** in the toolbar

Login to *Head Node*

On *Head Node* enter:

``ssh <username>@192.168.10.100``

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


##### Step 7 - Test MPI

**Test 1**
On the *Head Node* enter:

```
cd ~
mpiexec -f nodelist hostname
```

You should get an output similar to the following:

```
head
node1
```

**Test 2**

On the *Head Node* enter:

```
cd ~
mpiexec -f nodelist -n 6 ~/mpich3/build/examples/cpi
```

You should get an output similar to the following:
[image]

**_Note:_** Each process shows which node it was executed on. You should see both head and node1 displayed. This shows that MPI is sending and executing the script on both nodes in the cluster.

Congratulations! This cluster is ready to execute MPI code.

---

## Slurm on Head Node

##### Step 1 - Install needed packages

Execute:

``sudo apt-get install slurm-wlm slurmctld slurmd``

##### Step 2 - Develop configuration file

Edit */etc/slurm-llnl/slurm.conf* and add or edit to match the following:

```
# slurm.conf file generated by configurator.html.
# Put this file on all nodes of your cluster.
# See the slurm.conf man page for more information.
#
ControlMachine=head
ControlAddr=192.168.10.5
#BackupController=
#BackupAddr=
#
AuthType=auth/munge
#CheckpointType=checkpoint/none
CryptoType=crypto/munge
#DisableRootJobs=NO
#EnforcePartLimits=NO
#Epilog=
#EpilogSlurmctld=
#FirstJobId=1
#MaxJobId=999999
#GresTypes=
#GroupUpdateForce=0
#GroupUpdateTime=600
#JobCheckpointDir=/var/slurm/checkpoint
#JobCredentialPrivateKey=
#JobCredentialPublicCertificate=
#JobFileAppend=0
#JobRequeue=1
#JobSubmitPlugins=1
#KillOnBadExit=0
#LaunchType=launch/slurm
#Licenses=foo*4,bar
#MailProg=/bin/mail
#MaxJobCount=5000
#MaxStepCount=40000
#MaxTasksPerNode=128
MpiDefault=none
#MpiParams=ports=#-#
#PluginDir=
#PlugStackConfig=
#PrivateData=jobs
ProctrackType=proctrack/pgid
#Prolog=
#PrologFlags=
#PrologSlurmctld=
#PropagatePrioProcess=0
#PropagateResourceLimits=
#PropagateResourceLimitsExcept=
#RebootProgram=
ReturnToService=1
#SallocDefaultCommand=
SlurmctldPidFile=/var/run/slurm-llnl/slurmctld.pid
SlurmctldPort=6817
SlurmdPidFile=/var/run/slurm-llnl/slurmd.pid
SlurmdPort=6818
SlurmdSpoolDir=/var/lib/slurmd
SlurmUser=slurm
#SlurmdUser=root
#SrunEpilog=
#SrunProlog=
StateSaveLocation=/var/lib/slurmd/slurmctld
SwitchType=switch/none
#TaskEpilog=
TaskPlugin=task/none
#TaskPluginParam=
#TaskProlog=
#TopologyPlugin=topology/tree
#TmpFS=/tmp
#TrackWCKey=no
#TreeWidth=
#UnkillableStepProgram=
#UsePAM=0
#
#
# TIMERS
#BatchStartTimeout=10
#CompleteWait=0
#EpilogMsgTime=2000
#GetEnvTimeout=2
#HealthCheckInterval=0
#HealthCheckProgram=
InactiveLimit=0
KillWait=30
#MessageTimeout=10
#ResvOverRun=0
MinJobAge=300
#OverTimeLimit=0
SlurmctldTimeout=120
SlurmdTimeout=300
#UnkillableStepTimeout=60
#VSizeFactor=0
Waittime=0
#
#
# SCHEDULING
#DefMemPerCPU=0
FastSchedule=1
#MaxMemPerCPU=0
#SchedulerRootFilter=1
#SchedulerTimeSlice=30
SchedulerType=sched/backfill
SchedulerPort=7321
SelectType=select/linear
#SelectTypeParameters=
#
#
# JOB PRIORITY
#PriorityFlags=
#PriorityType=priority/basic
#PriorityDecayHalfLife=
#PriorityCalcPeriod=
#PriorityFavorSmall=
#PriorityMaxAge=
#PriorityUsageResetPeriod=
#PriorityWeightAge=
#PriorityWeightFairshare=
#PriorityWeightJobSize=
#PriorityWeightPartition=
#PriorityWeightQOS=
#
#
# LOGGING AND ACCOUNTING
#AccountingStorageEnforce=0
#AccountingStorageHost=
#AccountingStorageLoc=
#AccountingStoragePass=
#AccountingStoragePort=
AccountingStorageType=accounting_storage/none
#AccountingStorageUser=
AccountingStoreJobComment=YES
ClusterName=cluster
#DebugFlags=
#JobCompHost=
#JobCompLoc=
#JobCompPass=
#JobCompPort=
JobCompType=jobcomp/none
#JobCompUser=
#JobContainerType=job_container/none
JobAcctGatherFrequency=30
JobAcctGatherType=jobacct_gather/none
SlurmctldDebug=3
#SlurmctldLogFile=
SlurmdDebug=3
#SlurmdLogFile=
#SlurmSchedLogFile=
#SlurmSchedLogLevel=
#
#
# POWER SAVE SUPPORT FOR IDLE NODES (optional)
#SuspendProgram=
#ResumeProgram=
#SuspendTimeout=
#ResumeTimeout=
#ResumeRate=
#SuspendExcNodes=
#SuspendExcParts=
#SuspendRate=
#SuspendTime=
#
#
# COMPUTE NODES
NodeName=head CPUs=1 State=UNKNOWN
NodeName=node1 CPUs=1 State=UNKNOWN
PartitionName=vmcluster Nodes=head,node1 Default=YES MaxTime=INFINITE State=UP
```

##### Step 3 - Verify Slurm Controller is running

``scontrol show daemons``

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
sudo systemctl enable munge

sudo systemctl start munge
```

_**Note:**_ The _systemctl enable munge_ may show a failed notification but its fine. Just move to the next command.

##### Step 6 - Enable Slurm Controller

``sudo systemctl enable slurmctld``

Complete the automatic install:

``sudo apt-get upgrade -y``

##### Step 7 - Set permissions on Slurm folder

```
sudo chown -R slurm:slurm /var/lib/slurmd
```

Reboot:

``sudo reboot``

##### Step 8 - Verify Munge and Slurm are running

``sudo service munge status``

Should show _Active: active (running)_

``sudo service slurmctld status``

Should show _Active: active (running)_

##### Step 9 - Verify Slurm has started the PartitionName

``sinfo``

Should show two entries. Look for _head_ under nodelist. It's state should be _idle_. The other entry is for _node1_ that we have not set up yet.

## Slurm on Compute Node

**On _head node_**

##### Step 1 - Copy Slurm configuration file and Munge key to _node1_ <username's> home directory:

``sudo cat /etc/munge/munge.key | ssh <username>@node1 "cat >> ~/munge.key"``

``sudo cat /etc/slurm-llnl/slurm.conf | ssh <username>@node1 "cat >> ~/slurm.conf"``

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

Complete Slurm daemon auto install:

``sudo apt-get upgrade -y``

##### Step 6 - Set Slurm folder permissions

``sudo chown -R slurm:slurm /var/lib/slurmd``

##### Step 7 - Reboot both nodes

Execute on both nodes:

``sudo reboot``


---

## Save Your Cluster Snapshot

Once your cluster is working properly you will want to take a snapshot of all nodes. This will allow you to work forward from here but to have a restore point if things don't work out with future changes.

##### Step 1 - Shutdown All nodes

Execute the shutdown on all nodes:

``sudo shutdown -h now``

##### Step 2 - Snapshot Your Nodes

In VirtualBox right click the node in the left column

In the upper right hand corner of VirtualBox click **Snapshots**

Click the left purple camera icon to take a snapshot of the current machine state

Give the node a name that includes its node name and stage **Example** ``Head Node (MPI Stage)`` or ``Head Node (HADOOP/MPI Stage)``

Click **OK** and you are done

Do this for all nodes and you are safe to begin making changes and producing

**_Note:_** You can snapshot the node anywhere you want by following these instructions. In this case take advantage of the description box after naming the snapshot.

---

## Set up Hadoop on the Cluster

**This section is incomplete**

##### Step 1 - Create new user and group

Create hduser and hadoop group:

```
sudo addgroup HADOOP
sudo adduser --ingroup hadoop hduser
sudo usermod -a -G sudo hduser
```

**_Note:_** All other actions from now on must be completed under the *hduser* account
Exit out and login as *hduser*

##### Step 2 - Disable IPv6

Edit /etc/sysctl.conf:

``sudo nano /etc/sysctl.conf``

Add the following lines to the end of the file:

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Apply changes:

``sudo sysctl -p``


##### Step 3 - Download latest Apache Hadoop

```
cd ~
wget http://apache.osuosl.org/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
```

Extract files:

``tar xvfz hadoop-2.7.3.tar.gz``

Create Hadoop folder under /usr/local:

``sudo mkdir /usr/local/hadoop``

Move the Hadoop folder to /usr/local/hadoop:

```
sudo mv hadoop-2.7.3 /usr/local/hadoop
sudo chown hduser:hadoop -R /usr/local/hadoop
```

Create Hadoop temp directories for Namenode and Datanode:

```
sudo mkdir -p /usr/local/hadoop_tmp/hdfs/namenode
sudo mkdir -p /usr/local/hadoop_tmp/hdfs/datanode
sudo chown hduser:hadoop -R /usr/local/hadoop_tmp
```

##### Step 4 - Update Hadoop Configuration Files

Update $HOME/.bashrc

``sudo nano ~/.bashrc``

Add environment variables to .bashrc file:

```
# -- HADOOP ENVIRONMENT VARIABLES START -- #
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-armhf
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
# -- HADOOP ENVIRONMENT VARIABLES END -- #
```

Change to the /usr/local/hadoop/etc/hadoop directory:

``cd /usr/local/hadoop/etc/hadoop``

Edit hadoop-env.sh:

``sudo nano hadoop-env.sh``

Change the following line:

``JAVA_HOME=``

To:

``JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-armhf``

Edit core-site.xml:




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
cat /home/<username>/.ssh/id_rsa.pub >> /home/<username>/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub | ssh <username>@192.168.10.100 "cat >> .ssh/authorized_keys"
```

Save new SSH keys to keychain:

```
ssh-agent bash
ssh-add
```

#### Restore VirtualBox snapshot

In VirtualBox right click the node in the left column

In the upper right hand corner of VirtualBox click **Snapshots**

Select the snapshot you wish to restore from the list

Click the second icon with the loopback green arrow to restore that snapshot

You will be prompted if you want to save a copy of the current machine state. This is a personal choice and is advised if you think you may resolve the situation causing the restore later.

**_Note:_** Remember the rule to save and save often!
