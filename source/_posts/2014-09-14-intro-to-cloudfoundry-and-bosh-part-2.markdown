---
layout: post
title: "Intro to Cloud Foundry and Bosh, part 2"
date: 2014-09-14 18:44
comments: true
categories:
---

When last we were here, I was giving a [broad overview](/blog/2014/09/07/intro-to-cloudfoundry/) of Cloud Foundry and Bosh,
comparing them to [Heroku](http://heroku.com) and other PaaS's. Today, we're going to go over spinning
up a Cloud Foundry instance from scratch, all on your handy local laptop. These
instructions assume you are using OS X, though it should work for any platform
that [bosh-lite](https://github.com/cloudfoundry/bosh-lite) runs on. In fact, this
tutorial closely follows the bosh-lite documentation, but is more tailored to beginners.
Depending on the amount of bandwidth you have (downloading machine images can take
a while) and the speed of your CPU (this does involve some package compilation,
which can also take a while), this process might take up to an hour. I also assume
that you have a working install of [homebrew](http://brew.sh). It isn't absolutely
necessary, but it sure makes things easier.

First is the prerequisites. You need [Virtual Box](http://virtualbox.org) and [Vagrant](http://vagrantup.com)
installed. As of this writing, Virtual Box is at 4.3.16 and Vagrant is at 1.6.5.
You should be at those versions or better. You also need [spiff](https://github.com/cloudfoundry-incubator/spiff)

    brew tap xoebus/homebrew-cloudfoundry
    brew install spiff

and the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli)

    brew tap pivotal/tap
    brew install cloudfoundry-cli

and finally the [Bosh CLI](https://rubygems.org/gems/bosh_cli)

    gem install bosh_cli

Spiff is a tool that combines multiple Bosh manifests into one, which makes it
easier to re-use components in Bosh. The Cloud Foundry CLI is what you will be
using to push your final web app to your running Cloud Foundry instance. And the
Bosh CLI is the tool that allows you to interact with bosh, running commands and
connecting to virtual machines that are running inside of Bosh.

Remember - the order is: virtual machine in VirtualBox that runs Bosh, a tool that
helps provision multiple virtual machines and/or containers at a time. We'll be
using Bosh to provision a cluster of containers running *within* that first machine,
and that cluster is what will be running the Cloud Foundry software. Cloud Foundry is the software that makes
it easy to spin up even more containers, which are the small slices of VMs that is all you need to run web
applications. Refer back to this paragraph if you start to lose track of what the
various relationships are.

Next, create a directory and clone the following two git repos into it:

    https://github.com/cloudfoundry/bosh-lite
    https://github.com/cloudfoundry/cf-release

Unless otherwise directed, run the rest of the commands in the top of the bosh-lite
git repo.

Start the virtual machine that has been pre-configured to act as an "all-in-wonder"
bosh director.

    vagrant up --provider virtualbox

The first time you run that, vagrant will make you wait for a while as it fetches
the bosh-lite-ubuntu-trusty box from the internet. Feel free to peruse [twitter](http://twitter.com/gnuconsulting) in
the interim.

After bosh-lite is up and running in the vagrant VM, "login" to the bosh service
by first targeting it and then logging in with the username/password combo of "admin/admin".

    bosh target 192.168.50.4 lite
    bosh login

That makes it so that when you run the bosh cli commands, it knows which bosh instance
to talk to. In the future when you're a bosh wizard, controlling multiple bosh instances
with the flick of your wrists, you will run the 'bosh targets' command to list out
all of the different instances that you've logged into, and switch between them using
the alias that you've given them. In this case, we've aliased this bosh instance to 'lite',
so you'd switch to it with 'bosh target lite'. For now, just know that's
how the bosh command knows where to go.

Before we move on, use 'vagrant ssh' to log into the bosh-lite VM. Take a poke around,
that's the machine that's going to be hosting Cloud Foundry for you. You'll see it's
running a bunch of different things: nginx, postgresql, resque, and some unusual
things like 'warden-linux' and 'bosh-agent'. All of those put together are "bosh".

Next we're going to use bosh to provision Cloud Foundry on your VM. From within
the bosh-lite git repo (i.e., not within the Vagrant VM), run:

    ./bin/provision_cf

That script isn't very long - 75 lines at this writing - but it's going to do a lot.
This is when bosh shines. It's using the manifests from the cf-release git repo and
combining them (using spiff) into one big manifest (take a look at bosh-lite/manifests/cf-manifest.yml
to see it all put together) and then building out an infrastructure based on that.
You'll have plenty of time until bosh is done running, so take this time to go through
the cf-manifest.yml file. You'll see that's it's fairly logical (if long). It goes
through and defines various instances, along with their IP address, disk space allocated,
what template to use, etc. It also defines the networks and subnets, giving them a
name and a range of IPs.

While you're looking at that, bosh is downloading all of the "stemcells" (VM templates)
and building packages to use with those stemcells and produce functional machines. This part
takes the longest of all, and heats up my laptop the most (16 minutes on a brand-new
top-of-the-line Macbook Pro). I probably haven't posted
anything to [twitter](http://twitter.com/gnuconsuting) since you were downloading the vagrant instance, so instead go
remind yourself that even though this is large and complicated, [it's not rocket science](http://itsnotrocketscience.info).
Thank you to my friend Phil Seibel for that.

Now that Cloud Foundry has been provisioned, run

    bosh vms

to see a list of all of the VMs it is using. It should look something like

    +------------------------------------+---------+---------------+--------------+
    | Job/index                          | State   | Resource Pool | IPs          |
    +------------------------------------+---------+---------------+--------------+
    | unknown/unknown                    | running | small_errand  | 10.244.0.186 |
    | unknown/unknown                    | running | small_errand  | 10.244.0.178 |
    | unknown/unknown                    | running | small_errand  | 10.244.0.182 |
    | api_z1/0                           | running | large_z1      | 10.244.0.138 |
    | etcd_z1/0                          | running | medium_z1     | 10.244.0.42  |
    | ha_proxy_z1/0                      | running | router_z1     | 10.244.0.34  |
    | hm9000_z1/0                        | running | medium_z1     | 10.244.0.142 |
    | loggregator_trafficcontroller_z1/0 | running | small_z1      | 10.244.0.10  |
    | loggregator_z1/0                   | running | medium_z1     | 10.244.0.14  |
    | login_z1/0                         | running | medium_z1     | 10.244.0.134 |
    | nats_z1/0                          | running | medium_z1     | 10.244.0.6   |
    | postgres_z1/0                      | running | medium_z1     | 10.244.0.30  |
    | router_z1/0                        | running | router_z1     | 10.244.0.22  |
    | runner_z1/0                        | running | runner_z1     | 10.244.0.26  |
    | uaa_z1/0                           | running | medium_z1     | 10.244.0.130 |
    +------------------------------------+---------+---------------+--------------+

If you 'vagrant ssh' back into the bosh-lite machine and look at the running processes,
you'll see each of those processes are children of various 'wshd' processes. 'wshd' is
the [warden](http://docs.cloudfoundry.org/concepts/architecture/warden.html) daemon that I mentioned in my previous post, that forms the basis of Cloud
Foundry and is akin to lxc or docker. For example, the 'etcd_z1/0' container, when
viewed from the bosh-lite machine:

    root      8310  0.0  0.0   1120   240 ?        S<s  19:14   0:00 wshd: 4v0cs3g0arf
    root      9062  0.0  0.0    188    32 ?        S<s  19:14   0:00  \_ runsvdir -P /etc/service log: rsyslogd: warning: ~ action is deprecated, consider using the 'stop' statement instead [try http://www.rsyslog.com/e/2307 ] rsyslogd: action '*' treated as ':omusrmsg:*' - please change syntax, '*' will not be supported in the future [try http://www.rsyslog.com/e/2184 ] rsyslogd: Could no open output pipe '/dev/xconsole': No such file or directory [try http://www.rsyslog.com/e/2039 ] ........
    root      9070  0.0  0.0    168     0 ?        S<s  19:14   0:00  |   \_ runsv ssh
    root      9077  0.0  0.0  61364  2256 ?        S<   19:14   0:00  |   |   \_ /usr/sbin/sshd -D
    root      9071  0.0  0.0    168     0 ?        S<s  19:14   0:00  |   \_ runsv agent
    root      9087  0.0  0.0    184     4 ?        S<   19:14   0:00  |   |   \_ svlogd -tt /var/vcap/bosh/log
    root      9088  0.0  0.1 349308  7412 ?        S<l  19:14   0:02  |   |   \_ /var/vcap/bosh/bin/bosh-agent -I warden -P ubuntu -C /var/vcap/bosh/agent.json
    root      9072  0.0  0.0    168     0 ?        S<s  19:14   0:00  |   \_ runsv monit
    root      9084  0.0  0.0    184     0 ?        S<   19:14   0:00  |   |   \_ svlogd -tt /var/vcap/monit/svlog
    root      9351  0.0  0.0  91480  1548 ?        S<l  19:14   0:03  |   |   \_ /var/vcap/bosh/bin/monit -I -c /var/vcap/bosh/etc/monitrc
    root      9073  0.0  0.0    168     0 ?        S<s  19:14   0:00  |   \_ runsv rsyslog
    syslog   10815  0.0  0.0 205124  2456 ?        S<l  19:14   0:00  |       \_ rsyslogd -n -c5
    vcap     10783  0.9  0.4 292512 26404 ?        S<l  19:14   1:11  \_ /var/vcap/packages/etcd/etcd -snapshot -data-dir=/var/vcap/store/etcd -addr=10.244.0.42:4001 -peer-addr=10.244.0.42:7001 -name=etcd_z1-0 -peer-heartbeat-timeout=50 -peer-election-timeout=1000
    root     10785  0.0  0.0  17976   664 ?        S<   19:14   0:00  |   \_ /bin/bash /var/vcap/jobs/etcd/bin/etcd_ctl start
    root     10787  0.0  0.0  17980   652 ?        S<   19:14   0:00  |   |   \_ /bin/bash /var/vcap/jobs/etcd/bin/etcd_ctl start
    root     10804  0.0  0.0   4340   636 ?        S<   19:14   0:00  |   |   |   \_ logger -p user.info -t vcap.etcd_ctl.stdout
    root     10788  0.0  0.0   4348   584 ?        S<   19:14   0:00  |   |   \_ tee -a /dev/fd/63
    root     10786  0.0  0.0  17976   660 ?        S<   19:14   0:00  |   \_ /bin/bash /var/vcap/jobs/etcd/bin/etcd_ctl start
    root     10789  0.0  0.0  17984   656 ?        S<   19:14   0:00  |       \_ /bin/bash /var/vcap/jobs/etcd/bin/etcd_ctl start
    root     10802  0.0  0.0   4340   624 ?        S<   19:14   0:00  |       |   \_ logger -p user.error -t vcap.etcd_ctl.stderr
    root     10791  0.0  0.0   4348   584 ?        S<   19:14   0:00  |       \_ tee -a /dev/fd/63
    vcap     10826  0.0  0.0 106472  4592 ?        S<l  19:14   0:00  \_ /var/vcap/packages/etcd_metrics_server/bin/etcd-metrics-server -index=0 -etcdAddress=127.0.0.1:4001 -natsAddresses=10.244.0.6:4222 -natsUsername=nats -natsPassword=nats -port=5678 -username= -password=
    root     10828  0.0  0.0  17980   672 ?        S<   19:14   0:00      \_ /bin/bash /var/vcap/jobs/etcd_metrics_server/bin/etcd_metrics_server_ctl start
    root     10830  0.0  0.0  17980   652 ?        S<   19:14   0:00      |   \_ /bin/bash /var/vcap/jobs/etcd_metrics_server/bin/etcd_metrics_server_ctl start
    root     10844  0.0  0.0   4340   636 ?        S<   19:14   0:00      |   |   \_ logger -p user.info -t vcap.etcd_metrics_server_ctl.stdout
    root     10832  0.0  0.0   4348   580 ?        S<   19:14   0:00      |   \_ tee -a /dev/fd/63
    root     10829  0.0  0.0  17976   664 ?        S<   19:14   0:00      \_ /bin/bash /var/vcap/jobs/etcd_metrics_server/bin/etcd_metrics_server_ctl start
    root     10831  0.0  0.0  17984   660 ?        S<   19:14   0:00          \_ /bin/bash /var/vcap/jobs/etcd_metrics_server/bin/etcd_metrics_server_ctl start
    root     10847  0.0  0.0   4340   636 ?        S<   19:14   0:00          |   \_ logger -p user.error -t vcap.etcd_metrics_server_ctl.stderr
    root     10834  0.0  0.0   4348   580 ?        S<   19:14   0:00          \_ tee -a /dev/fd/63

However, each of those *is* separated out into their own containers. To ssh into a container,
use 'bosh ssh':

    bosh ssh etcd_z1/0

That will prompt for a password, use 'admin'. Once you're logged in, it should
actually look exactly like the *ps* output I pasted above, except that there's nothing
else running (and the PIDs will be different - the wshd process will be PID 1, for instance).

Believe it or not, we're just about ready to start pushing websites to your Cloud Foundry cluster (as long as they don't require a database backend - that's later).
We're going to create an ["org" and "space"](http://www.activestate.com/blog/2014/01/cloud-foundry-orgs-and-spaces-stackato-30) within CF for us to use.

    cf api --skip-ssl-validation https://api.10.244.0.34.xip.io
    cf auth admin admin
    cf create-org me
    cf target -o me
    cf create-space development
    cf target -s development

For the next bit, I'm going to cheat and send you elsewhere. I helped write
some documentation on pushing your first ruby project to a CF instance, which
you can find [here](https://trycf.starkandwayne.com/docs/#!index.md#Build_an_example_project). Scroll
down to "Build a Ruby project" and follow the instructions exactly as given.

You're back? Good. You should now have a small ruby application running in your
test Cloud Foundry instance, happily telling the world "Hello!". Interestingly, if
you go ahead and run 'bosh vms' again to see your new, shiny ruby app, you'll notice
that there are no new containers provisioned. That's because this new app is not
being handled by bosh, and it isn't running the bosh-agent. In fact, it's container
running within a container - the 'runner_z1/0' container. If you 'bosh ssh
runner_z1/0' and run 'ps axfuw', you'll notice that there is another wshd process
with a child of 'rackup' underneath it (actually, bash then rackup, but who's counting).

My next blog post will deal with setting up a database for your applications to use
as a service. Until then, have fun doing things like 'cf scale &lt;appname&gt; 3' to
see how easy it is to add more "droplets" (execution units, containers, whatever you
want to call them). Also spend some time playing around with the cf and bosh command-lines.
How do you view logs from individual containers? What happens if you kill the wshd process
of your sinatra example app from "the outside" (i.e., in the bosh-lite VM)? Kick
the tires, it's surprisingly difficult to break. If you do get in trouble, try 'bosh cloudcheck'.
And if worst comes to worst, you get to delete the Vagrant VM and start again, hardly
the end of the world. And you'll probably learn a little more each time you rebuild.

Happy Cloud Foundrying!

Places I blatantly stole commands from and whatnot:

https://trycf.starkandwayne.com/docs/#!index.md

https://github.com/cloudfoundry/bosh-lite

https://github.com/cloudfoundry/bosh-lite/blob/master/docs/deploy-cf.md
