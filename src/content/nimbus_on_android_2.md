---
title: "Building Nimbus on Android: Part 2 / Eth 2"
date: "2020-02-20"
description: "Building an Eth2 client on a phone"
cover: "clouds.jpg"
path: "/blog/nimbus_on_android_2"
draft: false
---


<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@quentinreyphoto?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Quentin Rey"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Quentin Rey</span></a>

Originally published [here](https://our.status.im/building-nimbus-on-android-2/)

I'm back.  If you read my [last article](https://our.status.im/building-nimbus-on-android/), then you know that Nimbus is not just for your desktop.  That said, you're probably asking yourself, "Why do I care about running Nimbus on my phone inside a Linux sandbox?"  Running an Ethereum 1.0 node on an ARM device is so [2017](https://pgaleone.eu/raspberry/ethereum/archlinux/2017/09/06/ethereum-node-raspberri-pi-3/).  And, given the [current size](bitinfocharts.com) of the Ethereum 1.0 chain (2.12GB as of this writing), you're probably right.

What you're really asking is, "How can I get in the Eth 2.0 game?  What I really want to do is run a Nimbus beacon node on the new open testnets!" If you asked this question, you have come to the right place.

Note: Before diving into the fun, I want offer a friendly reminder that the steps below require downloading a rather large amount of data so only do this when your Android device is connected to WiFi if you have a limited cellular data plan.

# Building Nimbus for Eth 2.0 on Android
Before we go anywhere, follow the [Termux and Ubuntu on Termux](https://our.status.im/building-nimbus-on-android/) sections of my previous article to get your Linux sandbox ready to play in.

## Build Nimbus for Eth 2.0
Before we get started, start the Termux app and then open up the Ubuntu proot as discussed in the previous article.

Like with Nimbus for Eth 1.0, we have a few external dependencies to grab first.

`apt install build-essential, git golang-go libpcre3-dev`

Now, let's check and make sure we have the right version of Go installed since Nimbus requires Go 1.12+.

`go version` should produce an output of `go version go1.12` or higher.  If not, see the bottom of the article for help straightening this out.

Next, clone the Nimbus Beacon Chain repo.

```
git clone https://github.com/status-im/nim-beacon-chain
cd nim-beacon-chain> []
```

Run `make` and Nimbus will compile and build out all the necessary internal Nim dependencies.  You should see the below message once everything is done.
![Successfully installed Nimbus build dependencies](../images/image_1.jpg)

Now, like the instructions say, run `make` again to actually compile Nimbus for Eth 2.0.  Now, go grab a cup of coffee as this will take a while.  It took my OnePlus 6T something like 20-30 minutes to finally finish.  You should see this output once it's finished.
![Successfully built Nimbus](../images/image_2.jpg)

## Running Nimbus for Eth 2.0

Now that we've got Nimbus built, let's explore what you can do with it.


### Running the entire Eth 2.0 universe on your own phone

Let's start small.  You can run an entire simulated Eth 2.0 testnet on your phone, beacon nodes, validators, Eth 2, oh my!

`make VALIDATORS=192 NODES=6 USER_NODES=1 eth2_network_simulation`

This command will fire up a local Eth 2 network with 192 validators and 7 beacon nodes, one of which you can bring up on your own to see the sync process in action.

Open a second terminal in Termux, and open the Ubuntu proot in this terminal as well. The below steps will set appropriate environment variables and start up the user beacon node referenced above.

```cd nim-beacon-chain
./env.sh
./tests/simulation/run_node.sh 0 #
```
Here you can see the user node starts up, joins the network, and starts getting blocks to sync the chain.
![](image_3.jpg)

### Running a Beacon Node on the Public Testnet

Now, let's talk to some other nodes.

Run `make testnet0` to fire up your Nimbus beacon node and join testnet0. 

Nimbus will prompt you for your Goerli Eth1 private key to to become a validator.
![](image_4.jpg)
Just press `Enter` to skip for now as we're just going to run a beacon node.

*drum roll*

![](image_5.jpg)

We're off, genesis block received, block pool up and running, looking for peers.

![](image_6.jpg)

Now we're making history, syncing blocks, participating in consensus, helping secure the network, enforcing fork choice rules, building the future, all from the comfort of a phone. 

## Addendum - Help with Go versioning

If for whatever reason, `apt` installs a version of Go that's older than 1.12, do the following:
* `apt remove golang-go` to remove the old version that `apt` installed.
* Grab the newest version of go [here](https://golang.org/dl/) appropriate for your architecture.  Given that we're running this on Android, it will most likely be the ARMv6, ARMv8, or ARM64.  
* Follow [these instructions](https://golang.org/doc/install) to install the newest version of Go and set your `GOPATH` appropriately.