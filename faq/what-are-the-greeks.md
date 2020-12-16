# What are the greeks?

Buying an option, whether it's a call or put, is known as buying premium; selling or shorting an option, whether it's a call or put, is known as selling premium. This terminology implies a certain equivalency between calls and put. Indeed, calls and puts share many characteristics.

The greeks of calls and puts are calculated from the price of the underlying asset, the strike price of the option, the estimate of volatility of the underlying asset, the time to expiration of the option, the current interest rate and any dividends payable on the underlying asset before the expiration date of the option.

### Delta

The delta of a long call is positive; the delta of a long put is negative. The delta is reversed for short calls and puts. This can be understood by knowing that, all things being equal, a long call makes money if the underlying asset price goes up, and a long put makes money if the underlying asset price goes down. One of the things you will probably watch the most when trading is the delta of your position. The delta of a position is simply the sum of the quantity of each option times each option's delta. 

The main factor in the delta of an option is where the underlying asset price is relative to the strike price of the option. So, a call that starts out with very little delta can have very large delta if the underlying asset price rises sufficiently. Your exposure in the underlying asset increases as the underlying asset price rises.

Remember that delta is only a theoretical approximation of your exposure in the underlying asset. So, don't be surprised if your options don't have prices that match what your delta predicted. With the underlying asset price, time, and volatility changing, you may have to monitor the delta of your position vigilantly to make sure you have the exposure you want.

### Gamma

Gamma is the greek that gets your delta going. If you look at delta as the "speed" of your option position, gamma is the "acceleration". The gamma of long options, calls or puts, is always positive; of short options, always negative. Gamma is highest for the ATM strike, and slopes off toward the ITM and OTM strikes. One good way of interpreting gamma is that long gamma "manufactures" deltas in the direction the underlying asset is moving. That is, positive gamma is why long calls get more positive delta when the underlying asset price rises, and why long puts get more negative deltas when the underlying asset price falls. That's why short gamma can be so dangerous. When your speculation on an underlying asset's price is wrong, short gamma makes it worse. With a small gamma, your position delta probably won't change much. The more gamma your position has, your position delta can change a great deal and probably needs close monitoring.

But if you think the price of an underlying asset is going to move a great deal very quickly, you want to buy an option with relatively high gamma. The high positive gamma will get you more deltas if the underlying asset price moves the way you want it to, and reduce your deltas if the underlying asset price moves against you.

### Theta

Theta measures the daily whittling down of an option's value. It's inescapable. Long calls and puts have negative theta and, all other things being equal, lose money as time passes. Short calls and puts have positive theta and, all other things being equal, make money as time passes. The theta of options is indirectly proportional to gamma. When gamma is big and positive, theta tends to be big and negative. That's the trade-off. A position that has a lot of gamma \(good for fast changing underlying assets\) also has lots of theta that is continuously eroding its value.

It's highest for the ATM strike, and slopes off to the ITM and OTM options, and responds to the passage of time and changes in volatility the same way that gamma does.

### Vega

Vega measures how much the value of an option changes when the implied volatility of that option changes. Long calls and puts both have positive vega and, all things being equal, make money when implied volatility rises. Short calls and puts both have negative vega and, all things being equal, make money when implied volatility falls. Implied volatilities move up and down, sometimes in large amounts. When markets are sluggish, implied volatilities often drop, combining with theta to make long option positions cry out for mercy.

The more time there is until expiration, the higher the vega is for an option. Vega also depends on where the price of the underlying asset is relative to the strike price of the option. Like gamma and theta, vega is highest for the ATM options, and drops for the OTM and ITM options. So, ATM options with lots of time to expiration are the most sensitive to changes in implied volatility.

The theoretical assumptions made here are only as good as the data input. Stress testing with changes in overall implied volatility and at each individual strike will help you understand this concept.

From a purely theoretical standpoint calls and puts would be perfectly opposite were it not for the probability assumption, which implies that calls can theoretically go further in the money than puts and therefore have bigger deltas than puts when they are both at-the-money or equidistant from the money.

Sometimes there may be minuscule differences between the greeks of puts and calls at the same strike. But generally speaking, gamma, theta, and vega are the same for long calls and long puts. A fair wager would be that there is no way to make or save money by playing for any differences.

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._

