# Series Struct

## Overview

The Series is the fundamental data structure in the SIREN protocol. It represents an [`ERC20`](glossary.md#erc20) option, and its name comes from the [TradFi](glossary.md#traditional-finance) term for an option with a specific underlying token, option type ([Put](glossary.md#put-option) or [Call](glossary.md#call-option)) [expiration date](glossary.md#expiration-date), and [strike price](glossary.md#strike-price). Each Series has an associated [`bToken`](glossary.md#bToken) and [`wToken`](glossary.md#wToken) which represent the [long](https://www.investopedia.com/terms/l/long.asp) and [short](https://www.investopedia.com/terms/s/short.asp) position, respectively, in the Series. The `SeriesController` manages the lifecycle of all Series, from Series creation in [`SeriesController.createSeries`](series-controller.md#createSeries), the minting of `bToken` and `wToken` in [`SeriesController.mintOptions`](series-controller.md#mintOptions), to the burning of `bToken` and `wToken` in [`SeriesController.exerciseOption`](series-controller.md#exerciseOption), [`SeriesController.claimCollateral`](series-controller.md#claimCollateral), and [`SeriesController.closePosition`](series-controller.md#closePosition).

### Series Id

Each Series is identified by a unique `uint64` called the `seriesId`. The `SeriesController` uses a monotonically increasing index for each new Series is creates, and stores them in the [series array](series-controller.md#series).

### Covered Calls and Puts

All options on the SIREN protocol are for [covered](glossary.md#covered-options) options, which means any account that wishes to mint and sell [option tokens](glossary.md#option-tokens), such as the [`MinterAmm`](minter-amm.md#overview), must transfer into the `SeriesController` the full collateral necessary to fulfill the option's obligation to buy (Calls) or sell (Puts) the underlying token. For instance, to mint 1.5 unit of `bToken` on a WBTC Call Series with expiration of Friday 8am July 2nd 2021 and a strike price of $60,000, the seller must first call `ERC20.approve` and give the `SeriesController` an allowance of 1.5 WBTC. If instead of a Call, this Series was a Put but all the other parameters remained the same, then the seller would first need to `ERC20.approve` the `SeriesController` for `1.5 * $60,000 = 90,0000 USDC`. This collateral ensures that no matter what how the price of the underlying token changes, the `SeriesController` will always be able to redeem option tokens for their correct payout.

## Struct Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `expirationDate`  | uint40  | The block timestamp this Series will expire, and then be redeemable                        |
| `isPutOption`  | bool  | `true` if these Series should be [Put](glossary.md#put-option) options, and false if they should be [Call](glossary.md#call-option) options                       |
| `tokens`  | The `underlyingToken`, `priceToken`, and `collateralToken` for this Series                       |
| `strikePrice`  | uint256  | The [strike price](glossary.md#strike-price) at which the Series will become [in the money](glossary.md#in-the-money-option)                       |
