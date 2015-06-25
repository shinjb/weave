# Weave - weaving containers into applications

[![Build Status](https://travis-ci.org/weaveworks/weave.svg?branch=master)](https://travis-ci.org/weaveworks/weave) [![Integration Tests](https://circleci.com/gh/weaveworks/weave/tree/master.svg?style=shield&circle-token=4933c7dabb3d0383e62117565cb9d16df7b1a811)](https://circleci.com/gh/weaveworks/weave) [![Coverage Status](https://coveralls.io/repos/weaveworks/weave/badge.svg)](https://coveralls.io/r/weaveworks/weave)

Weave creates a virtual network that connects Docker containers
deployed across multiple hosts and enables their automatic discovery.

![Weave Virtual Network](/docs/virtual-network.png?raw=true "Weave Virtual Network")

Applications use the network just as if the containers were all
plugged into the same network switch, with no need to configure port
mappings, links, etc. Services provided by application containers on
the weave network can be made accessible to the outside world,
regardless of where those containers are running. Similarly, existing
internal systems can be exposed to application containers irrespective
of their location.

![Weave Deployment](/docs/deployment.png?raw=true "Weave Deployment")

Weave can traverse firewalls and operate in partially connected
networks. Traffic can be encrypted, allowing hosts to be connected
across an untrusted network.

With weave you can easily construct applications consisting of
multiple containers, running anywhere.

Weave works alongside Docker's existing (single host) networking
capabilities, so these can continue to be used by containers.

## Installation

Ensure you are running Linux (kernel 3.8 or later) and have Docker
(version 1.3.1 or later) installed. Then install weave with

    sudo curl -L git.io/weave -o /usr/local/bin/weave
    sudo chmod a+x /usr/local/bin/weave

CoreOS users see [here](https://github.com/fintanr/weave-gs/blob/master/coreos-simple/user-data) for an example of installing weave using cloud-config.

Weave respects the environment variable `DOCKER_HOST`, so you can run
it locally to control a weave network on a remote host.

## Quick Start Screencast

<a href="http://youtu.be/k6r7yuSr0hE" alt="Click to watch the screencast" target="_blank">
  <img src="/docs/hello-screencast.png" />
</a>

## Example

Say you have docker running on two hosts, accessible to each other as
`$HOST1` and `$HOST2`, and want to deploy an application consisting of
two containers, one on each host.

First start weave on $HOST1:

    host1$ weave launch && weave launch-dns && weave launch-proxy

this runs the weave router, DNS and proxy, each in their own
container. Next we configure our `DOCKER_HOST` environment variable to
point to the proxy, so that containers launched via the docker command
line are automatically attached to the weave network:

    host1$ eval $(weave proxy-env)

Finally we run our application container; this happens via the proxy
so it is automatically allocated an IP address and registered in DNS:

    host1$ docker run --name a1 -ti ubuntu

That's it! If our application consists of more than one container on
this host we simply launch them with `docker run` as appropriate.

Next we repeat similar steps on `$HOST2`...

    host2$ weave launch $HOST1 && weave launch-dns && weave launch-proxy
    host2$ eval $(weave proxy-env)
    host2$ docker run --name a2 -ti ubuntu

The only difference, apart from the name of the application container,
is that we tell our weave that it should peer with the weave on
`$HOST1` (specified as the IP address or hostname, and optional
`:port`, by which `$HOST2` can reach it). NB: if there is a firewall
between `$HOST1` and `$HOST2`, you must open the weave port (6783 by
default; this can be overriden by setting `WEAVE_PORT`) for TCP and
UDP.

Note that we could instead have told the weave on `$HOST1` to connect to
`$HOST2`, or told both about each other. Order does not matter here;
weave automatically (re)connects to peers when they become
available. Also, we can tell weave to connect to multiple peers by
supplying multiple addresses, separated by spaces. And we can
[add peers dynamically](http://docs.weave.works/weave/latest_release/features.html#dynamic-topologies).

The router, DNS and proxy need to be started once per host. The
relevant container images are pulled down on demand, but if you wish
you can preload them by running `weave setup` - this is particularly
useful for automated deployments, and ensures that there are no delays
during later operations.

Now that we've got everything set up, let's see whether our containers
can talk to each other...

In the container started on `$HOST1`...

    root@a1:/# ping -c 1 -q a2
    PING a2.weave.local (10.160.0.2) 56(84) bytes of data.
    --- a2.weave.local ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.341/0.341/0.341/0.000 ms

Similarly, in the container started on `$HOST2`...

    root@a2:/# ping -c 1 -q a1
    PING a1.weave.local (10.128.0.2) 56(84) bytes of data.
    --- a1.weave.local ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.366/0.366/0.366/0.000 ms

So there we have it, two containers on separate hosts happily talking
to each other.

## Find out more

 * [Documentation homepage](http://docs.weave.works/weave/latest_release/)
 * [Getting Started Guides](http://weave.works/guides/)
 * [Features](http://docs.weave.works/weave/latest_release/features.html)
 * [Troubleshooting](http://docs.weave.works/weave/latest_release/troubleshooting.html)
 * [Building](http://docs.weave.works/weave/latest_release/building.html)
 * [How it works](http://docs.weave.works/weave/latest_release/how-it-works.html)

## Contact Us

Found a bug, want to suggest a feature, or have a question?
[File an issue](https://github.com/weaveworks/weave/issues), or email
help@weave.works. When reporting a bug, please include which version of
weave you are running, as shown by `weave version`.

Follow weave on Twitter:
[@weaveworks](https://twitter.com/weaveworks).

Read the Weave blog:
[Weaveblog](http://weaveblog.com/).

IRC:
[#weavenetwork](https://botbot.me/freenode/weavenetwork/)