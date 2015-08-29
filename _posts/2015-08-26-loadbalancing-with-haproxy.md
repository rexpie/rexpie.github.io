---
layout: post
title:  "Load Balancing with HAProxy"
tags: haproxy, load balancing, mosquitto
---

# Need to cluster

IBM has a [Redbook](http://www.redbooks.ibm.com/redbooks/pdfs/sg248054.pdf) for its own IOT service environment. The book also described the topologies of enterprise MQTT service architectures. So, to cluster some mosquitto brokers, we need some other tools.

Mosquitto is a single thread process. To scale out, we need tools to better monitor and manage them.

# Cue in load balancing

I followed the steps in this nice [tutorial](https://serversforhackers.com/load-balancing-with-haproxy), others you googled might be just as good, as HAProxy is very easy to install, configure and run.

I changed the configuration to match our need for mqtt connections.

{% highlight cfg %}
listen mqtt
  bind *:1883
  bind *:1884
  mode tcp
  maxconn 200000
  option tcplog
  option redispatch
  balance leastconn
  server serv1 192.168.0.36:1883 source 0.0.0.0:1024-65535 check
  server serv2 192.168.0.136:1883 source 0.0.0.0:1024-65535 check

listen stats *:1936
  stats enable
  stats uri /
{% endhighlight %}

## Configurations explained

## `option redispatch`
This is used to tell HAProxy to retry to another backend server when one of the servers goes down, rather than giving a 503 error back to the fronted.

## `balance leastconn`
This is the balancing algorithm, we expect to have many TCP connections with little traffic and low spikes so the we should use this method to pan out the number of connections evenly.

## `source 0.0.0.0:1024-65535`
See this nice [email exchange](http://comments.gmane.org/gmane.comp.web.haproxy/5594), also see haproxy's [configuration manual](https://cbonte.github.io/haproxy-dconv/configuration-1.5.html#4-source).

Basically, 

## Monitoring HAProxy

The HAProxy comes with a handy web interface for quick monitoring. The stats block of the configuration shows how to do this. I left out the `auth` config so I don't have to type in user name and password every now and then.

![image]({{ site.url }}/assets/haproxystat.png "HAProxy Stats Screenshot")

# Conclusion

HAProxy can be used to load balance a cluster of single-threaded mosquitto servers in a LAN environment. We can even setup multiple clusters globally.

Next we maybe can try out the provisioning service as to auto configure the MQTT clients.