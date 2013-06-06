Bitver.se
=========

**WORK IN PROGRESS**

Key/value store.

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
4. C, D, E, and F are chosen (possibly each with a bouty of their own?)
5. C asks for any source files they need from B (which are cryptographically signed)
6. B and C both compute the filter. 
7. B signs and sends their signature to C.
8. C signs B's envelope AND the solution and sends both to A.
9. A signs C's envelope and sends it to the root blockchain.
10. The bounty is split between A, B, and C.

The network would collaborate to maintain a ledger of how reliable nodes were at various tasks, which would determine the prices. This job would be mining.
