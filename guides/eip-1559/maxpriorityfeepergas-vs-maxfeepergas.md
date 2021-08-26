---
description: >-
  This guide will walk you through the difference between two EIP-1559 methods:
  maxPriorityFeePerGas and maxFeePerGas and help you understand when to use each
  of them.
---

# 🧐maxPriorityFeePerGas vs maxFeePerGas

Sending a transaction on Ethereum post London fork uses these two new gas price fields: `maxFeePerGas` and [`maxPriorityFeePerGas`](../../documentation/apis/ethereum/eth_maxpriorityfeepergas.md). We won't go into detail on the incentive theory behind fee markets here. Instead, we will dive into the difference between these two fields and when you might want to use one vs the other \(or both\).

If you've gone through our [tutorial on sending an EIP 1559 transaction](https://docs.alchemy.com/alchemy/guides/eip-1559/send-tx-eip-1559) then you've seen that we recommend using only the `maxPriorityFeePerGas` field. We did this for simplicity, but it's not always the better field to use.

To understand why, let's first make sure we're familiar with all the terms.

### What is the `baseFeePerGas`? <a id="what-is-the-base-fee-per-gas"></a>

Before talking about the two gas price limit fields, we need to understand what the base fee is. Post London fork, each block has a `baseFeePerGas` associated with it. You can see the base fee for the pending \(upcoming\) block with:

```text
web3.eth.getBlock("pending").then((block) => console.log("baseFee", Number(block.baseFeePerGas)));
```

The above snippet assumes you've set up your [Alchemy Web3 client](https://docs.alchemy.com/alchemy/documentation/alchemy-web3). If you haven't , check out the [EIP 1559 tutorial](https://docs.alchemy.com/alchemy/tutorials/sending-txs/eip-1559) for some sample code.

The base fee is the bare minimum you will be charged to send a transaction on the network. The base fee is set _by the network itself_, not by miners. The base fee changes block by block, based on how full the previous block was.

### What is [`maxPriorityFeePerGas`](../../documentation/apis/ethereum/eth_maxpriorityfeepergas.md)? <a id="what-is-max-priority-fee-per-gas"></a>

In the previous section we mentioned that the base fee is determined by the network. Well, it is also _**burned**_ when the block is mined, meaning that the miner does not get the base fee as a reward for mining a block. So how do miners get compensated for the computational work of mining?

When you submit a transaction you will also provide a "tip" to the miner. This is the `maxPriorityFeePerGas` field. The bare minimum you should tip the miner is 1 wei, otherwise there is no incentive for the miner to execute the transaction. The higher your tip, the more likely your transaction will be included in the block. This concept is pretty much the same as pre-London fork.

### And finally, what is `maxFeePerGas`? <a id="and-finally-what-is-max-fee-per-gas"></a>

Now that we know about base fee and tip, `maxFeePerGas` is super simple. It's just the sum of the two:

`maxFeePerGas = baseFeePerGas + maxPriorityFeePerGas`

### When to use `maxPriorityFeePerGas` vs. `maxFeePerGas` <a id="when-to-use-max-priority-fee-per-gas-vs-max-fee-per-gas"></a>

As mentioned in the tutorial, the most _guaranteed_ way to have your transaction included in the block is to just specify a `maxPriorityFeePerGas` field \(which is a tip\). In this case, Alchemy/geth/etc will look up the pending `baseFee` and then set the `maxFeePerGas` field accordingly \(to the sum of the base fee and the tip\). All you have to do is decide how much tip to provide, which you can get by simply calling the `eth_maxPriorityFeePerGas` method on Alchemy.

Although this is the simplest method, it may not be the cheapest. There are two dimensions to consider when submitting your transaction: speed and cost. If you bid high, you get mined earlier. If you bid low, you get mined later or not at all. When you submit only the `maxPriorityFeePerGas` field, the defaults will fill in the base fee for you. Recall that the base fee depends on how full previous blocks were. If many of the previous blocks were full, then the base fee can end up being quite high! And since we are filling in the value for you, you can end up paying a surprisingly high gas price.

To avoid this pitfall you can supply the `maxFeePerGas` field. If you supply _only_ this field, then the tip will be filled in for you, however, there are no guarantees on _when_ your transaction will be mined. You can also supply both the `maxFeePerGas` and the `maxPriorityFeePerGas` fields for full control.

### Let's see them in action <a id="lets-see-them-in-action"></a>

Just so that we 100% understand the concepts, let's submit a transaction that is guaranteed to fail by setting the `maxFeePerGas` to less than the sum of `baseFeePerGas` and `maxPriorityFeePerGas`. This code is an extension of the code in the [EIP 1559 transaction sending tutorial](https://docs.alchemy.com/alchemy/guides/eip-1559/send-tx-eip-1559).

```text
web3.eth.estimateGas({
  to: toAddress,
  data: "0xc6888fa10000000000000000000000000000000000000000000000000000000000000003"
}).then((estimatedGas) => {
  web3.eth.getMaxPriorityFeePerGas().then((tip) => {
    web3.eth.getBlock("pending").then((block) => {
      const baseFee = Number(block.baseFeePerGas);
      const max = Number(tip) + baseFee - 1; // less than the sum

      sendTx(web3, {
        gas: estimatedGas,
        maxPriorityFeePerGas: Number(tip),
        maxFeePerGas: max,
        to: toAddress,
        value: 100,
      });
    });
  });
});
```

The output is just a constant stream of "waiting…"

```text
Transaction sent! 0x7648502609b5617fc23fac9daf92a10e8c5ecdd8fe709132c2ce50899685072a
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
Attempting to get transaction receipt...
```

Basically you will be waiting until the base fee of a block drops low enough that the sum of base fee plus tip is less than your `maxFeePerGas`, which could be a very long time or never depending on the network congestion.

### Conclusion <a id="hkcau-conclusion"></a>

Choosing to use `maxPriorityFeePerGas` vs. `maxFeePerGas` for sending transactions is completely dependent on what your priorities are. If you care about speed and want to get your transaction into the earliest block possible, you should use the default base fee and just set \(a high\) `maxPriorityFeePerGas`. However, if you care more about saving on gas, and less about transaction speed, you can set your own `maxFeePerGas` and potentially risk the transaction getting mined at a later block \(or earlier block if you choose to set it to be higher than the base fee + tip\).

**If you're interested in learning more, or have feedback, suggestions, or questions, reach out to us in** [**Discord**](https://alchemy.com/discord)**! Get started with Alchemy today by** [**signing up for free**](https://alchemy.com/?r=affiliate:5494a54b-6ae1-4d33-9016-c331c0dcdc1f)**.**
