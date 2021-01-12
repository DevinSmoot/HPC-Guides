## Raspberry Pi Cluster Guide

Parts of this guide were developed using an article by Garrett Mills titled [Building a Raspberry Pi Cluster](https://glmdev.medium.com/building-a-raspberry-pi-cluster-784f0df9afbd) on Medium.com. Please see this article for further detailed information. The below guide gets right to the deployment instructions. This guide does deviate greatly at points to follow a more linear path for faster deployment and less back and forth between nodes.

---

### Parts List

- 8 x Raspberry Pi 3 Model B (7 for compute nodes and 1 for head node)
- 8 x MicroSD cards
- 8 x micro-USB power cables
- 1 x 8-port 10/100/1000 network switch
- 1 x 10-port USB power-supply
- 1 x 128GB USB flash drive

---

### Before starting:

Download and install Raspberry Pi OS Lite. The easiest way to do this is by using the new Raspberry Pi Imager tool provided [here](https://www.raspberrypi.org/software/).

Use this tool to install the Raspberry Pi OS Lite image directly to your microSD card for your head node. Instructions are provided on the above linked page.

Now navigate to the microSD card and create an empty file named `ssh` with no extension. This can be done in Windows using Notepad or Notepad++. This will allow SSH to be enabled and allow us to connect to the Raspberry Pi node without needing to use a monitor and keyboard.

---

### Initial configuration

SSH into the Pi with puTTY using the IP address of the Raspberry Pi.

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

Reboot the Raspberry Pi and login as the new _**tempuser**_ account using puTTY.

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

### Connect and Mount Flash Drive

##### Find the drive identifier

Plug the flash drive into on of the USB ports on the head node. To figure out its device location use the `lsblk` command.

```
<insert code snippet from using lsblk command here>
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
UUID="65077e7a-4bd6-47ea-8014-01e0655cc31"
```

Now, edit `fstab` to mount the drive on boot:

```
sudo nano /etc/fstab
```

Add the following line to the end of the file:

```
UUID=65077e7a-4bd6-47ea-8014-01e0655cc31 /hpc ext4 defaults 0 2
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
sudo apt install
```
