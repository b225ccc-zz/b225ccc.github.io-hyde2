---
layout: post
title: HAProxy healthcheck failures
excerpt: Fix "Connection reset by peer" errors in /var/log/messages
comments: true
tags: 
  - openstack
  - haproxy
  - redhat
  - rhop7
---

If you see errors in `/var/log/messages` like this:

```
May 15 03:38:14 n833 ironic-api: 10.190.17.1 - - [15/May/2016 03:38:14] "GET / HTTP/1.0" 200 338
May 15 03:38:14 n833 ironic-api: Traceback (most recent call last):
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/wsgiref/handlers.py", line 86, in run
May 15 03:38:14 n833 ironic-api: self.finish_response()
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/wsgiref/handlers.py", line 128, in finish_response
May 15 03:38:14 n833 ironic-api: self.write(data)
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/wsgiref/handlers.py", line 212, in write
May 15 03:38:14 n833 ironic-api: self.send_headers()
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/wsgiref/handlers.py", line 270, in send_headers
May 15 03:38:14 n833 ironic-api: self.send_preamble()
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/wsgiref/handlers.py", line 197, in send_preamble
May 15 03:38:14 n833 ironic-api: self._write('Server: %s\r\n' % self.server_software)
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/socket.py", line 324, in write
May 15 03:38:14 n833 ironic-api: self.flush()
May 15 03:38:14 n833 ironic-api: File "/usr/lib64/python2.7/socket.py", line 303, in flush
May 15 03:38:14 n833 ironic-api: self._sock.sendall(view[write_offset:write_offset+buffer_size])
May 15 03:38:14 n833 ironic-api: File "/usr/lib/python2.7/site-packages/eventlet/greenio/base.py", line 377, in sendall
May 15 03:38:14 n833 ironic-api: tail = self.send(data, flags)
May 15 03:38:14 n833 ironic-api: File "/usr/lib/python2.7/site-packages/eventlet/greenio/base.py", line 359, in send
May 15 03:38:14 n833 ironic-api: total_sent += fd.send(data[total_sent:], flags)
May 15 03:38:14 n833 ironic-api: error: [Errno 104] Connection reset by peer
```

Comment out the healthcheck for ironic in `/etc/haproxy/haproxy.cfg`:

```
listen ironic
  bind 10.190.17.2:6385 transparent
  bind 10.190.17.3:6385 transparent
  #option httpchk GET /
  server 10.190.17.1 10.190.17.1:6385 check fall 5 inter 2000 rise 2
```

{% highlight js %}
listen ironic
  bind 10.190.17.2:6385 transparent
  bind 10.190.17.3:6385 transparent
  #option httpchk GET /
  server 10.190.17.1 10.190.17.1:6385 check fall 5 inter 2000 rise 2
{% endhighlight %}

And restart haproxy:

``` bash
$ sudo systemctl restart haproxy.service
```

---

Reference(s):

1. [https://bugzilla.redhat.com/show_bug.cgi?id=1246525](https://bugzilla.redhat.com/show_bug.cgi?id=1246525)
