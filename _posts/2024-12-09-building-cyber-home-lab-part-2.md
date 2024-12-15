---
title: "Building a CyberSecurity Home Lab with Proxmox: Part 2 - Kali Linux Setup"
date: 2024-12-10 16:41 -0800
description: A Guide on Building a Cybersecurity Home Lab using Proxmox
categories: [Security,Home Lab]
tags: [homelab,security,proxmox,linux,networking]     # TAG names should always be lowercase
published: true
---
![title-banner](/images/homelab-guide/diagrams/front-banner-part2.png)  


Banner Background by [AndreaCharlesta](https://www.freepik.com/free-vector/background-wave-gradient-minimalist-style_73392382.htm#fromView=image_search_similar&page=2&position=32&uuid=071173e2-ea25-4619-8e4f-8b709eabb291)

Welcome back to the second part! I apologize for the delay, the christmas season has brought a cold and busy shopping for gifts. Lets jump right in with installing Kali Linux. Once Kali Linux is installed we will start configuring the pfSense Firewall through the web interface which is alot better to configure rather than doing it through the actual pfSense VM.

## Downloading Kali Linux
There are two types of flavors of Kali Linux:
- Kali Purple: I call it the SOC In-A-Box that has tools more orientated towards defensive purposes  
- Kali Linux: Standard kali linux orientated towards offensive purposes  

For the purpose of the lab, I will be downloading Kali Purple but you are more than welcome to choose either and the set up will be slightly different and both will do everything you need.

![kali-version](/images/homelab-guide/part2/kali-version.png)


Latest version of Kali Linux: **`2024.3a`**
Links:  
[Get Kali Purple](https://cdimage.kali.org/kali-2024.3/kali-linux-2024.3a-installer-purple-amd64.iso)  
[Get Kali Linux](https://cdimage.kali.org/kali-2024.3/kali-linux-2024.3-installer-amd64.iso)

Once you have picked your flavor of choice. Go ahead and upload this to your ISO image storage. Refer to the previous post on how to upload it.

## Kali Linux VM Creation
The process is similar to when we created the pfSense VM. Minor changes to the ram and memory.  

![kali-1](/images/homelab-guide/part2/kali-1.png)  

Lets create the VM. Give it a name and Hit Next  

![kali-2](/images/homelab-guide/part2/kali-2.png)

Let's select our kali linux iso image.

![kali-3](/images/homelab-guide/part2/kali-3.png) 

For the system tab, leave everything default and hit next. On the Disks tab select your **`Storage`** and Size I did **`100GB`** to give me space to download extra tools if needed.  


![kali-4](/images/homelab-guide/part2/kali-4.png)

Single core is enough but if you can I would give it 2 just in case.

![kali-5](/images/homelab-guide/part2/kali-5.png) 

I gave mine **`8GB`** of Memory because I have 64GB but it will do fine with 4GB (4096 MiB).

![kali-6](/images/homelab-guide/part2/kali-6.png)  

For our network interface, give it **`vmbr1`** which is our Lab LAN

![kali-7](/images/homelab-guide/part2/kali-7.png)

Confirm everything looks good and click Finish.

## Kali Linux Installation  
Once the VM is created and boot it up and let it initialize until you see the screen below.

![kali-8](/images/homelab-guide/part2/kali-8.png)  

Press enter on **`Graphical Install`**.

![kali-9](/images/homelab-guide/part2/kali-9.png) 

![kali-10](/images/homelab-guide/part2/kali-10.png) 

![kali-11](/images/homelab-guide/part2/kali-11.png) 

Select your language, location, and keyboard layout. Straight forward stuff.

![kali-12](/images/homelab-guide/part2/kali-12.png) 

The hostname is used to identify the kali machine on the network. I usually leave it as **`kali`** but you can name it anything you like.

![kali-13](/images/homelab-guide/part2/kali-13.png) 

This won't apply to you probably. Leave it blank.

![kali-14](/images/homelab-guide/part2/kali-14.png) 

Enter your name to create your user account.

![kali-15](/images/homelab-guide/part2/kali-15.png) 

Enter your **`username`** for the home directory.

![kali-16](/images/homelab-guide/part2/kali-16.png) 

Go ahead and give it a memorable but strong password.

![kali-17](/images/homelab-guide/part2/kali-17.png)

Set your time zone.

![kali-18](/images/homelab-guide/part2/kali-18.png) 

Use **`Guided- use entire disk`** and click continue.

![kali-19](/images/homelab-guide/part2/kali-19.png) 

Select the **`sda`** drive and click continue.

![kali-20](/images/homelab-guide/part2/kali-20.png)

Select **`All files in one partition`** then Continue.

![kali-21](/images/homelab-guide/part2/kali-21.png) 

Select **`Finish partitioning and write changes to disk`** and click contiune.

![kali-22](/images/homelab-guide/part2/kali-22.png) 

Select **`Yes`** to **`Write Changes to disks?`** and hit **`Continue`**. It will start installing so you can take a small break here maybe grab a snack or something.

![kali-23](/images/homelab-guide/part2/kali-23.png) 

Once it has finished, it will ask you to select a desktop environment and default tools. Xfce is kali's default environment and it standard. **`Gnome`** is a step up and visually appealing while still being lightweight. **`KDE Plasma`** is the top trim with all the visual features but it does take up more resources and would recommend to dedicate at least **`2 cores`** and **`4GB RAM`** core to keep it running smoothly. I chose KDE Plasma because I have the resources to spare. It does not matter which desktop environment you choose it's all the same.  

**`Leave the tools all checked as default.`**

![kali-24](/images/homelab-guide/part2/kali-24.png) 

Select **`Yes`**

![kali-25](/images/homelab-guide/part2/kali-25.png)  

Select **`/dev/sda`** and hit **`Continue`**. Hit **`Continue`** on the next window to reboot.

![kali-26](/images/homelab-guide/part2/kali-28.png)  

> **Stuck on Reboot**  
> If the reboot gets stuck. Right click the VM on the proxmox menu and click **`Stop`** .This will force shutdown the vm and start it up.
> ![kali-27](/images/homelab-guide/part2/kali-27.png)
{: .prompt-info }  

Reboot your system and it will take you to the login screen. Enter the password you created earlier.

![kali-28](/images/homelab-guide/part2/kali-29.png) 
![kali-29](/images/homelab-guide/part2/kali-30.png)  

Open up terminal and run **`ip a`** and verify kali has an IP from DHCP server we setup in the last part.

![kali-31](/images/homelab-guide/part2/kali-31.png) 
Before anything is done, let's run updates first:

```bash
sudo apt update && sudo apt upgrade && sudo apt autoremove
```
It will prompt for your password and if you want to continue. This might take a while, so perfect time for a snack. This command will grab the latest updates/upgrades and apply them to kali. Afterwards it will then clean up the unused packages.  

## Clean Up
We are going to clean up the kali VM. After shutting down navigate to your VM and Navigate to Hardware.

![cleanup](/images/homelab-guide/part1/cleanup.png)

Select **``CD/DVD Drive``** and click Remove. This will remove the ISO image from the VM as it is no longer needed. 

Keep an eye out for the next part where we will start configuring the Firewall through the pfSense Web interface.


