---
description: >-
  SIREN is a distributed protocol for creating, trading, and redeeming
  fully-collateralized options contracts for any ERC-20 token on Ethereum.
---

# Introduction

## About SIREN

The developers and designers at SIREN have been working on a protocol for decentralized options trading, and we’d like to share our thoughts on it with you here. Options are a financial primitive from which one can build many different more complex financial instruments. At their core, options give a trader the choice to buy or sell an asset at a predetermined price at a known time in the future. This is useful for protecting yourself \(a.k.a hedging\) against possible price changes in the asset, as well as speculating on these price changes.

Please continue reading for a high-level description of the unique components we’re working on to allow you to create and trade options for any ERC20 asset. We describe the core mechanics of how Siren Options are written and traded using an Automated Market Maker specifically designed for options, and the role the SIREN governance token plays in coordinating the options markets.

## Governance

There is a governance token for Siren Markets, called SIREN. This token allows holders to create new option markets, and determine the fee rate for writing, closing, and redeeming an option. These fees accrue to SIREN token holders. Upon launch these fees will be set to 0 to reduce the friction in using Siren options, but will likely increase if the DeFi community adopts Siren options.

![](.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

