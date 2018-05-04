---
layout: post
title: Windows 8 In Place Upgrade - Important Lessons
date: 2012-08-20 14:40:54.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Windows 8
tags:
- TrueCrypt
- Upgrade
- Windows 7
- Windows 8
meta:
  _edit_last: '1'
  _thumbnail_id: '552'
author:
  login: admin
  email: administrator@trycatch.me
  display_name: Eoin
  first_name: Eoin
  last_name: Campbell
---
<p><img class=" wp-image-552 " title="Windows 8 &amp; TrueCrypt Security" src="{{ site.baseurl }}/assets/295909-windows-8-security.jpg" alt="Windows 8 &amp; TrueCrypt Security" width="165" height="165" /> Windows 8 &amp; TrueCrypt Security</p>
<p style="text-align: justify;">Windows 8 RTM was released last week and since Rainmaker are MSDN Subscribers, I decided last Friday during a quiet few hours that I'd upgrade my work/development laptop to the new OS. It has not been going smoothly. Here are a few of the problems I've run into so far.</p>
<ul style="text-align: justify;">
<li>Make sure the ISO you're installing from matches the existing installation on your machine. In my case, the in-place edition (Windows 7 Enterprise) was incompatible with the ISO we'd downloaded (Windows 8 Enterprise N). That set me back several hours while pulling down a different ISO.</li>
<li>The Installation makes you take a "Leap of Faith". You need to un-install a number of "Incompatible" pieces of Software. On my Dell Latitude that involved uninstalling the Broadcomm BlueTooth Drivers, the Intel Wireless &amp; Wired Proset Drivers, Magic ISO Reader... pretty much everything that allowed the laptop to communicate with the outside world. Make sure you have the drivers handy on a USB key in case you need to abort the install and get your laptop back to normal. I also had to get rid of the Avast Anti Virus Install for what it's worth.</li>
<li>But the pièce-de-résistance was that when I finally got the installer running, I had completely forgotten that my Laptop Hard-drive is protected by TrueCrypt which requires a password when the machine boots up. Alas, instead of password entry this morning, I was presented with a nasty "Missing Operating System" message. TrueCrypt obviously needs to host OS intact to run, even before the OS loads up.</li>
</ul>
<p style="text-align: justify;">So right now I'm 5 hours into what will probably end up being an 8-hour full disk decrypt. Fingers are tentatively crossed that once the disk is decrypted the Windows 8 install will happily continue on where it left off. <strong>Optimistic?</strong></p>
<p style="text-align: justify;"><strong>**UPDATE**</strong></p>
<p style="text-align: justify;">That went surprisingly smoothly in the end. Once the TrueCrypt Disk Decryption finished and the machine rebooted, the setup processed continued exactly where it had left off and completed without incident. Shiny new Windows 8 Install.</p>
<p style="text-align: justify;"><em>~Eoin C</em></p>
