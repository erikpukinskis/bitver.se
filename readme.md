Bitver.se
=========

**is a distributed key/value store and compute engine.**

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
      root: 113,
      checks: 1,
      count: 100
      expiration: 1440
      inputs: [
        {source: '91vv', amt: 79}
      ],      
    }
    
A bouty just means you're putting up some money for the network to do something. The bounty has several required properties. 

**program** is a reference to what you want people to do. In this you're asking for the `scots-proverbs` program. `21` is just a reference to where the program is stored. That's discussed in the "Blocktree" chapter later on.

**root** is a reference to where in the Blocktree the bounty can be fulfilled.

**checks** is the number of times the results of the program need to be verified.

**count** is how many times you're paying for the program to run.

**expiration** is the number of minutes you want the bounty to run for.

**inputs** are the addresses for the money ('vers') you're using to pay for the bounty and the amount to take from each one.

When you want to pay for something to happen, you just sign one of these, and broadcast it to the network.

The Blocktree
-------------

Every piece of information in the Bitverse lives in what looks like the roots of a tree. The very top of the root system, the big central trunk, is Root 1. The branches below that are roots 2 and 3, and below those are 4, 5, 6, and 7.

Having all of these different roots is absolutely critical because they allow you to have a choice between expensive+durable information and cheap+transient information. 

At the top of the root system, in nodes with low numbers, any bounties that are funded are, for all intents and purposes, *permanent*. If you publish your poem in root 1, it's extraordinarily unlikely that anyone could ever prevent that distribution from happening. We'll describe how this works later, but someone would have to take control of more than half of the entire network in order to erase it.

The deeper you go into the Bitverse, things become less and less durable. If you publish your poem in root 10,000, it will probably last a day or so, but it will almost certainly disappear within the week.

Bounty Confirmation
-------------------

When a bounty is signed, it needs to be written into the blocktree. Bounties all go into Root 1, which is basically the "accounting" root for Bitverse. It's very similar to the Bitcoin blockchain, except miners do a little extra work to verify that the bounties are valid.

<when>When someone puts your bounty into a block</when>, it's considered active and the money attached is considered spent. <when>If the balance of your wallet (91vv) was 80 verscoins</when>, <when>and you broadcast the above bounty</when>, <then>it does down to 1.</then>.

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
      root: 5,
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

In order for this to work, these have to be limited somehow, so you can't just buy total coverage of the bitmask. OR maybe make the bitmask algorithm something that can't be predicted so even with an army of drone identities you still can only control your tiny fraction of the Bitverse enthalpy. And to commit fraud against even modestly redundant roots would require half of the network for a long time. (I think? hope?)


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
        {reliability: 0.9, price: 30, root: 113},
        {reliability: 0.95, price: 45, root: 90}
    ]

That would allow you to see how much money you'd need for a certain reliability. You could also leave out the reliability and provide a price range and find out what kind of reliability you can get for what you want to pay. I suspect that will be a common way to monetize "browsing". You just volunteer that every 10 seconds you'll pay for whatever you're looking at as long as it's under a certain amount... say, 1 ver.

Anonymity
---------

Ideally you could send data to another person and pay for it without the fact that you communicated with that person being discoverable by looking at the blockchain.

To do this, creating a block may have to require each of the working parties presenting signed work, a third party then batching that work up into a block and divying out rewards, then each of the working parties signing off on the block (agreeing that the rewards are fairly calculated) and then (and only then) is the block valid and ready to be broadcast and written into the blockchain.

Essentially, each block would be a tumbler.

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

* http://www.i2p2.de/
* https://gnunet.org
