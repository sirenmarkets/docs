# SIREN Automated Market Maker

## Overview

Bootstrapping liquidity is core to creating a thriving options protocol. Not only does liquidity get fractured by combination of strike prices and expirations, it also requires sophistication on the part of liquidity providers in order to ensure fair and sustainable pricing.

To ensure liquidity on day 1 we designed a custom SIREN Automated Market Maker (AMM) that uses a novel combination of a constant-product bonding curve and options minting to trade both bTokens and wTokens. At the core of the protocol are the MinterAmm smart contracts. Each MinterAmm can trade several Siren Series, each sharing the same collateral token. For example, one MinterAmm contract can trade multiple strikes of WETH/USDC Calls, while another one can trade multiple WETH/USDC Puts. Once a Series is created anyone can interact with it in a permissionless manner. The solvency of a position is ensured at all times by the collateral locked in the Settlement Layer.

Therefore, Siren is designed to address the following issues:

- Trading multiple series from a single pool: Single AMM can trade multiple series that share the same collateral tokens
- Passive LP participation: Liquidity providers ("LPs") don’t need to worry about rolling their liquidity to another pool at expiration. The AMM manages redeeming expired Series and recycling the collateral token back to the pool to be used again for undewriting options
- Solving theta decay loss: Because the price is quoted using a Black-Scholes model, AMM is time-decay-aware
- No arbitrage needed: There is no need for arbitrage in order to quote reasonable prices

Our AMM is designed to address these issues observed in other prototypes and production-grade AMMs while preserving a mint + trade bonding curve approach. Furthermore, Siren AMM uses an on-chain Black-Scholes approximation coupled with a Price Oracle (Chainlink). It uses the bonding curve to determine slippage for each trade, but not the starting price. You can think of it as if a virtual pool is being dynamically calculated for each trade based on the current price from the Oracle.

The current design for our AMM uses pooled LP funds across multiple options series. This design is favorable to LPs who want to be able to deposit their money and passively receive returns. Our LPs are not obliged to continuously close out and create new positions for each series as they expire, which would have significant gas costs for the casual investor. Our pooled structure is particularly beneficial in the early stages of the project when there are few series with limited liquidity.

## Functionality

### Case 1 - Buying option contracts from the AMM

1. The trader enters the # of contracts (Calls or Puts) which they want to buy in the input field on the right panel.
2. AMM checks the current underlying price, b/wToken balances and existing collateral in the respective pool ($UNI, $SUSHI, $YFI, $WETH for Calls or $USDC for Puts) to calculate the Premium (shown here above the “WBTC Required” text) to be paid by the trader.
3. The trader pushes the Buy button.
4. The trader confirms the Premium amount in their wallet (Metamask, etc.).
5. The trader executes the transaction in his wallet (Metamask, etc.).
6. The Premium is moved from the trader’s wallet to the AMM (to the respective Pool).
7. Minting process:

- The AMM uses LP collateral to mint the b/wTokens specified by the “# of Contracts” field
- AMM moves the Collateral_In from the Pool to the allocated seriesVault. This Collateral_In consists of:
  - the Premium paid by the trader, plus
  - LP collateral previously provided to the AMM.

8. bTokens are sent to the Trader’s wallet which they can see in the Portfolio tab; while wTokens stay in the Pool  (presenting Covered Call / Covered Put).


### Case 2 - Selling option contracts back to the AMM

The trader enters # of contracts (Calls or Puts) which they want to sell back in the input field on the right panel.
The AMM checks the current underlying price, time to expiration, b/wToken balances linked to the respective Pool ($UNI, $SUSHI, $YFI, $WETH for Calls or $USDC for Puts) and existing Collateral in the pool to calculate the Premium (shown here above the “WBTC Expected” text) to be repaid to the trader.
The trader pushes the Sell button.
The trader approves the AMM to transfer the bToken from the trader’s wallet  (Metamask, etc.).
The Trader executes the transaction in his wallet.
The trader calls the bTokenSell function on the AMM
Closing process:
bTokens are matched with the respective wTokens (held in the Pool) and both are burned.
The Collateral_Out in the seriesVault becomes unlocked.
The Reverse Premium (the first part of the unlocked Collateral_Out) is paid to the Trader.
The rest of the unlocked Collateral_Out is returned to the Pool.

### Case 2.1 - Selling an option when there is not enough wTokens in the Pool (rare case)

1. Steps from #1 to #4 and from #6 to #8 are the same as for Case 2 (“normal” Selling).
2. Only for step #5 are there some differences: some of bTokens are not burned but stayed in the Pool. This happens because there are not enough wTokens to close out the bToken.
If there are enough bToken held by the AMM to service the trade, then the AMM does not need to mint additional bToken, and instead simply sells its existing bToken to the trader. If the trade size is larger than the amount of bToken held by the AMM, then the AMM will make up the difference by minting it

### Case 3 - Depositing liquidity

LP provides some amount of collateral for the Pool liquidity.
LP will be given a corresponding amount of lpTokens to track ownership. The amount of lpTokens is calculated based on total Pool value which includes:
collateral token
active b/wTokens
expired/unclaimed b/wTokens
In order to calculate correct amounts of lpTokens we do the following:
Claim expired wTokens and bTokens
Add value of all active bTokens and wTokens at current prices
Add value of collateral
AMM calculates the total Pool value
The necessary amount of lpTokens is minted and transferred to the LP.


### Case 4 - Withdrawing liquidity

LP specifies what amount of lpTokens to be withdrawn from the Pool.
When withdrawing user can specify if they want their pro-rata b/wTokens to be automatically sold to the pool for collateral (the “sell tokens” checkmark):
If LP chooses not to sell then they get pro-rata of all tokens in the pool (collateral, bTokens, wTokens).
If LP choses to sell then their bTokens and wTokens will be sold pro-rata to the pool for collateral. The price impact of selling will cause the LP to receive less collateral than the fair market value of the bTokens and wTokens.
AMM burns the respective amount of lpTokens
In order to calculate the correct amounts of the Pool value and lpTokens we do the following:
Claim expired wTokens and bTokens
Subtract the value of all active bTokens and wTokens at current prices
Subtract the value of collateral
LP will get pro-rata collateral asset

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

