---
layout: post
title: Salt floating IP custom grain
comments: true
tags: 
  - salt
  - openstack
---

Writing Salt states, I find quite often that I need to know if the minion I'm on has a floating IP and, if so, what it is.  So, I created custom grain for `floating_ip`.
<!--more-->

Here's the code:

~~~ python
#!/use/bin/env python

import requests

def floating_ip():

    url = 'http://169.254.169.254/2009-04-04/meta-data/public-ipv4'

    try:
        r = requests.get(url, timeout=1)
    except requests.exceptions.ConnectionError as e:
        return {'floating_ip': False}
    except:
        return {'floating_ip': False}

    if r.status_code is 200:
        float = r.text
    else:
        float = False

    return {'floating_ip': float}
~~~

Simply drop this script in your custom grains directory (`_grains`) and sync it to all the minions with `salt '*' saltuitl.sync_grains` (or `salt '*' saltuitl.sync_all`).

Here's an example of getting the floating IP on a minion:

~~~ shell
root@nv001:~# salt-call grains.get floating_ip
[INFO    ] Starting new HTTP connection (1): 169.254.169.254
local:
    10.190.21.23
~~~

On a host with no floating IP:

~~~ shell
root@mesos-slave-02:~# salt-call grains.get floating_ip
[INFO    ] Starting new HTTP connection (1): 169.254.169.254
local:
    False
~~~

You can use this custom grain like any other grain in your state files with:

~~~ jinja
{% raw %}{% set floating_ip = salt['grains.get']('floating_ip', '') %}{% endraw %}
~~~

---

Reference(s):

1. [https://docs.saltstack.com/en/latest/topics/targeting/grains.html#writing-grains](https://docs.saltstack.com/en/latest/topics/targeting/grains.html#writing-grains)
