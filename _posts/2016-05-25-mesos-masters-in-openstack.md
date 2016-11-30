---
layout: post
title: Running Mesos masters in OpenStack
comments: true
tags: 
  - salt
  - mesos
---

When a Mesos master starts, by default, it tries to bind to the IP defined in `/etc/mesos-master/ip`.  Normally, the IP address in this file matches the IP on one of your interfaces and there is no issue.   However, if you're running a Mesos master in an OpenStack environment, and you expect to use a floating IP, this will cause the mesos-master process to fail to start with an error like:

>May 24 20:18:32 nv001 mesos-master[32280]: F0524 14:18:32.307010 32280 process.cpp:895] Failed to initialize: Failed to bind on 10.190.21.23:5050: Cannot assign requested address: Cannot assign requested address [99]
<!--more-->

The trick is to leave the `--ip` flag set to the local/private IP and use the `--advertise-ip` flag to define the IP that is used to communicate with other masters (the floating ip).

In my environment, with a Mesos master on a private IP of `10.10.10.30` and a floating IP of `10.190.21.23`, here's what the config looks like:

1. Local IP:

   ~~~ sh
root@nv001:~# ip a show eth0 | grep 'inet '
    inet 10.10.10.30/24 brd 10.10.10.255 scope global eth0
   ~~~

2. Floating IP:

   ~~~ sh
root@nv001:~# curl -s http://169.254.169.254/2009-04-04/meta-data/public-ipv4
10.190.21.23
   ~~~

3. `/etc/mesos-master/ip`:

   ~~~ sh
root@nv001:~# cat /etc/mesos-master/ip
10.10.10.30
   ~~~

4. `/etc/mesos-master/advertise_ip`:

   ~~~ sh
root@nv001:~# cat /etc/mesos-master/advertise_ip
10.190.21.23
   ~~~

Note that if your entire mesos master cluster is running in the same private network in OpenStack, you can just let them communicate via their private IPs (assuming your security policy allows it).
{: .notice}

---

Reference(s):

1. [http://mesos.apache.org/documentation/latest/configuration/](http://mesos.apache.org/documentation/latest/configuration/)
2. [http://stackoverflow.com/questions/33108406/not-able-to-access-mesos-ui-using-floating-public-ip-on-openstack](http://stackoverflow.com/questions/33108406/not-able-to-access-mesos-ui-using-floating-public-ip-on-openstack)
