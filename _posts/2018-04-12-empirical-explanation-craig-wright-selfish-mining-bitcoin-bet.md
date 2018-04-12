---
layout: post
title:  "Empirical explanation for Craig Wright's selfish mining Bitcoin bet"
tags: [bitcoin, data, data analysis, probability, statistics, r]
---
Last year, Craig Wright and Peter Rizun bet 1 Bitcoin on this problem:

{% include figure.html src="selfish-mining-bet.png" alt="Selfish Mining Bet" height="400" class="center" %}

[Evan Klitzke has a good explanation](https://eklitzke.org/demystifying-the-selfish-mining-bet) of the problem and a theoretical explanation of why Peter is correct. As Evan explains, because “mining approximates independent trials of a random process” the expected time for an honest miner to find a block is always 15 minutes, regardless of how much time they’ve been trying to find a block. Each time a miner increments a nonce and computes a block’s hash, they have a nearly equal chance of finding the nonce that “gives the block’s hash the required zero bits”, to quote Satoshi. No progress is made no matter how many hashes the miners have performed. The time between mined blocks is [memoryless](https://en.wikipedia.org/wiki/Memorylessness).

But we don’t need to rely on a theoretical argument alone. In the bet, the honest miners control ⅔ of the hash power, making the expected time for an honest miner to find a block 15 minutes, as Evan explains:

`T_expected = 10 / alpha`

Alpha of ⅔ yields 15. To empirically show why Peter is correct using real Bitcoin block time data, we’ll need to adapt the problem. By looking at the entire Bitcoin network (100% of hashpower) instead of only a subset of miners, we should expect 10 minutes until the next block is found, no matter how much time it has been since the last block was found.

Let’s find out if the data supports this. [BigQuery recently published all historical Bitcoin blockchain data](https://cloud.google.com/blog/big-data/2018/02/bitcoin-in-bigquery-blockchain-analytics-on-public-data). It’s an easy way to analyze Bitcoin data. I brought the data into R for analysis and visualization (full code below).  

One simplifying assumption: in order to analyze time between mined blocks, I sorted blocks by their timestamp. However, this likely introduces some error because some blocks’ timestamps may be unreliable. A given block’s timestamp may even be earlier than the timestamp of a prior block. [From Bitcoin wiki](https://en.bitcoin.it/wiki/Block_timestamp): “A timestamp is accepted as valid if it is greater than the median timestamp of previous 11 blocks, and less than the network-adjusted time + 2 hours.” For a more sophisticated approach, see the paper “[Block arrivals in the Bitcoin blockchain](https://arxiv.org/abs/1801.07447)” by R. Bowden, H.P. Keeler, A.E. Krzesinski, P.G. Taylor. [1]

Looking at data from 2016 [2], the mean block time is 9.61, and the distribution looks [exponential](https://en.wikipedia.org/wiki/Exponential_distribution), as expected. [3]

{% include figure.html src="2016-block-times.png" alt="Histogram of 2016 block times" height="400" class="center" %}

Now, if we only look at blocks that are found more than 10 minutes since the prior block, we see that the mean time is 19.64, about 10 minutes after our 10 minute offset. The distribution looks nearly identical, only it’s offset by 10.

{% include figure.html src="time-to-next-block-gt-10.png" alt="Histogram of time to next block, for blocks not found in first 10 minutes" height="400" class="center" %}

The same pattern can be seen for offsets up to about 65 minutes. Beyond 65 minutes, although the expected time until the next block is still 10 minutes, we can’t show it empirically due to a small sample size.

We can take this one step further by picking a set of random times and observing the mean time until the next block.

{% include figure.html src="block-times-random.png" alt="Histogram of time to next block from 10k random times" height="400" class="center" %}

Look familiar? It doesn’t matter what times you pick - 10 minutes since the last block was mined, totally random, every hour on the hour, your friends’ birthdays, whatever! The expected time until the next block will always be 10 minutes. [4]

[1] The paper takes a more sophisticated approach to identify blocks with unreliable timestamps, and then resamples them. They identified that 5% of blocks have unreliable timestamps.

[2] I chose 2016 because the BigQuery dataset has some [missing blocks in 2017](https://twitter.com/agrano/status/982909881077379072), which leads to outlier block times.

[3] The aforementioned paper [Block arrivals in the Bitcoin blockchain](https://arxiv.org/abs/1801.07447) rigorously investigates the assumption that Bitcoin blocks arrive according to a [Poisson process](https://en.wikipedia.org/wiki/Poisson_point_process), and hence time between blocks follow an [exponential distribution](https://en.wikipedia.org/wiki/Exponential_distribution). They note that Satoshi didn’t explicitly say that blocks arrive according to a Poisson process, but the [original whitepaper](https://bitcoin.org/bitcoin.pdf) includes calculations with this assumption: “Nakamoto [1] did not explicitly state that the blocks arrive according to a homogeneous Poisson process. However, the calculations in [1] are based on the assumption that the number of blocks that an attacker, mining at rate λ1, mines in the expected time z/λ2 that it takes for the community to mine z blocks at rate λ2, is Poisson with parameter λ1z/λ2, which is essentially equivalent to a Poisson process assumption.”

[4] For the same reason, [the times since the most recent blocks were mined will, on average, be 10 minutes](http://r6.ca/blog/20180225T160548Z.html).

Full code:

{% gist grano/4a2fc73093fd3af6707d710c7ddc6995 %}
