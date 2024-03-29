![Vixify Logo](https://i.ibb.co/TgqJpvx/logo2-color-transp.png "Vixify Logo")

# Vixify Blockchain

A modern pure Proof-of-Stake blockchain based on a verifiable delay functions (VDF) and a verifiable random function (VRF). Implements a synthetic Proof-of-Work using the VDF and VRF based on coin stakes and non-parallelizable mining.

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Summary

Vixify is a blockchain adopting a pure Proof-of-Stake consensus protocol based on a verifiable random function (VRF) and a verifiable delay function (VDF) that has the following properties: a) all addresses with a positive stake can participate in consensus; b) is fair regarding the stake and the distribution of rewards; b) is tolerant to several classic attacks such as Sybil attacks, "Nothing-at-stake" attacks and "Winner-takes-all" attacks.

## Blockchain Features

Vixify Blockchain has the following features:

* Proof-of-Stake - only stakeholders can participate in consensus and recieve rewards.
* Energy-efficient Single-thread Mining - Using a VDF allow the blockchain with blocks mined on a single-thread by each stakeholder. Under certain chip technologies the design if secure (for example, no miner has a chip technology that is x3 or x4 faster than the current state of the art in commercial chips).
* Secure - Using a verifiable random function (VRF) allows next-block miner to be unpredictable, discouraging attacks on stakeholders nodes.
* Catastrophic Failure-tolerant - supports catastrophic >50% stake failure or network fragmentation, unlike PBFT Proof-of-Stake blockchains that stop working under catastrophic conditions.
* Ethereum-compatible - Smart Contract build in Solidity have a huge community and support.
* Scaling - because block time standard deviation is very small we have predictable block times (beyond average block time), this encourages synchronicity (most of the time) and better scaling of block sizes and transactions per second.

## What are VDFs and VRFs (VXFs!)

Verifiable delays functions (VDFs) are essentialy cryptographic hash functions computing several steps of computation that cannot be paralelized but the computation can be verified much faster, or very fast. They have been proposed as solution to energy inefficient parallelizable Proof-of-Work consensus because of their non-paralelizable properties but they raised some concerns regading winner-takes-all scenarios for nodes with very fast specialized hardware, such as ASIC hardware. Under certain speed assumptions of the players we can handle the winner-takes-all scenarios, so we don't loose the fairness.

In more detail, a VDF is implemented by a tuple of three algorithms [1]:

* *VDFSetup(λ, T) → pp* is a randomized algorithm that takes a security parameter *λ* and a time
bound *T*, and outputs public parameter *pp*,
* *VDFEval(pp, x) → (y, π)* takes an input *x ∈ X* and outputs a *y ∈ Y* and a proof *π*.
* *VDFVerify(pp, x, y, π) → {accept, reject}* outputs accept if *y* is the correct evaluation of the VDF
on input *x*.

Verifiable random functions (VRFs) are defined using a public-key pair *(VK,SK)* and have the property that using a private key *SK* allows to hash a plain-text *x* into a hash *y* that can be verified using a public or verification key *VK*. VRFs are more useful than just signing a hash of the data because sometimes (for example with ECDSA) using signatures gives signatures that are not unique, then allowing a malicious miner to generate many signatures to search for one that make him the block proposer, bypassing the protocol [5][6].

VRFs are being popularized and use for leader-selection by the Algorand Blockchain project, although they use voting for Byzantine Fault-tolerance and do not use VDF as part of their consensus algorithm.

VRF syntax and properties are as follows [1]. A VRF is a triple of algorithms *VRFKeygen*, *VRFEvaluate*, and *VRFVerify*:

 - *VRFKeygen(r) → (VK, SK)*. On a random input, the key generation algorithm produces a verification key *VK* and a secret key *SK* pair.
 - *VRFEval(SK, x) → (y, ⍴)*. The evaluation algorithm takes as input the secret key *SK*, a message *X* and produces a pseudorandom output string *Y* and a proof *⍴*.
 - *Verify(VK, x, y, ⍴) → {accept, reject}*. The verification algorithm takes as input the verification key *VK*, the message *x*, the output *Y* and the proof *⍴*. It outputs *accept* if and only if it verifies that *Y* is the output produced by the evaluation algorithm on inputs SK and X.

## Vixify Consensus

Vixify Blockchain has a consensus algorithm that has very similar properties to Nakamoto Consensus, because is based on what we call: Synthetic Proof-of-Work. We define the later as having the following properties:

