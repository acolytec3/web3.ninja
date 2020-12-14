---
title: A Bright IDea
date: "2020-12-14"
description: "Building tooling for the BrightID project"
cover: ./bright_idea/blowing_glass.jpg
path: "/blog/bright_idea"
draft: false
---

<span>Photo by <a href="https://unsplash.com/@johanneswre?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Johannes W</a> on <a href="https://unsplash.com/s/photos/glass-blowing?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

For the first time in a while, I participated in a hackathon, specifically the [GR8 ](https://gitcoin.co/hackathon/gr8/) hosted by [Gitcoin](https://gitcoin.co).  The particular bounty/context I participated in was building for [BrightID](https://brightid.org).  BrightID is a "social identity" protocol which can be leveraged by other applications to combat [sybil attacks](https://en.wikipedia.org/wiki/Sybil_attack).  A good use case in the web3 dev world would be using BrightID as a human verification tool to ensure that a faucet for distributing testnet tokens isn't drained by a determined bad actor.  Think of it as an alternative to CAPTCHAs and asking you to tweet at the faucet in order to get your tokens.

My project for the hackathon is a Two-fer, a Javascript/Typescript SDK for integrating an application with BrightID and then a demo app that uses all of the functions exposed by the SDK and also can serve as a test harness for validating that a new integration is working right.  

## The SDK

The SDK is [published on NPM](https://www.npmjs.com/package/brightid_sdk) and the repo is [here](https://github.com/acolytec3/brightIdSDK).  It gives an application owner all the tools needed to integrate an app with BrightID and abstracts away the tedium of constructing the HTTP calls and parsing responses that would be necessary if you do the integration from scratch.  There are 3 basic pieces of integrating an app and the SDK supports all of them:

### Linking

The first thing an application does to integrate with BrightID is provide a link/QR code for app users to leverage to link their BrightID with the application.  The application generates an arbitrary "contextID" which can be any arbitrary **and** unique string that is useful for the application, so it could be a user name, a [uuid](https://www.npmjs.com/package/uuid), an Ethereum address, etc. Once this is done, the application can then query the BrightID nodes using that contextID to determine if the associated BrightID is unique, without ever exposing the BrightID itself to the application.  

### Sponsoring

One challenge that can face new users on an application when confronted with a BrightID linking request is they may not have a verified BrightID, since it's a social identity protocol that requires a user to have a BrightID linked to other BrightID users in order to evidence the uniqueness of the user.  In these cases, the application can "sponsor" a user directly within the BrightID network and the SDK provides the necessary functionality to achieve that.  A detailed explanation of sponsorship can be found [here](https://medium.com/brightid/brightid-sponsorships-5327a8d39f1e).

## The Demo App

The app is deployed [here](http://acolytec3.github.io/brightid_test_app).  It's **not** intended to be a production ready app so don't treat it like one.  

It can be used for testing out the various pieces of app integration, including generating deeplinks, checking a contextId's verification status with regard to a specific application, sponsoring a contextId, and also exposes the testing interface that allows an application to temporarily change the status of a BrightID for that specific application context.  It also displays API responses each time you trigger a function for ease of troubleshooting when issues occur.

In order to use the sponsoring and testing features of the demo app, you will need to work with a BrightID node operator to create an application context.  More about that [here](https://www.brightid.org/#use-in-your-project).  You can also join the [BrightID Discord](https://discord.gg/GrqWwbdtSy) for more details.

So, have fun and hope this is helpful!
