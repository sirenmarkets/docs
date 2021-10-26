# Option Pricing

The use of an AMM in lieu of an order book necessitates a different system of price determination.  In the order book model, price is discovered by matching buys and sells at user-entered prices.  SIREN’s AMM determines the premium price for an option series through a combination of the underlying price, Black Scholes, and the price impact of the trade.

## Underlying Price

The underlying price refers to the price of the underlying assets the option is being written off of.  If SIREN’s AMM were determining the price of an ETH call option series, this underlying price would be ETH’s current value as stated by a price oracle.  Currently SIREN makes use of Chainlink as a price oracle.

## Black Scholes

The Black-Scholes price formula makes use of the underlying price from above in combination with strike price, time to expiry, risk-free rate, and volatility to estimate the theoretical value of an options series.  It is regarded as one of the standard ways to price an options contract.

## Price Impact

In traditional order-book based markets, as a trade grows larger it consequently clears out more and more lower priced orders such that the average price for a large trade would be higher than that of a smaller trade (the larger the trade, the more expensive it becomes to complete as it chugs through the order-book leaving only higher and higher priced orders to consume).  This market depth -the number and size of the orders (liquidity) in the orderbook- determines how large of an order the market can process without significant price impact.
Because SIREN does not use an order-book, there is no inherent unavoidable price impact; theoretically, a trade that would consume all the liquidity in the SIREN AMM could be completed at a single static price.   

However, SIREN deliberately implements price impact to achieve a simulation of market depth to ensure a trade of any size can be handled without draining all liquidity reserves in the AMM.  With price impact the liquidity can never run out because the price impact increases infinitely as trade size increases.  This ensures there will always be liquidity available for others to trade.

Additionally, price impact creates an “urgency premium”; if a trader wants to complete a very large trade all at once it will cost them more, benefitting SIREN’s LP’s in the form of increased premium profit. 

These factors together form the basis for SIREN AMM’s option pricing.  SIREN factors in price impact before the trade is executed, and has guardrails to ensure buyers are quoted an accurate price.
