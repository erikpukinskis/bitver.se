Bitver.se
=========

Trustless key/value store and compute engine.

**WORK IN PROGRESS**

Requirements
------------

For something like this to be successful it must:

* Work even if you are the only person you know on it
* Make it possible to find cool stuff even if you know nothing about the universe

Benefits
--------

* Prepaid permanent data storage
* No trust required
* Scales to infinite size
* Always on, infinitely scaleable software services, prepaid until the end of time
* Unblockable services: if you describe it and fund it, it exists


Basics
------

Imagine a key/value store. To write to the key/value store you send a publish command out to a Bitcoin-like network that mirrors and confirms that data. Unlike Bitcoin, however, instead of a single blockchain, data is stored in a tree of chains (see Blocktree below).

Requests for data would also be broadcast to the network. These would simply be a set of keys which would request source inputs with transformations applied. For example:

* ("erik/bitver.se/readme.md") would simply be a request for this document.
* ("photo filters/black and white")("erik/photos/3j903") would be a request for a photo I took with a black and white filter applied to it.

Lets say I put out a request for that photo with that filter. Someone who has both the photo and the filter on their machine would then broadcast an offer to do the computation. Or they might have one file and they put out their own request for the other file, and then offer to do the computation. Regardless, someone offers to do it.

The network then calculates "prices" for the transaction at various levels of reliability. After all, the person who offered could just send me the photo with no filter applied, or send me nothing! What would be generated by the network is a hash of reliabilities, hops and estimated speeds to prices:

    {
      [r 0.5, 0, 400ms] => 0.1
      [r 0.8, 1, 2s] => 0.7
      [r 0.99, 2, 1.5s] => 1.3
      [r 0.99999, 5, 20s] => 11.04
    }

If I like the looks of any of these, I would then put out a bounty for one of those reliabilities, funded by my wallet. 

The offerer then accepts the bounty. 

If it's a non-zero numbre of hops, the network decides on a set of nodes that can act as intermediaries. So let's say as an example:

1. A requests
2. B offers
3. They settle on [r 0.8, 1, 2s] => 0.7
4. C, D, E, and F are chosen (possibly each with a bouty of their own?). This is called a "Relay Lineup"
5. C asks for any source files they need from B (which are cryptographically signed)
6. B and C both compute the filter. 
7. B signs and sends their signature to C.
8. C signs B's envelope AND the solution and sends both to A.
9. A signs C's envelope and sends it to the root blockchain.
10. The bounty is split between A, B, and C.

The network would collaborate to maintain a ledger of how reliable nodes were at various tasks, which would determine the prices. This job would be mining.


Keyspaces
---------

Keyspaces can be purchased, with a validation script attached. This script might only allow writes which...

* are signed with an indentity which has sent a certain amount of money to a specific address.
* which conform to the laws of physics in a game world
* which are 140 characters or less and put in sub-keyspaces (a la Twitter handles)

It's really up to the purchaser. They could just throw an arbitrary javascript function into access.js at the root of their keyspace perhaps.

Mining
------

Mining would happen through a variety of transactions:

* Relaying/verifying computations/data as described above
* Generating a Relay Lineup, with proof of work
* Generating transaction prices (with proof of work?)

All of these would be written into the blockchain, and verified by every host on that chain. The actual computations (step 6 above) would not be.


Bounty Prices
-------------

In general, the goals for the reward system are to:

* Incentivize participation. It should make financial sense for people to make computers available to do work for the network.
* Disincentivize fraudulent activity. 

There's a couple ways this could go. The price of a bounty could be just based on the "cost" of a transaction:

* Number of chained parties
* Trustedness of chained parties

Or, it could be a simple auction-based system, where a party makes a series of offers until one is accepted.

Or, another possibility that's appealing to me is that you can actually buy tokens that are redeemable for specific services. So, I could purchase tokens and assign them for distributing, say, my album. These could potentially have built-in appreciation so that I could purchase in advance, say, 100 downloads a day. Then anyone could download those for free, literally for the rest of time.

That, of course, opens up a some trouble. Namely: as a provider of that data, the incentive is for you to request those files yourself, and send them to yourself.

So, in order to head that off, perhaps these incentives should be disbursed to people, not just for distributing the data, but for *being available* to distribute it.  So, people sign up for the pool, and whether or not the data is requested, the funds are disbursed to the pool.

Of course, if you say you're available and you fail to deliver the data then you would be demoted from the pool, or dropped to a lower priced tier.


Blocktree
---------

In order to keep the blockchain size manageable, while still allowing large volumes of data and transactions to be circulated, the blockchain is actually a binary tree of blockchains which represent an enormous address space. 

Each individual chain would be some size... say 1 GB.

No client stores the entire blocktree, but instead they store some set of branches.

When the transaction prices are generated, they take into account the depth in the tree. Blockchains are only available at certain intervals. The root blockchain might become available for verification once every ms, such that it got massively verified and was all but computationally impossible to rewrite.

But futher up the tree transactions are cheaper and the blockchains are available less often, so they have less verifications.

At the very top of the tree transactions would be totally unverified "take it or leave it" data.

At the bottom of the tree, anything a certain distance from the root would be for all intents and purposes immutable... information written in the stars themselves.

Questions:

* It's not clear how the branches should be split up. Perhaps they can just branch arbitrarily and we let market forces prune them. 
* Or maybe there's a way to allow for arbitrary precision and arbitrary branch sizes. So that the tree becomes a kind of a continuous data triangle.


Anonymity
---------

Ideally you could send data to another person and pay for it without the fact that you communicated with that person being discoverable by looking at the blockchain.

To do this, creating a block may have to require each of the working parties presenting signed work, a third party then batching that work up into a block and divying out rewards, then each of the working parties signing off on the block (agreeing that the rewards are fairly calculated) and then (and only then) is the block valid and ready to be broadcast and written into the blockchain.

Essentially, each block would be a tumbler.

Possible implemenation technologies
-----------------------------------

* [WebRTC](http://www.webrtc.org/)

Related
-------

* http://www.i2p2.de/
* https://gnunet.org