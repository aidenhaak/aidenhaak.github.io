---
title: Setting Up An OpenTTD Server Part 2
date: 2021-01-29
description: I found myself wanting to scratch my OpenTTD itch again but I need to setup another server
images: ["images/new-grf-settings.png"]
type: post
---

I found myself wanting to scratch my OpenTTD itch again but I need to setup another server as I had long ago nuked the server for [part one]({{< ref "2015-11-08-setting-up-an-openttd-server" >}}). Turns out that part one is (unsurprisngly) a bit out of date after five years and this time I wanted to dabble in some custom graphics or GRFs for my server.

For the first problem there is a an easy fix thanks to an [OpenTTD docker image](https://hub.docker.com/r/bateau/openttd). As for the custom GRFs - well you'll just have to keep reading.

For this guide I'm assuming:

- That you're running OpenTTD locally on a Windows desktop.
- You have a Google Cloud account setup and you know your way around it a bit.
- The OpenTTD server will run on Ubuntu.

## Customising the Custom Graphics

I found the easiest way to get the graphics files setup was to do it in game via the New GRF settings

<img class="post-img" src="/images/new-grf-settings.png" alt="OpenTTD new GRF settings window" title="OpenTTD new GRF settings window"/>

Once you are happy with your custom GRFs (making sure that they are compatible with each other!) it's time fire up the command prompt and run OpenTTD as a local server using the following command:

{{< highlight batch >}}

openttd.exe -D

{{< / highlight >}}

This will make OpenTTD generate the config file and GRF files that our server can use. After a brief wait you can stop the OpenTTD server after it has finished generating the map and has started the game.

Next navigate to `C:\Users\<username>\Documents\OpenTTD` where you should see a `openttd.cfg` file and a `content_download` folder. These are the files we will be uploading to a storage bucket. It is a good idea to double check that the new GRFs are correctly included in the config file, you should see something like the following at the end of the `openttd.cfg` file:

{{< highlight cfg >}}

[newgrf]
44440111|9B505....AFB0CA|uk_renewal_set.3.04\pb_ukrs.grf = 
F1250007|997B3....46D7EB|firs_industry_replacement_set_3-3.0.12\firs.grf = 0 0 0 0 0 0 16 150 80 300
4A530117|CA321....D4A501|ecs__firs_vehicle_set-2014.11.26\efrefit.grf = 

{{< / highlight >}}


## Google Cloud Setup

We will be running OpenTTD on the [free F1-micro instance](https://cloud.google.com/free) but first we have a few things to setup.

### Storage

Firstly create a new [Storage bucket](https://cloud.google.com/storage) for your OpenTTD files then upload the `openttd.cfg` file and `content_download` folder from the `C:\Users\<username>\Documents\OpenTTD` folder to a `openttd` folder in the bucket. Remember to edit the `openttd.cfg` file to your desired settings before uploading as this is the configuration your server will be using.

After the upload has finished the folder in your bucket should look like the following.

<img class="post-img" src="/images/openttd-bucket-contents.png" alt="OpenTTD bucket contents" title="OpenTTD bucket contents"/>

### Firewall

We need to add a [firewall rule](https://console.cloud.google.com/networking/firewalls/) to the [VPC network](https://cloud.google.com/vpc/docs/vpc) the server will be running in to allow the OpenTTD traffic through. Create a firewall rule that allows TCP and UDP on your desired OpenTTD port -- by default this is port 3979.

<img class="post-img" src="/images/openttd-firewall.png" alt="OpenTTD firewall setup" title="OpenTTD Firewall Setup"/>

### Putting It All Together

Now it's time to create a new VM instance but there are a few settings that need to be configured. Firstly, make sure to use a Ubuntu image and then assign a public IP to your instance to ensure that it is reachable over the public internet.

<img class="post-img" src="/images/openttd-public-ip.png" alt="OpenTTD Public IP" title="OpenTTD Public IP"/>

Then copy the following bash script _after_ changing the `OPENTTD_BUCKET_NAME` in the script to the same name as the bucket you created earlier in this guide.

{{< highlight bash >}}

#!/usr/bin/bash

if [ "$EUID" -ne 0 ]
  then echo "Please run as root"
  exit
fi

# Use the same bucket name from earlier in the guide here.
OPENTTD_BUCKET_NAME="bucketname"

# Install some prerequisites.
apt-get update -y
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# Next steps copy pasted from: https://docs.docker.com/engine/install/ubuntu/

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

# Add the Docker repository.
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Install docker
apt-get update -y
apt-get install docker-ce docker-ce-cli -y

# Add the openttd group and user.
groupadd openttd -r
useradd openttd -r -g openttd,docker

OPENTTD_GROUP_ID=`id -g openttd`
OPENTTD_USER_ID=`id -u openttd`

# Add firewall exceptions.
apt install ufw -y
ufw allow 3979

OPENTTD_DIR="/etc/openttd"

# Download the openttd config file and GRFs.
mkdir -p $OPENTTD_DIR
gsutil cp -r "gs://$OPENTTD_BUCKET_NAME/openttd" $OPENTTD_DIR
chown openttd:openttd -R $OPENTTD_DIR

# Finally run the container.
docker run --name openttd -d \
    -e PUID=$OPENTTD_USER_ID \
    -e PGID=$OPENTTD_GROUP_ID \
    -v $OPENTTD_DIR:/home/openttd/.openttd \
    -p 3979:3979/tcp \
    -p 3979:3979/udp \
    bateau/openttd:latest

{{< / highlight >}}

And paste it into the start-up script section for the VM.

<img class="post-img" src="/images/openttd-startup-script.png" alt="OpenTTD start-up script" title="OpenTTD Start-up Script"/>

Now create the instance and sit back and wait for the magic to happen 🧙‍♂️

Add the server IP via the in-game browser and you should be able to join!

## Troubleshooting

If you can't connect to your OpenTTD server here are a few basic steps you can try. Firstly, make sure you can ping the server via its public IP. Secondly, verify that the docker container is actually running on the server. 

You can do this by running `docker ps -a` and you should see an `bateau/openttd` image in the results. If it's not there it's likely the startup script failed to run and you can try running the script manually. One easy way to do this is to upload the initialization script to the openttd bucket and run the following commands:

{{< highlight bash >}}

gsutil cp "gs://OPENTTD_BUCKET_NAME/init.sh"
chmod +x init.sh
sudo ./init.sh

{{< / highlight >}}

After the script has run check the docker container is running -- if OpenTTD still isn't running after this the problem is left as [an exercise for reader](https://abstrusegoose.com/395).

## Next Steps

Some next steps to consider for your server:

- Add a domain for your server.
- Add cron job to upload save games to storage bucket so you can relive your greatest maps!