---
description: >-
  SIREN is a distributed protocol for creating, trading, and redeeming
  fully-collateralized options contracts for any ERC-20 token on Ethereum.
---

# Introduction

## About SIREN

Siren Markets (‘SIREN’) is focused on creating a high-quality, seamless experience for sophisticated users and requires no third-party settling mechanism or order matching to complete option settlement on chain.

The developers and designers at SIREN have been working on a protocol for decentralized options trading, and we’d like to share our thoughts on it with you here. Options are a financial primitive from which one can build many different more complex financial instruments. At their core, options give a trader the choice to buy or sell an asset at a predetermined price at a known time in the future. This is useful for protecting yourself (a.k.a hedging) against possible price changes in the asset, as well as speculating on these price changes.

SIREN fills a unique market niche by catering specifically to sophisticated parties interested in holding and actively trading tokenized options contracts. Unlike any other cryptocurrency options platform, SIREN has tokenized both sides of an option contract, allowing for elements of an options ecosystem that can be traded as easily as a standard ERC-20. SIREN allows for both buyers and sellers to mint, buy, and trade in and out of options positions at any time.

In addition to our unique tokenization model, we have also separated our automated market making (AMM) layer from the settlement layer, which allows us to continuously modify the design of our market making, pool administration, and pricing as the trading volume and liquidity of our platform grows over time. Under this paradigm, a strategy that serves SIREN in our early stage of growth with a limited number of markets and available liquidity does not need to be the same strategy that serves during future high growth periods.

### Building the Documentation Locally

We use [GitBook](https://www.gitbook.com/) to build and host our docs. However, if you'd like to build them locally there are a couple steps you need to take to do so.

1. Run `npm install -g gitbook-cli` to download the `gitbook-cli` tool globally
2. Unfortunately, the `gitbook-cli` has been deprecated by the GitBook team and they haven't provided an official tool for building locally. In the meantime, you can follow the steps in [this GitHub issue](https://github.com/GitbookIO/gitbook-cli/issues/110#issuecomment-863706455) to get `gitbook-cli` working.
3. Once you've fixed `gitbook-cli`, you can run `gitbook build .` and `gitbook serve`. You can view your locally-built docs at http://localhost:4000

![](.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._
