---
title: Twoards Decentralized Price Oracles; Or how I learned to stop worrying and love the TWAP
date: 2022-12-24
---

Price oracles are a crucial necessity, but also a great weakness for DeFi. Most DeFi projects use centralized oracles
with no fallback, which means that they are a single point of failure. In this post, I will do a deep dive into oracles,
and especially decentralized ones, and provide multiple solutions to the problem. *This article is fairly technical and 
is not an introduction to oracles.*

## TLDR

If you can't be bothered to read the whole blog post, which is fine, here are some important takeaways:

- decentralized oracles are good, centralized oracles are bad
- decentralized oracles are hard to do properly
- a decentralized oracle should never take the face value of a single source. something like an index/TWAP is necessary
for the oracle to be usable
- having a centralized oracle with a fallback to a decentralized one is usually an acceptable tradeoff

## The current relationship between DeFi and oracles

Centralized oracles (mostly chainlink) are used by almost all top DeFi protocols. In my opinion this is a
rather reckless thing to do, as all protocols that use chainlink, with no good fallback, are at their mercy. No project
that relies on external centralized services can rightfully claim to be decentralized.

![Breakdown of oracles by TVL.](./defilamaOracleTvl.png)

Currently, around 65% of DeFi TVL is at the mercy of centralized oracles. Maker accounting for 30% of oracle TVL with
their oracle solution is a very nice thing to see, but it should be noted that they currently have way bigger systemic
risks than oracles. It should also be noted that a very large majority of projects *do not* have fallbacks in case 
chainlink experiences a catastrophic failure.

## Why centralized oracles are so widespread

When creating a DeFi protocol that needs to get price data for certain assets, it's important to make sure that the data
is as recent as possible and as resistant to manipulation as possible. Decentralized oracles face the same trillema 
blockchains do. If you prioritize speed, you lose security in the form of manipulation, and vice versa.

![Oracle trillema](./trillema.png)

Chainlink markets itself as a secure, decentralized, and fast oracle. While these claims are rather dubious, the
marketing worked, and in no small part of its plug-and-play nature, chainlink now secures over 40% of all DeFi TVL.
DeFi protocols and users assume that since it had no major incidents in the past, chainlink is indeed as decentralized
as the marketing suggests.