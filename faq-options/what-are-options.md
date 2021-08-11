# What are options?

## Terminology

An option is a contract which conveys to its buyer the right, but not the obligation, to buy or sell an underlying asset at a strike price on an expiration date, paying a premium for that right. Options are a financial primitive from which one can build many different more complex financial instruments. This is useful for protecting yourself \(a.k.a hedging\) against possible price changes in the asset, as well as speculating on these price changes.

When buying options, the potential loss is limited to the amount paid for the options \(the premium\). Unlike futures, the holder is not required to buy or sell the asset. Currently on SIREN you can buy cryptocurrency options from the market and you can sell them back to the pool as well.

## What option types can I trade?

SIREN offers American style options with physical delivery: buyers can exercise anytime before expiration by paying the strike amount of payment asset in exchange for the underlying.

## What are calls?

Buying a call is a strategy that's used if you think a cryptocurrency's price will rise and can be seen as a substitute for buying cryptocurrency while offering limited risk. It usually costs less to buy an option than it does to buy the underlying cryptocurrency, and is generally considered less risky than a position in cryptocurrency. You can "hodl" a cryptocurrency indefinitely, but you can only hold a call option through its expiration date. If the cryptocurrency price does not rise above your strike price by the expiration date, the call option will expire without being exercised.

You have to decide whether to buy a call with more or fewer days to expiration. First, an option with fewer days to expiration usually has a cheaper premium \(cost\) than an option with more days to expiration. This is because the more time there is until expiration, the greater probability there is that the price will change, and thus the greater probability that the option will go into the money and be profitable to exercise.

## What are puts?

Buying puts is a strategy that profits from a drop in a cryptocurrency's price. Shorting a cryptocurrency directly \(i.e. by selling it now and buying it for a lower price in the future\), rather than through a put, can have high margin requirements, and some brokers restrict shorting entirely. The alternative to shorting is to buy puts, which has limited risk but unlimited profit potential.

You have to decide whether to buy a put with more or fewer days to expiration. First, an option with fewer days to expiration usually has a cheaper premium \(cost\) than an option with more days to expiration.

## What are bTokens and wTokens?

In SirenSwap AMM both the buyer’s and writer’s side of the contract are tokenized. The buyer’s side \(bToken, which stands for buyToken\) gives the holder the right to purchase the underlying asset at a predetermined strike price. The writer’s side \(wToken, which stands for writeToken\) allows the holder to withdraw the collateral \(if the option was not exercised\) or withdraw the exercise payment \(if the option was exercised\) from the contract after expiration.

Initially LPs deposit a collateral asset into the SIREN AMM pool, e.g. for WBTC/USDC calls the collateral asset is WBTC. When traders buy options from the SIREN AMM, the collateral in the pool is used to mint a new token pair \(a bToken and a wToken\). bTokens are sent to the buyer while wTokens stay in the pool. When buyers make trades they pay a premium in the collateral asset. In essence, over time, LPs become covered option writers in a passive way, automatically underwriting contracts for which there is demand.

More detailed explanation of the SirenSwap AMM and b/wTokens see at the [AMM section](../siren-protocol/siren-amm.md)

## Risks of trading options

Options are designed to be a tool for transferring risk from one trader to another. Therefore, when you buy an option, you are limiting your risk by transferring it to the writer of this option. When you write an option, you are accepting the risk from whoever bought the option. An option writer must accept that their underlying asset may be sold at the option strike price, which may be less than the underlying value of the asset at the time of expiration.

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

