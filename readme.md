Bitver.se
=========

**is a distributed trust-free application hosting protocol.**

Conventions
-----------

In this document most transactions will be of a form like this:

    erik request: {
        some: 1,
        keys: 2,
        ...
    } (signature b1a1)

The name (erik) is just shorthand for the public key that is signing the transaction, which acts as the identifier for the identity that's issuing the transaction. It's followed by the command (request) and then a JSON hash of the parameters for the request. It's followed by the signature for the transaction, which may be used to reference it later.

Sometimes the name and/or signature is left out if they're not needed, but in practice every transaction would be signed.

Intro
-----

Let's say you want to provide a service that will give people a Scottish proverb. You compile a database of 10,000 proverbs and write a little program that gives you a proverb if you give it a number between 1 and 10,000:

    function(number) {
        return [
            "Ye maun tak the will for the deed",
            ...
            "A close mouth catches nae flees"
        ][number];
    }

You save that into the Bitverse with at `21/scots-proverbs`. But before we get into how that works, we have to talk about bounties.

Bounties
--------

    bounty: {
      program: '21/scots-proverbs',
      branch: 113,
      checks: 1,
      cap: '100/day'
      expiration: 1440
      payout: 0.01
      inputs: [
        {source: '91vv', amt: 79}
      ],      
    } (signature gg98)
    
A bouty just means you're putting up some money for the network to do something. The bounty has several required properties. 

**program** is a reference to what you want people to do. In this you're asking for the `scots-proverbs` program. `21` is just a reference to where the program is stored. That's discussed in the "Blocktree" chapter later on. The root of the program can also be an identity, like `erik/scots-proverbs`.

**branch** is a reference to where in the Blocktree the bounty can be fulfilled.

**checks** is the number of times the results of the program need to be verified.

**cap** is how many times you're paying for the program to run in a given time period.

**payout** is the payout per transaction. So the bounty will be spent at a rate of `payout` &times; `cap` per day.

**expiration** is the number of minutes you want the bounty to run for.

**inputs** are the addresses for the money ('vers') you're using to pay for the bounty and the amount to take from each one.

When you want to pay for something to happen, you just sign one of these, and broadcast it to the network.

The Blocktree
-------------

*Why do we need a blocktree instead of just a blockchain?*

Well, the idea is that every single user action is going into the tree, right? Every tweet, every favorite, every movement in a game world. And the way the blockchain works, every miner has to store the whole thing. It's just too much data.

Even if we're not storing any real data, even if we're just storing metadata, it's still way too much. And in practice, much of the metadata will be just as big as the data. For stuff like movement, or tweets, or favorites, the data just isn't that big.

So the idea is basically to shard it. A miner could find it lucrative just to host likes of photos in the keyspace from a8b-a8f, and that could be enough data to warrant an entire blockchain.

One trouble with this is that because everything is sharded, it becomes trivially easy to brute force an attack on one of these "blockshards".

This is somewhat solved by the fraud reporting. In principle all data is deterministic: you start with some set of inputs, and there are no side-effects; the inputs determine the outputs. So even if someone did brute force the data in one of these shards, you could go back and look at the computation and see if it was correct or not.  

But if the difficulty is really low on these chains, you could also spoof the fraud reporting, so I think the reason for the blocktree is so that you can "escalate" truthmaking.

So, let's say someone dumps a bunch of machines into a little blockshard containing a room in a game world and starts giving themselves lots of free stuff.

Another miner could then escalate to the parent blockshard, which would be more durable, which I guess just means there are more checks required and the bounties are larger, which would attract more miners and make the difficulty higher. Really the important thing is that the difficulty is higher.

Someone in that tree would then grab the relevant hashed data, verify the checksums, do the relevant calculations, and issue a ruling on whether it was valid or not. That would get broadcast and once it was included into a block, everyone would do the verification.

It's sort of like appeals in the US judicial system.

If there was actual fraud, the person who committed it could lose their reputation, which would hurt their ability to mine profitably. A fee could be exacted. If no fraud was found, the person reporting the fraud would have to pay.

