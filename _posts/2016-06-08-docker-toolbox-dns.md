---
layout: post
title: "Docker Toolbox DNS resolution (OSX El Capitan host, ubuntu:14.04 container)"
comments: true
tags: 
  - docker
  - ubuntu
---

There's lots of info online about Docker and related DNS issues.  These DNS issues can stem from a lot of different sources, but a workaround for many of these issues is to override the docker process "auto-discovered" DNS servers.  I couldn't easily find how to do that using [Docker Toolbox](https://www.docker.com/products/docker-toolbox) on OSX, so here's a quick howto.
<!--more-->

My specific errors had to do with resolving ubuntu's mirrors when doing an `apt-get update` during the `docker build` process.

~~~
Err http://archive.ubuntu.com trusty Release.gpg
  Could not resolve 'archive.ubuntu.com'
~~~

So, we need to override the auto-detected DNS servers for the docker process.  This is done in the `/var/lib/boot2docker/profile` file in the docker machine "default" VM.

The process:

1. From your shell:
   
   ~~~ sh
   ~ > docker-machine ssh default
   ~~~
   
2. Run the following `sed` command (substitute appropriate DNS server for your environment for `8.8.8.8`.  `8.8.8.8` is a public Google DNS server - so if you don't know what to use, this should work):

   ~~~ sh
   docker@default:~$ sudo sed -i '/EXTRA_ARGS/a--dns 8.8.8.8' /var/lib/boot2docker/profile
   ~~~
   
3. Verify the text was added:
   
   ~~~ sh
   docker@default:~$ grep -A1 EXTRA_ARGS /var/lib/boot2docker/profile
EXTRA_ARGS='
--dns 8.8.8.8
   ~~~

   Exit the VM with `exit`.

4. Restart the default VM:

   ~~~
   ~ > docker-machine stop
   ~ > docker-machine start
   ~~~

---

Reference(s):

1. [http://stackoverflow.com/questions/26424338/docker-daemon-config-file-on-boot2docker](http://stackoverflow.com/questions/26424338/docker-daemon-config-file-on-boot2docker)