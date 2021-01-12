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

---
