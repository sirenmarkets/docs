# Glossary

## Terms

### Arbitrage

A risk-less trade. See [this investopedia definition](https://www.investopedia.com/terms/a/arbitrage.asp)

### Automated Market Maker

An onchain contract, or set of contracts, which offers to buy and sell tokens at a price determined by an onchain formula. Often abbreviated to "AMM". Th most popular AMM is [Uniswap](https://uniswap.org/), which allows users to buy and sell [`ERC20`](#erc20) tokens onchain without any intermediary. Usually the onchain formula uses some function of the AMM's token reserves to calculate the price. For the Siren Protocol we've written an [options trading AMM](minter-amm.md#overview).

### Black Scholes Price Function

A differential equation for calculating an option's price using its [strike price](#strike-price), [expiration date](#expiration-date), and [implied volatility](#implied-volatility). It can also be used to calculate the implied volatility, given its price on the open market. For Siren, we use a Black Scholes approximation discovered by Brennan-Subrahmanyam from their paper "A Simple Formula to Compute the Implied Standard Deviation" (1988). See [the Wikipedia page](https://en.wikipedia.org/wiki/Black%E2%80%93Scholes_model#Black%E2%80%93Scholes_formula) for more details.

### bToken

An [ERC1155](#erc1155) token representing the long side of an option [Series'](#series) trade. The [`SeriesController`](#series-controller) creates `bTokens` in [`SeriesController.mintOptions`](series-controller.md#mintOptions), and the `bTokens` can be redeemed using [`SeriesController.exerciseOption`](series-controller.md#exerciseOption). `bTokens` always have the same decimals as their Series' [underlying token](#underlying-token). See the [option token section of the docs](TODO link) for a detailed explanation of `bTokens`.

### Call Option

A financial asset which gives the holder the right, but not the obligation, to **purchase** an underlying asset at a pre-determined [strike price](#strike-price). Traders purchase Call options because they think the price will increase in the future. If the underlying asset price increases to more than the strike price, then the trader can profit by exercising the option and selling the underlying token, pocketing the difference between the current [spot price](#spot-price) and the strike price.

### Cash Settled

See the excellent definition on [Investopedia](https://www.investopedia.com/terms/c/cash-settled-options.asp)

### Collateral Token

The `ERC20` token used to underwrite an [option series](#series). The Siren protocol trades covered calls & puts, which means each option must lock some amount of collateral so that when a trader exercises the option there is no risk of the trader being unable to receive their due.

### Covered Options

See the excellent definition of a Covered Call on [Investopedia](https://www.investopedia.com/terms/c/coveredcall.asp). Covered Puts are very similar, except rather than locking the [underlying token](#underlying-token) into the protocol, we lock the [price token](#price-token) (e.g. [`USDC`](#usdc)).

### Decimals

The number of leading zeroes in an `ERC20` token's balances. For example, [USDC](#usdc) has 6 decimals, and so 1 unit of USDC is equal to `1000000`. For a TradFi example, the US Dollar has 2 decimals, and so 1 dollar is `100` cents. The decimals field exists only to display ERC20 balances in a way that is easier for humans to read them. See [the decimals section of the ERC20 standard](https://eips.ethereum.org/EIPS/eip-20#methods) for more information.

### ERC1155

An Ethereum token standard for representing multiple fungible and non-fungible tokens using a single onchain contract. Siren uses ERC1155 for its [option tokens](#option-tokens) in order to save on gas when creating new [Series](#series). Rather than deploy a new contract (as would have to be done in [`ERC20`](#erc20)) for each new option token, the protocol uses a single [`ERC1155Controller`](erc1155-controller.md#overview) which contains sub-ids for each [`bToken`](#btoken) and [`wToken`](#wToken). See [the EIP description](https://eips.ethereum.org/EIPS/eip-1155) for more details.

### ERC20

An Ethereum token standard for representing a single fungible token. `ERC20` is by far the most popular standard for representing fungible assets onchain. Many popular protocols, such as Uniswap and Aave, buy, sell, lend and borrow `ERC20` tokens. However, when a protocol needs to create a large number of `ERC20` tokens, it is more gas efficient to use a standard like [`ERC1155`](#erc1155) because `ERC1155` does not deploy a new contract for every token. See [the EIP description](https://eips.ethereum.org/EIPS/eip-20) for more details.

### ERC20 Approval

In order for a contract (e.g. the [`MinterAmm`](minter-amm.md#overview)) to call the [`ERC20.transferFrom`](https://eips.ethereum.org/EIPS/eip-20#methods) function and pull tokens from a user's wallet, the user must first call the [`ERC20.approve`](https://eips.ethereum.org/EIPS/eip-20#methods) to permit the `MinterAmm` to pull the tokens. This ensures that a contract is only able to transfer a user's tokens when the user has explicitly given permission for the contract to do so.

### Expiration Date

The block time at which a [Series](#series) will transition from the `OPEN` state to the `EXPIRED` state. When a Series is expired its [`bTokens`](#bToken) and [`wTokens`](#wToken) can be redeemed for their locked collateral. But it can no longer be purchased on the AMM.

### Implied Volatility

See the excellent definition on [Investopedia](https://www.investopedia.com/terms/i/iv.asp)

### In The Money Option

See the excellent definition on [Investopedia](https://www.investopedia.com/terms/i/inthemoney.asp)

### Liquidity Providers

Also abbreviated as "LPs". Any account that adds their [`ERC20`](#erc20) tokens to an AMM pool by calling the function [`MinterAmm.provideCapital`](minter-amm.md#providecapital) is a liquidity provider. LPs provide liquidity to the AMM so that the AMM can use it to mint and sell [`bTokens`](#btoken). In return, LPs earn yield from option [premiums](#premium)

### Logic Contract

One of the 2 contracts in the [EIP-1822 Universal Upgradeable Proxy Standard](https://eips.ethereum.org/EIPS/eip-1822). The logic contract is the contract which the [Proxy](#proxy-contract) delegates all of its function calls to. The logic contract contains all of the state variable definitions and function definitions you want the Proxy to be able to execute. The logic contract holds no state of its own, instead, the Proxy contract contains all of the state data.

### LP Token

The [`ERC20`](#erc20) token a [liquidity provider](#liquidity-provider) receives for providing capital to an [AMM](#automated-market-maker). The LP Token is like an IOU for liquidity providers's [collateral token](#collateral-token). An LP can burn their LP Token, and receive their share of the AMM's value.

### MinterAmm

A contract in the Siren Protocol which calculates [option premiums](glossary.md#premium)(#premium) for different Series, and exposes functions for buying and selling [`bTokens`](#bToken). See [the MinterAmm section](minter-amm.md#overview) for more info.

### Moneyness

Whether or not an option is [in the money](#in-the-money-option) or [out of the money](#out-of-the-money-option).

### Option Redemption

When either a [`bToken`](#btoken) is [exercised](series-controller.md#exerciseOption), or a [`wToken`](#wtoken) is [claimed](series-controller.md#claimCollateral).

### Option Payouts

The amount of [collateral token](#collateral-token) [option token](#option-tokens) holders receive for redeeming their option tokens. See the [`SeriesController.getExerciseAmount`](series-controller.md#getExerciseAmount) and [`SeriesController.getClaimAmount`](series-controller.md#getClaimAmount) for calculating the payouts for [`bTokens`](#btoken) and [`wTokens`](#wtoken), respectively.

### Option Tokens

"option tokens" is an umbrella term used to refer to both [`bTokens`](#btoken) and [`wTokens`](#wtoken).

### Owner

Certain contract functions can only be called by the SIREN protocol owner multisig address. This is a [Gnosis Safe Multisig](https://gnosis-safe.io/) contract deployed onchain which acts to manage the parts of the protocol that rely on a trusted administrator. Actions such as creating new [Series](#series), creating new [AMMs](#automated-market-maker), and upgrading contracts when vulnerabilities in the contracts are discovered. In the future we will transition this permissioned owner address to a DAO account, and the community of SI token holders can directly govern the protocol.

### Out of The Money Option

See the excellent definition on [Investopedia](https://www.investopedia.com/terms/o/outofthemoney.asp)

### Premium

The option price. The price offered by the AMM when buying or selling [`bTokens`](#btoken) or selling [`wTokens`](#wtoken). For the Siren protocol, the price is determined by the [Black-Scholes price function](#black-scholes-price-function).

### Price Impact

The price change in response to the trade size relative to the amount of liquidity in the [AMM](#automated-market-maker). Larger trades of [option tokens](#option-tokens) will result in a larger magnitude price compared to a smaller trade. This is because the bonding curve used by Siren's AMM, similar to that used by [Uniswap]((https://uniswap.org/)), is a hyperbola.

### Price Token

The `ERC20` token used to denominate the a [Series](#series) [strike price](#strikePrice). This will almost always be [USDC](glossary.md#usdc), but in the future could be different if we want to denominate prices in something other than USDC.

### Pro Rata

Pro rata is a Latin term used to describe a proportionate allocation. When an [AMM's](#automated-market-maker) pool value is distributed "pro-rata" to [LPs](#liquidity-providers), it is distributed in proportion to that LP's share of the total supply of LP Token for that AMM. If Alice owns 10% of an AMM's LP Token, then when she withdraws all of her LP Token then she will receive her pro-rata share (in this case 10%) of the AMM's total pool value. That total pool value will consist of [collateral token](#collateral token) and [option tokens](#option-tokens).

### Proxy Contract

One of the 2 contracts in the [EIP-1822 Universal Upgradeable Proxy Standard](https://eips.ethereum.org/EIPS/eip-1822). The Proxy contract holds all of the state data, whereas the [logic contract](#logic-contract) defines the state variables themselves and the functions the Proxy delegates to. Splitting the contracts this way allows for upgradeability; when the [owner](#owner) wishes to upgrade the contract, they call a function which points the Proxy contract to a new logic contract.

### Put Option

A financial asset which gives the holder the right, but not the obligation, to **sell** an underlying asset at a pre-determined [strike price](#strike-price). Traders purchase Put options because they think the [spot price](#spot-price) of the underlying asset will decrease in the future. If the underlying asset's spot price decreases to less than the strike price, then the trader can profit by purchasing the underlying token at the spot price and then exercising the option (which sells the recently purchased underlying asset), pocketing the difference between the strike price and the current spot price.

### Series

A Solidity struct representing an option with a specific [expiration date](#expiration-date), [strike price](#strike-price), and [underlying token](#underlying-token). The words "option" and "series" are often used interchangeably throughout the Siren Protocol. Each Series has a pool of locked [collateral token](#collateral-token) which is used to mint [`bToken`](#bToken) and [`wToken`](#wToken). A Series is identified by its unique [`seriesId`](#seriesId). See the [`SeriesController.series`](series-controller.md#series) for how to query individual Series.

### Series ID

A monotonically increasing identifier for a [Series](#series). The Series ID is used in various protocol functions for interacting with a specific Series. For example, [`SeriesController.mintOptions`](series-controller.md#mintOptions). You can access all of the data associated with a Series using [`SeriesController.series`](series-controller.md#series).

### Settlement Bot

An offchain process which checks for recently expired [Series](#series.md#overview), and executes the [`PriceOracle.setSettlementPrice`](price-oracle.md#setSettlementPrice) function. This ensures that Series use accurate [settlement prices](#settlement-price) when calculating their [payouts][#option-payouts].

### Settlement Price

The token price a [`Series`](series.md#overview) will use when for calculating its [payouts][#option-payouts] after the Series has expired. This price is set by the [`PriceOracle`](price-oracle.md#overview) using an [offchain bot process](#settlement-bot).

### Spot Price

The price of a token on the open market. Spot prices will differ slightly between different exchanges (e.g. Coinbase, Binance, Uniswap, etc.), but tend to remain close to one another because price differences between exchanges represent [arbitrage](#arbitrage) opportunities. The Siren Protocol obtains the spot prices of the various tokens it trades using onchain oracles, such as Chainlink.

### Slippage

A change in the execution price due to other transactions (trades, deposits, withdrawals) landing on-chain between the time the user broadcasted their transaction and the transaction getting mined in a block. There is no guarantee on the Ethereum blockchain that a user's transaction will be mined at a particular time, and so transactions from other users may be included in the chain prior to that transaction. These prior transactions can change the balance of reserves in the [AMM](#automated-market-maker), and cause the premium paid by the user to change. Slippage is the amount by which the premium changes.

### Strike Price

See the excellent definition on [Investopedia](https://www.investopedia.com/terms/s/strikeprice.asp)

### Theta Decay

A technical term for how an option's price goes down as time advances towards its [expiration date](expiration-date). Theta is one of the [option Greeks] defined as the derivative of the option price with respect to time. The Siren protocol includes an AMM which is aware of theta decay, which allows it to offer correct option prices.

### Traditional Finance

A blanket term for all of the institutions, practices, and services in finance prior to DeFi. Wall Street, Goldman-Sachs, The Federal Reserve, and others are all part of TradFi.

### Underlying Token

The `ERC20` token whose price determines the [moneyness](#moneyness) of the Series. As the price of the underlying token changes, so does the [premium](#premium) of Series with this underlying token. For [WBTC](#wbtc) [Call](glossary.md#call-option) and WBTC [Put](glossary.md#put-option) Series, the underlying token will be WBTC.

### USDC

A stablecoin pegged to USD. It is an audited and trusted stablecoin used by many DeFi protocols wishing to trade USD onchain, or denominate other assets in units of USD. Visit [its website](https://wbtc.network/) for more info.

### WBTC

A stablecoin pegged to the value of 1 Bitcoin. It is an audited and trusted stablecoin used by many DeFi protocols wishing to trade BTC in the Ethereum ecosystem. Visit [its website](https://www.coinbase.com/usdc/) for more info.

### wToken

An [ERC1155](#erc1155) token representing the short side of an option [Series'](#series) trade. The [`SeriesController`](#series-controller) creates `wTokens` in [`SeriesController.mintOptions`](series-controller.md#mintOptions), and the `wTokens` can be redeemed using [`SeriesController.claimCollateral`](series-controller.md#claimCollateral). `wTokens` always have the same decimals as their Series' [underlying token](#underlying-token). See the [option token section of the docs](TODO link) for a detailed explanation of `wTokens`.
