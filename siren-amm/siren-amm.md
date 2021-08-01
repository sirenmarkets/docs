# SIREN Automated Market Maker

## Overview

Bootstrapping liquidity is core to creating a thriving options protocol. Not only does liquidity get fractured by combination of strike prices and expirations, it also requires sophistication on the part of liquidity providers in order to ensure fair and sustainable pricing.

To ensure liquidity on day 1 we designed a custom SIREN Automated Market Maker ("AMM") that uses a novel combination of a constant-product bonding curve and options minting to trade both [bTokens and wTokens](https://docs.sirenmarkets.com/faq-options/what-are-options#what-are-btokens-and-wtokens). At the core of the protocol are the MinterAmm smart contracts. Each MinterAmm can trade several Series, each sharing the same collateral token. For example, one MinterAmm contract can trade multiple strikes of WETH/USDC Calls, while another one can trade multiple WETH/USDC Puts. Once a Series is created anyone can interact with it in a permissionless manner. The solvency of a position is ensured at all times by the collateral locked in the [Settlement Layer](https://docs.sirenmarkets.com/siren-protocol/settlement-layer).

Therefore, SIREN AMM is designed to address the following issues:

- Trading multiple series from a single pool: Single AMM can trade multiple series that share the same collateral tokens
- Passive Liquidity Providers ("LPs") participation: LPs don’t need to worry about rolling their liquidity to another pool at expiration. The AMM manages redeeming expired Series and recycling the collateral token back to the pool to be used again for undewriting options
- Solving theta decay loss: Because the price is quoted using a Black-Scholes model, AMM is time-decay-aware
- No arbitrage needed: There is no need for arbitrage in order to quote reasonable prices

Our AMM is designed to address these issues observed in other prototypes and production-grade AMMs while preserving a mint + trade bonding curve approach. Furthermore, SIREN AMM uses an on-chain Black-Scholes approximation coupled with a Price Oracle (currently Chainlink). It uses the bonding curve to determine slippage for each trade, but not the starting price. You can think of it as if a virtual pool is being dynamically calculated for each trade based on the current price from the Oracle.

The current design for our AMM uses pooled LP funds across multiple options series. This design is favorable to LPs who want to be able to deposit their money and passively receive returns. Our LPs are not obliged to continuously close out and create new positions for each series as they expire, which would have significant gas costs for the casual investor. Our pooled structure is particularly beneficial in the early stages of the project when there are few series with limited liquidity.

## Functionality

### Case 1 - Buying options

1. A trader enters the *# of contracts* (Calls or Puts) to be to bought in the input field on the right panel on the *Trade* tab.
2. AMM checks the current underlying price, **b/wToken** balances and existing **Free_Collateral** in the respective Pool ($UNI, $SUSHI, etc. for Calls or $USDC for Puts) to calculate the **Premium_In** to be paid by the trader.
3. The trader pushes the *Buy* button.
4. The trader confirms the **Premium_In** amount for the transaction in a wallet linked (Metamask, etc.).
5. The trader executes the transaction in the wallet.
6. The **Premium_In** is moved from the trader’s wallet to the Pool.
7. Minting process:
- The AMM uses **Collateral_In** to mint the b/wTokens specified by the *# of Contracts* (1 contract equals 1 **bToken**). This **Collateral_In** consists of:
  - the **Premium_In** paid by the trader, plus
  - the **LP_Collateral** taken from the **Free_Collateral** in the Pool
- The [SeriesController](https://docs.sirenmarkets.com/siren-protocol/settlement-layer) moves the **Collateral_In** from the Pool to the allocated **SeriesVault**
8. **bTokens** are sent to the trader’s wallet, their quantity can be seen in the *Portfolio* tab.
9. **wTokens** stay in the Pool (presenting [Covered Call](https://www.investopedia.com/terms/c/coveredcall.asp) / Covered Put).


### Case 2 - Selling options

1. A trader enters *# of contracts* (Calls or Puts) to be sold back to the AMM in the input field on the right panel on the *Trade* tab. 1 contract equals 1 **bToken**, the total quantity can be seen in the *Portfolio* tab.
2. The AMM checks the current underlying price, time to expiration, **b/wToken** balances linked to the respective Pool ($UNI, $SUSHI, etc. for Calls or $USDC for Puts) and existing **Free_Collateral** in the Pool to calculate the **Premium_Out** to be repaid to the trader.
3. The trader pushes the *Sell* button.
4. The trader confirms the # of **bToken** for the transaction in a wallet linked (Metamask, etc.).
5. The trader executes the transaction in the wallet.
6. The **bTokens** are moved from the trader’s wallet to the Pool.
7. Closing process:
- **bTokens** are matched with the respective **wTokens** (held in the Pool) and both are burned
- The **Collateral_Out** in the **SeriesVault** becomes unlocked. This **Collateral_Out** consists of:
  - the **Premium_Out** to be paid to the trader, plus
  - the **LP_Collateral** to be returned to the Pool
8. The **Premium_Out** is paid to the trader's wallet.
9. The SeriesController moves the **LP_Collateral** to the Pool.

### Case 2.1 - Selling options when there are not enough wTokens in the Pool (rare case)

1. Steps from #1 to #4 and from #6 to #8 are the same as for the Case 2 (“standard” Selling).
2. Only for step #5 are there some differences: some of **bTokens** are not burned but stay in the Pool. This happens because there are not enough **wTokens** to close out the **bTokens**.
3. For instance, new trader wants to buy some contracts: if there are enough **bTokens** held by the AMM to service the trade, then the AMM does not need to mint additional **bTokens**, and instead simply sells its existing **bTokens** to new trader. If the trade size is larger than the amount of **bTokens** held by the AMM, then the AMM will make up the difference by minting it.

### Case 3 - Depositing liquidity

1. A LP deposits some amount of **LP_collateral** for the respective Pool liquidity.
2. LP will be given a corresponding amount of **LP_Tokens** to track ownership. The amount of **LP_Tokens** is calculated based on total Pool value which includes:
- collateral tokens ($UNI, $SUSHI, etc.)
- active **b/wTokens**
- expired/unclaimed **b/wTokens**
3. In order to calculate correct amounts of **LP_Tokens** we do the following:
- Claim expired **b/wTokens**
- Add value of all active **b/wTokens** at current prices
- Add value of the **LP_ollateral**
4. The AMM calculates the new total Pool value.
5. The necessary amount of **LP_Tokens** is minted and transferred to the LP, their quantity can be seen in the *Pool* tab.

### Case 4 - Withdrawing liquidity

1. LP specifies what # of **LP_Tokens** to be withdrawn from the Pool, the total quantity can be seen in the *Pool* tab.
2. When withdrawing LPs can specify if they want their pro-rata **b/wTokens** to be automatically sold (the *sell tokens* checkmark) to the respective Pool for collateral ($UNI, $SUSHI, etc.):
- If LP chooses NOT to sell then they get pro-rata of assets the pool (**LP_collateral**, **bTokens**, **wTokens**), the received **b/wTokens** quantity can be seen in the *Portfolio* tab
- If LP chooses to sell then their **b/wTokens** will be sold pro-rata to the Pool. The price slippage impact of selling will cause the LP to receive less collateral than the fair market value of the **b/wTokens**
3. The AMM burns the respective amount of **LP_Tokens**.
4. In order to calculate the correct new total Pool value and **LP_Tokens** we do the following:
- Claim expired **b/wTokens**
- Subtract the value of all active **b/wTokens** at current prices
- Subtract the value of the **LP_collateral**
5. LP will get assets depending on *sell tokens* choice.

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

