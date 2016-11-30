---
layout: post
title: RedHat Openstack Platform, TripleO, Heat and Single Interface no VLANs
comments: true
tags: 
  - openstack
  - redhat
  - tripleo
  - heat
  - rhop7
---

The default "Basic Scenario" network heat templates that come with RedHat OpenStack Platform 7 assume a single NIC with VLAN trunking.  After a lot of trial and error and Googling, using a single NIC with *no* VLANs wasn't very intuitive.  I finally stumbled across [this blog post](http://blog.nemebean.com/content/network-isolation-tripleo-home-networks) that gave me hope.  I adapted the templates referenced there for my environment and got this working.  See below for the details.

<!--more-->

I forked cybertron's tripleo-network-templates repo (just to make sure I don't lose the source files) to https://github.com/b225ccc/tripleo-network-templates.  In the README.md, see the instructions for the "Simple" templates and instructions.

In my setup, interface `p1p1` is External and `em1` is for provisioning and all other functions (API, storage, etc.)

My external subnet is `10.190.17.64/26` and provisioning is `10.190.17.0/26`.

Here's the steps I performed with detail.  All these are executed from the RedHat Director.

1. Get the templates and move the files into place

   ~~~ sh
cd ~
git clone https://github.com/b225ccc/tripleo-network-templates.git
cd tripleo-network-templates
cp -r /usr/share/openstack-tripleo-heat-templates ~/simple-templates
cp ~/tripleo-network-templates/simple/network-isolation.yaml ~/simple-templates/environments/network-isolation.yaml
   ~~~

2. Edit the following files to conform to your environment.  See the end of the post for the gists.

   1. `~/tripleo-network-templates/simple/network-environment.yaml`
   2. `~/tripleo-network-templates/simple/nic-configs/compute.yaml`
   3. `~/tripleo-network-templates/simple/nic-configs/controller.yaml`

   

3. Command to deploy the overcloud

   ~~~ sh
openstack overcloud deploy \
 --templates ~/simple-templates \
 -e ~/tripleo-network-templates/simple/network-environment.yaml \
 -e ~/simple-templates/environments/network-isolation.yaml \
 --control-flavor control --compute-flavor compute \
 --ntp-server pool.ntp.org \
 --neutron-network-type vxlan \
 --neutron-tunnel-types vxlan
   ~~~

---

Gists

{% gist b225ccc/3f6cf74e155b8f8a587cff4a60a932c7 %}

{% gist b225ccc/b57088f24449fa03169c02ce61a7e109 %}

{% gist b225ccc/742687a75610a431778dd70f3d3ed713 %}

---

Reference(s):

1. [http://blog.nemebean.com/content/network-isolation-tripleo-home-networks](http://blog.nemebean.com/content/network-isolation-tripleo-home-networks)
2. [https://github.com/cybertron/tripleo-network-templates](https://github.com/cybertron/tripleo-network-templates)
3. [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/chap-Installing_the_Overcloud.html#sect-Creating_the_Basic_Overcloud](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/chap-Installing_the_Overcloud.html#sect-Creating_the_Basic_Overcloud)
