---
layout: post
title: "Building a CyberSecurity Home Lab with Proxmox: Part 5 - Active Directory Lab Setup - Part 1"
description: A Guide on Building a Cybersecurity Home Lab using Proxmox
date: 2025-01-19 13:20 -0800
categories: [Security,Home Lab]
tags: [homelab,security,proxmox,]
media_subpath: /images/homelab-guide/part5/
---

![title](/images/homelab-guide/diagrams/front-banner-part5.png)

Banner Background by [Andrea Charlesta](https://www.freepik.com/free-vector/background-wave-gradient-minimalist-style_73392382.htm#fromView=image_search_similar&page=2&position=32&uuid=071173e2-ea25-4619-8e4f-8b709eabb291)

Welcome back! I hope everyone is having a great beginning to their year. I want to give a shoutout to the firefighters in LA fighting those wildfires, you guys have my utmost respect. In this part we are going to be setting up and configuring out Active Directory Lab. It will consist of three VMs. The first VM we will be creating will serve as our domain contorller and DHCP for the two VMs. The Domain Controller (DC) will be ran on Windows Server 2022. The other two VMs will be running Windows 10.

You are more than welcome to run this Active Directory lab with just one Windows 10 client and it will work just fine. I am running it with two because there are certain payloads/attacks that require two client machine, one such being NTLM Relay Attack. We will also be setting up snapshots to revert back to a state before any attacks. This helps make our lives easier by not having to remake the VMs after each attack.

> Microsoft Evaluation Trial Period
> Don't worry about this, Microsoft gives a trial license of x amount of times depending on the OS. It will function completely fine after the trial period is over.
{: .prompt-info }

## Windows ISO Downloads

**Windows Server 2022 (64-bit):** https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US

**Windows 10 Enterprise (64-bit):** https://go.microsoft.com/fwlink/p/?LinkID=2208844&clcid=0x409&culture=en-us&country=US

**VirtIO Download**: https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso

> VirtIO Drivers
> We will need VirtIO drivers for our Windows VM otherwise it will not run on proxmox. Download it normally like you would any ISOs. I will be using the **`Stable virtio-win ISO`** which is what I linked above.  
{: .prompt-info}

## Windows Server VM Creation
Setting up a Windows Machine on Proxmox takes a bit more work than a traditional Linux but its still straightforward. Let's jump right in.

![vm-setup-1](/images/homelab-guide/part5/vm-setup-1.png)

Let's create a new VM. I am going to call it **`lab-DC`** and give the **`VM-ID: 300`**. Click **`Next`**.


![vm-setup-2](/images/homelab-guide/part5/vm-setup-2.png)

Select the Windows Server 2022 ISO and the Virtio Drivers. Make sure it looks like mine!

![vm-setup-3](/images/homelab-guide/part5/vm-setup-3.png)

Match up your **`System`** tab with the screenshot above. As you saw, alot more options are used for Windows lol.

![vm-setup-4](/images/homelab-guide/part5/vm-setup-4.png)

Same thing as the last one. Ensure yours matches the screenshot above. Click **`Next`**.

![vm-setup-5](/images/homelab-guide/part5/vm-setup-5.png)

I will give is an extra core. Set the type to **`x86-64-v2-AES`**. Click **`Next`**.

![vm-setup-6](/images/homelab-guide/part5/vm-setup-6.png)

Provide it **`4096 MiB`** of Memory. Click **`Next`**.

![vm-setup-7](/images/homelab-guide/part5/vm-setup-7.png)

For the Bridge we will use **`vmbr2`** and the VLAN tag will be **`20`**. Click **`Next`**.

![vm-setup-8](/images/homelab-guide/part5/vm-setup-8.png)

Make sure everything looks good. Click **`Confirm`**.

![vm-setup-9](/images/homelab-guide/part5/vm-setup-9.png)

Head over to **Options** -> **Boot Order**. Make sure it matches mine. Boot up the VM afterwards.

![](/images/homelab-guide/part5/vm-setup-10.png)

## Windows Server Installation

![windows-1](/images/homelab-guide/part5/windows-setup-1.png)

Select **`Windows Server 2022 Standard Evaluation (Desktop Experience)`**. Click **`Next`**.

![windows-2](/images/homelab-guide/part5/windows-setup-2.png)

Select **`Custom: Install Windows only (Advanced)`**.

![windows-3](/images/homelab-guide/part5/windows-setup-3.png)

Click **`Load Driver`**

![windows-4](/images/homelab-guide/part5/windows-setup-4.png)

Click **`Browse`**.

![windows-6](/images/homelab-guide/part5/windows-setup-6.png)

Expand the **`CD Drive (D:) virtio-win0.1.266 -> amd64`**. Select the folder **`2k22`**. Click **`Ok`**.

![windows-7](/images/homelab-guide/part5/windows-setup-7.png)

Click **`Next`**.

![windows-8](/images/homelab-guide/part5/windows-setup-8.png)

Click **`Next`**.

![windows-9](/images/homelab-guide/part5/windows-setup-9.png)

Sit back and take a break. Let this finish.

![windows-10](/images/homelab-guide/part5/windows-setup-10.png)

Create a password for the administrator account.

![windows-11](/images/homelab-guide/part5/windows-setup-11.png)

On the VM you should see an arrow. Click on it and it will bring up a menu. You want to select the box with the three squares.This will simulate **Ctrl + Alt + Del**. Afterwards enter your password.

## Windows Server Configuration

Once inside, let's open **CMD** and type **`ipconfig`**. We should verify that we recieved an ip from our DHCP server to which we did.

![ipconfig-1](/images/homelab-guide/part5/ipconfig-1.png)


![ipconfig-2](/images/homelab-guide/part5/ipconfig-3.png)

Click on **Network and Internet Settings**.

![ipconfig-3](/images/homelab-guide/part5/ipconfig-4.png)

Click **`Change Adapter Options`**.

![ipconfig-4](/images/homelab-guide/part5/ipconfig-5.png)

Right Click **`Ethernet`** and Click **`Properties`**.

![ipconfig-5](/images/homelab-guide/part5/ipconfig-6.png)

Select **`Internet Protocol Version 4 (TCP/IPv4)`** and Click **`Properties`**.

![ipconfig-6](/images/homelab-guide/part5/ipconfig-7.png)

Enter the details as shown and then click **`Ok`**.

IP address: **`192.168.20.10`**  
Subnet Mask: **`255.255.255.0`**    
Default gateway: **`192.168.20.20`**  
Preferrred DNS Server: **`192.168.20.20`**  
Alternate DNS Server: **`8.8.8.8`**  

![ipconfig-8](/images/homelab-guide/part5/ipconfig-2.png)

Now that our DC has a static IP. We will turn off the DHCP Server on pfSense through our kali VM. The DC will be made into a DHCP server so before we start the DHCP. We will disable the pfSense DHCP. Navigate to **Servers -> DHCP Server -> AD_LAB** and uncheck the **Enable** box.


![renaming-1](/images/homelab-guide/part5/renaming-1.png)

Let's rename our PC to something else. Head over to **`About This PC`** -> **`Rename this PC`**

![renaming-2](/images/homelab-guide/part5/renaming-2.png)

We will rename to **`DC1`**.

![renaming-3](/images/homelab-guide/part5/renaming-3.png)

Click **`Restart`**.

## Active Directory and Domain Controller Configuration 

![dc-config-1](/images/homelab-guide/part5/dc-config-1.png)

On the Server Manager Window, click **`Manage`** and select **`Add Roles and Features`**.

![dc-config-2](/images/homelab-guide/part5/dc-config-2.png)

Click **`Next`** until you reach **Server Roles**.

![dc-config-3](/images/homelab-guide/part5/dc-config-3.png)

Select **`Active Directory Domain Services`** and **`DNS Server`**. Click **`Next`**.

![dc-config-4](/images/homelab-guide/part5/dc-config-4.png)

Click **`Add Features`** until you reach the **`Confirmation`** page and click **`Install`**.

![dc-config-5](/images/homelab-guide/part5/dc-config-5.png)

Let it finish and then Click **`Close`**.

### Domain Controller Configuration
Let's head back to the server manager and select <u>Promote this server to a domain controller</u>.
![dc-config-6](/images/homelab-guide/part5/dc-config-6.png)


![dc-config-7](/images/homelab-guide/part5/dc-config-7.png)

A Configuration wizard should open up. Select **`Add a new forest`** and we will name the root domain **`ad.lab`**. If you choose to name is something different make sure it is two words separated by a period. click **`Next`**.

![dc-config-8](/images/homelab-guide/part5/dc-config-8.png)

Set a password for the **DSRM** and click **`Next`**.

![dc-config-9](/images/homelab-guide/part5/dc-config-9.png)

Nothing to do here. click **`Next`**.

![dc-config-10](/images/homelab-guide/part5/dc-config-10.png)

The NetBIOS domain name should already be filled out for you. Click **`Next`**.

![dc-config-11](/images/homelab-guide/part5/dc-config-11.png)

Spam click **`Next`** until you hit this page and click **`Install`**. It will prompt you to install after it finishes. You should also notice the name on the login page has changed. The domain is not attached before the username and means we have successfully set up the domain controller.

### DHCP Installation

![dhcp-config-1](/images/homelab-guide/part5/dhcp-config-1.png)

Head on back to the server manager. Click Manage -> **`Add Roles and Features`**.

![dhcp-config-2](/images/homelab-guide/part5/dhcp-config-2.png)

Click **`Next`** until you get to server roles. Select **`DHCP Server`** and then click **`Add Features`**.

![dhcp-config-3](/images/homelab-guide/part5/dhcp-config-3.png)

Click **`Next`** until you get to the confirmation and click **`Install`**.

![dhcp-config-4](/images/homelab-guide/part5/dhcp-config-4.png)

Let it do its thing. Then hit **`Close`**

### DHCP Configuration

Once it has finishing installing. Head back over to the server manager and click the Flag and **`Complete DHCP Configurations`**.

![dhcp-config-5](/images/homelab-guide/part5/dhcp-config-5.png)

Clicking it should bring up a DHCP configuration wizard.

![dhcp-config-6](/images/homelab-guide/part5/dhcp-config-6.png)

Leave everything as default. Click **`Commit`**.

![dhcp-config-7](/images/homelab-guide/part5/dhcp-config-7.png)

Click **`Close`**.

![dhcp-config-1](/images/homelab-guide/part5/dhcp-config-8.png)

Click **Start** and type in DHCP and open it up.

![dhcp-config-9](/images/homelab-guide/part5/dhcp-config-9.png)

Navigate to the sidebar and select **`IPv4`** under **dc1.ad.lab**. Right Click and select **`New Scope`**.

![dhcp-config-10](/images/homelab-guide/part5/dhcp-config-10.png)

Name the scope **`VLAN 20 AD Lab`** and the description will be **`DHCP for Lab`**.

![dhcp-config-11](/images/homelab-guide/part5/dhcp-config-11.png)

Assign the following values:  
Start IP Address: **`192.168.20.50`**  
End IP Address: **`192.168.20.95`**  
Length: **`24`**  
Subnet Mask: **`255.255.255.0`**  

![dhcp-config-12](/images/homelab-guide/part5/dhcp-config-12.png)

Change the Days to **`365`**.

![dhcp-config-13](/images/homelab-guide/part5/dhcp-config-13.png)

Leave it at **Yes**. Click **`Next`**.

![dhcp-config-14](/images/homelab-guide/part5/dhcp-config-14.png)

Enter **`192.168.20.20`** and click **Add**. Hit **`Next`**

![dhcp-config-15](/images/homelab-guide/part5/dhcp-config-15.png)

Nothing to change here. Click **`Next`**.

![dhcp-config-16](/images/homelab-guide/part5/dhcp-config-16.png)

Leave it as **Yes**. Hit **`Next`**.

And... Thats it for the first part! We have basically finished setting up our Windows Server 2022. We set up the Domain Controller, DHCP, and Active Directory. Next up is to create the Windows 10 VMs which will act as our users in the Domain Controller and Active Directory. Stay tuned for the second part of the AD Lab Segment!

