1. **Money Investment**: To start mining and competing *miners have to invest money*. On our case the need to hold some special coin stake on their coinbase wallet, in Nakamoto Consensus miners invest in special and powerful hardware.
2. **Time Investment**: Depending on know much they invest they will have to also invest time. Their *time investment will be inversally proportial to their money investment*.
3. **Random Timer**: Miners *run a timer with random time and the winner is the one with the smallest time*. In Vixify Consensus a private random timer is generated using the VRF and the VDF functions. On Nakamoto Consensus, looking for the pre-image generated an specially difficult hash has this property of a *random timer*.
4. **Block Proposer is Unpredictable**: Miners (with less than 50% of the stake) don't know if their mined block or whose block will be chosen for consensus. *Next block proposer is not predictable*.
5. **Block Proposer Chances are Predictable**: Miners might have some probabilistic hint that they are going to be the next block proposer but they are never certain, unless they have more than 50% of the stake and then they can start to do some attacks on the blockchain and eventually getting 100% of the stake and total control of the blockchain.

To see the differences between Nakamoto Consensus and Vixify Consensus check the following simplified diagrams:

![Nakamoto Consensus diagram](https://i.ibb.co/QYsqKDK/Vixify-Consensus.png "Nakamoto Consensus diagram")

Notice that the overall structure is very similar but Vixify is more complex because we use the assymmetric keys for VRF Pseudo-randomness and also two functions to generate and verify VDF outputs.

![Vixify Consensus diagram](https://i.ibb.co/7bY7gBm/Vixify-Consensus-3.png "Vixify Consensus diagram")

The important thing is that to generate the number of *T* steps of the VDF only use: a) the Previous Block Hash, b) the private key SK of the miner's wallet, and c) the stake *S* of the miner. In this way the only attack that the Miner can make is to generate many wallets and keys wanting to move the Stake to another wallet with another secret key SK. But that attack is very cumbersome. That is, when you find a SK that serves to trout something then you have to transfer the Stake to that wallet and looses the opportunity because he must put a transaction in the next block to move stake to the new wallet.



## Race Conditions: Shortest VDF chain

In case of uncles (aka orphan or stale block) the network can decide which chain is the best one by comparing the length of the chains. Comparing the sum of all VDF difficulties is not enought because a chain a similar or longer VDF difficulty total sum can be fabricated by a Byzantine node, but a lot of chained blocks is difficult to generate because you not only need the time to run VDF you also need to have more than one account with stake if you want to increase the chances of a successful atack.

Summarizing: 

 - Longest chain in number of blocks: Okey.
 - Longest chain in total sum of VDF difficulty: Not good enough, an attacker maybe continue generating a branch where he is the winner in parallel and feed this bogus branch to network newcomers.
 - Average VDF difficult per block: Not good enough, an attacker maybe split its stake into many account to test many branches and see if he can have a branch with low difficulty blocks.




## VDF linear-mining and the Fastest Chip or Winner-takes-all Attack Protection

To avoid one miner being faster than the rest and getting all the rewards the difficulty of the VDF mining must be related to the stake owned by the miner. Also, remember that to allows for network fragmentation we allow any miner to propose blocks at any time, in case the miner with the smallest VDF linear difficulty is down, another miners are allowed to jump in an propose anotherblock. So, we need to introduce an extra protection because, as we allow a second miner to jump in if a first one fails, lets miners with faster CPUs/GPUs to jump always in front of the fair proposer designated by the protocol.

The idea is that there is a fixed temporal quantum and that we introduce an exponent variable multiplier.
That is, on the basis of the time block, the exponent is modified.
Put the initial exponent is 1 and the mult is 2 ^ 1
This means that to cheat you have to be the NSA or a super-player and make a chip more than twice as fast as the rest of the miners.
Then you are NSA winner take all you take all the blocks but the multiplier detects that the average block time was reduced then quickly increases the exponent put it to 2 then it is 2 ^ 2 = 4 and the NSA chip doesn't work anymore.
In practice it increases by 10%, but it is the idea. Put it from 2 ^ 1 to 2 ^ 1.1 but it's the same.
The concept is that as block time quickly controls the exponent then the exponential appearance of faster chips simply enlarges the exponential distance between slot 1 and 2, between the 2 and 3 etc.
The exponential part that is fixed in the hello-vixify is missing.
The issue is that the block time could lie a bit maybe, you decided it took maybe what you should but you mine it and share it before.
The same is difficult, you can't trick the blocktime pq much if you propagate something with timestamp of the future it would be an invalid block assuming clocks with a little bit of syncronia margin
In theory you cannot assume synchrony but in practice you can assume some level of statistical synchrony and time delta.

