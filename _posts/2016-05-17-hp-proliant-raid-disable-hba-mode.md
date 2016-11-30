---
layout: post
title: Enable/Disable hbamode on HP ProLiant DL360 with P420i RAID controller
comments: true
tags: 
  - hp
  - raid
---

Today I had to disable "hbamode" on a P420i RAID controller.  
Setting `hbamode=on` in the first place isn't a supported thing, so 
documentation on this is spotty.  I had a machine that had hbamode=on and I 
wanted to turn it back off.  
<!--more-->

You'll know if you're server is in hbamode if you see the following text 
during boot:

> Hardware RAID support is disabled via NVRAM Configuration Setting

Here's how to turn off hbamode:

1. Boot to Service Pack disk (at the time of writing, see [here](http://h17007.www1.hp.com/us/en/enterprise/servers/products/service_pack/spp/index.aspx) or just search for "hp proliant service pack")
2. Select interactive mode
3. Select HP Smart Storage Admin
4. Press Ctrl + Alt + d + b + x (i.e. hold Ctrl+Alt while pressing d, b, x in sequence).  You'll see a terminal appear.
5. Execute the following commands:
     
   ~~~ bash
   cd /opt/hp/hpssacli/bld/
   ./hpssacli controller slot=0 modify hbamode=off
   ~~~

6. Exit and reboot the server


---

Reference(s):

1. [https://hardforum.com/threads/hp-dl380p-gen8-p420i-controller-hbamode.1852528/](https://hardforum.com/threads/hp-dl380p-gen8-p420i-controller-hbamode.1852528/)
2. [http://community.hpe.com/t5/ProLiant-Servers-ML-DL-SL/Hardware-RAID-is-disabled-via-NVRAM-Controller-setting/td-p/4702761](http://community.hpe.com/t5/ProLiant-Servers-ML-DL-SL/Hardware-RAID-is-disabled-via-NVRAM-Controller-setting/td-p/4702761)
