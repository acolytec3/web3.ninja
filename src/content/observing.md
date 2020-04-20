---
title: Subspace - Observing DeFi
date: "2020-03-20"
description: "Building a simple defi dashboard with Subspace"
cover: ./observing/starttrails.jpg
path: "/blog/observing_defi"
draft: false
---

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@derekthomson?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Derek Thomson"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Derek Thomson</span></a>

Originally posted [here](https://our.status.im/subspace-observing-defi/)


Ever wanted to be part of the DeFi revolution but don't know where to start?  Ever wanted to incorporate event tracking into your Dapp but not eager to build out complicated event tracking logic in your backend? Well, my friend, now's your chance.  Subspace is calling.. 

## What is Subspace?

For those of you not familiar with [Subspace](https://subspace.embarklabs.io), it's a Javascript library from the Embark team for tracking smart contract state variables, events, address balances, etc on Ethereum.  It's built on top of [RxJS](https://rxjs-dev.firebaseapp.com/guide/observable) observables and can be used to easily connect smart contract events or state changes into your Dapp with a few simple steps.

## What's an observable?

Glad you asked.  According to the RxJS docs, "Observables are lazy Push collections of multiple values."  Okay...great, what can I do with a lazy push collection?  For our purposes, think of an observable as the stream of state changes an Ethereum smart contract public variable or the stream of events emitted by a smart contract. And, Subspace makes it easy to grab these events or state changes and connect them into a Dapp.  So, let's see what it looks like.

## What are we doing here?

Before we dig into Subspace, let's sketch out an idea for a Dapp that uses smart contract events.  In this article, we're going to build a simple DeFi dashboard that shows you the last 5 DAI -> ETH trades on the Uniswap DAI contract and the average ETH/DAI price (i.e. how many DAI it costs to buy 1 ETH).

We're going to use [React](https://reactjs.org/) as our framework. Love it or hate it, it is the most popular Javascript UI framework around today.

Since I will be the first to admit I'm not much of a UI/UX person, I've recently come across [Grommet](https://v2.grommet.io/), which is a component library built on top of React that is mobile-first, responsive, accessible, yadda yadda yadda.  Basically, it's great so we'll some of its components to make our Dapp look not completely terrible.

So, all that to say, don't hate.

## Build the Dapp already!

Okay, let's get down to business. First, let's get the boring stuff out of the way:
```
npx create-react-app subspace-demo
cd subspace-demo
yarn add rxjs web3 @embarklabs/subspace grommet grommet-icons styled-components
```

This will get all your dependencies installed.  Next, let's purge out the React boilerplate in the app and get Grommet set up.

Open up App.js and replace the imports with the following:
```js
import React, {useEffect, useState} from 'react';
import {
    Box,
    Grommet,
    DataTable,
    Text
} from 'grommet';
import { pipe } from 'rxjs';
import Subspace, {$latest} from '@embarklabs/subspace';
import Web3 from 'web3';
```

Next, replace the code in the `return` statement of the `App` function with this:
```js
 return (
    	<Grommet theme={theme}>
        	 <AppBar>Subspace DeFi Dashboard Demo</AppBar>
	</Grommet>
  );
```

Add this component:
```js
const AppBar = (props) => (
    <Box tag='header' direction='row' align='center' alignContent="center"  background='brand'
        pad={
            {
                left: 'medium',
                right: 'small',
                vertical: 'small'
            }
        }
        elevation='medium'
        style={
            {zIndex: '1'}
        }
        {...props}/>
);
```

Now we've got the skeleton of an app.  Let's wire it up to Ethereum.

## Setting up the plumbing

First, we need a web3 provider.  For simplicity's sake, we're going to take advantage of the openly accessible Ethereum gateway provided by the fine folks over at [Cloudflare](https://developers.cloudflare.com/distributed-web/ethereum-gateway/).
```const web3 = new Web3('https://cloudflare-eth.com');```

Next, we need a smart contract to interact with.  The ABI for the Uniswap Exchange contract.  You can get it [here](https://raw.githubusercontent.com/acolytec3/subspace-demo/master/src/contract/exchange_abi.json).  I put it in a subdirectory called `contract` but do what you will.  Then, just import it like any other dependency. 
`import exchangeABI from './contract/exchange_abi.json'`

Finally, let's set up Subspace and the smart contract we're going to be working with:
```js
const subspace = new Subspace(web3.currentProvider);
var dai = new web3.eth.Contract(exchangeABI, '0x2a1530C4C41db0B0b2bB646CB5Eb1A67b7158667');
const daiContract = subspace.contract(dai);
subspace.init();
```
 Now, we're reading to start doing something interesting.

## Getting the data

While there are any number of ways to do this, we're going to use React Hooks to manage our Subspace observables that pipe date to the front end.

The basic premise of this Dapp is that we're going to show the exchange rate on most recent 5 DAI to ETH trades and then compute an average. I'm going to set up 2 Subspace observables, one to stream the trades from the Uniswap exchange contract and then another to package that up into an array of the last 5 trades to put in a widget in the Dapp.

### Setting up the observables

We're going to use State and Effect Hooks to manage the observables and the data they return.  As a reminder, [State hooks](https://reactjs.org/docs/hooks-state.html) are a React concept for managing state in functional components.  Here's the declaration of the State Hooks.

```js
    const [txnObserver, setObservable] = useState();
    const [last5Observer, setLast5Observer] = useState();
    const [latestBlock, setBlock] = useState();
    const [last5, setLast5] = useState([]);
```

Next, we're going to set up three [Effect Hooks](https://reactjs.org/docs/hooks-effect.html) to define and then manage the subscriptions for the Subspace observables.  A full explanation of how Effect Hooks work is beyond the scope of this article but the basic point to remember here is that these hooks are intended to be side effects of the rendering process in the `App` component and each effect should be isolated from the others or you will get odd behavior.

### Observable declaration

The below hook defines our 2 observables.
```js
    useEffect(() => {
        web3.eth.getBlockNumber().then((block) => setBlock(block));
        if (typeof(latestBlock) != "number") 
            return;

        const EthPurchased$ = daiContract.events.EthPurchase.track({
            fromBlock: latestBlock - 50
        });
        const last5$ = EthPurchased$.pipe($latest(5));
        setObservable(EthPurchased$);
        setLast5Observer(last5$)
    }, [setObservable, setLast5Observer, latestBlock])
```

`EthPuruchase$` is using the Subspace [trackEvent](https://subspace.embarklabs.io/api.html#contract-methods) method to stream the `EthPurchase` event from the Uniswap exchange contract.  

Important note: When using the `trackEvent` method, make sure to specify the `fromBlock` parameter as I've done in the above code block.  If you don't, it will begin at the very first block where the contract was deployed and stream every event from then on.  In the example here, that could be tens of thousands of events.  The first time I did this, the app basically got stuck endlessly streaming old `EthPurchase` events going back to the beginning of Uniswap.

`last5$` creates a new observable that returns the last 5 trades streamed from `EthPurchase$` using the [`latest` operator](https://github.com/embarklabs/subspace/blob/353fee7d23f12ffa3a8f91fe095500335016586e/packages/core/src/operators.js#L84) that is exposed by Subspace.

### Setting up subscriptions

Now, let's fire up the subscriptions.

First, let's start streaming `EthPurchase` events.  This is a very simple example and just starts up the subscription and prints each transaction to the developer console.  An important thing to remember is to pass the observable's `unsubscribe` function in the return value of the effect.  This is an important feature of Effect hooks.  The hook runs at each component re-render or whenever an element in the dependency array changes and so its important to clean up the subscription before the re-render to avoid memory leaks.  React will call the function specified in the return before re-running the hook during re-render.

```js
    useEffect(() => {
        if ((txnObserver === undefined) || (typeof latestBlock != "number")) {
            return;
        } else {
            txnObserver.subscribe((trade) => {
                console.log(trade);
            });
        }
        return txnObserver.unsubscribe;
    }, [txnObserver, latestBlock]);
```

Next, the magic.  Our `last5Observer` observable is the one that produces the data we actually use in the dapp's UI.

```js
    useEffect(() => {
        if (last5Observer === undefined) {
            return;
        } else {
            last5Observer.subscribe((fiveTrades) => {
                const prices = fiveTrades.map(trade => {
                    const txnDetails = new TradeDetails(trade.tokens_sold, trade.eth_bought);
                    return {'block': trade.blockNumber, 'rate': txnDetails.exchangeRate}
                });
                setLast5(prices);
            });
        }
        return last5Observer.unsubscribe;
    }, [last5Observer]);
```

The magic here is that under the hood,  Subspace stores all the events that are streamed from `EthPurchase$` in a local database and then `last5Observer` uses the `latest` operator to grab the "latest" 5 events and store them in the `last5` state variable maintained by the `App` component's state hook we set up earlier.  Inside the subscribe function, we're transforming the data in the `EthPurchase` events to provide the block number of each transaction and its exchange rate.  Any experts reading this will know that you can use RxJS operators like `pipe` and `map` to transform this data within the initial observable declaration but the above approachs makes it clear what we're doing. 

### Displaying the data

Now, let's do something with this data.  Since this is just a demo, we have only 2 visualizations of the data sourced from Subspace but this should give you an idea how powerful Subspace is.

First, let's use the data from the `last5Observer`.  

```js
const Tradelist = (props) => (
    <Box direction='column' align='center' pad="medium">
        <DataTable columns={
                [
                    {
                        property: 'block',
                        header: <Text>Block</Text>,
                        primary: true
                    }, {
                        property: 'rate',
                        header: <Text>ETH/DAI</Text>
                    }
                ]
            }
            data={
                props.last5
            }/>
    </Box>
)
```

The `TradeList` component makes use of the [`DataTable`](https://v2.grommet.io/datatable) component from Grommet to show the block number and exchange rate for the 5 last trades from Uniswap.  This table updates dynamically as new `EthPurchase` events are detected by Subspace.  Each time an event is detected, it gets piped from `txnObserver` to `last5Observer` and then updated in the `last5` State hook which causes React to then re-render the page with the latest trades.  Neat!

Next, let's show the average exchange rate of those trades.  This is just plain old React JSX and Javascript at this point but here's the code to display the average rate.  Note, the `Text` element is another Grommet component.

```js
    <Text margin="medium" textAlign="center">Average Exchange Rate on 5 latest Uniswap DAI->ETH trades = {
         (last5.reduce((a,b) => a + b.rate, 0) / last5.length).toFixed(6)
    }</Text>
```

## Wrapping up

And that's it.  In this walkthrough, we've piped in some Ethereum smart contract data, done some transforomations, and then pushed it to a front-end, all using Subspace for the backend and the most cutting edge version of React for the front-end.  Not bad for a few minutes coding.

Hit up <https://acolytec3.github.io/subspace-demo> to see the whole thing in action.