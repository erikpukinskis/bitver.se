Bitver.se
=========

Trustless key/value store and compute engine.

**WORK IN PROGRESS**

Blockchain is a tree. Mining incentives are such that branches closer to the root are more heavily mirrored and thus more costly to spoof. Anything below a certain threshold distance to the root are considered "permanent".

Requests are document chains. You might request a PNG and then request a filter. 

That request goes out to the network, where free agents are looking for work. 

Someone who has both of those files on their machine offers to do the computation. Or they might have one file and they put out their own request for the other file, and then offer to do the computation. Regardless, someone offers to do it.

The network then calculates "prices" for the transaction at various trustability. This is a hash of reliabilities, hops and estimated speeds to prices:

    {
      [r 0.5, 0, 400ms] => 0.1
      [r 0.8, 1, 2s] => 0.7
      [r 0.99, 2, 1.5s] => 1.3
      [r 0.99999, 5, 20s] => 11.04
    }

The requester then puts out a bounty for one of those reliabilities.

And the offerer takes it on.

If it's a non-zero numbre of hops, the network then decides on a set of nodes that can act as intermediaries. So let's say as an example:

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

At the bottom of the tree would be immutable data, information written in the stars themselves.