Then the blockchain in the lower shard would have to get rolled back somehow. Or the transactions would just be marked as invalid in a new block perhaps. More on this in *Fraud Reporting*.

So in order to mine you have to store not only the relevant shard, but all of the shards above it. For that reason the tree probably can't be crazy deep? And periodically in order to verify fraud reports, you'd have to grab data from a lower shard? Although maybe that wouldn't even have to be verified... Maybe when a fraud investigation is done the blockchain just stores the signatures of the miners who signed off on it and the resolution, not the actual issues and data and such.

---

Every piece of information in the Bitverse lives in what looks like the roots of a tree. The very top of the root system, the big central trunk, is Branch 1. The branches below that are branches 2 and 3, and below those are 4, 5, 6, and 7.

Having all of these different branches is absolutely critical because they allow you to have a choice between expensive+durable information and cheap+transient information. 

At the top of the root system, in branches with low numbers, any bounties that are funded are, for all intents and purposes, *permanent*. If you publish your poem in Branch 1, it's extraordinarily unlikely that anyone could ever prevent that distribution from happening. We'll describe how this works later, but someone would have to take control of more than half of the entire network in order to erase it.

The deeper you go into the Bitverse, things become less and less durable. If you publish your poem in Branch 10,000, it will probably last a day or so, but it will almost certainly disappear within the week.

Bounty Confirmation
-------------------

When a bounty is signed, it needs to be written into the blocktree. Bounties all go into Branch 1, which is basically the "accounting" branch for Bitverse. It's very similar to the Bitcoin blockchain, except miners do a little extra work to verify that the bounties are valid.

<when>When someone puts your bounty into a block</when>, it's considered active and the money attached is considered spent. <when>If the balance of your wallet (91vv) was 80 vers</when>, <when>and you broadcast the above bounty</when>, <then>it does down to 1.</then>.

The block stores the bounty more or less as-is, but does add one additional property:

    bounty {
      ...
      fulfillerMask: "br",
      ...
    }

The fulfillerMask is a mask that determines who can serve as a "check" for fulfilling the bounty as we'll see in the next section.

Bounty Fulfillment
------------------

A bounty with zero checks can be fulfilled without any double-checking. It's the absolute lowest of the low in terms of reliability. If you put out a zero-check bounty for distributing your scots proverbs, anyone could just offer to fulfill your bounty, but instead of sending the actual proverbs they could just send random words, and they'd still get a share of bounty[1].

<small>[1] This is discouraged, since they could potentially take a hit to their reputation by doing this, making it harder for them to earn future bounties. But as the publisher or a recipient, you would have no way to know whether you actually got a proper proverb</small>

The way Bitverse solves this problem is by checking results through a series of "checks". Our bounty specifies two checks, so in order to get a share of the bounty, the following would need to happen:

First, someone broadcasts a request for proverb 7777 to the branch that has the bounty (113):

    request: {
      program: '21/scots-proverbs',
      checks: 1,
      
      proverb: 7777
    } (signature dj91)

The only required properties for a request are the program and the number of checks, but for `scots-proverbs` you also need to provide the proverb number.

Next, a fulfiller ('bradley') who is mining on 113 obtains the program, runs it, generating a proverb, signs the proverb and broadcasts it to the network:

    result: {
        request: 'dj91',
        hashedData: 'o00r'
        by: 'bradley'
    } (signature f09a)

Note that only a fulfiller whose identifier ('bradley') matches the lineup mask is allowed to do this.

Then, another miner who matches the mask ('brenda') does the same thing, grabbing the proverbs and looking up the correct one. If it matches the one from bradley, they take bradley's signature, bundle it with the result, and send it to the requester:

    result {
        request: 'dj91',
        hashedData: 'o00r',
        data: 'Our ain reek's better than ither folk's fire',
        checks: [
            {'bradley': 'f09a'}
        ]
    } (signature fk01)
    
The requester then broadcasts a receipt to the rest of 113:

    receipt {
        request: 'dj91',
        checks: [
            {bradley: 'f09a'},
            {brenda: 'fk01'}
        ]
    }

