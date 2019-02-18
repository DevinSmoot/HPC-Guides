# Ubuntu Supercomputing Virtual Cluster Setup Guide

* * *

## Acknowledgements

Project lead: **Devin Smoot**, _SWOSU, Weatherford, OK_

Contributor/Implementation tester: **Madison Matli**, _SWOSU, Weatherford, OK_

Contributor/Implementation tester: **Hayden Webb**, _SWOSU, Weatherford, OK_

* * *

## To do list

-   Add new sections:
    -   Table of contents
    -   What this guide covers
    -   What you need for this guide (Hardware/software)
    -   Who this guide is directed at (audience)
    -   Conventions
    -   Errata (Include known issues)
    -   Fleshed out troubleshooting guide

* * *

## Definition of repository

This repository and guide is designed to guide the setup of a Ubuntu
supercomputing cluster. This cluster consists of a basic Ubuntu Server install
that is combined with the MPICH3 system. This gives the cluster MPI capability.
By default OpenMP libraries are included with GCC which is also installed in
the process of setting up MPICH3.

**_This guide is built to work with Ubuntu Server 16.04.5_**

[Ubuntu Server 16.04.5](http://releases.ubuntu.com/xenial/ubuntu-16.04.5-server-amd64.iso)

* * *

## Set up Head Node

Starting Notes: You will begin by downloading the VirtualBox software. VirtualBox will allow students' to download Linux operating systems inside of a virtual environment without overwritting the user's current operating system (Windows, macOS).

> #### Step 1 - Install VirtualBox

Download and install Oracle VirtualBox
<https://www.virtualbox.org/wiki/Downloads>

> #### Step 2 - Create Virtual Machine

Create a virtual machine in virtualbox by starting VirtualBox and clicking the **New** button

<img src="images\part1step2.png" alt="Step 2 VirtualBox" style="width: 200px;"/>

> #### Step 3 - Create Virtual Machine Continued

Set _Name:_ to **Head Node**

Set _Type:_ to **Linux**

Set _Version:_ to **Ubuntu (64-bit)**

Set _Memory size_ to **2048**

Set _Hard disk_ to **Create a virtual hard disk now**

Click **Create**

<img src="images\part1step3.png" alt="Step 3 VirtualBox" style="width: 450px;"/>

> #### Step 4 - Create Virtual Hard Disk

Set _File location_ to **Head Node**

**_Note:_** File location can be changed by clicking the icon to the right of the _File location_ input box

Set _File size_ to **80.00**

Set _Hard disk file type_ to **VDI (VirtualBox Disk Image)**

Set _Storage on physical hard disk_ to **Dynamically allocated**

Click **Create**

<img src="images\part1step4.png" alt="Step 4 VirtualBox" style="width: 450px;"/>

> #### Step 5 - Set Processors and Network Adapters

Right click the VM for _Head Node_ in the left column of VirtualBox and click on **Settings**

Click **System** and select the **Processor** tab

Change _Processor(s)_ to **2**

<img src="images\part1step5a.png" alt="Step 5a VirtualBox" style="width: 450px;"/>

Click **Network** and select the **Adapter 2** tab

Check **Enable Network Adapter**

Set _Attached to:_ to **Internal Network**

Set _Name:_ to **cluster**

<img src="images\part1step5b.png" alt="Step 5b VirtualBox" style="width: 450px;"/>

> #### Step 6 - Start-up the Head Node VM

Download and install Ubuntu Server 64-bit ISO
<https://www.ubuntu.com/download/server>

Select the Head Node VM from the left column and click **Start**

Click the icon to the right of the drop-down box and navigate to the downloaded _Ubuntu Server 64-bit ISO_

Click **Start**

<img src="images\part1step6.png" alt="Step 6 VirtualBox" style="width: 450px;"/>

> #### Step 7 - Install Ubuntu Server

Select **Install Ubuntu Server**

Select **English**

Select **United States**

Select **No** for _Detect keyboard layout_

Select **English (US)**

Select **English (US)**

Select **enp0s3** for _Primary network adapter_

Set _hostname_ to **head**

Set _New user full name_ to first initial and last name

Set _Username_ to last name and first initial

Choose a password for your account or use your student id number

Agree to use weak password

Select **No** for _Encrypt your home directory_

Select **Yes** for _Is this time zone correct?_

Select **Guided - use entire disk and set up LMV**

Select the default drive

Select **Yes** to _Write the changes to disks and configure LVM_

Keep the default drive size

Select **Yes** to _Write the changes to the disk_

Leave _HTTP Proxy_ empty and Continue

Select **No automatic updates**

Hit _Tab_ to move the cursor to **OpenSSH server** and press `Space` to select it

**_Note:_** OpenSSH package is selected when a \* is shown in the box under the cursor

Press `Enter` to continue the installation

Select **Yes** to _Install the Grub boot loader to the master boot record_

<img src="images\part1step7.png" alt="Step 7 VirtualBox" style="width: 450px;"/>

> #### Step 8 - Set Static IP Address for Secondary Connection

Start the _Head Node_ and login using the username and password created during the install process

Edit the network interfaces file:

    sudo nano /etc/network/interfaces

Add the secondary interface to the file:

    # Secondary Interface - cluster connection enp0s8
    auto enp0s8
    iface enp0s8 inet static
    address 192.168.10.5
    netmask 255.255.255.0
    network 192.168.10.0

Save and exit

Restart the interface:

    sudo ifup enp0s8

Edit the hosts file:

    sudo nano /etc/hosts

Add to the end of the file:

    192.168.10.5    head
    192.168.10.100  node0

Save and exit

> #### Step 9 - Set up IPv4 Traffic Forwarding

Enable traffic forwarding and make it permanent:

    sudo nano /etc/sysctl.conf

Add the following to the end of the file:

    # Enable IPv4 forwarding
    net.ipv4.ip_forward = 1

    # Disable IPv6
    net.ipv6.conf.all.disable_ipv6 = 1
    net.ipv6.conf.default.disable_ipv6 = 1
    net.ipv6.conf.lo.disable_ipv6 = 1

Save and exit

Enable the new rules:

    sudo sysctl -p

Enter iptables rules:

    sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
    sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE
    sudo bash -c "iptables-save > /etc/iptables.rules"

Edit _/etc/network/interfaces_ file:

    sudo nano /etc/network/interfaces

Add the following line to the end of the file:

    pre-up iptables-restore < /etc/iptables.rules

Save and exit.

Now reboot the system:

    sudo reboot

> #### Step 10 - Update the system packages and kernel

    sudo apt udpate && sudo apt upgrade -y

> #### Step 11 - Set up SSH keys

**VERIFY AT THE COMMAND PROMPT THAT YOU ARE UNDER YOUR USER ACCOUNT AND NOT EXECUTING CODE AS SUPER USER OR ROOT**

Generate an SSH key:

    cd ~
    ssh-keygen -f ~/.ssh/id_rsa -t rsa -N ''

Copy the key to the authorized_keys file:

    cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys

* * *

## MPI

> #### Step 1 - Create Directories

Install some required compilers and packages:

    sudo apt install make build-essential gfortran

Create _/software_ directory:

    sudo mkdir -p /software/lib/mpich_3

Create hpc user group:

    sudo groupadd hpc

Add user to hpc user group:

    sudo usermod -aG hpc <username>

Take ownership of _/software_:

    sudo chown -R <username>:hpc /software

Change to the _mpich-3.2_ directory and create _build_ and _install_ directories:

    cd /software/lib/mpich_3

    mkdir build install

> #### Step 2 - Download and install

Download MPICH3 package:

    wget http://www.mpich.org/static/downloads/3.3/mpich-3.3.tar.gz

Untar the package:

    tar xvfz mpich-3.3.tar.gz

Change to _build_ directory to begin building the install:

    cd build

Configure the install:

    /software/lib/mpich_3/mpich-3.3/configure  --prefix=/software/lib/mpich_3/install

Compile the install:

    make
    make install

Add MPI location to system environment variable PATH:

    export PATH=$PATH:/software/lib/mpich_3/install/bin

Make the PATH change permanent by adding it to the profile file:

    sudo nano ~/.bashrc

Add the following to the end of the file:

    export PATH="$PATH:/software/lib/mpich_3/install/bin"

Save and exit

> #### Step 3 - Create Node List

Create a list of nodes for MPI to use:

    cd ~

    sudo nano nodelist

Save and exit.

Add the _head node_ ip address to the file:

    192.168.10.5

> #### Step 4 - Test MPI

    cd ~

**Test 1**

    mpiexec -f nodelist hostname

Output:

    head

**Test 2**

    mpiexec -f nodelist -n 2 /software/lib/mpich_3/build/examples/cpi

Output:
<img src="images\part2step4.png" alt="Step 4 MPI" style="width: 450px;"/>

Shutdown the head node:

    sudo shutdown -h now

* * *

## Set up Cluster Compute Node

> #### Step 1 - Clone the Virtual Machine

In VirtualBox right click the _Head Node_ in the left column and select **Clone**

Set _Name_ to **Compute Node 1**

Click **Next**

Select **Full clone**

Click **Clone**

> #### Step 2 - Set Static IP Address

In VirtualBox select _Compute Node 1_ in the left column

With _Compute Node 1_ selected click **Start** in the toolbar

Login to _Compute Node 1_

Edit _/etc/network/interfaces_ file:

    sudo nano /etc/network/interfaces

Remove all of the following lines:

    # Secondary Interface - cluster connection enp0s8
    auto enp0s8
    iface enp0s8 inet static
    address 192.168.10.5
    netmask 255.255.255.0
    network 192.168.10.0

Also remove the line below:

    pre-up iptables-restore < /etc/iptables.rules

Under the line `auto enp0s3` change or add the following:

    iface enp0s3 inet static
    address 192.168.10.100
    netmask 255.255.255.0
    gateway 192.168.10.5
    dns-nameservers 8.8.8.8

Save and exit

Shutdown the _Compute Node 1_:

    sudo shutdown -h now

> #### Step 3 - Change Compute Node 1 Network Adapters

In VirtualBox right click _Compute Node 1_ in the left column

Select **Settings**

Click **Network** and select **Adapter 2**

Uncheck the **Enable Network Adapter** box

Next, select **Adapter 1** tab

Set _Attached to:_ to **Internal Network**

Set _Name:_ to **cluster**

> #### Step 4 - Set hostname

In VirtualBox select _Compute Node 1_ in the left column

With _Compute Node 1_ selected click **Start** in the toolbar

Login to _Compute Node 1_

Edit the hostname file:

    sudo nano /etc/hostname

Change `head` to `node0`

Save and exit

Edit the hosts file:

    sudo nano /etc/hosts

Change

    127.0.0.1      head

to

    127.0.0.1      node0

Save and exit

Now reboot _Compute Node 1_

    sudo reboot

Wait for _Compute Node 1_ to reboot before continuing

> #### Step 5 - SSH into Compute Node 1 to Acquire Authentication key

In VirtualBox select _Head Node_ in the left column

With _Head Node_ selected click **Start** in the toolbar

Login to _Head Node_

On _Head Node_ enter:

    ssh <username>@192.168.10.100

Type `yes` and press `Enter` when asked _Are you sure you want to continue connection (yes/no)?_

Type `exit` and press `Enter` to return to _Head Node_

Verify _Head Node_ by checking the command prompt for `<username>@head:~$`

> #### Step 6 - Add Compute Node 1 to the nodelist File on Head Node

On the _Head Node_ edit the nodelist:

    cd ~
    sudo nano nodelist

Add `192.168.10.100` to the second line

Save and exit

**On _Both nodes_**

Edit the /etc/sudoers file:

    sudo visudo

Add this line to the end of the file:

    <username> ALL=NOPASSWD: /usr/bin/rsync *

On the _Head Node_ deploy Head Node SSH Key to the compute node.

Issue the following command for each node:

    rsync -a --rsync-path="sudo rsync" ~/.ssh/authorized_keys <username>@nodeX:~/.ssh/authorized_keys

> #### Step 7 - Test MPI

**Test 1**

On the _Head Node_ enter:

    cd ~
    mpiexec -f nodelist hostname

You should get an output similar to the following:

    head
    node0

**Test 2**

On the _Head Node_ enter:

    cd ~
    mpiexec -f nodelist -n 6 /software/lib/mpich_3/build/examples/cpi

You should get an output similar to the following:

<img src="images\part2step7.png" alt="Step 7 MPI Compute Node" style="width: 450px;"/>

**_Note:_** Each process shows which node it was executed on. You should see both head and node0 displayed. This shows that MPI is sending and executing the script on both nodes in the cluster.

Congratulations! This cluster is ready to execute MPI code.

* * *

## Slurm on Head Node

> #### Step 1 - Install needed packages

Execute:

    sudo apt-get install slurm-wlm slurmctld

> #### Step 2 - Develop configuration file

Edit _/etc/slurm-llnl/slurm.conf_ and add or edit to match the following:

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
    NodeName=node0 CPUs=1 State=UNKNOWN
    PartitionName=vmcluster Nodes=head,node0 Default=YES MaxTime=INFINITE State=UP

> #### Step 3 - Verify Slurm Controller is running

    scontrol show daemons

> #### Step 4 - Create Munge authentication keyboard

    sudo /usr/sbin/create-munge-key

> #### Step 5 - Fix Munge issue so it will load

    sudo systemctl edit --system --full munge

Change this line:

    ExecStart=/usr/sbin/munged

To:

    ExecStart=/usr/sbin/munged --syslog

Save and exit.

Start Munge:

    sudo systemctl start munge

_**Note:**_ The _systemctl enable munge_ may show a failed notification but its fine. Just move to the next command.

> #### Step 6 - Enable Slurm Controller

    sudo systemctl enable slurmctld

Complete the automatic install:

    sudo apt-get upgrade -y

> #### Step 7 - Set create and set permissions on Slurm folder

    sudo mkdir -p /var/lib/slurmd
    sudo chown -R slurm:slurm /var/lib/slurmd

Reboot:

    sudo reboot

> #### Step 8 - Verify Munge and Slurm are running

    sudo service munge status

Should show _Active: active (running)_

    sudo service slurmctld status

Should show _Active: active (running)_

> #### Step 9 - Verify Slurm has started the partition

    sinfo

Should show two entries. Look for _head_ under nodelist. It's state should be _idle_. The other entry is for _node0_ that we have not set up yet.

* * *

## Slurm on Compute Node

> #### Step 1 - Install Slurm

**On _compute node_**

    sudo apt-get install slurmd slurm-client

> #### Step 2 - Copy Slurm configuration file and Munge key to _node0_:

**On _head node_**

Copy _munge.key_ file:

    sudo rsync --rsync-path="sudo rsync" /etc/munge/munge.key <username>@node0:/etc/munge/munge.key

Copy _slurm.conf_ file:

    sudo rsync -a --rsync-path="sudo rsync" /etc/slurm-llnl/slurm.conf <username>@node0:/etc/slurm-llnl/slurm.conf

**On _compute node_**

> #### Step 3 - Fix Munge issue so it will boot

    sudo systemctl edit --system --full munge

Change this line:

    ExecStart=/usr/sbin/munged

To:

    ExecStart=/usr/sbin/munged --syslog

Save and exit.

> #### Step 4 - Finish Munge setup

    sudo systemctl start munge

_**Note:**_ The _systemctl enable munge_ may show a failed notification but its fine. Just move to the next command.

> #### Step 5 - Enable Slurm daemon

    sudo systemctl enable slurmd

> #### Step 6 - Set Slurm folder permissions

    sudo mkdir -p /var/lib/slurmd
    sudo chown -R slurm:slurm /var/lib/slurmd

> #### Step 7 - Reboot both nodes

Execute on both nodes:

    sudo reboot

* * *

## Save Your Cluster Snapshot

Once your cluster is working properly you will want to take a snapshot of all nodes. This will allow you to work forward from here but to have a restore point if things don't work out with future changes.

> #### Step 1 - Shutdown All nodes

Execute the shutdown on all nodes:

    sudo shutdown -h now

> #### Step 2 - Snapshot Your Nodes

In VirtualBox right click the node in the left column

In the upper right hand corner of VirtualBox click **Snapshots**

Click the left purple camera icon to take a snapshot of the current machine state

Give the node a name that includes its node name and stage **Example** `Head Node (MPI Stage)` or `Head Node (HADOOP/MPI Stage)`

Click **OK** and you are done

Do this for all nodes and you are safe to begin making changes and producing

**_Note:_** You can snapshot the node anywhere you want by following these instructions. In this case take advantage of the description box after naming the snapshot.

* * *

## Troubleshooting

If having trouble with using rsync commands:

> #### Setup Rsync:

**On _Both nodes_**

Edit the /etc/sudoers file:

    sudo visudo

Add this line to the end of the file:

    <username> ALL=NOPASSWD: /usr/bin/rsync *

> #### Host Verification Key Error

In case of _host verification key_ error when executing MPI follow the steps for deleting and regenerating SSH keys.

On _Head Node_ as user delete previous SSH keys:

    rm -rf ~/.ssh
    mkdir ~/.ssh

Generate new SSH keys:

    cd ~
    ssh-keygen -f ~/.ssh/id_rsa -t rsa -N ''

Copy new SSH keys nodes:
Copy SSH keys to authorized keys:

    cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys

    sudo rsync -a --rsync-path="sudo rsync" ~/.ssh/authorized_keys <username>@nodeX:~/.ssh/authorized_keys

> #### Restore VirtualBox snapshot

In VirtualBox right click the node in the left column

In the upper right hand corner of VirtualBox click **Snapshots**

Select the snapshot you wish to restore from the list

Click the second icon with the loopback green arrow to restore that snapshot

You will be prompted if you want to save a copy of the current machine state. This is a personal choice and is advised if you think you may resolve the situation causing the restore later.

**_Note:_** Remember the rule to save and save often!
