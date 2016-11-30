---
layout: post
title: Oracle VM 'No locks available' error
comments: true
tags: 
  - oracle_vm
  - ubuntu
  - nfs
---

If you're seeing errors in Oracle VM Manager about "No locks available" and 
you are using an NFS file system, you might need to make sure that the 
`statd` service is running on your NFS server.
<!--more-->

I was trying to set up repositories on an NFS share
Trying to set up repositories on an NFS share kept failing with an error 
(in `/var/log/ovs-agent.log`) "No locks available".  After poking around 
for awhile, I found that this was because the `rpc-statd` service was not
running on the NFS server.

Once I started the service, the repository creation completed without any 
issues.

~~~ bash
$ systemctl start rpc-statd.service
$ systemctl enable rpc-statd.service
~~~

Here is the error and stack trace:

~~~
[2016-05-18 11:53:16 29267] ERROR (service:97) catch_error: Lock file /OVS/Repositories/0004fb00000300007faf8c5bdb4064bb/.ovsrepo failed: [Errno 37] No locks available
Traceback (most recent call last):
  File "/usr/lib64/python2.6/site-packages/agent/lib/service.py", line 95, in wrapper
    return func(*args)
  File "/usr/lib64/python2.6/site-packages/agent/api/repository.py", line 294, in create_repository
    set_repository_info(repo_mount_point, info)
  File "/usr/lib64/python2.6/site-packages/agent/api/repository.py", line 95, in set_repository_info
    lock.acquire(wait=360)
  File "/usr/lib64/python2.6/site-packages/agent/lib/filelock.py", line 54, in acquire
    (self.filename, e))
LockError: Lock file /OVS/Repositories/0004fb00000300007faf8c5bdb4064bb/.ovsrepo failed: [Errno 37] No locks available
~~~