I'm not sure how the signatures for all this would work, but basically the receipt would have to be signed in such a way that you could tell that bradley and brenda both signed the same data and the requester signed those signatures.

At that point, the receipt can be incorporated into the blockchain and the bounty is distributed between bradley, brenda, and the requester.

There are a number of important things happening here.

#### Tracking only receipts

This system would never work if all of the data getting process *had* to be broadcast to the network and stored in the blocktree. The blocktree, even at the lowest levels, is at least somewhat permanent, and there's just not enough storage in the world to store everyone's transient data. So the results of the program are passed to the recipient and the only thing stored in these blockchains is the receipt.

#### Fulfiller masking

Bitverse identities can be created unlimitedly[2], and so without the fulfiller mask, anyone could just generate three identities, have one request the proverb, a second one sign an empty result, a third one "verify" that empty result, and then have the first one generate the receipt and then all share the bounty, *without ever having even downloaded the program, let alone run it*.

If that were the case, bounties would just be immediately sucked dry by people pretending to fulfill them for themselves.

What the fulfiller mask does is specific a certain set of identities which can fulfill requests, making it statistically improbable (more and more improbable as the number of checks goes up) that the chain of signatures is all completely faked.

[2] 


Bounty Payout
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

Clocking In
-----------

In order to disincentivize frivolous moving of data around just to earn out bounties, miners can "clock in" on a bounty and if no one makes any requests, the bounty will just be split between all of the clocked in miners:

    tim clockin {
        bounty: gg98,
        duration: 60,
    } (signature 01kr)

Miners can, of course, claim they are available for bounties they have no intention of ever filling, but they could get caught with their pants down during a spot check.

I'm not sure exactly how that would work, but perhaps a byproduct of the blockchain mining would be various bounties for spotchecks are issued:

    spotcheck {
        clockin: 01kr,
        fulfiller: 'g0',
    }

At that point, someone whose public key was in the fulfiller mask could issue a request to tim, the normal way. If they got a proper response from tim they could sign it and they'd get a nice bonus.

If no one claimed the spotcheck bonus, tim's reputation would take a hit. So it's risky claiming to be able to fulfill bounties that you can't actually fill. 

Fraud Reporting
----------------

If you got lucky and you controlled an identity that matched the fulfiller mask for a request you wanted to defraud, and you try to pass along fake data, what would happen?

If the number of checks is low, it's possible you could get away with it. That's the downside of cheap/low-check bounties.

But if another fulfiller saw your result broadcasted on the network and noticed that your result was wrong, they could then publish a fraud report to the network:

    fraud {
        result: 'f09a'
    } (signature ka02)

Other fulfillers could then verify the fraud report and add their own signatures. Once there were enough signatures the result would be considered fraudulent and it would go into the blockchain, where it would be verified by everyone:

    fraud {
        result: 'f09a',
        signatures: ['ka02', 'gj01', 'fk01']
    }

Identities caught committing fraud would be knocked down to a lower level of the blocktree.

try to pass along fake data, and and someone inside 

*Open issues*

* How to resolve data consistency issues that arise when fraud 

Expiration
----------

Just as fraud reports discourage people from sending bad information to try to get paid, expiration discourages people from sending good information to themselves to try to get paid.

The way it works is that bounties are set to expire after a certain amount of time. If a request is made and fulfilled, a certain portion of the bounty is paid out and a part of it is returned to the identity that sponsored the bounty. 

But if the bounty expires before a request is made, *the entire bounty* is split amongst everyone who was prepared to fulfill the bounty. So if you want to fulfill the bounty, you broadcast your availability to the network and much of the time you can earn a payout without actually doing anything:

    availability: {
        program: '21/scots-proverbs',
    } (signature b0ba)

Of course, you could bluff and say you're prepared to fulfill the bounty without actually having any intention to do that. In order to prevent that, other miners could essentially call your bluff by requesting the program and then claiming a reward if you failed to broadcast a result:
    
    request: {
      program: '21/scots-proverbs',
      checks: 1,
      fulfiller: 'b0ba'
      
      proverb: 7777
    } (signature dj91)

