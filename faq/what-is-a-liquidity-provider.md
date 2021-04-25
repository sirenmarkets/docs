# What is a liquidity provider?

## Terminology

A liquidity provider (LP), is one who deposits a collateral asset into the SIREN AMM pool, e.g. for WBTC/USDC calls the collateral asset is WBTC. LPs are covered option writers in a passive way, automatically underwriting contracts for which there is demand.

When traders buy options from the SIREN AMM, the collateral in the pool is used to mint a new token pair (bToken, which stands for buyToken, and wToken, which stands for writeToken). bTokens are sent to the buyer while wTokens stay in the pool. When buyers make trades they pay a premium in the collateral asset.

## Providing capital

When an LP wishes to provide collateral to the pool, the pool calculates the total value of all assets it holds in order to properly calculate the amount of new LP tokens to mint. You can see it in percentage in the Share (%) column on the Pool tab. 

When LP deposits liquidity AMM performs the following steps:  
1. Claims all expired unclaimed wTokens in order to release their locked collateral or payment tokens (USDC) into the AMM pool.
2. Calculates total pool value as a sum of collateral tokens + payment tokens + bTokens + wTokens. To calculate the value of the payment token the SIREN protocol uses a price Oracle (Chainlink); to calculate the value of b/wTokens the SIREN protocol uses the price Oracle + Black-Scholes approximation.
3. Calculates the relative value of the new LP contribution versus the existing Total Pool Value and mints the appropriate number of LP tokens to represent their proportional share of the pool.

## Earnings (APY)

LPs earn yield in three ways:
- Collecting options premiums when traders buy options
- Trading slippage when traders buy options or sell them back to the pool.
- SI reward tokens via the SIREN LPP (button CLAIM on the Pool tab)

## Withdrawing capital

When an LP withdraws their capital, the AMM might contain a combination of collateral, payment token, active b/wTokens and expired unclaimed b/wTokens. When an LP withdraws, a mixture of collateral and w/bTokens are sent to their address.

Withdrawing capital involves the following steps:
1. Claiming all expired unclaimed wTokens in order to release their locked collateral or payment token into the AMM pool.
2. Withdrawing all pro-rata assets.

## Risks to being an LP

As option writers, LPs in aggregate carry the risk of option being exercised. A rational buyer would only trigger an exercise if the option is ITM. Said a different way, they would only trigger an exercise if the strike price is lower (for calls) and higher (for puts) than current underlying price (i.e. calls buyers can obtain the underlying asset cheaper than it trades in the market). When an exercise happens, some of the collateral locked inside of wTokens is exchanged for the payment token at a ratio determined by the option strike price.

Here are a few examples:
- **A liquidity pool only has collateral at time of deposit and no trading activity while staking**. An LP receives back the same amount of deposited collateral upon withdrawal.
- **A liquidity pool has wTokens or trading activity while staking, but all OTM**. LP gets back less collateral upon withdrawal compared to amount of originally deposited collateral, but also receives wTokens. At expiration of the option market, wTokens unlock the remaining original collateral plus the option writing premiums/yield.
- **A liquidity pool has wTokens or trading activity while staking, some ITM**. LP gets back less collateral upon withdrawal compared to amount of originally deposited collateral, plus wTokens. At expiration, wTokens unlock some of the original collateral from options exercise. There here may or may not be loss to the LP, depending on if the profits via premiums + slippage + SI rewards is greater than the value lost through the ITM wTokens.
- **A liquidity pool has trading activity while staking, mostly ITM**. Same as above, with a higher chance of loss, but a loss is still not guaranteed. The potential loss depends on the premium at which bTokens were sold to buyers that later exercised.

Diversification is a technique that can help reduce LP risk. Since a single AMM can trade multiple markets, LP's exercise risk is spread across multiple strike prices and expirations.

## Providing Liquidity Summary

Three steps:
1. An LP provides liquidity and obtains a share in the asset Pool consisting of collateral token (e.g. SUSHI, USDC), payment token (e.g. USDC) and wTokens (previously written contracts).
2. Liquidity in the pool is used to write new option contracts which allows LPs to earn APY from option premiums, trading slippage and SI rewards.
3. When an LP withdraws, they obtain the proportional share of the collateral and unexpired wTokens. To unlock the collateral in wTokens the LP can wait for contract expiration or perform a “closing” in the Portfolio.

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

