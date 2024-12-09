---
layout: ../../layouts/post.astro
title: "Understanding L2 Transaction Lifecycle: From Submission to L1 Finality"
pubDate: 2024-12-09
description: "This is the first post of my new Astro blog."
author: "querty"
excerpt: "When you deposit assets from a Layer 2 to a centralized exchange, you often encounter a waiting period of around 25 minutes before the funds become available. Why such a delay? This article breaks down the entire journey of a Layer 2 transaction: from its initial submission to its final settlement on Ethereum. We'll dive into key concepts like the OP-Stack architecture, blob transactions, and the fascinating two-phase commit process that ensures your transaction's security through cryptoeconomic guarantees worth billions of dollars."
image:
  src:
  alt:
tags: ["rollup", "consensus", "ethereum"]
---

Recently, when depositing ETH from Base to Binance, I encountered this explicit message: *'Please note that L2 networks require its corresponding L1 transaction to reach finality (64 blocks) before deposits can be unlocked. ETA ~25 min'*. 

This constraint raises two interesting technical questions: why is L1 finality crucial in this context, and where does this 25-minute delay specifically come from? Let's analyze the underlying mechanisms of Ethereum rollups to understand these points.

## Initial Transaction Processing

Let's take a concrete example: Alice wants to send 1 ETH to Bob on Base, a Layer 2 using the OP-Stack. This seemingly simple transaction involves several key actors in its processing chain.

It all begins when Alice initiates the transaction from her favorite wallet, specifying Bob's address and the amount of 1 ETH. The wallet then transmits this transaction to the configured RPC node, which relays it to the Sequencer. This essential component of the architecture integrates the transaction into the next block before broadcasting it to the **P2P** (peer-to-peer) network.
## Data Availability: Publishing to L1

At this stage, the transaction is marked as "**unsafe**" by the OP-Stack. Why? Simply because the block data hasn't yet been published on Ethereum, a crucial step to ensure the transaction's security.

The next step is to publish this block on Ethereum, but not just any way: since the introduction of [**EIP-4844**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-4844.md), we transmit it via what a new transaction type: blob transactions (**Type 3**). The OP-Batcher handles the behind-the-scenes work, publishing all this data on L1. This step takes about ten minutes, from inclusion in the L2 block to publication on L1.

Once published on L1, our transaction becomes "**safe**" on Base. But that's not all - there's still the crucial step of "**finality**" to consider. In Ethereum Proof-of-Stake, finality represents an impressive **cryptoeconomic guarantee**: any attempt at modification would require burning at least 33% of the total staked ETH on the network, approximately $72 billion at current prices.

<figure style="text-align: center">
  <img src="/posts/l2-transaction-lifecycle/beacon-deposit-contract-balance.webp" alt="Ethereum 2.0 finality" style="margin: auto; display: block;" />
  <figcaption style="text-align: center"><i>Total ETH staked in the <a href="https://etherscan.io/address/0x00000000219ab540356cbb839cbe05303d7705fa">Beacon Deposit Contract</a></i></figcaption>
</figure>

## Ethereum Finality: A Two-Step Process

This finality is ensured by [**Casper FFG**](https://arxiv.org/abs/1710.09437) (Friendly Finality Gadget), a mechanism inspired by **BFT** (Byzantine Fault Tolerance) consensus protocols. Its operation relies on several key concepts:

### Epoch Structure and Roles

An epoch is divided into 32 slots of 12 seconds each, totaling 6.4 minutes. For each epoch:

- A block proposer is randomly designated per slot, responsible for creating and proposing new blocks
- The entire validator set is divided into 32 committees (one per slot), ensuring decentralized block verification
- Each validator participates only once per epoch, making the process efficient and fair
- The first slot contains a checkpoint (or the most recent block if this slot is empty), serving as a reference point for finality

### The Two-Phase Commit Process

1. **_Justification Phase:_**
 Each slot's committee validators cast their votes, known as attestations, for the current epoch's checkpoint. A checkpoint transitions to a "**justified**" state once it accumulates votes from two-thirds of the total validator set. This supermajority requirement is crucial - it provides robust protection against potential network splits.
 
2. **_Finalization Phase:_**
When a justified checkpoint's child checkpoint (in the subsequent epoch) also achieves justified status, something important happens: the parent checkpoint, along with all blocks in its epoch, become finalized. This two-step approach isn't arbitrary - it creates a system where finality, once achieved, is both secure and irreversible. By requiring consecutive justified checkpoints, the protocol ensures that the network has reached a strong consensus about the chain's state.

<figure style="text-align: center">
  <img src="/posts/l2-transaction-lifecycle/2pc.webp" alt="Casper FFG" style="margin: auto; display: block;" />
  <figcaption style="text-align: center"><i>Two-Phase Commit workflow</i></figcaption>
</figure>

## Understanding the Timeline

Let's break down why transactions need these specific timeframes:

1. **_L2 to L1 Publication (≈10 minutes):_**
    - The OP-Batcher collects and batches L2 transactions
    - Preparation and submission of blob transactions
    - Network propagation and inclusion in L1 blocks
2. **_L1 Finality Process (≈13 minutes):_**
    - Best case: Transaction data lands at the start of an epoch, requiring two full epochs
	- Common case: Transaction data arrives mid-epoch, requiring the completion of the current epoch plus two more

This is why Binance sets a 25-minute waiting period - it accounts for the entire process while building in safety margins to ensure reliable transaction finality. 

### Resources 
- [Upgrading Ethereum](https://eth2book.info/capella/part2/consensus/casper_ffg/#the-mechanics-of-casper-ffg)
- [How does the NEW Ethereum work?](https://www.preethikasireddy.com/post/how-does-the-new-ethereum-work)