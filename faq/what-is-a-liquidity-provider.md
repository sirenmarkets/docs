# What is a liquidity provider?

### Overview

A liquidity provider \(also known as an "LP"\), is one who deposits a collateral asset into the SIREN AMM pool, e.g. for WBTC/USDC calls the collateral asset is WBTC. When traders buy options from the SIREN AMM, the collateral in the pool is used to mint a new token pair \(a `bToken`, which stands for buyToken, and a `wToken`, which stands for writeToken\). bTokens are sent to the buyer while wTokens stay in the pool. When buyers make trades they pay a premium in the collateral asset.

LPs are covered option writers in a passive way, automatically underwriting contracts for which there is demand.

### **Providing capital**

When an LP wishes to provide collateral to the pool, the pool calculates the total value of all assets on its balance in order to properly calculate the amount of new LP tokens to mint. 

### Withdrawing capital

When an LP withdraws their capital, the AMM might have any combination of collateral token, payment token, active b/wTokens and expired unclaimed b/wTokens. Pro-rata w/bTokens are sent to their address.

### **Risks**

As option writers, LPs carry the risk of option being exercised. Since buyers are motivated to exercise their options only when it is profitable to do so, it means LPs can incur losses when this happens. When exercise happens a collateral token locked inside of wTokens is exchanged for a payment token at a ratio determined by the option strike price. 

A rational buyer would only trigger an exercise if the strike price is lower than current underlying price \(i.e. they can buy the underlying asset cheaper than it trades in the market\). In other words, LPs are forced to sell the underlying asset for less than they could've sold it in the market. This risk is present for option writes in all options markets in traditional finance.

LPs should maintain situational awareness and understand their risks. Here are a few examples:

* **A liquidity pool only has collateral at time of deposit and no trading activity while staking** = LP receives back same amount of deposited collateral upon withdrawal + accumulated SI rewards
* **A liquidity pool has wTokens or trading activity while staking, but all OTM** = LP gets back less collateral upon withdrawal compared to amount of originally deposited collateral + wTokens \(at expiration wTokens unlock the difference missing from their collateral plus option writing premiums/yield\)  + accumulated SI rewards
* **A liquidity pool has wTokens or trading activity while staking, some ITM** = LP gets back less collateral upon withdrawal compared to amount of originally deposited collateral + wTokens \(at expiration wTokens unlock the some of their missing collateral plus some payment token from options exercise. There may or may not be loss.\)  + accumulated SI rewards
* **A liquidity pool has trading activity while staking, mostly ITM** = same as above, more chance of loss, but still not guaranteed loss \(depends on the premium at which bTokens were sold to buyers that later exercised\)

There are factors that can help reduce LP risk:

1. Slippage creates a separate revenue stream that offsets potential exercise losses.
2. Diversification. Since a single AMM can trade multiple markets, LP's exercise risk is spread across multiple strike prices and expirations.

### **Rewards / Yield**

LPs earn yield in three ways:

1. Collecting option premium when traders buy options
2. Slippage when traders buy or sell options
3. SI reward tokens via the [SIREN LPP](https://sirenmarkets.medium.com/expanding-the-siren-lpp-c69969e25d41)

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

