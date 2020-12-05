---
description: The SIREN Automated Market Maker Version 2
---

# SIREN AMMv2

### Overview

The SIREN Automated Market Maker Version 2 \(AMM v2, or simply stated - "v2"\) is designed to address issues observed in other prototypal and production-grade AMMs while preserving a mint + trade bonding curve approach. The issues it is designed to address are:

* **Trading multiple markets from a single pool.** v2 can trade multiple markets that share the same collateral and payment assets. For example, it can trade multiple strikes of WBTC calls.
* **Passive LP participation.** In v2, liquidity providers \("LPs"\) don’t need to worry about rolling their liquidity to another pool at expiration. As long as the AMM has active markets, LPs can keep their deposits.
* **Solving theta decay loss.** Because the price is quoted using a Black-Scholes model, v2 is time-decay-aware.
* **No arbitrage needed.** There is no need for arbitrage in order to quote reasonable prices.

SIREN v2 uses on-chain Black-Scholes approximation coupled with a price Oracle \(Chainlink\). It uses the bonding curve to determine slippage for each trade, but not the starting price. You can think of it as if a pool is being dynamically created for each trade based on the current oracle price.

### **How it works**

Initially LPs deposit a collateral asset into the SIREN AMM pool, e.g. for WBTC/USDC calls the collateral asset is WBTC. When traders buy options from the SIREN AMM, the collateral in the pool is used to mint a new token pair \(a `bToken`, which stands for buyToken, and a `wToken`, which stands for writeToken\). bTokens are sent to the buyer while wTokens stay in the pool. When buyers make trades they pay a premium in the collateral asset. In essence, over time, LPs become covered option writers in a passive way, automatically underwriting contracts for which there is demand.

At the core of the system is a `MinterAmm` smart contract. Each MinterAmm can trade up to 6 SIREN options contracts, each sharing the same collateral and payment tokens. For example, one MinterAmm contract can trade multiple WBTC/USDC calls, while another one can trade multiple WBTC/USDC puts.

Note that SIREN AMM v2 only trades bTokens, but not wTokens. Trading of wTokens may become available in the future.

### **Trading mechanics**

From a technical perspective, v2 functions in the following stepwise fashion:

1. Calculates bToken price using Black-Scholes approximation. It accepts the following inputs: current collateral price, implied volatility, option strike, time to expiration.
2. Based on the current pool collateral and b/wToken balance, determines the maximum-sized bToken / wToken pool that matches the price from step 1.
3. Executes the trade in the pool.
4. Cleans up all residual b/wTokens to ensure there is always maximum collateral available in the pool.

### **Providing capital**

Depositing capital involves the following steps:

1. Claiming all expired unclaimed wTokens in order to release their locked collateral or payment tokens into the AMM pool.
2. Calculates total pool value as a sum of collateral + payment token + bTokens + wTokens. To calculate the value of the payment token v2 uses a price Oracle; to calculate the value of b/wTokens v2 uses the price Oracle + Black-Scholes approximation.
3. Calculates the amount of LP tokens to mint.

When an LP wishes to provide collateral to the pool, the pool calculates the total value of all assets on its balance in order to properly calculate the amount of new LP tokens to mint. 

### **Withdrawing capital**

Withdrawing capital involves the following steps:

1. Claiming all expired unclaimed wTokens in order to release their locked collateral or payment token into the AMM pool.
2. Withdrawing all pro-rata assets.

When an LP withdraws their capital, the AMM might have any combination of collateral token, payment token, active b/wTokens and expired unclaimed b/wTokens. Pro-rata w/bTokens are sent to their address.

![](https://docs.google.com/drawings/u/0/d/s9a8nH8V_7Jtu9CB-HVwIlg/image?w=506&h=361&rev=1&ac=1&parent=1fJw3DA1aLUfiivW16HC8TUGruvmxQKmzL7h0UvuD9gU)

### **Pricing**

The price that the AMM quotes for each trade is determined by two components: marginal price and slippage. The marginal price is calculated on-chain based on a formula that takes as parameters the underlying price, time to expiration, option strike and implied volatility. In addition, each trade incurs slippage that is based on the size of the trade and available assets in the pool - the larger the trade relative to the size of the pool the higher the slippage. LPs benefit from this slippage as it provides a revenue stream in addition to collecting option premiums.

### **Contract expiration**

When contracts expire the AMM uses wTokens on its balance to claim the collateral and/or payment locked inside options markets back into the pool. This newly recycled collateral can now be used to underwrite options for newer markets, hence providing LPs with a passive way to underwrite options without having to manually roll expiring contracts.

### **LP risks**

As option writers LPs carry the risk of option being exercised. Since buyers are motivated to exercise their options only when it is profitable to do so it means LPs incur losses when this happens. When exercise happens collateral token locked inside of wTokens is exchanged for payment token at a ratio determined by the option strike price. 

A rational buyer would only trigger an exercise if the strike price is lower than current underlying price \(i.e. they can buy the underlying asset cheaper than it trades in the market\). In other words, LPs are forced to sell the underlying asset for less than they could've sold it in the market. This risk is present for option writes in all options markets in traditional finance.

There are factors that reduce LP risk:

1. Slippage creates a separate revenue stream that offsets potential exercise losses.
2. Diversification. Since a single AMM can trade multiple markets, LPs exercise risk is spread across multiple strike prices and expirations.

### **LP rewards / sources of yield**

LPs earn yield in three ways:

1. Collecting option premium when traders buy options
2. Slippage when traders buy or sell options
3. SI reward tokens via the [SIREN LPP](https://sirenmarkets.medium.com/expanding-the-siren-lpp-c69969e25d41)

![](.gitbook/assets/image.png)

**LAST UPDATED:** 5 December 2020

_NOTE: This is a living document that will continue to be updated as the SIREN protocol evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and specific technical guidance may also be provided from time to time in the SIREN Telegram._  
  
  
****

### \*\*\*\*

  
****

###  ****

  
  


