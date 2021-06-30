# PriceOracle

## Mainnet Contract Address

TODO

## Overview

The `PriceOracle` performs two necessary functions for the SIREN protocol:

First and most importantly, the `PriceOracle` stores the settlement price at expiration for every [Series'](TODO glossary) underlying asset. On every Friday at 8am UTC, an offchain bot process is scheduled to fetch and set the current price for every asset traded on the SIREN protocol. Since all [`Series'`](TODO glossary) expiration dates are aligned to Friday 8am UTC, this allows any expired `Series` to fetch the settlement price of the underlying token at expiration and use it along with the `Series'` [strike price](TODO glossary) to calculate the payouts for [`bToken`](TODO glossary) and [`wToken`](TODO glossary) holders.

Second, the `PriceOracle` gives all other protocol contracts a single place to access onchain price data. Currently we use only Chainlink's oracles, but other's like Uniswap V3 could easily be used. Contracts like the [`MinterAmm`](TODO glossary) and [`SeriesController`](TODO glossary) use the `PriceOracle` to fetch the current price for the tokens they use. It simplifies the protocol when all other contracts need only have a single address with a known interface for fetching onchain prices.

## Functions