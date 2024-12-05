---
title: "Building a CyberSecurity Home Lab with Proxmox: Part 1 - Network and pfSense Setup"
description: A Guide on building a cybersecurity home lab using Proxmox
date: 2024-11-28 18:09:00 -0800
categories: [cybersecurity, homelab]
tags: [homelab,security,proxmox,networking]     # TAG names should always be lowercase
published: true
---

![network-diagram](/images/homelab-guide/diagrams/Front%20Banner%20Logo.png)

Welcome! In this project, we will be setting a Cybersecurity Home Lab using the Open Source Hypervisor Proxmox. I was originally going to do ESXI and Virtualbox but after what Broadcom did to their customers and the community, I decided to go another route. That's where I found proxmox and after hours of frustration and tears I finally grasped it! 

That being said, I currently have Proxmox running natively on a separate system (specifications below) that I have built for a homelab. I strongly recommend you do the same as Proxmox is a Type 1 Hypervisor and works best when its installed as the OS.

Throughout the entire project, I will be posting modules instead of one giant post to make it easier to follow along. The modules will consist of a different component of the lab that we will set up together.

## Shoutouts
With that being said, I want to give a couple of shoutouts to the following Home Lab guides that helped me create my personal homelab and inspired me to create my own:
- [Building Blue Team Home Lab Part 1 - Introduction \| facyber](https://facyber.me/posts/blue-team-lab-guide-part-1/)

- [Building a Virtual Security Home Lab: Part 1 - Network Topology \| David Varghese](https://blog.davidvarghese.net/posts/building-home-lab-part-1/)

- [So you want to be a SOC Analyst? Part 1 \| Eric Capuano](https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-part)
  
- [Ultimate Cyber Security Home Lab with Proxmox \| Corey Jones](https://medium.com/h7w/ultimate-cyber-security-home-lab-with-proxmox-part-1-setting-up-the-network-2f70c91606ff)

## Project Overview
- pfSense (Firewall & Gateway)
- Kali Linux (Management VM)
- Active Directory Lab (Domain controller and Windows Machine)
- Security VMs (Ubuntu Server and Windows VM (LimaCharlie EDR))
- Sandbox (Capture the Flag Practice)
- Malware Lab (Linux and Windows)

![network-diagram](/images/homelab-guide/diagrams/network-diagram.svg)

## My Hardware
![system-picture](/images/homelab-guide/part1/mysystem.JPG)
- <b>Case</b>: Fractual Node 804
- <b>Motherboard</b>: Asus Prime B650M-A AX II
- <b>CPU</b>:AMD Ryzen 5 7600x
- <b>RAM</b>: G.SKILL 64GB DDR5 6000
- <b>PSU</b>: BeQuiet 750W ATX 80 Gold
- <b>GPU</b>: Nvidia GTX 1070
- <b>Storage</b>: 1TB SSD, 250GB NVME, 4 1TB Mechanical Hard Drive (RAID 1+0)

## Recommended Hardware from Proxmox
- Intel 64 or AMD64 with Intel VT/AMD-V CPU flag.
- Memory, minimum 2 GB for OS and Proxmox VE services. Plus designated memory for guests. For Ceph or ZFS additional memory is required, approximately 1 GB memory for every TB used storage.
- Fast and redundant storage, best results with SSD disks.
- OS storage: Hardware RAID with batteries protected write cache (“BBU”) or non-RAID with ZFS and SSD cache.
- VM storage: For local storage use a hardware RAID with battery backed write cache (BBU) or non-RAID for ZFS. Neither ZFS nor Ceph are compatible with a hardware RAID controller. Shared and distributed storage is also possible.
- Redundant Gbit NICs, additional NICs depending on the preferred storage technology and cluster setup – 10 Gbit and higher is also supported.
- For PCI(e) passthrough a CPU with VT-d/AMD-d CPU flag is needed.

## Enabling Virtualization
Before installing Proxmox we need to make sure virtualization is enabled on the system.On Windows this can be checked by opening up Task Manager -> Performance and checking if Virtualization is set to Enabled.

![system-picture](/images/homelab-guide/part1/windows-taskmanager.png)
If for whatever reason you do not have virtualization enabled, then you will need to enabled as it is disabled in the BIOS. If you do not see virtualization at all, you are out of luck and your system does not support it. There many flavors of BIOS's that I cannot cover so instead I will link below some resources on how to enable your Virtualization through your BIOS! 

> [!INFO] Virtualization Tip 
> I always found googling my motherboard name + virutalization does the trick in finding a guide.

[Enabling CPU Virutalization by ninjaOne](https://www.ninjaone.com/blog/how-to-enable-cpu-virtualization-in-your-computer-bios/)

## Installing Proxmox
### Downloading Proxmox
As I am writing this guide the latest version of Proxmox is 8.3.
You can download it from this link: [Proxmox VE 8.3 ISO Installer](https://www.proxmox.com/en/downloads)

Now that we have the ISO image, you will need a usb thumb drive with at least 5GB of space and make sure to remove any information off of it because we will be making it a bootable drive. I used [Rufus](https://rufus.ie/en/) to burn the ISO onto the usb drive.

Lastly, I will not go step by step through the proxmox setup because Craft Computing made an amazing video explaining it that I would not be able to create anything nearly as helpful. With that being said please watch his video! 

[Craft Computing Proxmox Setup](https://www.youtube.com/watch?v=sZcOlW-DwrU&t=334s)

## Setting Up Open Switch and Linux Bridge
Before we get down and dirty with setting up pfSense we first must configure our virtual switch for our VLANS and our virtual WAN/LAN for pfSense. Lets navigate over to our node's shell.

![shell](/images/homelab-guide/part1/proxmox-shell.png)

Let's run an update command because why not.
```bash
apt update
```

> [!IMPORTANT]  Errors
> Most likely you will get an error saying something like "updating from repo can't be done securely" this is due to not having a valid subscription.
> 
>Below I will link a video going over this issue and a link to Proxmox Documentation on switching over to the No-Subscription Repository


[Proxmox Update No Subscription Repository Video](https://www.youtube.com/watch?v=DzHRhu3On7o)

[Package Repositories](https://pve.proxmox.com/wiki/Package_Repositories)

```bash
apt install openvswitch-switch
```
## VLAN Configuration
Once it finished installing, we are good to set up our bridges and VLANs! First we will do our linux bridge that will act as the lan.
![linuxbridge](/images/homelab-guide/part1/linuxbridge.png)

Once we click linux bridge we will be presented with the window below.

![lab-lan-bridge](/images/homelab-guide/part1/labLanBridge.png)

Go ahead and give the bridge a name or leave it the same as me. You can also assign it an IP like 10.10.1.0/24 in my case I have it set to 192.168.1.0 because my home network uses the 10.10.xx.xx scheme and I encountered some issues. 

Leaving a comment is useful so you don't forget what this bridge was made for. Volia! We created our LAN for our lab environment. 

![ovsbridge](/images/homelab-guide/part1/ovsbridge.png)

Now, we can create the bridge for our virtual switch. Located in the same place as the Linux Bridge just further down the list.

![vlaninterface](/images/homelab-guide/part1/vlaninterface.png)

Same thing as the other bridge. We give it a name and a comment to remember what it is. This interface will allow us to select the VLAN we want to assign to each VM.

![ovsintport](/images/homelab-guide/part1/ovsintport.png)
Let's go ahead and create our VLANs using the OVS IntPort.

![vlans](/images/homelab-guide/part1/VLAN-IntPort.png)

We will be having four total VLANs. You can choose how to name each one but I will keep it simple and do 10,20,30,40. The process is the same for each one. Give it a name, set the Bridge to be the our VLAN Interface (vmbr2), give it an IP, and give it a tag. You can also add a comment as well. I actually went back later and added this. I referred to the network diagram and left a comment corresponding to what I designated each vlan for.

After creating the Vlans you should have something like this.

VLAN    |        IP       |     Tag
- VLAN 10: 192.168.10.10/24, 10
- VLAN 20: 192.168.20.20/24, 20
- VLAN 30: 192.168.30.30/24, 30
- VLAN 40: 192.168.40.40/24, 40

![applyconfig](/images/homelab-guide/part1/Apply-Config.png)

After all is set and done. Click Apply Configuration and we are ready to move onto setting up the pfSense Virtual Machine.

## pfSense Setup
### Downloading pfSense
![pfSenseISO](/images/homelab-guide/part1/pfSense-ISO.png)

Before we create the VM, we want to either upload or Download the ISO Image to our storage to use. For me, I will be uploading my ISO Image into ISO (Darla) but yours will be different, most likely if you are running default it will be something like "storage".

[pfSense ISO Download](https://sgpfiles.netgate.com/mirror/downloads/)

### Creating pfSense VM
Let's create the VM! Click on Create VM in the Top Right.

![vm-creation-1](/images/homelab-guide/part1/vm-creation-1.png)

Set your VM ID to be anything you want. I am doing 101. Other than that click Next.

![vm-creation-2](/images/homelab-guide/part1/vm-creation-2.png)

We will select the pfSense ISO from the storage. Click Next.

![vm-creation-3](/images/homelab-guide/part1/vm-creation-3.png)

Leave everything default. Next.

![vm-creation-4](/images/homelab-guide/part1/vm-creation-4.png)

Select your storage and at least 32GB will be enough for pfSense but I will use 40GB. Click Next.

![vm-creation-5](/images/homelab-guide/part1/vm-creation-5.png)

Default is fine but I have core to spare so I will do 2.

![vm-creation-6](/images/homelab-guide/part1/vm-creation-6.png)

Default memory of 2GB works fine for pfSense. Next!

![vm-creation-7](/images/homelab-guide/part1/vm-creation-7.png)

Leave the Bridge as default. After the creation of the vm we will feed it the second bridge.

![vm-creation-8](/images/homelab-guide/part1/vm-creation-8.png)

Verify everything looks good and click finish.

![vm-creation-9](/images/homelab-guide/part1/vm-creation-9.png)

Go ahead and navigate back to the newly created VM. Go to hardware -> Add -> Network Device

![network-adapter](/images/homelab-guide/part1/network-adapter-addon.png)

Go ahead and add vmbr 1 (lab lan) and vmbr2 (vlan interface) to the VM.

![vm-startup](/images/homelab-guide/part1/vm-startup.png)

We are now finished with setting it up and we can finally start it up!

## pfSense Installation

![pfsense-installation-1](/images/homelab-guide/part1/pfsense-installation1.png)

It'll take a few minutes but eventually it'll boot to this screen. Hit Accept

![pfsense-installation-2](/images/homelab-guide/part1/pfsense-installation2.png)

Leave it all default. Hit Install

![pfsense-installation-3](/images/homelab-guide/part1/pfsense-installation3.png)

Leave it at default again. Hit ok

![pfsense-installation-4](/images/homelab-guide/part1/pfsense-installation4.png)

Nothing to change here.. Hit Select and install

![pfsense-installation-5](/images/homelab-guide/part1/pfsense-installation5.png)

Same thing. Hit ok unless you know what you're doing.

![pfsense-installation-6](/images/homelab-guide/part1/pfsense-installation6.png)

Hit Space and then Ok.

![pfsense-installation-7](/images/homelab-guide/part1/pfsense-installation7.png)

Looks scary but its harmless. Hit yes and take a breather and maybe some water.

![pfsense-installation-8](/images/homelab-guide/part1/pfsense-installation8.png)
Once it finishes you might see a screen like this. Just got ahead and type exit and it will reboot.

![pfsense-installation-10](/images/homelab-guide/part1/pfsense-installation10.png)

Once it finishes rebooting, we'll need to configure the interfaces manually.

Should VLANs be set up now? **`n`** \
Enter the WAN Interface name: **`vtnet0`**
  
![pfsense-installation-11](/images/homelab-guide/part1/pfsense-installation11.png)

Enter the LAN interface name: **`vtnet1`**

![pfsense-installation-12](/images/homelab-guide/part1/pfsense-installation12.png)

Enter the Optional 1 interface name: **`vtnet2`** \
Do you want to proceed?: **`y`**

We will set up OPT1 in the pfSense interface so don't worry if it doesn't have an IP Address


### Configuring LAN (vtnet1)
 Personally, I do not need to configure the lan as this is the ip address I want to use, however for those using a 10.10.x.x or another ip scheme I will go over it!

> [!INFO]
> Your WAN will have an IP assigned by your DHCP server, I blocked out mine for security reasons.

![pfsense-installation-13](/images/homelab-guide/part1/pfsense-installation14.png)

Enter **`2`** to select the Interface to assign

![lanconfig](/images/homelab-guide/part1/lan-config.png)

Enter an option: **`2`** \
Enter the number of the interface you wish to configure: **`2`** \
Configure IPv4 Address lan interface via DHCP? **`n`** \
Enter the new LAN IPv4 Address: **`192.168.1.1`** \
Enter the new LAN IPv4 subnet bit count: **`24`**

![lan-config2](/images/homelab-guide/part1/lan-config2.png)
Configure IPv6 Address LAN interface via DHCP6? **`n`** \
Enter the new LAN IPv6 address: **`NONE`** \
Do you want to enable the DHCP Server on LAN? **`y`** \
Enter the Start address of the Range: **`192.168.1.50`** \
Enter the end address of the Range: **`192.168.1.100`** \
Do you want to revert to HTTP as the webConfigurator Protocol?: **`n`**

![lan-config3](/images/homelab-guide/part1/lan-config3.png)

Once you have reached this part we are essentially done with the configuration of pfSense and in the next we will continue configuring pfsense through the Web Interface via Kali Linux.

## Clean Up
Doesn't hurt to clean up your environment a bit. After shutting down navigate to your VM and Navigate to Hardware.

![cleanup](/images/homelab-guide/part1/cleanup.png)

Select **``CD/DVD Drive``** and click Remove. This will remove the ISO image from the VM as it is no longer needed. Feel free to remove pfSense ISO image from your storage drive if you want the extra space.

Keep an Eye out for the next part where we will set up Kali Linux and start configuring the Firewall on pfSense.
 







