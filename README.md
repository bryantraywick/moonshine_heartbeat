# Moonshine Heartbeat

Heartbeat is a tool for providing a high availibility resource (http, mysql,
etc) from a set of two or more servers. You want heartbeat if:

* You need high availability
* Moving requests from one server to a hot spare at any point is acceptable in
  your application

We use this at Rails Machine for providing a hot spare for single points of
failure, like software load balancers.

### A plugin for [Moonshine](http://github.com/railsmachine/moonshine)

This Moonshine plugin allows you to easily integrate heartbeat into your
deployment.

### Instructions

* <tt>script/plugin install git://github.com/railsmachine/moonshine_heartbeat.git</tt>
* Configure settings in <tt>config/moonshine.yml</tt> (see Configuration below)
* Invoke the recipe(s) in the Moonshine manifest for your high availaibility resource
    recipe :heartbeat

### Basic Configuration

A basic heartbeat setup with two haproxy servers would look something like
this in <tt>moonshine.yml</tt>

<pre><code>
  :heartbeat:
    :resources:
      :haproxy1: 12.34.56.80
    :nodes:
      :haproxy1: 12.34.56.78
      :haproxy2: 12.34.56.79
    :password: sekret
</code></pre>

The <tt>resources</tt> key is a hash of preferred nodes and the resources that
these nodes provide. In this case, we have two servers, haproxy1 and haproxy2.
haproxy1 is the preferred (or primary) node. In this case, the resource that
the primary node should present is an IP address - 12.34.56.80.

#### <tt>resources</tt> Requirements

* The name of the preferred node must match the output of <tt>uname -n</tt> on
  that server.
* All nodes below must have their firewall configured to allow incoming
  traffic on the ip address specified for the preferred node.

The <tt>nodes</tt> key is a hash of two or more nodes that will selected from
to provide the high availibility resources specified in the <tt>resources</tt>
hash. Their ip address is required here to setup heartbeat pings between them.

#### <tt>nodes</tt> Requirements

* The name of the each node must match the output of <tt>uname -n</tt> on
  that server.

The <tt>password</tt> is a simple key used for each heartbeat server to
authenticate with each other.

You'd then point your DNS to the high availibiltiy resource that heartbeat
monitors - 12.34.56.80. If haproxy1 goes down, haproxy2 will take over this IP
and continue serving requests. When haproxy1 comes back up, it'll take back
the high availibility IP.

### Advanced Configuration

Some examples of more custom configuration follow:

<pre><code>
  :heartbeat:
    :resources:
      :haproxy1: -
        12.34.56.80
        haproxy
        god
    :nodes:
      :haproxy1: 12.34.56.78
      :haproxy2: 12.34.56.79
    :password: sekret
    :deadtime: 10
    :warntime: 5
    :initdead: 20
    :interface: eth1
    :udpport: 6009
    :auto_failback: no
    :ping: 12.34.56.1
</code></pre>

Please see the "heartbeat wiki"[http://wiki.linux-ha.org/ha.cf] for more
more information on these advanced config options.

### Bcast/Ucast/Mcast

This plugin defaults to UDP ucast heartbeats because we've found them to be
the most reliable. There is experimental (untested) support for mcast - see
the [ha.cf](https://github.com/railsmachine/moonshine_heartbeat/blob/master/lib/templates/ha.cf)
template on Github for details

***

Unless otherwise specified, all content copyright &copy; 2014, [Rails Machine, LLC](http://railsmachine.com)
