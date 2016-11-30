---
layout: post
title: "Script for adding cert to Java keystore on OSX"
comments: true
tags: 
  - java
  - osx
---

I came across a (yet another) Java error on OSX El Capitan today where a site's cert that I was trying to access was not in the keystore.  I had already whitelisted the host, so I'm not sure why this additional step is needed.  Here's a script for automating adding the cert.
<!--more-->

I googled around to find the process and found a [post](http://www.sebsgarage.com/2012/09/adding-java-ssl-certificate-mac-osx/) describing the process and referencing an "InstallCert.java" program.  The original link  to the program in that post was dead, but someone imported it into github [here](https://github.com/escline/InstallCert).

This got me the program I needed, but there are still a few commands you need to run that were easily scriptable, so I wrote a quick bash script.

I forked the above repo to [here](https://github.com/b225ccc/InstallCert) and added a script called `add.sh`.  Note that this script is specifically for OSX and tested on 10.11.5.  However, if you are able to compile the "InstallCert.java" program, all you should need to do to the script is modify the `$KEYSTORE` variable to get this to work.

To use, do the following:

1. Clone the repo

   ~~~ sh
   git clone https://github.com/b225ccc/InstallCert.git
   cd InstallCert
   ~~~

2. Compile `InstallCert.java`

   ~~~ sh
   javac InstallCert.java
   ~~~

3. Run `add.sh`

   ~~~ sh
   ./add.sh host-182a-mgmt
   ~~~
   
   The last command in the script (importcert) requires sudo, so you'll be prompted for your admin password.
   

If all goes well, you should see a message:

> Certificate was added to keystore

Done.

---

Reference(s):

1. [https://github.com/b225ccc/InstallCert](https://github.com/b225ccc/InstallCert)
2. [http://solutions.unixsherpa.com/2011/02/18/add-certificate-to-java-keystore-on-os-x/](http://solutions.unixsherpa.com/2011/02/18/add-certificate-to-java-keystore-on-os-x/)
3. [http://stackoverflow.com/questions/11936685/how-to-obtain-the-location-of-cacerts-of-the-default-java-installation](http://stackoverflow.com/questions/11936685/how-to-obtain-the-location-of-cacerts-of-the-default-java-installation)
