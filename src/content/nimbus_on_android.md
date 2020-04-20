---
title: Building Nimbus on Android
date: "2020-01-14"
description: "Building an Eth client on a phone"
cover: ./nimbus/clouds.jpg
path: "/blog/nimbus_on_android"
draft: false
---

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@fkaymak?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Faruk Kaymak"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Faruk Kaymak</span></a>

Originally posted [here](https://our.status.im/building-nimbus-on-android/)

# Building Nimbus on Android

Or, how I built an Ethereum client designed for resource-restricted devices on a resource-restricted device.

Nimbus's tag line is "an Ethereum 2.0 Sharding Client for Resource-Restricted Devices."  My thought when reading this is if it's designed for resource-restricted devices, let's see if we can build it on a resource-restricted device.  And when I say "resource-restricted device," I'm not talking about a Raspberry Pi, though you can certainly [build and run Nimbus on a Raspberry Pi](https://github.com/status-im/nimbus#raspberry-pi).  What I'm talking about is building and running Nimbus on an Android device, in my case, a Oneplus 6T.  What follows is a guided tour of how to get a development environment up and running on your Android device and how leverage that to build and run Nimbus.

## Environment

Before you can build Nimbus, you need something more foundational, a development environment.  

### [Termux](termux.com)
First stop on our road to building Nimbus is [Termux](termux.com).  Termux is a complete Linux distribution packaged up inside an Android app and gives you access to a complete operating system, including a terminal emulator, and even any of several Linux GUIs if you want, no root required.  Once you've installed Termux and fire it up, you'll be greeted with a very familiar site, the Bash terminal prompt.  If you've ever used LInux, this is the same Bash shell you've used on any other distribution under the sun.  So far, so good. 

Now that we have a Linux environment available to us, we need to get all the [Nimbus prerequisite packages](https://github.com/status-im/nimbus#prerequisites).  As a reminder, Nimbus requires RocksDB, PCRE, GNU Make, Git, etc.  The issue is that Termux has a relatively limited set of packages available to it.  While you can install Make, Git, and maybe some of the other needed Posix utilities, any attempt to `apt install RocksDB` will result in an unfriendly `package not found` error message from the Termux package manager.  What's a wouldbe phone coder to do?

### Ubuntu on Termux
Our second stop is to leverage one of Termux's many awesome features, the ability to support [chroot](https://en.m.wikipedia.org/wiki/Chroot).  Since Termux's default package manager doesn't have the packages we need, we're going to have to somehow get access to a package manager that does, which means setting up a guest Linux distribution in a chroot on Termux.  There are a variety of options for other Linux distributions that are supported but we're going with Ubuntu. Follow the instructions [here](https://wiki.termux.com/wiki/Ubuntu) to get Ubuntu installed.  Once installed, you can access the Ubuntu environment via the `start_ubuntu.sh` script.  When I installed Ubuntu, that script was located in `~/jails/ubuntu` so I made a simple bash script called `ub.sh` that was placed in my home Termux directory to simplify getting into the Ubuntu environment.

```sh
#!/bin/bash

./jails/ubuntu//start-ubuntu.sh
```

Now, we've got the basic OS setup that we need to be able to build Nimbus.  It'd be nice to have a nice code editor to work with like [VSCode](https://code.visualstudio.com/) but that isn't possible on your phone, or is it?

### BLACKICECoder
Our last stop on this section of the tour is getting a code editor.  I use Visual Studio Code as my editor of choice and was hunting around the web for something similar to use on my phone.  Sure, there's [Stackblitz](stackblitz.com) and [Code Sandbox](codesandbox.io) that will theoretically work in your phone's browser, but that's not going to work for building Nimbus locally.  What I stumbled across was a little tool called [BLACKICECoder](https://github.com/raynoppe/BLACKICEcoder).  BlackIceCoder is essentially a stripped down version of VSCode that runs in your phone's browser that is specifically designed to run on Termux.  Follow the instructions on the repo to get it up and running.

One important note: BlackIceCoder should be installed in your base Termux install and not from within the Ubuntu chroot.

Below is a simple script that makes starting BlackIceCoder easier.

```sh
#!/bin/bash

php -S 127.0.0.1:1028 -t ~/BLACKICEcoder
```

When you run this script in Termux, open your phone's browser and go to localhost:1028 and the IDE will be ready and waiting for you to get down to coding.

## Build nimbus

Okay, if you've made it this far, we're finally at the point where we ready to actually build an Etheruem client on our phone.  

* First, load up your Ubuntu chroot (simply `./ub.sh` if you built my helper script from above)
* Install the prerequisites: `apt install librocksdb-dev libpcre3-dev git`
* Clone the Nimbus repo: `git clone https://github.com/status-im/nimbus.git`
* Run `make` to get all the needed submodules
* Run `make nimbus` to build the Nimbus binary. 

If all goes well, you should see output like below:
```sh
root@localhost:~/nimbus# make nimbus
Building: build/nimbus
[NimScript] exec: nim c --out:build/nimbus -d:chronicles_log_level=TRACE --verbosity:0 --hints:off --warnings:off -d:usePcreHeader --passL:"-lpcre" nimbus/nimbus.nim
root@localhost:~/nimbus#
```

Now, let's see what we can do!

## Run nimbus

This is the moment we've been waiting for.  Run `build/nimbus` to fire up the client.  Below is an example of the output you can expect from the client.  Note, I've turned off discovery and set the log level to TRACE so I can see what the client is doing.  If you just run `build/nimbus` by itself, you probably won't see much output.

```sh
root@localhost:~/nimbus# build/nimbus --nodiscover --log-level=TRACE
Nimbus Version 0.0.1 [linux: arm64, rocksdb, e651975c]
Copyright (c) 2018-2020 Status Research & Development GmbH
DBG 2020-01-04 01:30:33+00:00 UPnP                                       topics="nat" tid=23766 file=nat.nim:64 msg="Internet Gateway Device found."
DBG 2020-01-04 01:30:33+00:00 UPnP: added port mapping                   topics="nat" tid=23766 file=nat.nim:122 externalPort=30303 internalPort=30303 protocol=TCP
DBG 2020-01-04 01:30:33+00:00 UPnP: added port mapping                   topics="nat" tid=23766 file=nat.nim:122 externalPort=30303 internalPort=30303 protocol=UDP
INF 2020-01-04 01:30:33+00:00 RLPx listener up                           tid=23766 file=p2p.nim:87 self=enode://89e925220f113521a1d6ced3f5ad45575ad7127c7e6307501c00874208e0d8010925d30d3b48ef9715c923a728578a7f1f8e9df3507027f0b76ab99d3e9a30dc@172.72.14.93:30303
INF 2020-01-04 01:30:33+00:00 Discovery disabled                         tid=23766 file=p2p.nim:109
TRC 2020-01-04 01:30:33+00:00 Waiting for more peers                     tid=23766 file=p2p.nim:112 peers=0
```

## Change nimbus

Now, yea, we've finally built Nimbus and gotten it up and running. That's cool and all but throwing BlackIceCoder into the mix is where the magic happens.  Let's get down to coding!

* Open Termux
* Run `./black.sh` and BlackIceCoder will start.
* Open your browser and go to localhost:1028.
* Find the directory where the Nimbus repo lives (probably something like jails->ubuntu->ubuntu-fs->root->nimbus) in the file drawer on the right
* Select whichever file you want to start working on, say `nimbus.nim` and get to work!  You'll see something like the below image.
![BlackIceCoder](/BlackIceCoderNimbus.jpg)
* Once you've finished your edits, save your changes.
* Switch back to Termux and open a new terminal session by swiping over from the left side of the screen and tapping on "New Session"
* Run `./ub.sh` to get to your Ubuntu chroot and navigate to your nimbus repo directory.
* Run `make nimbus` again, run `build/nimbus` and test your changes.  

And that's it.  Nwow we're actually coding, building, and running Nimbus entirely within a virtual Linux environment on an Android device.  As of this writing, I've actually used this set up for a few commits on a PR I'm making for the Nimbus repo so it actually does work.  It's not a replacement for my main development setup but if I'm away from my PC and have a sudden inspiration or the urge to code, it lets me scratch that itch.

## Additional notes

There are a few things to be aware of when working on Nimbus with the above setup.

### Nimbus notes
* The above should theoretically work on any reasonably modern Android device though your mileage will vary if you have a low powered device without alot of space or RAM.  That said, this works pretty well on a reasonably specced device like my Oneplus 6T.
* The `make nimbus` command fails sometimes inexplicably and throws an "OSError".  This is probably just a resource constraint issue so don't have lots of other apps running while you're doing this.  Remember, we are running an Ubuntu emulator insde of a Linux emulator inside of an Android app, so just be mindful of the limitations of your device.  
* `make nimbus` will probably take a lot longer to compile Nimbus on your phone than on your high-pwoered workstation so don't get frustrated if it takes a couple of minutes to compile.  I've seen it take more than 5 minutes to compile.
* When running Nimbus, remember that you're running this on your phone, so probably best to always run it with the `--nodiscover` argument if you're not connected to Wifi or you might chew up a lot data while it tries to sync whatever chain you're connected to.
* If you run Nimbus, it doesn't really respect the `Ctrl+c` or `Ctrl+z` commands to shut down gracefully.  The only way I've found to kill Nimbus is to open a new terminal window and then run `killall nimbus` from the Termux terminal.  

### BlackIceCoder notes
* BlackIceCoder doesn't support syntax highlighting and color for Nim.  That said, it still does autocomplete and parentheses matching like a champ.
* Sometimes, after you've compiled and run Nimbus and then switch back to BlackIceCoder in the browser, you'll find that it's completely refreshed and showing the default screen again rather than still being open to the file you were editing last.  I think this is just a system limitation with how much Android can keep in memory at any one time. As such, always save your changes before switching between the browser and Termux.
