---
layout: post
title:  "Raspberry Pi Torrent SeedBox Setup"
date:   2024-12-30 9:42:08 AM
categories: raspberry pi torrent transmission irssi seedbox autodl
---
# Setting up Auto Torrent Uploads to a Raspberry Pi Torrent SeedBox using Irssi and Transmission
I'm writing this post to document my Torrent SeedBox setup. In my case I run the Irssi IRC client locally to
login to IRC and auto download torrents matching certain criteria. These locally downloaded torrents are
uploaded to my Raspberry Pi which runs Transmission and from there it is downloaded and seeded automatically.
The Raspberry Pi runs 24/7, so once downloaded, it remains seeding until I choose to remove it.

## Setup the Raspberry Pi
I have Ubuntu Server Edition (22.04.5) running on my Raspberry Pi. To get Transmission running, just do the
following...

{% highlight shell %}
sudo apt install transmission-daemon
{% endhighlight %}

To configure Transmission you will need to stop the service first with...

{% highlight shell %}
sudo systemctl stop transmission-daemon.service
{% endhighlight %}

Now you can edit `/etc/transmission-daemon/settings.json` and then start transmission with...

{% highlight shell %}
sudo systemctl start transmission-daemon.service
{% endhighlight %}

Some important settings to modify and take note of...

> rpc-username
>
> rpc-password
>
> rpc-whitelist

I based these instructions off [these](https://pimylifeup.com/raspberry-pi-transmission/)

## Setup the Transmission Uploader

I have written a small Python program intended to be called by the Irssi client to upload any auto downloaded
torrent to the Transmission daemon running on the Raspberry Pi. It can be found [here](https://github.com/Hammit/transmission-rpc-uploader)

Clone the repo locally and read the [Overview](https://github.com/Hammit/transmission-rpc-uploader?tab=readme-ov-file#overview)
and [Installation](https://github.com/Hammit/transmission-rpc-uploader?tab=readme-ov-file#installation)
instructions to get it setup.

The `.env` values can be filled based on your Raspberry Pi Transmission setup.

## Setup Irssi and AutoDl

Install Irssi IRC client with `sudo apt install irssi irssi-scripts`

Now we need to install autodl-irssi, which is a script for Irssi to auto download announced torrent files in IRC.
autodl-irssi can be found [here](https://autodl-community.github.io/autodl-irssi/). Follow the [Installation](https://autodl-community.github.io/autodl-irssi/installation/)
instructions.

Now, you need to configure AutoDl so that when a torrent is auto downloaded, the Transmission Uploader is called
and the torrent is uploaded to the Transmission daemon. Refer to the [instructions](https://github.com/Hammit/transmission-rpc-uploader?tab=readme-ov-file#using-from-irssi)
in my repo for the relevant options

