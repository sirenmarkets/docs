# What are greeks?

## Terminology

"Greeks" is a term used in the options market to describe the different dimensions of risk involved in trading options. These variables are called Greeks because they are typically associated with Greek symbols. The Greeks represent the sensitivity or relationship of the option to underlying parameters (asset price, volatility, time). Traders use different Greek values, such as delta, theta, and others, to assess options risk and manage option portfolios. 

The greeks of calls and puts are calculated from the price of the underlying asset, the strike price of the option, the estimate of volatility of the underlying asset, the time to expiration of the option, the current interest rate, and any dividends payable on the underlying asset before the expiration date of the option.

## Delta

Delta measures the rate of change of the theoretical option value with respect to changes in the underlying asset's price. Delta is the first derivative of the value of the option with respect to the underlying instrument's price. For example, if an option has Delta of 0.65, this means that if the underlying asset changes in price by $1 per token, the option value on it will also change by $0.65 per contract, all else being equal. We say “theoretical” option value and not just option value because the delta relies on a the option value calculated by Black-Scholes, and so does not always correspond to the present market value of the option. 

Delta is positive when buying a call, Delta is negative when buying a put. Delta is reversed when writing calls and puts. This can be understood by knowing that, all things being equal, a buying a call makes money if the underlying asset price goes up, and buying a put makes money if the underlying asset price goes down. A call that starts out with very little Delta can have a very large Delta if the underlying asset price rises sufficiently.

## Gamma

If you look at Delta as the "speed" of your option position, Gamma is the "acceleration". It is the second derivative of the value of the option with respect to the underlying instrument’s price. Gamma when buying options, calls or puts, is always positive; when writing options - always negative. Gamma is highest for the ATM strike, and slopes off toward the ITM and OTM strikes. One good way of interpreting Gamma is that a buyer’s side Gamma "manufactures" Deltas in the direction the underlying asset is moving. That is, positive Gamma is why buying a call gets more positive Delta when the underlying asset price rises, and why buying a put gets more negative Deltas when the underlying asset price falls.

A writer's side Gamma can be dangerous. When your speculation on an underlying asset's price is wrong, a writer's side  Gamma makes it worse. But if you think the price of an underlying asset is going to move a great deal very quickly, you want to buy an option with relatively high Gamma. The high positive Gamma will get you more Deltas if the underlying asset price moves the way you want it to, and reduce your Deltas if the underlying asset price moves against you.

## Theta

The term theta refers to the rate of daily decline in the option value due to the passage of time. This means an option loses value as time moves closer to the expiration date, as long as everything is held constant. Buyer’s side calls and puts have negative theta and, all other things being equal, lose money as time passes. Writer’s side calls and puts have positive theta and, all other things being equal, make money as time passes. 

The theta of options is indirectly proportional to gamma. When gamma is big and positive, theta tends to be big and negative. A position that has a lot of gamma (good for fast changing underlying assets) also has lots of theta that is continuously eroding its value. Theta is highest for the ATM strike and slopes off to the ITM and OTM options, and it responds to the passage of time and changes in volatility the same way that gamma does.

## Vega

Vega measures how much the value of an option changes when the implied volatility (IV) of that option changes. Buyer’s side calls and puts both have positive vega and, all things being equal, make money when implied volatility rises. Writer’s side calls and puts both have negative vega and, all things being equal, make money when implied volatility falls.

Like gamma and theta, vega is highest for the ATM options, and drops for the OTM and ITM options. So, ATM options with lots of time to expiration are the most sensitive to changes in implied volatility. The more time there is until expiration, the higher the vega is for an option. Vega also depends on where the price of the underlying asset is relative to the strike price of the option.

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

