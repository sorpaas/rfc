## Abstract

This ECIP describes the details of delaying Difficulty Bomb growth for one year. It’s suggested to change Difficulty Bomb formula to stop growing for period of time from January 2017 to December 2017, which is around blocks 3,000,000 and 5,000,000 accordingly.

## Motivation

For the next phase of Ethereum Classic it’s planned to make a set of updates that could possible break backward compatibility and make serious changes to Ethereum Classic economic model. It includes changes to Monetary Policy, PoW/PoS or Hybrid, Replay Attack, DAG growths and Difficulty Bomb itself. All of these topics are important and must be solved as soon as possible.

Such changes to the protocol can cause another fork, and it’s very dangerous to introduce just part of protocol changes at this moment. It’s very likely that after the Difficulty Bomb will be diffused it will be near impossible to make any other changes to the protocol. That is one of the main reasons why Difficulty Bomb was introduced initially.

To reduce risk of a potential fork during protocol upgrades and to keep development resources focused on working on the next phase of the Ethereum Classic, it seems reasonable to implement Difficulty Bomb diffusion at the very last step in the series of protocol upgrades.

Most of this upgrades to the protocol requires significant time for finding optimal model, implementation and testing. It’s too reckless to make this changes without proper validation. With optimistic estimations it requires at least 6 months, which is not giving us enough time before Difficulty Explosion moment. Also consider the fact that this is an estimation for the ideal situation.

One of the most feasible solutions at this moment will be delaying difficulty growth to the moment when all other changes will be ready to use.

Delaying the Difficulty Bomb is a trivial change to the protocol. It keeps all the logic on place, just adds a pause in X blocks to stop difficulty growing for a some period of time. If we choose 2,000,000 blocks, it will a pause it for about a year, which will give us time to implement and test all protocol upgrades for the next version.

## Specification

Current formula for a minimal difficulty is

````
block_diff = parent_diff
     + parent_diff / 2048 * max(1 - (block_timestamp - parent_timestamp) / 10, -99)
     + int(2**((block.number / 100000) - 2))
````

The most important part related to Difficulty Bomb is:

````
2**((block.number / 100000) - 2)
````

It’s a lower bound for minimal difficulty, which grows exponentially with next step every 100,000 blocks. `-2` is added to start growth only after the block 200,000

With this formula minimal difficulty is:
* Block 1 == 2**(-2) == 0
* Block 100,000 == 2**(-1) == 0
* Block 200,000 == 2**0 == 1
* Block 300,000 == 2**1 == 2
* Block 1,920,000 == 2**17 == 131,072
* Block 2,000,000 == 2**18 == 262,144
* Block 2,500,000 == 2**23 == 8,388,608
* Block 3,000,000 == 2**28 == 268,435,456
* Block 4,000,000 == 2**38 == 274,877,906,944 or 274 GH
* Block 5,000,000 == 2**48 == 281 TH
* Block 5,200,000 == 2**50 == 1126 TH
* Block 6,000,000 == 2**58 == 288230 TH

Current difficulty used for mining is 8 TH. For a comparison, difficulty in Forked ETH Blockchain is 71 TH.

To pause difficulty growth `(block.number / 100000) - 2` must be fixed to a concrete value for the range 3,000,000 to 5,000,000, then continue growing.

Instead of value `2` it’s proposed to use `n` with a dynamic formula as following:

Constants:
````
pause_block = 3000000 //15 Jan 2017
cont_block = 5000000 //15 Dec 2017
delay = (cont_block - pause_block) / 100000 //20
fixed_diff = (pause_block / 100000) - 2 //28
````

Final formula:
````
if (block.number < pause_block) {
    explosion = (block.number / 100000) - 2    
} else if (block.number < cont_block) {
    explosion = fixed_diff
} else { // block.number >= cont_block    
    explosion = (block.number / 100000) - delay - 2
}

block_diff = parent_diff
      + parent_diff / 2048 * max(1 - (block_timestamp - parent_timestamp) / 10, -99)
      + int(2**explosion)
````

With this changes min difficulty value will become:
* Block 2,500,000 == 2**23 == 8,388,608
* Block 3,000,000 == 2**28 == 268,435,456
* Block 4,000,000 == 2**28 == 268,435,456
* Block 5,000,000 == 2**28 == 268,435,456
* Block 5,200,000 == 2**30 == 1 TH
* Block 6,000,000 == 2**38 == 274 TH

## Links

Difficulty growth model: https://docs.google.com/spreadsheets/d/1ZXNrSCNV0HGWU7zOTUyIIRUGv5M44P6wiAZclpY4Y2Q/edit
