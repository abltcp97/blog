---
layout: post
title: "Building a CyberSecurity Home Lab with Proxmox: Part 6 - Active Directory Lab Setup - Part 2"
description: A Guide on Building a Cybersecurity Home Lab using Proxmox
date: 2025-01-22 10:37 -0800
categories: [Security,Home Lab]
tags: [homelab,security,proxmox,]
---

![title](/images/homelab-guide/diagrams/front-banner-part6.png)

Banner Background by [Andrea Charlesta](https://www.freepik.com/free-vector/background-wave-gradient-minimalist-style_73392382.htm#fromView=image_search_similar&page=2&position=32&uuid=071173e2-ea25-4619-8e4f-8b709eabb291)

Welcome back! In the last part, we installed and configured Windows Server 2022, installed the AD domain, and configured DHCP on the DC. In this part, we will continue with the final configurations for our Windows Server and finally adding Windows machines to our environment.

## Windows Server Configuration

### Certificate Services Setup

Let's head over to our Server VM. In the Server manager, select **`Add Roles and Features`**.

![server-config-1](/images/homelab-guide/part6/server-config-1.png)


Keep clicking **`Next`** until you hit <u>Select Server Roles</u>. Select <u>Active Directory Certificate Services</u> then hit **`Next`**.


![server-config-2](/images/homelab-guide/part6/server-config-2.png)

Click **`Add Features`**.   

![server-config-3](/images/homelab-guide/part6/server-config-3.png)

Hit **`Next`** until you reach <u>Certification Authority</u> and select it. Click **`Next`** at the bottom.

![server-config-4](/images/homelab-guide/part6/server-config-4.png)

Click **`Next`** until you get the <u>Confirmation</u> page and click **`Install`**.

![server-config-5](/images/homelab-guide/part6/server-config-5.png)

**`Restart`** your windows server. Once it's back up head over to the sever manager and click the flag. Then **`Configure Active Directory Certifcate Services on the..`**

![server-config-6](/images/homelab-guide/part6/server-config-6.png)

When you reach this page, select **`Certification Authority`** then hit **`Next`**.

![server-config-7](/images/homelab-guide/part6/server-config-7.png)

Keep clicking **`Next`** and leave everything default. Once you reach the <u>Confirmation</u> screen go ahead and click **`Configure`**. Let it do its thing.

![server-config-8](/images/homelab-guide/part6/server-config-8.png)

Click **`Close`** once its finshed. And Boom! We are doing configuring the Certificate Authority!

![server-config-9](/images/homelab-guide/part6/server-config-9.png)

## Active Directory User Configuration

### Domain Users

Now, we move on to creating the AD users for our domain. Can't have a house without any people right?

![server-config-10](/images/homelab-guide/part6/server-config-10.png)

Click the Start Menu and type <u>Active Directory Users and Computers</u>.

![server-config-11](/images/homelab-guide/part6/server-config-11.png)

Right Click on **`ad.lab`**. Then **`New -> User`**.

![server-config-12](/images/homelab-guide/part6/server-config-12.png)

Go ahead and enter the details. I just entered my Name and my username. Since I am the admin. Click **`Next`**.

![server-config-13](/images/homelab-guide/part6/server-config-13.png)

Create a super secure password. Select <u>Password Never Expires</u>. Click **`Next`**.

![server-config-14](/images/homelab-guide/part6/server-config-14.png)

Double click **`Domain Admins`**.

![server-config-15](/images/homelab-guide/part6/server-config-15.png)

Navigate to  **`Members`** tab. Click **`Add`**

![server-config-16](/images/homelab-guide/part6/server-config-16.png)

Enter the name of the user we just created earlier. Click **`Check Names`** and then **`Ok`**.

![server-config-17](/images/homelab-guide/part6/server-config-17.png)

Click **`Apply`**.

![server-config-18](/images/homelab-guide/part6/server-config-18.png)

Sign out and Click <u>Other User</u>. Sign in with the username and password of the user we made earlier.


### Domain User 1 Setup

Now, that we are signed in as a Domain Admin. We are going to create our first standard user and then afterwards will create a VM to sign in with that user. 

![server-config-20](/images/homelab-guide/part6/server-config-20.png)

Navigate back to <u>Active Directory Users and Computers</u>. Right click <u>ad.lab -> New -> User</u>.

![server-config-21](/images/homelab-guide/part6/server-config-21.png)

Create a user with any name you want. I chose John Snow.

![server-config-22](/images/homelab-guide/part6/server-config-22.png)

Give it a password. Select **`User cannot change password`** and **`Password Never Expires.`**. Click **`Next`**.

![server-config-23](/images/homelab-guide/part6/server-config-23.png)

Completely optional but I went ahead and created a second user. I named the second user Jane Seymour because why not.

### VM Snapshots
After doing all that work, it would be a shame if something went bad and you had to start all over again. Introducing Snapshots! This wonderful features allows us to save a snapshot of a VM and revert back to it if anything ever happens. This is especially useful if you decide to toggle between a vulnerable and a non-vulnerable AD Configuration. I will quickly show you how its set up.

![server-config-24](/images/homelab-guide/part6/server-config-24.png)

Navigate to your proxmox server and click on the Windows Server VM. Click <u>Snapshots</u>. Notice it says <u>The current guest configuration dos not support taking new snapshots</u>. Oh no! Its over we can never use it. Just joking!

![server-config-25](/images/homelab-guide/part6/server-config-25.png)

Navigate to **`Hardware`** tab. Notice how **`TPM State`** is in a **`.raw`** type. Proxmox only support **`.qcow2`** for snapshots. So what we are going to do is simply select it and remove it. Boom! Done!

![server-config-26](/images/homelab-guide/part6/server-config-26.png)

Now, you should have the option to **`Take Snapshot`**.

![server-config-27](/images/homelab-guide/part6/server-config-27.png)

Give it a name. Since this Ad Lab is "Vanilla" and I haven't made it vulnerable I am going to call it that and give it a quick description so I remember what the snapshot is. Click **`Take Snapshot`**

![server-config-23](/images/homelab-guide/part6/server-config-28.png)

And... That's it! Let it finish and click the X and you have a snapshot!


### Making the AD Lab Vulnerable
So this section is completely option in making the AD Lab vulnerable and will require changing some settings. I will not be doing this but if you decide you want to. Check out David Varghese's guide on his AD lab and he a section where he goes over the settings you need to disable to this. I will link it [here](https://blog.davidvarghese.net/posts/building-home-lab-part-7/#making-ad-lab-exploitable). He does a great job guiding and I took inspiration from him! Shoutout to you David!

## Windows 10 VM Setup
Now, we will setup the Windows VMs for the users we have created! It'll be straightforward you probably won't need help from me but in case you do here it is!

![Win10-1](/images/homelab-guide/part6/WinSetup-1.png)

Let's create a new VM. Fill it out with the neccessary details. Then hit **`Next`**.

![Win10-2](/images/homelab-guide/part6/WinSetup-2.png)

We will select our Windows 10 ISO and set the OS to <u>Microsoft Windows</u>. We will also add the <u>VirtlO</u> drivers. Hit **`Next`**.

![Win10-3](/images/homelab-guide/part6/WinSetup-3.png)

Make sure it matches my image above. Click **`Next`**.

![Win10-1](/images/homelab-guide/part6/WinSetup-4.png)

I gave it 64GB because I have space to spare. You can give it half and be alright. Make sure to select **`Write Back`** for Cache.

![Win10-5](/images/homelab-guide/part6/WinSetup-5.png)

Give it 2 Cores and make sure Type is set to **`x86-64-v2-AES`**.

![Win10-6](/images/homelab-guide/part6/WinSetup-6.png)

Give it **`4096 MiB`**. It'll be sufficient! 

![Win10-7](/images/homelab-guide/part6/WinSetup-7.png)

We will assign it **`vmbr2`** and the **`20`** vlan tag.

![Win10-8](/images/homelab-guide/part6/WinSetup-8.png)

Make sure everything looks good on the screen and hit **`Confirm`**.

![Win10-9](/images/homelab-guide/part6/WinSetup-9.png)

Head over to **`Hardware -> Boot Order`**. Ensure that the Windows ISO is on top so that we boot directly into Windows Environment.

![Win10-10](/images/homelab-guide/part6/WinSetup-10.png)

This Windows setup should look familiar! Go ahead and click **`Next`**.

![Win10-11](/images/homelab-guide/part6/WinSetup-11.png)

Click on <u>Custom: Install Windows Only (advanced)</u>.

![Win10-12](/images/homelab-guide/part6/WinSetup-12.png)

Pretty much the same process as the server. Click on **`Load Driver`**.

![Win10-13](/images/homelab-guide/part6/WinSetup-13.png)

Click **`Browse`**.

![Win10-14](/images/homelab-guide/part6/WinSetup-14.png)

Select the **`w10`** folder and click **`Ok`**.

![Win10-15](/images/homelab-guide/part6/WinSetup-15.png)

Hit **`Next`** once it's finished loading.

![Win10-16](/images/homelab-guide/part6/WinSetup-16.png)

Let it do its thing. Once finished it will restart.

![Win10-17](/images/homelab-guide/part6/WinSetup-17.png)

Click **`Domain Join Instead`**.

![Win10-18](/images/homelab-guide/part6/WinSetup-18.png)

This windows 10 VM will belong to John Snow. So I will give it John.

![Win10-19](/images/homelab-guide/part6/WinSetup-19.png)

Give John a memorable password.

![Win10-20](/images/homelab-guide/part6/WinSetup-20.png)

Annoying Security questions lol.

![Win10-21](/images/homelab-guide/part6/WinSetup-21.png)

I don't like being tracked so I disable all the settings.

![Win10-22](/images/homelab-guide/part6/WinSetup-22.png)

Unless you love cortana. Click **`Not Now`**.

![Win10-23](/images/homelab-guide/part6/WinSetup-23.png)

Once you have booted into the desktop. Navigate over to **`This PC -> Properties`**.

![Win10-24](/images/homelab-guide/part6/WinSetup-24.png)

Click **`Advanced System Settings`**.

![Win10-25](/images/homelab-guide/part6/WinSetup-25.png)

Click **`Change`** under **`Computer Name`**.

![Win10-26](/images/homelab-guide/part6/WinSetup-26.png)

I will name the computer **`Win10-VM1`**. Let's enter our domain **`ad.lab`**. Then Click **`OK`**.

![Win10-27](/images/homelab-guide/part6/WinSetup-27.png)

Click **`OK`**.

![Win10-28](/images/homelab-guide/part6/WinSetup-28.png)

Go ahead and click **`OK`** to restart your PC.

![Win10-29](/images/homelab-guide/part6/WinSetup-29.png)
 
Login with the credentials of our AD User. In this case it is John so if you remember earlier his username is **`jsnow`** and the password is whatever we set.

![cmd](/images/homelab-guide/part6/cmdwindow.png)

We can also verify that the user we have signed in as is part of the domain by doing **`whoami`** in a cmd window. We can also verify the IP we have by doing **`ipconfig`** then checking it to the Leases in the DC.

![dhcpleases](/images/homelab-guide/part6/dhcpleases.png)

And BOOM! They match! 

![reservations](/images/homelab-guide/part6/reservations.png)
If you would like, we can set our Windows Machine to have a static IP by Right click and selecting **`Add to Reservation`**

And... That's all folks! You have successfully set up and added a Windows 10 system onto a domain! Repeat these exact same steps to setup another system. The only thing you need to change is the name when creating the user.

## Conclusion
Well Done! You have now reached the true end of the AD Lab Module! We have set up a Domain Controller on Windows Server 2022 and set up a Windows 10 Machine configured under the same domain. On the DC we set up DHCP, AD Certficate Services, and configured the domain.


### Additional AD Setups
There are so many more things you can do with the Active Directory Lab. Below are links to videos  that go more in-depth into Active Directory and it's features!

- [How to setup a Basic Home Lab Running Active Directory](https://www.youtube.com/watch?v=MHsI8hJmggI)
- [How to build an Active Directory Hacking Lab](https://www.youtube.com/watch?v=xftEuVQ7kY0)

### Attacking your AD Lab

Below are some links to resources on the types of attacks/hacking you can perform againist your AD environment. Remember to take snapshots so you can revert back to a previous state!

- [Hack your AD Lab](https://benheater.com/hack-your-virtualbox-ad-lab/)
- [AD Methodology by HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology)

In our next module we will cover setting up our Security Lab that involves Security Onion and Splunk! Stay tuned!

