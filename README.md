Bitver.se
=========

Blockchain stores key/value/reference to governance code

Create spaces:

    # DIM
    # TYPE
    # DATUM
    # string
    # numeric
    # function
    # struct

    {
      type: dimension,
      name: "root"
    }
    {type: dimension, name: "x", in: "root", type: numeric}
    {type: dimension, name: "y", in: "root", type: numeric}
    {type: dimension, name: "z", in: "root", type: numeric}
    {type: dimension, name: "t", in: "root", type: numeric}
    {type: dimension, name: "color" in "root", type: "hexColor"}
    {type: dimension, name: "width", in: "root", type: numeric}
    {
      in: "root",
      type: "struct",
      name: "point"
      of: ["x", "y", "z", "color", "root/width", "governance"] # if you don't add a namespace it 
                                                              # just assumes you mean the same one as the struct
    }
    {
      in: "root",
      type: "function",
      name: "permanent"
      function: function() {
        return false;
      }
    }
    {
      in: "root",
      type: "point",
      x: 0,
      y: ,
      z: -8
      t: 0,
      width: 4,
      color: "#FFCC00"
      changeGovernor: "permanent"
    }
    {type: dimension, name: "height", in: "root", type: numeric}
    {type: dimension, name: "depth", in: "root", type: numeric}
    {
      in: "root",
      type: interface,
      name: "volume"
      any: ["x", "y", "z", "length", "width", "height"]
    }
    {
      in "root",
      type: "struct",
      name: "box",
      of: []
    }
    {
      in: "root",
      name: "main",
      governor: {
        interface: "volume",
        function: function(other) {
          this.type ~= ["box", "link"] &&
          this.x%10 == this.y%10 == this.z%d == 0 &&
          this.length == this.width == this.height == 10
          volumeAt(x,y,z) = nil
        }
    }
    {
      in: "root/main",
      type: "universe",
      name: "Someone's backyard",
      x: 0,
      y: 0,
      z: 0,
      t: 0,
      length: 10,
      width: 10,
      height: 10,
    }

Let's inject a little prose here. Each hash is a datum. Clients sign a datum by creating a transaction:


    TRANSACTION {
      data: [{
        type: dimension,
        name: "turtle"
      }],
      sources: {
        "a3j90j29a2d" => 43929,
        "ajd90ja2ja" => 219
      }
      permanance: 100,
      signature: "d20d9qjdqj029d",
    }

Data is the stuff to add or change in the world. This document begins with 16 examples of data. The Bitverse blockchain is just like the Bitcoin blockchain, except instead of confirming transfers of wealth, it confirms the validity of information about the Bitverse.

To store a structure in the Bitverse blockchain for other people to see and interact with, you simply sign your data and publish it. With infinite free storage space and bandwidth, this would be enough to create a virtual world. Anyone could upload whatever they want and it would get cryptographically signed into the blockchain.

But since space and bandwidth are scarce, we need a way to decide what to store where. Bitverse solves this problem by requiring payment to write data. Bitverse miners confirm transactions by mining "creative currency". This currency can then be spent writing data into the blockchain.

In addition, each transaction specifies a target "permanence". Each level of permanence, from 1 to 100 has it's own difficulty associated with it, which adjust independently. Miners choose a permanence to work on, and search for nonces with the required level of difficulty. When they find a nonce, they can write a block and verify a set of transactions at that specific permanence level. There are effecively 100 different blockchains, one for each permanence.

Lastly, a huge difference between the Bitcoin blockchain and the Bitverse blockchain is that in Bitcoin transactions must be valid to enter the blockchain. In Bitverse, no transaction is ever truly confirmed. Instead, one can merely estimate the probability of a transaction's validity based on it's set of confirmations. 

HOW DOES THIS WORK?

HOW ARE TRANSACTIONS VETTED?

HOW DOES THE DIFFICULTY PLAY IN TO IT?

IS IT JUST A "RICH PEOPLE GET TO BREAK THE RULES" THING?

    BLOCK {
      confirm: ["32dj90jd9023jd", "3q0jd03ad930"]
      deny: ["92jd0ja9dj09ajd", "29j9aj09jd290a"]
    }

turtle dimension includes a link to it's original self, of type fork:

    {
      in: "turtle",
      type: "link/fork",
      x:0, y:0, z:0, t:0,
      w: 10, h: 10,
      to: "turtle?t=0"
    }
    It's turtles all the way down.


