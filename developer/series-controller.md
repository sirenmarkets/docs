# SeriesController

## Mainnet Contract Address

TODO

## Overview

The `SeriesController` implements all of the logic involving the lifecycle of [option tokens](TODO glossary). It uses the `ISeriesController.Series` struct to represent all [option series](TODO glossary).

Any contract wishing to redeem SIREN [option tokens](TODO glossary) will need the `SeriesController` [contract address](./series-controller.md#mainnet-contract-address).

`SeriesController.mintOptions` can be used to lock `X` amount of collateral token to a [`Series`](TODO glossary) and in return mint `X` [`bToken`](TODO glossary) and `X` [`wToken`](TODO glossary). This `bToken` can then be sold at its [premium](TODO glossary) to those wishing to purchase ERC20 options, and if after expiry the `Series` is [in the money](TODO glossary) then the `bToken` can be redeemed for a portion of the `Series's` locked collateral The `wToken` can be kept until the `Series` expires at which point the remaining locked collateral can be claimed. The `SeriesController` exposes functions for redeeming the `bToken` (`SeriesController.exerciseOption`) and `wToken` (`SeriesController.claimCollateral`).

Since `X` `bToken` + `X` `wToken` is worth `X` collateral token [^1], there is a function `SeriesController.closePosition` which acts like the inverse of `SeriesController.mintOptions`; `X` `bToken` + `X` `wToken` can be burned in return for `X` collateral token.

## Functions

### View/Pure Functions

#### latestIndex

```solidity
  function latestIndex(
  ) public view returns (uint64)
```

Returns the index to be used for the next call to `SeriesController.createSeries`. Each `Series` is identified by a monotonically incrementing `uint64`.

#### erc1155Controller

```solidity
  function erc1155Controller(
  ) public view returns (address)
```

Returns the address of this `SeriesController`'s `ERC1155Controller`. This is the contract that implements all of the [`ERC1155`](https://eips.ethereum.org/EIPS/eip-1155) token functions.

#### priceDecimals

```solidity
  function priceDecimals(
  ) public view returns (uint8)
```

Returns the value of `8`, which is the number of leading 0's in all of the oracle price values used by the `SeriesController`. More specifically, all of the current market prices returned by `SeriesController.getSettlementPrice` and `SeriesController.getCurrentPrice` use `8` leading decimals. For example, if the current price of `WBTC`[TODO glossary] was $34,000, then the value returned by `SeriesController.getCurrentPrice` for a `WBTC` Call `Series` would be `34_000 * 1e8`.

#### state

```solidity
  function state(
    uint64 seriesId,
  ) public view returns (ISeriesController.SeriesState)
```

Returns the state of given `Series`, which depends on the current `block.timestamp`. If `block.timestamp` is before the `Series`' [expiration date](TODO glossary), then it is in the state `OPEN`, otherwise it is in the state `EXPIRED`

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |

#### series

```solidity
  function series(
    uint64 seriesId,
  ) external view override returns (ISeriesController.Series memory)
```

Returns a specific `Series` by its ID. This struct contains information about the [option series](TODO glossary).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |

#### calculateFee

```solidity
  function calculateFee(
    uint256 amount,
    uint16 basisPoints
  ) public pure override returns (uint256)
```

Returns the fee amount in units of [collateral token](TODO glossary) when exercising [`bToken`](#exerciseOption), claiming [`wToken](#claimCollateral), or [closing a position](#closePosition).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `amount`  | uint256  | The amount of collateral token to take a percentage fee of                       |
| `basisPoints`  | uint16  | The fee percentage, expressed in basis points (100 basis points = 1%)                       |

#### getExerciseAmount

```solidity
  function getExerciseAmount(
    uint64 _seriesId,
    uint256 _bTokenAmount
  ) public view override returns (uint256, uint256)
```

Returns the payout amount for exercising `_bTokenAmount` of this `Series`'s [`bToken`](TODO glossary), as well as the exercise fee, both in units of [collateral token](TODO glossary). See the documentation section on [protocol math](TODO point to this) for how the exercise payout is calculated.

This is useful to call before a call to [`SeriesController.exerciseOption](#exerciseOption) so you can see what the payout will be prior to exercising the `bToken`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |
| `_bTokenAmount`  | uint256  | The amount of bToken to exercise                       |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `buyerShare`          | uint256   | The amount of collateral the exerciser receives  |
| `feeAmount` | uint256 | The amount of collateral the exerciser pays to the protocol for exercising |

#### getClaimAmount

```solidity
  function getClaimAmount(
    uint64 _seriesId,
    uint256 _wTokenAmount
  ) public view override returns (uint256, uint256)
```

Returns the payout amount for claiming `_wTokenAmount` of this `Series`'s [`wToken`](TODO glossary), as well as the exercise fee, both in units of [collateral token](TODO glossary). See the documentation section on [protocol math](TODO point to this) for how the claim payout is calculated.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |
| `_wTokenAmount`  | uint256  | The amount of wToken to claim                       |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `writerShare`          | uint256   | The amount of collateral the claimer receives  |
| `feeAmount` | uint256 | The amount of collateral the claimer pays to the protocol for claiming |

#### strikePrice

```solidity
  function strikePrice(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the strike price of this `Series`. For [Calls](TODO glossary) this is the price at which the [`bToken`](TODO glossary) holder may buy the underlying asset, and for [Puts](TODO glossary) this is the price at which the [`bToken`](TODO glossary) holder may sell the underlying asset. The strike price always has 8 decimals, and is expressed in USD. For example, a `strikePrice` of `3400000000000` indicates the price is $34,000.
##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### expirationDate

```solidity
  function expirationDate(
    uint64 _seriesId,
  ) external view override returns (uint40)
```

Returns the expiration date of the `Series`, in units of block time (i.e. seconds past epoch). Prior to this date the `Series` is in the `OPEN` state and can mint options and close positions, but cannot redeem [option tokens](TODO glossary). At and after this date the `Series` is in the `EXPIRED` state and can redeem option tokens, but can no longer mint or close out positions.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### underlyingToken

```solidity
  function underlyingToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the ERC20 token whose price determines the [moneyness](TODO glossary) of the `Series`. For example, both WBTC Call [option series](TODO glossary) and WBTC Put option series share [WBTC](TODO glossary) as their underlying token, because the price of WBTC determines whether the `Series` is in or out of the money. If this a [Call](TODO glossary) `Series` then this will be the same token as the [collateralToken](#collateralToken)

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### priceToken

```solidity
  function priceToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the ERC20 token used to denominate the [`strikePrice`](#strikePrice). This will almost always be [USDC](TODO glossary), but in the future could be different if we want to denominate prices in something other than USDC. If this a [Put](TODO glossary) `Series` then this will be the same token as the [collateralToken](#collateralToken)

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### collateralToken

```solidity
  function collateralToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the ERC20 token used to underwrite this `Series'` option positions. This token will be used to denominate all option prices in the [`MinterAmm`](TODO glossary)

If this is a [Call](TODO glossary) `Series` then the collateral token will be equal to the [underlyingToken](#underlyingToken), because a Call gives the holder the right to **buy** the underlying. And if this is a [Put](TODO glossary) `Series` then the collateral token will be equal to the [priceToken](#priceToken), because a Call gives the holder the right to **sell** the underlying.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### wTokenIndex

```solidity
  function wTokenIndex(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the [`ERC1155`](TODO glossary) token index of the [`wToken`](TODO glossary) for this `Series`. This index can then be used on the [`ERC1155Controller`](erc1155-controller.md) to query for `wToken` balances.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### bTokenIndex

```solidity
  function bTokenIndex(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the [`ERC1155`](TODO glossary) token index of the [`bToken`](TODO glossary) for this `Series`. This index can then be used on the [`ERC1155Controller`](erc1155-controller.md) to query for `bToken` balances.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### isPutOption

```solidity
  function isPutOption(
    uint64 _seriesId,
  ) external view override returns (bool)
```

Returns true if this `Series` is for a [Put](TODO glossary) option, and false if it's for a [Call](TODO glossary) option.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |
                      |

#### getSeriesERC20Balance

```solidity
  function getSeriesERC20Balance(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the amount of collateral token locked in this `Series`. The amount of locked tokens increases with every call to [`SeriesController.mintOptions`](#mintOptions) and decreases with every call to [`SeriesController.exerciseOption`](#exerciseOption), [`SeriesController.claimCollateral`](#claimCollateral), and [`SeriesController.closePosition`](#closePosition). This function is also useful for calculating part of the TVL (Total Value Locked) in the protocol.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

#### getCollateralPerOptionToken

```solidity
  function getCollateralPerOptionToken(
    uint64 _seriesId,
    uint256 _optionTokenAmount
  ) public view override returns (uint256)
```

Returns the amount of [collateral token](TODO glossary) locked in the `Series` for every `_optionTokenAmount` of [option tokens](TODO glossary). For [Call](TODO glossary) options this is simply equal to `_optionTokenAmount`; 1 `bToken` or 1 `wToken` are backed by 1 collateral token. However, for [Puts](TODO glossary) it's slightly more complicated. Becuase a Put option uses [USDC](TODO glossary) as the collateral token, and a Put option gives the `bToken` holder the right to **sell** the [underlying token](TODO glossary) at the [strike price](TODO glossary), 1 `bToken` or 1 `wToken` must be backed by `1 * strike_price * decimals_coefficient` amount of collateral token. The `decimals_coefficient` is term which divides out the [price token](TODO glossary) and underlying token decimals, leaving only the correct decimals of the collateral token remaining. 

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |
| `_optionTokenAmount`  | uint256  | The amount of `wToken` or `bToken`                         |

#### getSettlementPrice

```solidity
  function getSettlementPrice(
    uint64 _seriesId
  ) external view override returns (bool, uint256)
```

Returns a tuple of 2 values, the first indicates whether or not the [settlement price](TODO glossary) has been set by the [settlement bot](TODO glossary) and second value is the settlement price, or `0` if it has not been set yet. The settlement price will have 8 decimals, and use the same units as the [priceToken](TODO glossary). For example, if the settlement price of a certain `Series` is $34,000, then `strike_price = 34_000 * 1e8`.

##### Setting the Settlement Price

The settlement price will be equal to `0` if the current [block timestamp](TODO glossary) is prior to the [`Series`](TODO glossary) [expiration date](TODO glossary). It's also possible the settlement date will be equal to `0` for a small amount of time (1-15 minutes) after the expiration date. This is because the [EVM](TODO glossary) does not provide any functionality for automatically calling contract functions on a predefined schedule (such as [cron](https://en.wikipedia.org/wiki/Cron)), and so an offchain process (currently an AWS lambda function) must send a transaction calling `PriceController.setSettlementPrice` and set the current underlying token's price manually. The EVM gives no guarantees on when a transaction will be included in the blockchain, so the protocol cannot guarantee the settlement price will be set exactly at the `Series`' expiration date.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the `Series`                         |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `isSet`          | bool   | true if the `Series` price at expiration (the settlement date) has been set, false otherwise  |
| `settlementPrice` | uint256 | The price of the `Series`' underlying token at the time of the `Series`' expiration date |

#### getCurrentPrice

```solidity
  function getCurrentPrice(
    address underlyingToken,
    address priceToken
  ) public view override returns (uint256)
```

Returns the current price of the [underlying token](TODO glossary) in denominated in units of the [priceToken](TODO glossary). For example, if the current price of `WBTC`[TODO glossary] was $34,000, then the value returned by `SeriesController.getCurrentPrice` for a `WBTC` `Series` would be `34_000 * 1e8`. This price is fetched via the [`PriceOracle](TODO glossary), which in turn fetches it from an onchain [oracle](TODO glossary).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `underlyingToken`  | address  | An `ERC20` token                        |
| `priceToken`  | address  | An `ERC20` token                       |


### Mutating Functions

#### createSeries

```solidity
  function createSeries(
    ISeriesController.Tokens calldata _tokens,
    uint256[] calldata _strikePrices,
    uint40[] calldata _expirationDates,
    address[] calldata _restrictedMinters,
    bool _isPutOption
  ) external onlyOwner
```

Creates one or more `Series` structs with the given parameters. This function has the `onlyOwner` modifier so it can only be called by the [protocol admins](TODO glossary).

The choice of [underlyingToken](TODO glossary) determines the `ERC20` token the `Series` will be for (i.e. the token whose price movements will determine the [long](TODO glossary) and [short](TODO glossary) payouts). The [priceToken](TODO glossary) will be the coin to denominate the price in (e.g. for a WBTC option denominated in USDC, the underlying token will be WBTC and the price token will be USDC, but for a WBTC option denominated in ETH, the underlying token will be WBTC and the price token will be ETH). And the [collateralToken](TODO glossary) determines the `ERC20` token used to underwrite the [covered](TODO glossary) options. For [Put](TODO glossary) options the `collateralToken` must be equal to the `priceToken`, and for [Call](TODO glossary) options the `collateralToken` must be equal to the `underlyingToken`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_tokens`  | ISeriesController.Tokens calldata  | The `underlyingToken`, `priceToken`, and `collateralToken` for these `Series`                        |
| `_strikePrices`  | uint256[]  | An array of the [strike prices](TODO glossary) at which the `Series` will become [in the money](TODO glossary)                       |
| `_expirationDates`  | uint40[]  | An array of the block timestamps these `Series` will expire. These must be after the current block timestamp                       |
| `_restrictedMinters`  | address[]  | An array of [`MinterAmm`](minter-amm.md) addresses which will be the only AMM's permitted to mint [option tokens](TODO glossary) for these `Series`                       |
| `_isPutOption`  | bool  | `true` if these Series should be [Put](TODO glossary) options, and false if they should be [Call](TODO glossary) options                       |

#### mintOptions

```solidity
  function mintOptions(
    uint64 _seriesId,
    uint256 _optionTokenAmount
  ) external override whenNotPaused
```

Creates new `_optionTokenAmount` amount of a given `Series`'s `wTokens` and `bTokens` by locking collateral in that `Series`. The [option tokens](TODO glossary) are then sent to the `msg.sender`. One of the fundamental formulas for the SIREN protocol is:

`X collateralToken ==> X wToken and X bToken`

or, for every `X` amount of [collateral token](TODO glossary) a minter locks in the protocol, the minter receives `X` amount of `wToken` and `X` amount of `bToken`.

The [`MinterAmm`](minter-amm.md) contracts are the only accounts permitted to call this function, and they use this to mint additional option tokens using [LP](TODO glossary) liquidity and sell them to traders.

The exact amount of collateral required depends on whether or not the given `Series` is a [Put](TODO glossary) or a [Call](TODO glossary) option. See [the function `SeriesController.getCollateralPerOptionToken`](#getCollateralPerOptionToken) for more details on computing the exact amount of collateral needed to mint a given amount of option tokens.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |
| `_optionTokenAmount`  | uint256 | The amount of `wTokens` or `bTokens` to mint                       |

#### exerciseOption

```solidity
  function exerciseOption(
    uint64 _seriesId,
    uint256 _bTokenAmount,
    bool _revertOtm
  ) external override whenNotPaused
```

Burns `_bTokenAmount` amount of [`bToken`](TODO glossary) and transfers the `bToken`'s redemption value 

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |
| `_bTokenAmount`  | uint256 | The amount of `bTokens` to exercise                       |
| `_revertOtm`  | bool | `true` if you want this function call to revert if the `Series` is [OTM](TODO glossary), `false` if you want it to continue executing even if it's [ITM](TODO glossary). Passing `true` will save gas but will require the calling context to handle the error (possibly resulting in the entire transaction reverting). Passing `false` will use more gas, but will effectively be a no-op and preclude the calling context from worrying about their transaction reverting                       |

[^1]: See the article on [Protocol Math](TODO link to protocol math page)
