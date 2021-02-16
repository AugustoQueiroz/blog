---
title: 'Automatically logging my internet speed throughout the day'
subtitle: 'inb4: Yeah, my internet sucks'
category: tech
---

I found myself on the [speedtest.net](https://speedtest.net) site recently, and for the first time I noticed they have apps on all major platforms... as well as a cli for linux (you can find the instructions for installing it [here](https://www.speedtest.net/apps/cli)). Cool, this means I'm halfway done on an old project I had of logging my internet speeds throughout the day to check whether or not my ISP was managing to deliver the (already abysmal enough) 25Mbps they promise. Actually, halfway there is definitely understating it. The cli allows you to run a speedtest from the command line (no duh), and it has an argument for defining what format you want the output to be in, between human-readable (that is actually quite pretty, imo), csv, tsv, json, jsonl, and json-pretty. So

{% highlight bash %}
$ speedtest -f json >> results
{% endhighlight %}

Will run a speedtest for me, and output the results in json format, but not pretty, so it's all in a single line, into a file names results. This is cool, because ultimately I will write the output to a json lines format (aka each line of the file will be a json object instead of the entire file being a single object, meaning I can't just load the entire file as json, but I can just append to it without worrying about it being a valid json).

As I said, this is more than half the battle, now I just need to automate this to run every hour. This will be done by my RaspberryPi that is connected through ethernet. Recently I've decided to just put anything I want to always be running on there (at least for now), since it was just ~~collecting dust~~ sitting idly unused. To make it run every hour, I went for setting up a systemd service and a timer. The `.service` timer looks like this:

{% highlight bash linenos %}
[Unit]
Description=Internet Speed Logger

[Service]
User=pi
Type=oneshot
ExecStart=/usr/bin/sh -C 'speedtest -f json >> /path/to/results_file'
{% endhighlight %}

Here I'm saying I want the command above to be run once when the service is started. An important thing is that I want the user to be set to my own user, otherwise I wouldn't have write permission to my own file. The `.timer` file follows:

{% highlight bash linenos %}
[Unit]
Description=Speed Test Logger

[Timer]
OnActiveSec=0s
OnUnitActiveSec=1h

[Install]
WantedBy=timers.target
{% endhighlight %}

Where `OnActiveSec=0s` means I want the timer to fire for the first time as soon as the service starts, and `OnUnitActiveSec=1h` means I want the timer to go off 1h after the service is started (that is, 1h after the last time the timer went off!)

Both these files should be saved to `/etc/systemd/system/`. Then you must run the following commands:

{% highlight bash prompt %}
$ sudo systemctl daemon-reload
$ sudo systemctl start <your_service_name>.timer
{% endhighlight %}

This should start the timer and, so, have it fire immediately. If you expect to turn off the computer where this is running, and don't want to restart it manually everytime

{% highlight bash %}
$ sudo systemctl enable <your_service_name>.timer
{% endhighlight %}

## A couple considerations...

### 1. Why not use jsonl, since the results file *will* be a json lines document?

Because the jsonl format that `speedtest` outputs to is a lot more verbose than I need, and would require a lot more processing when I ultimately want to explore the results.

### 2. `journalctl -u <your_service_name>` is your friend

In case there are any errors with your service file, or with your service itself, this is how you can get the logs from it. And, actually, remember: this is where you can get the logs of whatever program you set to run. So be mindful of what and how you're logging!

### 3. There may be other options to replace `speedtest`

A friend has found out that on arch the package `speedtest-cli` works as well, but is open source. If for some reason you don't want to use the Ookla speedtest, or if you can't manage to install it on your distro, you can check out your package managers repos to see if there is an alternative. This might slightly change how the cli is used: with the arch package instead of passing `-f json` to get the result as a json you should use `--json`.