```
difficulty = slow-moving variable self-regulated by average block time (for example, miners can move this deterministically by 1% each block up or down).
minerStake = current block # of coins stake of the miner address holding the coins
stake = minerStake / totalCoins 
slot = int(round(1/stake))
miner_vrf_seed = vrf_sign( prev_block_hash, miner_private_key ).    # VRF is just a deterministic signature, bijective.
random.set_seed(miner_vrf_seed)
slotRange = [1:slot+1] # the range of possible integer slots for a given miner holding stake on a given address.
slotNumber = random.random_integer_on_range(slotRange) # slotNumber = a deterministic slot number based for address or miner holding stake on a given blocknumber.
vdfSteps = 2 ^ ( difficulty * slotNumber )   # this number is like the average mining time of traditional PoW.
```

Notice that on changes of average block time, the vdf steps for each miner move exponentially. This is the Winner-takes-all Protection. A VDF winner can only used a tecnology advantage for a limited number of blocks. Let's say this winner has a 50% increase in VDF speed and can jump from slot `S` to slot `S-1`. Then if the average block time is reduced the vdf steps grows exponentially for all miners and the winner and every slot is penalized.

For example we show 10 examples of VDF steps for a miner with a stake of 25% (25 from a total of 100 coins):

```
VRF Miner Seed = 59
Mining Slot = 2.0423
Exponential Mining Slot = 3.1427
Slot Translated to VDF Steps = 31427
========================================
VRF Miner Seed = 84
Mining Slot = 3.4570
Exponential Mining Slot = 14.8692
Slot Translated to VDF Steps = 148692
========================================
VRF Miner Seed = 5
Mining Slot = 5.1277
Exponential Mining Slot = 93.1995
Slot Translated to VDF Steps = 931995
========================================
VRF Miner Seed = 46
Mining Slot = 1.2000
Exponential Mining Slot = 1.2457
Slot Translated to VDF Steps = 12457
========================================
VRF Miner Seed = 76
Mining Slot = 3.2316
Exponential Mining Slot = 11.6076
Slot Translated to VDF Steps = 116076
========================================
VRF Miner Seed = 50
Mining Slot = 4.4278
Exponential Mining Slot = 43.1993
Slot Translated to VDF Steps = 431993
========================================
VRF Miner Seed = 47
Mining Slot = 3.0313
Exponential Mining Slot = 9.3149
Slot Translated to VDF Steps = 93149
========================================
VRF Miner Seed = 71
Mining Slot = 3.2542
Exponential Mining Slot = 11.8994
Slot Translated to VDF Steps = 118994
========================================
VRF Miner Seed = 2
Mining Slot = 1.0458
Exponential Mining Slot = 1.0516
Slot Translated to VDF Steps = 10516
========================================
VRF Miner Seed = 47
Mining Slot = 3.0313
Exponential Mining Slot = 9.3149
Slot Translated to VDF Steps = 93149
========================================
```


## Installation

Vixify requires [Python ](https://python.org/) v3+ to run.

Install the dependencies and devDependencies and start the server.

```sh
$ cd bla
$ python install -d
$ bla bla
```

For production environments...

```sh

$ pip install 
$ blabl apython
```

### References

[1] Dan Boneh, Benedikt Bunz, Ben Fisch, "A Survey of Two Verifiable Delay Functions", August 22, 2018,  https://crypto.stanford.edu/~dabo/pubs/papers/VDFsurvey.pdf
[2] Dan Boneh, Joseph Bonneau, Benedikt Bunz, and Ben Fisch, "Verifiable Delay Functions" https://eprint.iacr.org/2018/601.pdf
[3] (Minimal) Go implementation of Algorand, https://github.com/ericderegt/algorand
[4] Implementing Algorand Agreement, https://nickgreenquist.github.io/blog/projects/2019/01/04/algorand.html
[5] Cryptographic Sortition in Blockchains: the importance of VRFs https://medium.com/witnet/cryptographic-sortition-in-blockchains-the-importance-of-vrfs-ad5c20a4e018
[6] What is the difference between signatures and VRF? https://crypto.stackexchange.com/questions/50681/what-is-the-difference-between-signatures-and-vrf
[7] Block Gas Limits vs. Transactional Throughput: A Performance Analysis of the Ubiq Platform https://www.whiteblock.io/wp-content/uploads/2019/07/ubiq-report.pdf
[8] Bobtail: Improved Blockchain Security with Low-Variance Mining https://arxiv.org/pdf/1709.08750.pdf
[9] Announcing our Verifiable Random Function (VRF) Rust library https://medium.com/witnet/announcing-our-verifiable-random-function-vrf-rust-library-2e042c29a4f7
[10] (Similar Approach, but not exactly the same) Nakamoto Consensus with VDF and VRF https://ethresear.ch/t/nakamoto-consensus-with-vdf-and-vrf/5671 
