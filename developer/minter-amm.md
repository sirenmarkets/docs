# MinterAmm

## Mainnet Contract Address

TODO

## Overview

The `MinterAmm` is an [Automated Market Maker](TODO glossary) or "AMM" designed specifically for trading options for ERC20 tokens. It exposes functions for [buying](#bTokenBuy) and [selling](#bTokenSell) [`bToken`](TODO glossary) as well as [selling](#wTokenSell) [`wToken`](TODO glossary)[^1]. All prices will be in units of the AMM's [collateral token](TODO glossary), which for a [Call](TODO glossary) AMM will be equal to the underlying token (e.g. a WBTC Call AMM's collateral token is WBTC), and for a [Put](TODO glossary) AMM be equal to [USDC](TODO glossary). It uses the collateral token added by [liquidity providers](TODO glossary) or LP's to mint `bTokens` and `wTokens`, keeping the `wTokens` for itself and selling the `bTokens` to traders. The AMM exposes functions for [providing](#provideCapital) and [withdrawing](#withdrawCapital) liquidity in the AMM pool. It is often useful to know the total value of all the option tokens and collateral token that comprise a pool, and there [is a function](#getTotalPoolValue) for that as well.

### Option Token Pricing

When a trader goes to buy or sell option tokens via the `MinterAmm`, the price suffers from slippage. It is not possible to guarantee an exact option token price, because the price depends on the reserves of the AMM and those reserves might change if another transaction which affects the reserve amounts gets mined before the original trader's transaction gets mined. This is also true for when an LP withdraws their liquidity from the AMM and sets `sellTokens = true`, which will sell the `wToken` owed to trader back to AMM in return for collateral token. This is why all of these functions take some slippage function argument, either `collateralMinimum` when selling option tokens, or `collateralMaximum` when buying option tokens.

However, it's still useful for the trader to know what the expected price will be, and if there are no changes in the reserves in between when the trader broacasts the transaction and when the trader's transaction gets mined, this will indeed be the price of the trade. The AMM exposes functions for calculating the expected price when [selling `bTokens`](#bTokenGetCollateralOut), [buying 'bTokens'](bTokenGetCollateralIn), and [selling `wTokens`](#wTokenGetCollateralOut).

For more details on the pricing formulas used throughout the AMM, see the [protocol math section](TODO point to this)
## Functions

[^1]: It is currently not possible for the AMM to buy `wToken` because of a technical blocker in the SIREN protocol's [Black-Scholes](TODO glossary) approximation algorithm. The algorithm over-prices `bToken`'s and underprices `wToken`'s, and so if the AMM wer eto sell `wToken`'s then through arbitrage the AMM would quickly be drained of all `wToken`'s. For that reason, we cannot add functionality for selling `wToken`'s to the AMM.