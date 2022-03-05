---
title: Setting Up An OpenTTD Server
date: 2015-11-08
description: I recently setup an OpenTTD server and I thought I might document the steps I took to do so.
draft: false
images: ["images/openttd-multi-screen.jpg"]
type: post
---

_Update: Check out [part two]({{< ref "2021-01-29-setting-up-an-openttd-server-part-2" >}}) for a more up to date guide._

I recently setup an [OpenTTD](https://www.openttd.org/en/) server and I thought I might document the steps I took to do so.

I choose to use a [Digital Ocean](https://www.digitalocean.com/) (DO) to host my server since I've been wanting to try them out for some time. It also helps that their cheapest hosting option is rather cheap - only $5 per month. And since I don't think OpenTTD will be particularily memory intensive I am fairly certain that the extra 512MB of RAM for the $10 per month DO droplet will be needed.

After creating my $5 per month droplet running *Ubuntu 14.04 LTS* I followed DO's handy [initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04) to get my server ready for installing OpenTTD.

## Installing OpenTTD

The first step to installing OpenTTD is downloading it:

{{< highlight bash >}}

wget https://binaries.openttd.org/releases/1.5.2/openttd-1.5.2-linux-ubuntu-trusty-amd64.deb

{{< / highlight >}}

Then to install OpenTTD and it's dependencies:

{{< highlight bash >}}

sudo dpkg -i openttd-1.5.2-linux-ubuntu-trusty-amd64.deb
sudo apt-get -f install

{{< / highlight >}}

## Installing Graphics

Before you can kick off your OpenTTD server you need to install some graphics. If you don't have your original Transport Tycoon Deluxe CDs handy the bare minimum for this is to download [OpenGFX](http://dev.openttdcoop.org/projects/opengfx) which provides free replacements for the game's graphics. To install it:

{{< highlight bash >}}

wget https://binaries.openttd.org/extra/opengfx/0.5.2/opengfx-0.5.2-all.zip
unzip opengfx-0.5.2-all.zip
tar -xvf opengfx-0.5.2.tar
mv opengfx-0.5.2 ~/.openttd/baseset

{{< / highlight >}}

## Adding An OpenTTD Service

With OpenTTD installed one of the last steps is to setup an OpenTTD service. Luckily for us, [Frodus](https://bitbucket.org/frodus/) has kindly created an OpenTTD init script [available here](https://bitbucket.org/frodus/openttd-init/wiki/Home).

First we need to download and unzip it:

{{< highlight bash >}}

wget https://bitbucket.org/frodus/openttd-init/downloads/openttd-init-1.1.1.zip
unzip openttd-init-1.1.1.zip -d openttd-init

{{< / highlight >}}

Then we need to add a symlink of the openttd file to `/etc/init.d`, update its permissions and add the OpenTTD service to the Ubuntu startup scripts.

{{< highlight bash >}}

sudo ln -s ~/openttd-init/openttd /etc/init.d/openttd
chmod 755  ~/openttd-init/openttd
sudo update-rc.d openttd defaults

{{< / highlight >}}

Finally rename `~/openttd-init/config.example` to `config` and edit the variables to suitable values.

## OpenTTD Configuration

The last step before starting to play is setting up the `openttd.cfg` file - a list of the available options is on the [OpenTTD wiki](https://wiki.openttd.org/Openttd.cfg). A good overview of what options to change is also available on the [OpenTTD Coop Wiki](https://wiki.openttdcoop.org/Map_Preparation).

One setting of great importance is the `sever_port` (default value 3979) setting. Whatever option you choose, make sure to add a firewall rule. If you are using the default Ubuntu firewall `ufw` it is as simple as:

{{< highlight bash >}}

sudo ufw allow 3979/tcp
sudo ufw reload

{{< / highlight >}}

`iptables` configuration is left as an exercise to the reader.

Some of the other settings I like to change are:

- [`min_active_clients`](https://wiki.openttd.org/Min_active_clients) set this to 1 so the game is paused when noone is connected.
- [`inflation`](https://wiki.openttd.org/Inflation) set this to false since after a while money is no object.
- [`forbid_90_deg`](https://wiki.openttd.org/Forbid_90_deg) set this to false as 90 degree turns are ugly.
- [`train_acceleration_model`](https://wiki.openttd.org/Train_acceleration_model) set this to realistic otherwise slopes will seriously slow your trains down.
- [`vehicle_breakdowns`](https://wiki.openttd.org/Vehicle_breakdowns) breakdowns are annoying so no breakdowns please.
- [`no_servicing_if_no_breakdowns`](https://wiki.openttd.org/No_servicing_if_no_breakdowns) no servicing when there are no breakdowns.

## Joining Your Server

Once you have setup your config it's finally time to fire up your OpenTTD server:

{{< highlight bash >}}

sudo service openttd start

{{< / highlight >}}

To join your server in the Multiplayer settings window click the *Add Server* button and add your server with the port number like so:

<img class="post-img" src="/images/openttd-multi-screen.jpg" alt="OpenTTD multiplayer settings window" title="OpenTTD multiplayer settings window"/>

And after joining you should be greeted with your shiny new OpenTTD game. Happy network building!
