---
layout: post
title: "Intro to CloudFoundry and Bosh"
date: 2014-09-07 12:00
comments: true
categories: tech
---

I've started looking into [CloudFoundry](http://cloudfoundry.org) and
[Bosh](http://bosh.cloudfoundry.org) for work, and something that I've noticed is
a lack of "mid-range" documentation. It's quite
possible I'm blind, but I've seen a lot of 30,000' things ("CloudFoundry
will accelerate your velocity!") and some great docs for the people who already
know what they're talking about ("The Director uses the CPI to tell the IaaS to
launch a VM") but I haven't seen any introductions written with an eye towards someone who
is not a PaaS expert, but is also not a manager. This is my attempt to fill
that gap.

To start with, CloudFoundry is an open-source version of [Heroku](http://heroku.com) (in intent, and
it's even compatible in [some ways](https://devcenter.heroku.com/articles/third-party-buildpacks)). It's an implementation of a "Platform as a
Service" ([PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service)) product,
that you can run either [in-house](http://openstack.org) or on [various](http://aws.amazon.com)
[cloud](http://rackspace.com) [platforms](http://www.vmware.com/products/vcloud-hybrid-service/) that provide you
with "normal" virtual machines. It's been explicitely designed to be agnostic about
what sort of machine it's running on (as long as that machine is linux, for now). Unlike
traditional machines where, if you want to deploy a website you have to do ALL THE
THINGS including configuring the webserver, the IP address, the database (if you
use one), take care of logging, etc., in a PaaS they try and abstract as much of that
away as possible. All you have to do is push your code (along
with a [config file](http://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)
that determines things like how much RAM, CPU, and disk space
you need and what services you will be talking to) and you get your own little
slice of a machine that is isolated into it's own container. Using a PaaS is the
natural end-game of someone that is trying to follow the [12 Factor App](http://12factor.net)
manifesto. A good case can be made that using a PaaS should be the default for any
newly written code, unless you have very good reasons why it won't work. Of course,
existing shops with their legacy code and processes are a different story - migrating
even a medium-sized project from the "traditional" approach over to using a PaaS
can be quite a bit of work.

To divert for a second, a container is kinda like a virtual machine, in that it gets its own IP address and disk space, but
it's not a "full fledged" machine - it shares a kernel with the other containers
and the host machine. This is done using a software component of CloudFoundry called
[Warden](http://docs.cloudfoundry.org/concepts/architecture/warden.html), which
works very similarly to [LXC](https://linuxcontainers.org) or
[Solaris Zones](http://en.wikipedia.org/wiki/Solaris_Containers) (in fact, early
versions of Warden used LXC as the base). If you want to scale up the performance
of your app, the way of doing that is to add additional instances. CloudFoundry
will take care of making sure that requests to your app get routed to both
instances. It will also restart the container if it detects things like running
out of RAM or that the application is crashing.

Bosh is sort of like Puppet or Chef, but working at the infrastructure layer vs.
working with individual resources on each machine. By that I mean you describe how
you want your infrastructure to look and then bosh will make it look that way. With
puppet you describe how you want a given machine to look, and it's up to you to make
the machines all work together. It uses virtual-machine templates (called stemcells), code,
configuration and scripts (called 'releases'), and a manifest file that describes how the infrastructure,
code, network architecture, and everything else combines together. All three of those
components wrapped up is a "deployment", and can be pushed using bosh. Bosh will
provision machines, upload your code, install packages, and then monitor them to
make sure everything is healthy, redeploying machines if something breaks. If you
want to do something like add more virtual machines, you modify the manifest and
redeploy. You can also update the stemcell (maybe switch from Ubuntu to CentOS)
without changing anything else and redploy. And of course you can change the actual
code you're pushing and redeploy. Each time, if it sees something that is different
on the running machines versus what's in the manifest, bosh will rectify the situation,
whether it needs to shut down a machine and redeploy it entirely (newly-updated
stemcell), just deploy new packages, or whatever.

Bosh is used to deploy CloudFoundry onto your infrastructure and if it sounds like
it kind of works like CloudFoundry, that's because it does. However, bosh is aimed
at building out infrastructure and is a bit
[unwieldy](http://docs.cloudfoundry.org/bosh/create-release.html) for J. Random
Developer to use just to deploy their webapp. Deciding how to setup networking, managing
packages, those are all the sorts of things that a PaaS abstracts away from you.
However, bosh does a bang-up job of building out large, complicated software like
CloudFoundry.

The next article will go over installing bosh, then using bosh to install
CloudFoundry, and then using CloudFoundry to host an app.
