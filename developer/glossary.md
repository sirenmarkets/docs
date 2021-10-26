# Glossary

### ATM

At the money. A situation where the options strike price is identical to the current market price of the underlying security.  If ETH is trading at 3500 and the strike of an ETH call is 3500, that call is ATM.

### Automated Market Maker

A bundle of contracts which accepts and manages liquidity.  The entity that replaces centralized order books, which both buyers and sellers interact with.  The AMM determines price through a combination of token reserves, price function, and an oracle.  

### Bilateral Tokenization

The tokenization of both short and long sides of a contract.  SIREN employs bilateral tokenization to keep track of both long and short sides of the contract universally by counting the balance of bTokens/wTokens. This also allows for the passive/automatic implementation of the writer’s role in SIREN protocol. 

### Black Scholes Price Formula

An equation for calculating an option’s price based on strike, expiration, and implied volatility.  This formula is used in combination with a price oracle and token reserves by the SIREN AMM to determine option pricing.

### bToken

A token representing the long side of an option series trade.  Created by the Series Controller, at expiry bTokens only have value when an option is ITM.  Keeps track of both long and short positions universally by counting the balance of b/wTokens.

### Cash-Secured Put

A put option in which the writer deposits an amount equal to the strike price into an account in case the option is exercised. This ensures the writer will be able to meet the obligations of the contract.  SIREN’s wTokens simulate this implementation with locked collateral in put contracts.

### Cash-Settlement

a settlement method used for options contracts where upon expiration, instead of delivery of the actual underlying asset, an equivalent cash position is transferred. Cash-settlement alleviates the buyer from paying the strike price in order to exercise an option.

### Collateral Token

The ERC20 Token used to underwrite an option series.  SIREN protocol requires each option to contain locked up collateral to guarantee payment to traders upon exercise.

### Covered Call

Similar to a cash-secured put, when the writer selling calls owns an equivalent amount of the underlying security thus ensuring the seller’s ability to deliver the shares if the buyer chooses to exercise.  SIREN’s wTokens achieve this with locked collateral in call contracts.

### European Option

A version of an options contract that limits execution to its expiration date. Investors are not able to exercise the option early.

### Expiration Date

The block time when an option series expires and is no longer purchasable on the AMM.  When a series is expired its bTokens and wTokens can be redeemed for their locked collateral.

### ITM

In the money. Refers to an option that has value in a strike price that is favorable in comparison to the prevailing market price of the underlying asset.  That is, the asset price will have exceeded the strike price for a call on that asset, or the opposite in case of a put.

### Liquidity Provider

Abbreviated as LP’s, liquidity providers are users that add assets to an AMM pool.  SIREN LP’s provide liquidity to the AMM to be used for minting and selling bTokens.  LP’s earn pro rata yields from option premiums.

### LP Token

The token LP’s receive for providing capital to the AMM.  The LP token is an IOU for LP’s collateral token.  An LP can burn their LP token and receive their share of the AMM’s value.

### OTM

Out of the money. An OTM call will have a strike price higher than the market price of the underlying asset. An OTM put will have a strike price lower than the market price of the underlying asset.  If an option expires OTM, it has no value.

### Premium
 
The option price offered by SIREN AMM when buying or selling bTokens or selling wTokens.  SIREN AMM determines this price through a combination of a price oracle, Black-Scholes price function, and calculations of the liquidity reserves.

### Pro Rata

A Latin term used to describe a proportionate allocation.  When the AMM distributes value pro-rata to LP’s, it is distributed in proportion to each LP’s share of the total supply of LP tokens for that AMM.  If an LP has provided 10% of all liquidity to an AMM’s pool, that LP will receive 10% of premium yield from options minted off that liquidity pool.

### Series

A group of options with a specific expiration date, strike price, and underlying token.  A single SIREN AMM can trade multiple series that share the same underlying, collateral and price tokens.  A series is identified by a unique seriesID.

### Spot Price

The price of a token on the open market.  SIREN protocol obtains the spot prices of various tokens using onchain oracles such as Chainlink.

### Settlement Layer

The set of smart contracts responsible for the creation of SIREN options through the process of bilateral tokenization, minting tokens and locking up collateral.  Contains the SeriesController which keeps track of which tokens were created for which options series.  The Settlement Layer is responsible for the exercise of bTokens and the claiming of wTokens on option expiry.

### Settlement Price

Spot price of a series’ underlying token once the series has expired and payout is being calculated.

### Price Impact

The impact of a trade on its own price.  The larger the trade, the more expensive it will become to fulfill.  SIREN’s implementation of this is deliberate to ensure available liquidity and institute an “urgency premium” for the profit benefit of LP’s.

### Underlying Token

The token whose price determines the moneyless of the series.  As the price of the underlying token changes, so does the premium of the series.  For ETH Call and ETH Put series, the underlying token will be ETH.

### wToken

A token composed of the short side of the contract combined with the collateral, representing a covered call or cash-secured put.  wTokens are minted together with bTokens by the AMM as it locks up the collateral for an option series trade.  wTokens are held by the AMM and the process of LP’s claiming wTokens is managed by the Settlement Layer.