then some time later:

    fraud {
        type: 'non-availability',
        availability: 'b0ba',
        request: 'dj91'
    }
    
which would result in the fake fulfiller being demoted in the payout scheme.


Endowments
-----------

The Bitverse also has a kind of "financial API" which lets you give your money a life of it's own. For example you endow a "ver fountain" that gives a thousandth of the endowment to anyone who puts their wallet address into a document:

    42/fountain-payout:
    
    function() {
        var addresses = bitverse.readlines('12/fountain');
        var payout = bitverse.balance() / 1000 / addresses.length;
        
        var commands = addresses.map(function(address) {
            return {
                cmd: 'pay',
                to: address,
                amt: payout
            };
        };
        
        commands.push({
            cmd: 'write',
            location: '12/fountain',
            content: null
        });
        
        return commands;
    }

Then if you created a bounty that requrired a lot of checks[3]:

    bounty: {
      program: '42/fountain-payout',
      branch: 5,
      checks: 10,
      count: 100
      expiration: 1440
      inputs: [
        {source: '91vv', amt: 79}
      ],
      payout: 9,
    }

Note this bounty has a `payout` property, which specifies how much of the inputs should go to pay out bounties.  The rest of the inputs are available *to the script*. Then, if someone signs the output of the script and it has some pay commands in it, the money will be transferred:

    receipt: {
        request: ...,
        checks: ...,
        commands: [
            {
                cmd: 'pay',
                from: 'dj191',
                to: 'k100',
                amt: 4
            }
        ]
        signatures: [commands signed by all the checkers],
    }

This is of course quite dangerous if you put your bounty in a less reliable branch of the blocktree. If you're only requiring a single confirmation on this bounty then someone could probably fairly easily nab your vers fraudulently. But if you require sufficient checks it should be, for all intents and purposes, un-spoof-able.

<small>[3] You'd also need a bounty to pay for all of the times 42/fountain-payout has to read from and write to 12/fountain. But that's left as an exercise for the reader.</small>

Micropayments
-------------

One could also just pay to seed a file to the network, and then let their audience pay for their own distribution. People could even pre-orderers:

    bounty {
        program: '59/eriks-zookeeper-lovemonster',
        checks: 10,
        inputs: [
            {source: '0f00', amt: 1092}
        ],
        payout: 92,
        
        audience: 'me',
    } (signature jj68)
    
    request {
        program: '59/eriks-zookeeper-lovemonster',
        checks: 10
    } (signed by the same private key as the bounty)
    
Note there is no expiration on the bounty. Then when the album was ready, the producer could just seed it to some of those pre-orderers: 

    result {
        request: ...,
        data: 'this is where the actual album data would go, 
               but probably tacked on as a binary payload.'
        by: 'eriks'
    } (signature 1dk0)

Then the pre-orderer can fulfill *other* pre-orderers, and those pre-orderers can fulfill *other* pre-orderers and the rest of the distribution happens within the network.

But you will also notice the bounty only releases 92 of the 1092 vers provided. What this allows the program to do is pay itself some money. Effectively this let's the creator build in a price:

    59/eriks-zookeeper-lovemonster:
    
    function() {
    
        return {
            payload: [somehow the data payload is attached],
            commands: [{
                cmd: 'pay',
                to: 'j2ak',
                amt: bitverse.balance()
            }]
        }
    }

Fulfiller mask difficulty
-------------------------

Another possible way to achieve better durability without forcing the network to do lots of double-checks is by changing the specificity of the fulfiller mask. 

This could happen by varying the size of the fulfiller pool and the length of a mask-match that would be required. So if you didn't care so much about the reliability, you could just require the first letter in the mask match the fulfiller ID.  That would give an attacker a 1 in 36 chance of being able to fulfill a request fraudulently (assuming the masks are base 36).

But if you required the first 3 letters of the mask to match the fulfiller, then that brings the probability a given attacker could defraud you down to one in 46 thousand.

In order for this to work, these have to be limited somehow, so you can't just buy total coverage of the bitmask. OR maybe make the bitmask algorithm something that can't be predicted so even with an army of drone identities you still can only control your tiny fraction of the Bitverse enthalpy. And to commit fraud against even modestly redundant branches would require half of the network for a long time. (I think? hope?)


Estimating how much you'll need to pay
-------------------------------------

A useful program would be one that estimates how much jobs should cost:

You could send an unsigned bounty to it, and it would send you back a list of probabilities:

    request: {
        program: '2/estimate',
        ...
        reliability: 0.85-0.95,
        bounty: {
            program: '21/scots-proverbs',
            checks: 4,
            count: 1,
            ...
        }
    }

The output of that program would be something like this:

    reliabilities: [
        {reliability: 0.9, price: 30, branch: 113},
        {reliability: 0.95, price: 45, branch: 90}
    ]

That would allow you to see how much money you'd need for a certain reliability. You could also leave out the reliability and provide a price range and find out what kind of reliability you can get for what you want to pay. I suspect that will be a common way to monetize "browsing". You just volunteer that every 10 seconds you'll pay for whatever you're looking at as long as it's under a certain amount... say, 1 ver.

Game World Time
---------------

There could be timing discrepancies, but not if you make everything truly deterministic. In a game world, for example, I could place a bomb not just in a location, but in a moment in time.  Other people could also try to put something else in that same spot in that same moment, or could try to put it *before*, but that would all get resolved in the blockchain. Whatever goes into a block first, wins.

In that case, the miner/servers aren't so much running a simulation with a ticking clock, but instead are listening for events which could be happening at *any* time in the history of the game world, and deciding whether they could have happened. It's a 4d game world, and players are free to act whenever they like.

Of course it might be practical to also allow miners to declare moments to have "passed" so the game world is just randomly changing because they went back in time and did something different. But that would be up to the game designer. And at least within a small window of the present, the world would have to be treated as 4d.

Anonymity
---------

Ideally you could send data to another person and pay for it without the fact that you communicated with that person being discoverable by looking at the blockchain.

To do this, creating a block may have to require each of the working parties presenting signed work, a third party then batching that work up into a block and divying out rewards, then each of the working parties signing off on the block (agreeing that the rewards are fairly calculated) and then (and only then) is the block valid and ready to be broadcast and written into the blockchain.

Essentially, each block would be a tumbler.

Random other stuff
------------------

Someone could DOS your storage by just requesting a file to run out the bounty before anyone else could. It would cost them bandwidth, but that's it. I guess you could always just pay for the data if you wanted it.


***
***
***

Misc Older stuff
===========

A general principle for Bitver.se is that it provides vastness through infinite divisibility *not* expandability. The sensation of space comes from seeing a universe within a universe.


Benefits
--------

* Prepaid permanent data storage
* No trust required
* Scales to infinite size
* Always on, infinitely scaleable software services, prepaid until the end of time
* Unblockable services: if you describe it and fund it, it exists

Keyspaces
---------

Keyspaces can be purchased, with a validation script attached. This script might only allow writes which...

* are signed with an indentity which has sent a certain amount of money to a specific address.
* which conform to the laws of physics in a game world
* which are 140 characters or less and put in sub-keyspaces (a la Twitter handles)

It's really up to the purchaser. They could just throw an arbitrary javascript function into access.js at the root of their keyspace perhaps.

Questions
---------

* It's not clear how the branches should be split up. Perhaps they can just branch arbitrarily and we let market forces prune them. 
* Or maybe there's a way to allow for arbitrary precision and arbitrary branch sizes. So that the tree becomes a kind of a continuous data triangle.

Possible implemenation technologies
-----------------------------------

* [WebRTC](http://www.webrtc.org/)

Related
-------

* https://gnunet.org
* http://en.wikipedia.org/wiki/Kademlia
* https://bitmessage.org/wiki/Main_Page

* http://www.i2p2.de
* http://www.tixati.com/
* http://sipdht.sourceforge.net/
* http://jinroh.github.io/kadoh/
* https://github.com/isaaczafuta/pydht
* https://github.com/AArnott/IronPigeon
* https://github.com/irungentoo/ProjectTox-Core
