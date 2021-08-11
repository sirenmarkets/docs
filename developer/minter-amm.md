# MinterAmm

## Polygon Contract Address

[View on Polygonscan](https://polygonscan.com/address/0xca84835655a8e863d34875326419a46f823320f6)

## Source Code

[`MinterAmm.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/amm/MinterAmm.sol)

## Overview

The `MinterAmm` is an [Automated Market Maker](glossary.md#automated-market-maker) or "AMM" designed specifically for trading options for `ERC20` tokens. It exposes functions for [buying](#bTokenBuy) and [selling](#bTokenSell) [`bToken`](glossary.md#bToken) as well as [selling](#wTokenSell) [`wToken`](glossary.md#wToken)[^1]. Each `MinterAmm` trades one of the two types of option [Series](series.md#overview): [Puts](glossary.md#put-option) or [Calls](glossary.md#call-option). All [option premiums](glossary.md#premium) will be in units of the AMM's [collateral token](glossary.md#collateral-token), which for a Call AMM will be equal to the underlying token (e.g. a WBTC Call AMM's collateral token is WBTC), and for a Put AMM be equal to [USDC](glossary.md#usdc) (e.g. a WBTC Put AMM's collateral token is USDC). It uses the collateral token added by [liquidity providers](glossary.md#liquidity-providers) to mint `bTokens` and `wTokens`, keeping the `wTokens` for itself and selling the `bTokens` to traders. The AMM exposes functions for [providing](#provideCapital) and [withdrawing](#withdrawCapital) liquidity in the AMM pool. It is often useful to know the total value of all the option tokens and collateral token that comprise a pool, and there [is a function](#getTotalPoolValue) for that as well.

### Option Token Pricing

When a trader goes to buy or sell option tokens via the `MinterAmm`, the quoted premium includes consists of the component from the onchain [Black-Scholes approximation formula](glossary.md#black-scholes-price-function), as well as a [price impact](glossary.md#price-impact) component. It is not possible to guarantee an exact option token price, because the price depends on the reserves of [`bToken`](glossary.md#bToken) and [`wToken`](glossary.md#wToken) held by the AMM, and those reserves might change if another transaction which affects the reserve amounts gets mined before the original trader's transaction gets mined. This is also true for when an LP withdraws their liquidity from the AMM and sets `sellTokens = true`, which will sell the `wTokens` owed to trader back to AMM in return for collateral token. This is why all of these functions take some [slippage](glossary.md#slippage) function argument, either `collateralMinimum` when selling option tokens, or `collateralMaximum` when buying option tokens.

However, it's still useful for the trader to know what the expected price will be, and if there are no changes in the reserves in between when the trader broadcasts the transaction and when the trader's transaction gets mined, this will indeed be the price of the trade. The AMM exposes functions for calculating the expected price when [selling `bTokens`](#bTokenGetCollateralOut), [buying 'bTokens'](bTokenGetCollateralIn), and [selling `wTokens`](#wTokenGetCollateralOut).

For more details on the pricing formulas used throughout the AMM, see the [protocol math section](TODO point to this)

## Functions

### Modifiers

#### minTradeSize

```solidity
  modifier minTradeSize(uint256 tradeSize) {
    require(
      tradeSize >= MINIMUM_TRADE_SIZE,
      "Buy/Sell amount below min size"
    );
    _;
  }
```

Prevents low-valued trades from occurring. Due to numeric rounding issues with Solidity, if a user passes a low-valued argument in [`bTokenBuy`](#bTokenBuy), [`bTokenSell`](#bTokenSell), or [`wTokenSell`](#wTokenSell) unexpected behavior can occur. So to preclude this from happening we gate these function calls with the `minTradeSize` modifier

#### onlyOwner

```solidity
  modifier onlyOwner() {
    require(owner() == _msgSender(), "Ownable: caller is not the owner");
    _;
  }
```

Certain functions can only be called by the SIREN protocol owner multisig address. The functions [`updateImplementation`](#updateImplementation) and [`transferOwnership`](#transferOwnship) exist so that the owners can update the contracts in the event of a critical vulnerability that puts user's funds at risk.

### View/Pure Functions

#### lpToken

```solidity
  function lpToken(
  ) public view returns (ISimpleToken)
```

Returns the address of the `ERC20` contract used for this `MinterAmm`'s [LP Token](glossary.md#lp-token).

#### underlyingToken

```solidity
  function underlyingToken(
  ) public view returns (IERC20)
```

Returns the address of the `ERC20` token whose price determines the (glossary.md#moneyness) of the Series this AMM trades. For example, both WBTC Call [option series](glossary.md#series) and WBTC Put option series share [WBTC](glossary.md#wbtc) as their [underlying token](glossary.md#underlying-token), because the price of WBTC determines whether the Series is [in](glossary.md#in-the-money-option) or [out of](glossary.md#out-of-the-money-option) the money. If this AMM trades [Calls](glossary.md#call-option) then this will be the same token as the [collateralToken](#collateralToken).

#### priceToken

```solidity
  function priceToken(
  ) public view returns (IERC20)
```

Returns the address of the `ERC20` token used to denominate the [`strike price`](series.md#strikePrice) of the Series this AMM trades. This will almost always be [USDC](glossary.md#usdc), but in the future could be different when SIREN denominates prices in something other than USDC. If this AMM trades [Puts](glossary.md#put-option) then this will be the same token as the [collateralToken](#collateralToken)

#### collateralToken

```solidity
  function collateralToken(
  ) public view returns (IERC20)
```

Returns the address of the `ERC20` token this AMM uses for collateral. This is the token used to denominate all option prices (a.k.a. premiums) in the `MinterAmm`, as well as the liquidity provided by [LPs](glossary.md#liquidity-providers)

If this is a [Call](glossary.md#call-option) Series then the collateral token will be equal to the [underlyingToken](#underlyingToken), because a Call gives the holder the right to **buy** the underlying. And if this is a [Put](glossary.md#put-option) Series then the collateral token will be equal to the [priceToken](#priceToken), because a Put gives the holder the right to **sell** the underlying.

#### seriesController

```solidity
  function seriesController(
  ) public view returns (ISeriesController)
```

Returns the [`SeriesController`](series-controller.md#overview) associated with this `MinterAmm`. The `MinterAmm` uses the `SeriesController` to interact with the [Series](series.md#overview) it trades.

#### erc1155Controller

```solidity
  function erc1155Controller(
  ) public view returns (IERC1155)
```

Returns the [`ERC1155Controller`](erc1155-controller.md#overview) associated with this `MinterAmm`. The `MinterAmm` uses the `ERC1155Controller` to interact with the the [option tokens](glossary.md#option-tokens) of the [Series](series.md#overview) it trades.

#### tradeFeeBasisPoints

```solidity
  function tradeFeeBasisPoints(
  ) public view returns (uint16)
```

The fee, denominated in bips (i.e. hundredths of a percent), that the `MinterAmm` takes when it executes a trade.

#### volatilityFactor

```solidity
  function volatilityFactor(
  ) public view returns (uint256)
```

The volatility coefficient to use in the [Black-Scholes approximation formula](glossary.md#black-scholes-price-function). This value is calculated offchain to save gas, and updated by the contract [owner]((glossary.md#owner)). It is equal to the [implied volatility](glossary.md#implied-volatility) multiplied by `0.4`, multiplied by the square root of the number of seconds in a year (i.e. `implied_volatility * 0.4 * Math.sqrt(365 * 24 * 60 * 60)`)

#### MINIMUM_TRADE_SIZE

```solidity
  function MINIMUM_TRADE_SIZE(
  ) public view returns (uint256)
```

Equal to `1000`. This is the minimum trade sized used in the [`minTradeSize`](#minTradeSize) modifier.

#### getTotalPoolValue

```solidity
  function getTotalPoolValue(bool includeUnclaimed)
    public
    view
    returns (uint256)
```

Returns the total value of all the tokens held by the pool, denominated in units of [collateral token](glossary.md#collateral-token). The AMM's value stems from several different tokens: The [option tokens](glossary.md#option-tokens) it holds, as well as the [collateral token] is has received from [LP's](glossary.md#liquidity-providers) and from collecting [option premiums](glossary.md#premium).

The argument `includeUnclaimed` exists only as a way to save on gas when calculating total pool value on chain; if the caller knows all [option tokens](glossary.md#option-tokens) have been claimed (which will be true if [`claimAllExpiredTokens`](#claimAllExpiredTokens) has just been called), it can pass `false` for `includeUnclaimed` and skip calculating their value.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `includeUnclaimed`  | bool  | `true` if the value of expired of unclaimed `wToken` should be included in the total pool calculation, and `false` if it should not be included                         |

#### getAllSeries

```solidity
  function getAllSeries() external view returns (uint64[] memory)
```

Returns the array of [`seriesIds`](series.md#series-id) for the Series this AMM trades.

#### getSeries

```solidity
  function getSeries(uint64 seriesId)
    external
    view
    returns (ISeriesController.Series memory)
```

Returns the [`Series struct`](series.md#overview) with the given `seriesId` traded by this AMM.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |

#### getVirtualReserves

```solidity
  function getVirtualReserves(uint64 seriesId)
    public
    view
    returns (uint256, uint256)
```

Returns the balances of [`bToken`](glossary.md#bToken) and [`wToken`](glossary.md#wToken) available to the AMM for the given [`seriesId`](series.md#series-id). The AMM uses these balances as inputs into its bonding curve equation to compute the premiums for buying and selling [option tokens](glossary.md#option-tokens).

We use the word "virtual" because the balances returned by this function include not only the raw balances of `bToken` and `wToken` held by the AMM, but additionally those option tokens that could be minted using the AMM's [collateral token](glossary.md#collateral-token). The amounts returned will likely be slightly less than the full `bToken` and `wToken` amounts owned by the AMM, because the amounts returned must satisfy the ratio of the `wToken` and `bToken` prices computed by the onchain [Black-Scholes approximation formula](glossary.md#black-scholes-price-function). See the documentation section on [protocol math](TODO point to this) for the motivation behind calculating virtual reserve balances in this manner.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `bTokenVirtualBalance` | uint256 | The amount of `bToken` available on the AMM  |
| `wTokenVirtualBalance` | uint256 | The amount of `wToken` available on the AMM |

#### getPriceForSeries

```solidity
  function getPriceForSeries(uint64 seriesId)
    external
    view
    returns (uint256)
```

Returns the price of the [underlyingToken](glossary.md#underlying-token) of the given [`seriesId`](series.md#series-id) denominated in the [priceToken](glossary.md#price-token). The returned values always use `8` decimal places. For example, if the Series' underlying == WBTC and price == USDC, then this function will return `4500000000000` ($45_000 in human readable units).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |

#### calcPrice

```solidity
  function calcPrice(
    uint256 timeUntilExpiry,
    uint256 strike,
    uint256 currentPrice,
    uint256 volatility,
    bool isPutOption
  ) public pure returns (uint256)
```

Returns the price of a Series' [`bToken`](glossary.md#bToken) as a percentage of collateral locked by 1 [option token](glossary.md#option-tokens), multiplied by a scaling factor of `1e18`. For instance, if the price of 1 `bToken` is `0.1 * 1e18`, and the `bToken`'s Series has a [strike price](glossary.md#strike-price) of $35,000, then the price of 1 `bToken` in units of USDC is $3,500.

The scaling factor exists as a workaround to Solidity's lack of floating point numerics. If we did not use the scaling factor, then any division of the price variable would round down to 0. So a price of `0.5 * 1e18` and be thought treated logically as simply `0.5`.

`calcPrice` uses a [Black-Scholes approximation formula](Tglossary.md#black-scholes-price-function) for pricing `bTokens`. See the section [protocol math](TODO link to this) for more details on the approximation formula.

The price of a Series' `wToken` is always `1e18 - price_of_btoken`, so in the above scenario, the price of the `wToken` would be $35,000 - $3,500 = $31,500.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `timeUntilExpiry`  | uint256  | The number of seconds until the Series' [expiration date](glossary.md#expiration-date)                       |
| `strike`  | uint256  | The [strike price](glossary.md#strike-price) of the Series'. See [The Series struct parameters section](series.md#struct-parameters) for more details on the strike price                       |
| `currentPrice`  | uint256  | The [spot price](glossary.md#spot-price) of the Series' [underlying token](glossary.md#underlying-token), denominated in units of [price token](glossary.md#price-token) with 8 [decimals](glossary.md#decimals)                      |
| `volatility`  | uint256  | The AMM's [volatility factor](#volatilityFactor)                       |
| `isPutOption`  | uint256  | `true` if the Series is for a [Put](glossary.md#put-option) option, and `false` if it's for a [Call](glossary.md#call-option) option                       |

#### bTokenGetCollateralIn

```solidity
  function bTokenGetCollateralIn(uint64 seriesId, uint256 bTokenAmount)
    public
    view
    returns (uint256)
```

Returns the [premium](glossary.md#premium) for purchasing from the AMM `bTokenAmount` of [`bTokens`](glossary.md#bToken) on the given [`seriesId`](series.md#series-id) in units of [collateral token](glossary.md#collateral-token). The premium has two components: the price returned by the [Black-Scholes approximation formula](glossary.md#black-scholes-price-function), and the added cost of [price impact](glossary.md#price-impact).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `bTokenAmount`  | uint256  | The amount of `bToken` the caller wants to receive in return for paying the premium                       |

#### bTokenGetCollateralOut

```solidity
  function bTokenGetCollateralOut(uint64 seriesId, uint256 bTokenAmount)
    public
    view
    returns (uint256)
```

Returns the premium for selling to the AMM `bTokenAmount` of [`bTokens`](glossary.md#bToken) on the given [`seriesId`](series.md#series-id) in units of [collateral token](glossary.md#collateral-token). The premium has two components: the price returned by the [Black-Scholes approximation formula](glossary.md#black-scholes-price-function), and the [price impact](glossary.md#price-impact).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `bTokenAmount`  | uint256  | The amount of `bToken` the caller wants to sell to the AMM                       |

#### wTokenGetCollateralOut

```solidity
  function wTokenGetCollateralOut(uint64 seriesId, uint256 wTokenAmount)
    public
    view
    returns (uint256)
```

Returns the premium for selling to the AMM `wTokenAmount` of [`wTokens`](glossary.md#wToken) on the given [`seriesId`](series.md#series-id) in units of [collateral token](glossary.md#collateral-token). The premium has two components: the price returned by the [Black-Scholes approximation formula](glossary.md#black-scholes-price-function), and the [price impact](glossary.md#price-impact).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `wTokenAmount`  | uint256  | The amount of `wToken` the caller wants to sell to the AMM                       |

#### getCollateralValueOfAllExpiredOptionTokens

```solidity
  function getCollateralValueOfAllExpiredOptionTokens()
    public
    view
    returns (uint256)
```

Returns the amount of [collateral token](glossary.md#collateral-token) the AMM would receive by redeeming all expired [`bTokens`](glossary.md#bToken) and [`wTokens`](glossary.md#wToken) held by the AMM. This function is useful for offchain clients to calculate the total [collateral token](glossary.md#collateral-token) available to the AMM. The offchain clients can use this to calculate the collateral owed to [LPs](glossary.md#liquidity-providers) when The LPs withdraw their [LP Token](glossary.md#lp-token)

#### getOptionTokensSaleValue

```solidity
  function getOptionTokensSaleValue(uint256 lpTokenAmount)
    external
    view
    returns (uint256)
```

Returns the expected amount of [collateral token](glossary.md#collateral-token) an [LP](glossary.md#liquidity-providers) will receive for selling the [`bTokens`](glossary.md#bToken) and [`wTokens`](glossary.md#wToken) held by the AMM on the [LP's](glossary.md#liquidity-providers) behalf. This function is useful for offchain clients to calculate value of the [option tokens](glossary.md#option-tokens) owed to the holder of `lpTokenAmount` of LP Token.

For example, if `lpTokenAmount` is equal to 10% of the total supply of LP Token, then the return value of [`MinterAmm.getOptionTokensSaleValue`](minter-amm.md#getOptionTokensSaleValue) will be equal to the premiums for selling 10% of the `bTokens` and `wTokens` held by the AMM. See [`MinterAmm.bTokenGetCollateralOut`](minter-amm.md#bTokenGetCollateralOut) and [`MinterAmm.wTokenGetCollateralOut`](minter-amm.md#wTokenGetCollateralOut) for how to calculate the premiums when selling `bTokens` and `wTokens`.
##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `lpTokenAmount`  | uint256  | The amount of LP token that the caller will burn in return for collateral                       |

### Mutating Functions

#### provideCapital

```solidity
  function provideCapital(uint256 collateralAmount, uint256 lpTokenMinimum)
    external
```

Transfers the caller's [collateral token](glossary.md#collateral-token) to the AMM so it can be used for minting options, and in return sends [LP Token](glossary.md#lp-token) the caller can use to later withdraw their share of the AMM's [total value](#getTotalPoolValue).

The amount of LP Token the caller receives depends on the AMM's total value, which itself depends on the AMM's reserves of [option tokens](glossary.md#option-tokens) and [collateral token](glossary.md#collateral-token). The reserve amounts may change when the caller's transaction finally gets included in the blockchain, so this function takes an `lpTokenMinimum` argument to specify the minimum acceptable amount of LP Token the caller will receive, and any lower than this will cause the function call to revert.

[LPs](glossary.md#liquidity-providers) profit when the fees they acquire from selling options is greater than their losses from the options they sell going deeper [in the money](glossary.md#in-the-money-option).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `collateralAmount`  | uint256  | The amount of [collateral token](glossary.md#collateral-token) to transfer to the AMM                       |
| `lpTokenMinimum`  | uint256  | The minimum amount of [LP Token](glossary.md#lp-token) the caller is willing to receive for providing `collateraAmount` of collateral token                      |

#### withdrawCapital

```solidity
  function withdrawCapital(
    uint256 lpTokenAmount,
    bool sellTokens,
    uint256 collateralMinimum
  ) public
```

The caller burns `lpTokenAmount` of their [LP Token](glossary.md#lp-token) and in exchange receives their [pro-rata](glossary.md#pro-rata) share of the AMM's tokens. For example, if `lpTokenAmount` is equal to 10% of the total supply of LP Tokens, then the caller will receive 10% of the AMM's [collateral token](glossary.md#collateral-token) as well as 10% of each of the [option tokens](glossary.md#option-tokens) the AMM holds. If `sellTokens` is set to `true`, then the caller will receive only collateral token, because all of LP's option tokens will be sold to the AMM in exchange for collateral token. However, because of [price impact](glossary.md#price-impact) the value of the collateral token the LP receives may be less than 10% of the AMM's total value.

The amount of collateral token the caller receives depends on the AMM's total value, which itself depends on the AMM's reserves of [option tokens](glossary.md#option-tokens) and [collateral token](glossary.md#collateral-token). The reserve amounts may change when the caller's transaction finally gets included in the blockchain, so this function takes a `collateralMinimum` argument to specify the minimum acceptable amount of collateral token the caller will receive, and any lower than this will cause the function call to revert.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `lpTokenAmount`  | uint256  | The amount of LP Token to burn                       |
| `sellTokens`  | bool  | `true` if the caller wishes to sell their option tokens to the AMM and only receive [collateral token](glossary.md#collateral-token), and `false` if they wish to receive less collateral token but receive their [pro-rata](glossary.md#pro-rata) share of the AMM's option tokens which they can sell at a later time                       |
| `collateralMinimum`  | uint256  | The minimum amount of collateral token the caller wishes to receive for burning their LP Token                     |

#### claimAllExpiredTokens

```solidity
  function claimAllExpiredTokens() public
```

Redeems all the [option tokens](glossary.md#option-tokens) held by the AMM that have recently expired. See [`MinterAmm.claimExpiredTokens`](#claimExpiredTokens)

#### claimExpiredTokens

```solidity
  function claimExpiredTokens(uint64 seriesId) public
```

Redeems the given Series' expired [`bTokens`](glossary.md#bToken) and [`wTokens`](glossary.md#wToken) held by the AMM. If the Series is [out of the money](glossary.md#out-of-the-money-option) then all `bTokens` will be worthless, and the `wTokens` will be claimed for the collateral value locked when minting them. If the Series is [in the money](glossary.md#in-the-money-option) then both the `bTokens` and `wTokens` will have some non-zero value.

The AMM redeems all expired [option tokens](glossary.md#option-tokens) every time an LP provides or withdraws liquidity, so the AMM is constantly converting expired option tokens back into collateral it can use to mint further options

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |

#### bTokenBuy

```solidity
  function bTokenBuy(
    uint64 seriesId,
    uint256 bTokenAmount,
    uint256 collateralMaximum
  ) external minTradeSize(bTokenAmount) returns (uint256)
```

Purchases `bTokenAmount` of [`bToken`](glossary.md#bToken) from the AMM, but will revert if the premium calculated by the AMM exceeds `collateralMaximum`. The return value is the premium (in units of [collateral token](glossary.md#collateral-token)) the caller pays. Prior to calling this function, the caller must [approve](glossary.md#erc20-approval) the AMM for the `collateralMaximum` amount. See [`MinterAmm.bTokenGetCollateralIn`](#bTokenGetCollateralIn) for how the AMM calculates the premium.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `bTokenAmount`  | uint256  | The amount of `bToken` the caller wishes to purchase                       |
| `collateralMaximum`  | uint256  | The maximum amount of collateral the caller wishes to pay for the `bToken`. The actual premium can exceed this value due to [slippage](glossary.md#slippage)                       |

#### bTokenSell

```solidity
  function bTokenSell(
    uint64 seriesId,
    uint256 bTokenAmount,
    uint256 collateralMinimum
  ) external minTradeSize(bTokenAmount) returns (uint256)
```

Sells `bTokenAmount` of [`bToken`](glossary.md#bToken) to the AMM, but will revert if the premium calculated by the AMM goes below `collateralMinimum`. The return value is the premium (in units of [collateral token](glossary.md#collateral-token)) the caller receives. Prior to calling this function, the caller must [approve](glossary.md#erc20-approval) the AMM for the `bTokenAmount`. See [`MinterAmm.bTokenGetCollateralOut`](#bTokenGetCollateralOut) for how the AMM calculates the premium.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `bTokenAmount`  | uint256  | The amount of `bToken` the caller wishes to purchase                       |
| `collateralMinimum`  | uint256  | The minimum amount of collateral the caller wishes to receive for selling their `bToken`, The actual premium can go below this value due to [slippage](glossary.md#slippage)                       |

#### wTokenSell

```solidity
  function wTokenSell(
    uint64 seriesId,
    uint256 wTokenAmount,
    uint256 collateralMinimum
  ) external minTradeSize(wTokenAmount) returns (uint256)
```

Sells `wTokenAmount` of [`wToken`](glossary.md#wToken) to the AMM, but will revert if the premium calculated by the AMM goes below `collateralMinimum`. The return value is the premium (in units of [collateral token](glossary.md#collateral-token)) the caller receives. Prior to calling this function, the caller must [approve](glossary.md#erc20-approval) the AMM for the `wTokenAmount`. See [`MinterAmm.wTokenGetCollateralOut`](#wTokenGetCollateralOut) for how the AMM calculates the premium.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                       |
| `wTokenAmount`  | uint256  | The amount of `wToken` the caller wishes to purchase                       |
| `collateralMinimum`  | uint256  | The minimum amount of collateral the caller wishes to receive for selling their `wToken`, The actual premium can go below this value due to [slippage](glossary.md#slippage)                       |

#### setVolatilityFactor

```solidity
  function setVolatilityFactor(uint256 _volatilityFactor) public onlyOwner
```

Sets a new volatility factor for this AMM to use when pricing option premiums. See [`MinterAmm.calcPrice`](#calcPrice) for how it is used.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_volatilityFactor`  | uint256  | The new volatility factor, equal to `0.4 * implied_vol * sqrt(num_seconds_in_year)`                       |

#### updateImplementation

```solidity
  function updateImplementation(
    address _newImplementation
  ) external onlyOwner
```

Updates this `MinterAmm`'s [logic contract](glossary.md#logic-contract). The SIREN protocol's contracts use the [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822) standard for implementing upgradeable contracts. This allows us to update vulnerable contracts and keep users' [option tokens](glossary.md#option-tokens) safe. When the SIREN protocol has reached a certain level of stability, we can remove these safety guards and ensure no one on the Siren team can swap out the smart contract functionality.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newImplementation`  | address  | The address of the new logic contract to use for the `MinterAmm`'s function implementations                      |

#### transferOwnership

```solidity
  function transferOwnership(address newOwner) external onlyOwner
```

Removes ownership from the current [owner](glossary.md#owner) and assigns ownership to the `newOwner` address. See [the `onlyOwner` modifier](#onlyOwner) for the permissions granted to the protocol admin.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `newOwner`  | address  | The address of the new admin for this contract                      |

[^1]: It is currently not possible for the AMM to buy `wToken` because of a technical blocker in the SIREN protocol's [Black-Scholes](glossary.md#black-scholes-price-function) approximation algorithm. The algorithm over-prices `bToken`'s and underprices `wToken`'s, and so if the AMM were to sell `wToken`'s then through [arbitrage](glossary.md#arbitrage) the AMM would quickly be drained of all `wToken`'s. For that reason, we cannot add functionality for selling `wToken`'s to the AMM
