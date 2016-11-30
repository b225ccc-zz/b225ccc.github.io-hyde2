---
layout: post
title: Oracle VM NFS
comments: true
tags: 
  - oracle_vm
  - ubuntu
  - nfs
---

Today I had to provide NFS shares to an Oracle VM 3.4 demo cloud environment.  I 
decided to setup an NFS server with Ubuntu Server 16.04.
<!--more-->

After the OS was installed, I followed most of the guide 
[here](https://help.ubuntu.com/14.04/serverguide/network-file-system.html) to 
get NFS installed.

Once the NFS components are installed, I did the following things:

1. Created the export directories:

   ~~~ bash
   $ mkdir -p /srv/nfs/share1
   $ mkdir -p /srv/nfs/share2
   $ chmod 777 /srv/nfs/share1
   $ chmod 777 /srv/nfs/share2
   ~~~

   (I'm using one of the shares for the repositories and one for the server pool.)

2. Added the exports.  Note that I'm only access from the storage network that I 
created for this demo.

   ~~~ bash
   cat <<EOF >> /etc/exports
   /srv/nfs/share1	172.16.47.0/24(rw,sync,no_subtree_check)
   /srv/nfs/share2	172.16.47.0/24(rw,sync,no_subtree_check)
   EOF
   ~~~

3. Start and enable the appropriate services.  I kept getting a "No locks available" error in 
Oracle VM Manager when setting up the NFS share repositories until I started 
the rpc-statd service.  (See [http://b225ccc.github.io/articles/2016-05/orace-vm-nfs-locks](http://b225ccc.github.io/articles/2016-05/orace-vm-nfs-locks).)

   ~~~ bash
   $ systemctl restart nfs-server.service
   $ systemctl start rpc-statd.service
   $ systemctl enable nfs-server.service
   $ systemctl enable rpc-statd.service
   ~~~


---

Reference(s):

1. [https://help.ubuntu.com/14.04/serverguide/network-file-system.html](https://help.ubuntu.com/14.04/serverguide/network-file-system.html)
