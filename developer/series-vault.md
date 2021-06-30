# SeriesVault

## Mainnet Contract Address

TODO

## Overview

The `SeriesVault` acts as a simple storage contract for the [`SeriesController's`](TODO glossary) [`ERC20`](TODO glossary and [`ERC1155`](TODO glossary) tokens. Whenever the `SeriesController` receives collateral from minting options, it transfers the collateral to be stored in the `SeriesVault`. Currently the `SeriesVault` does not store any [option tokens](TODO glossary), but a future upgrade to the protocol might enable this so that we can implement [spreads](https://www.investopedia.com/articles/active-trading/032614/which-vertical-option-spread-should-you-use.asp).

The `SeriesVault` and `SeriesController` are separate contracts because we wanted to separate the [Series](TODO glossary) settlement layer logic (the `SeriesController`) from the Series collateral storage (the `SeriesVault`). This results in higher gas usage, but gives more flexibility for future changes to either contract.


## Functions