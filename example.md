Example Bitverse Session
========================

An example of how the world could get bootstrapped into existence, and the scots-proverbs app used.

Let's assume that we've been just writing empty blocks on branch 0 for a while and we've got a balance of 1 million vers in wallet a9j1.

First thing we do is create a bounty: 

    erik bounty: {
      program: 'erik/scots-proverbs',
      branch: '418',
      checks: 1,
      payout: 0.1
      cap: '10/day',
      inputs: [
        {source: 'a9j1', amt: 4000}
      ]
    }


