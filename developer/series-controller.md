# SeriesController

## Polygon Contract Address

[View on Polygonscan](https://polygonscan.com/address/0x716c543b39a85aac0240ba7ed07e79f06e1fed48)

## Source Code

[`SeriesController.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/series/SeriesController.sol)

## Overview

The `SeriesController` implements all of the logic involving the lifecycle of [option tokens](glossary.md#option-tokens). It uses the `ISeriesController.Series` struct to represent all [option series](glossary.md#series).

Any contract wishing to redeem SIREN [option tokens](glossary.md#option-tokens) will need the `SeriesController` [contract address](./series-controller.md#mainnet-contract-address).

`SeriesController.mintOptions` can be used to mint onchain [Call](glossary.md#call-option) and [Put](glossary.md#put-option) options. The call to `SeriesController.mintOptions` locks `X` amount of collateral token to a [Series](glossary.md#series) and in return mints `X` [`bToken`](glossary.md#bToken) and `X` [`wToken`](glossary.md#wToken) to the caller. This `bToken` can then be sold at its [premium](glossary.md#premium) to those wishing to purchase `ERC20` options, and if after expiry the Series is [in the money](glossary.md#in-the-money-option) then the `bToken` can be redeemed for a portion of the `Series'` locked collateral. The `wToken` can be kept until the Series expires at which point the remaining locked collateral can be claimed. The `SeriesController` exposes functions for redeeming the `bToken` (`SeriesController.exerciseOption`) and `wToken` (`SeriesController.claimCollateral`).

Since `X` `bToken` + `X` `wToken` is worth `X` collateral token [^1], there is a function `SeriesController.closePosition` which acts like the inverse of `SeriesController.mintOptions`; `X` `bToken` + `X` `wToken` can be burned in return for the `X` collateral token locked in the Series.

## Functions

### Modifiers

#### onlyOwner

```solidity
  modifier onlyOwner() {
    require(
        hasRole(DEFAULT_ADMIN_ROLE, msg.sender),
        "SeriesController: Caller is not the owner"
    );

    _;
  }
```

Certain functions can only be called by the SIREN protocol owner multisig address. The functions [`updateImplementation`](#updateImplementation) and [`transferOwnership`](#transferOwnship) exist so that the owners can update the contracts in the event of a critical vulnerability that puts user's funds at risk.

### View/Pure Functions

#### latestIndex

```solidity
  function latestIndex(
  ) public view returns (uint64)
```

Returns the index to be used for the next call to `SeriesController.createSeries`. Each Series uses a monotonically incrementing `uint64` as an index into the array of Series. To read the fields of any single Series struct, use [the `series` function](#series).

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

Returns the value of `8`, which is the number of leading 0's in all of the oracle price values used by the `SeriesController`. More specifically, all of the current market prices returned by `SeriesController.getSettlementPrice` and `SeriesController.getCurrentPrice` use `8` leading [decimals](glossary.md#decimals). For example, if the current price of `WBTC`[TODO glossary] is $34,000, then the value returned by `SeriesController.getCurrentPrice` for a `WBTC` Call Series would be `34_000 * 1e8`.

#### state

```solidity
  function state(
    uint64 seriesId,
  ) public view returns (ISeriesController.SeriesState)
```

Returns the state of given Series, which depends on the current `block.timestamp`. If `block.timestamp` is before the Series' [expiration date](glossary.md#expiration-date), then it is in the state `OPEN`, otherwise it is in the state `EXPIRED`. Only certain functions can be executed when the state is `OPEN`, and some only when the state is `EXPIRED`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |

#### series

```solidity
  function series(
    uint64 seriesId,
  ) external view override returns (ISeriesController.Series memory)
```

Returns a specific Series by its ID. This struct contains information about the [option series](glossary.md#series).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |

#### calculateFee

```solidity
  function calculateFee(
    uint256 amount,
    uint16 basisPoints
  ) public pure override returns (uint256)
```

Returns the fee amount in units of [collateral token](glossary.md#collateral-token) when exercising [`bToken`](#exerciseOption), claiming [`wToken](#claimCollateral), or [closing a position](#closePosition).

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

Returns the payout amount for exercising `_bTokenAmount` of this Series's [`bToken`](glossary.md#bToken), as well as the exercise fee, both in units of [collateral token](glossary.md#collateral-token). See the documentation section on [protocol math](TODO point to this) for how the exercise payout is calculated.

This is useful to call before a call to [`SeriesController.exerciseOption](#exerciseOption) so you can see what the payout will be prior to exercising the `bToken`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |
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

Returns the payout amount for claiming `_wTokenAmount` of this Series's [`wToken`](glossary.md#wToken), as well as the exercise fee, both in units of [collateral token](glossary.md#collateral-token). See the documentation section on [protocol math](TODO point to this) for how the claim payout is calculated.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |
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

Returns the strike price of this Series. For [Calls](glossary.md#call-option) this is the price above which the [`bToken`](glossary.md#bToken) holder's option is [ITM](glossary.md#in-the-money), and for [Puts](glossary.md#put-option) this is the price below which the [`bToken`](glossary.md#bToken) holder's option is [ITM](glossary.md#in-the-money). The strike price always has 8 [decimals](glossary.md#decimals). For example, a `strikePrice` of `3400000000000` indicates the price is $34,000.

The majority of SIREN options denominate prices (including strike prices) in units of [USDC](glossary.md#usdc), but in the future the protocol might use different denominations (e.g. units of ETH) for prices.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### expirationDate

```solidity
  function expirationDate(
    uint64 _seriesId,
  ) external view override returns (uint40)
```

Returns the expiration date of the Series, in units of block time (i.e. seconds past epoch). Prior to this date the Series is in the `OPEN` state and can mint options and close positions, but cannot redeem [option tokens](glossary.md#option-tokens). At and after this date the Series is in the `EXPIRED` state and can redeem option tokens, but can no longer mint or close out positions.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### underlyingToken

```solidity
  function underlyingToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the `ERC20` token whose price determines the (glossary.md#moneyness) of the Series. For example, both WBTC Call [option series](glossary.md#series) and WBTC Put option series share [WBTC](glossary.md#wbtc) as their underlying token, because the price of WBTC determines whether the Series is [in](glossary.md#in-the-money-option) or [out of](glossary.md#out-of-the-money-option) the money. If this is a [Call](glossary.md#call-option) Series then this will be the same token as the [collateralToken](#collateralToken).


##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### priceToken

```solidity
  function priceToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the `ERC20` token used to denominate the [strike price](#strikePrice). This will almost always be [USDC](glossary.md#usdc), but in the future could be different if we want to denominate prices in something other than USDC. If this a [Put](glossary.md#put-option) Series then this will be the same token as the [collateralToken](#collateralToken)

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### collateralToken

```solidity
  function collateralToken(
    uint64 _seriesId,
  ) external view override returns (address)
```

Returns the address of the `ERC20` token used to underwrite this `Series'` option positions. This token will be used to denominate all option prices in the [`MinterAmm`](minter-amm.md#overview)

If this is a [Call](glossary.md#call-option) Series then the collateral token will be equal to the [underlyingToken](#underlyingToken), because a Call gives the holder the right to **buy** the underlying. And if this is a [Put](glossary.md#put-option) Series then the collateral token will be equal to the [priceToken](#priceToken), because a Put gives the holder the right to **sell** the underlying.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### wTokenIndex

```solidity
  function wTokenIndex(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the [`ERC1155`](glossary.md#erc1155) token index of the [`wToken`](glossary.md#wToken) for this Series. This index can then be used on the [`ERC1155Controller`](erc1155-controller.md) to query for `wToken` balances.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### bTokenIndex

```solidity
  function bTokenIndex(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the [`ERC1155`](glossary.md#erc1155) token index of the [`bToken`](glossary.md#bToken) for this Series. This index can then be used on the [`ERC1155Controller`](erc1155-controller.md) to query for `bToken` balances.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### isPutOption

```solidity
  function isPutOption(
    uint64 _seriesId,
  ) external view override returns (bool)
```

Returns true if this Series is for a [Put](glossary.md#put-option) option, and false if it's for a [Call](glossary.md#call-option) option.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |
                      |

#### getSeriesERC20Balance

```solidity
  function getSeriesERC20Balance(
    uint64 _seriesId,
  ) external view override returns (uint256)
```

Returns the amount of collateral token locked in this Series. The amount of locked tokens increases with every call to [`SeriesController.mintOptions`](#mintOptions) and decreases with every call to [`SeriesController.exerciseOption`](#exerciseOption), [`SeriesController.claimCollateral`](#claimCollateral), and [`SeriesController.closePosition`](#closePosition). This function is also useful for calculating part of the TVL (Total Value Locked) in the protocol.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

#### getCollateralPerOptionToken

```solidity
  function getCollateralPerOptionToken(
    uint64 _seriesId,
    uint256 _optionTokenAmount
  ) public view override returns (uint256)
```

Returns the amount of [collateral token](glossary.md#collateral-token) locked in the Series for every `_optionTokenAmount` of [option tokens](glossary.md#option-tokens). For [Call](glossary.md#call-option) options this is simply equal to `_optionTokenAmount`; 1 `bToken` or 1 `wToken` are backed by 1 collateral token. However, for [Puts](glossary.md#put-option) it's slightly more complicated. Because a Put option gives the `bToken` holder the right to **sell** the [underlying token](glossary.md#underlying-token) at the [strike price](glossary.md#strike-price), 1 `bToken` or 1 `wToken` must be backed by `1 * strike_price * decimals_coefficient` amount of collateral token. The `decimals_coefficient` is term which divides out the [price token](glossary.md#price-token) and underlying token [decimals](glossary.md#decimals), leaving only the correct decimals of the collateral token remaining.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |
| `_optionTokenAmount`  | uint256  | The amount of `wToken` or `bToken`                         |

#### getSettlementPrice

```solidity
  function getSettlementPrice(
    uint64 _seriesId
  ) external view override returns (bool, uint256)
```

Returns a tuple of 2 values, the first indicates whether or not the [settlement price](glossary.md#settlement-price) has been set by the [settlement bot](glossary.md#settlement-bot) and second value is the settlement price, or `0` if it has not been set yet. A Series' settlement price is set by an offchain bot as soon as that Series expires. The settlement price will have 8 [decimals](glossary.md#decimals), and use the same units as the [priceToken](glossary.md#price-token). For example, if the settlement price of a certain Series is $34,000, then `settlement_price = 34_000 * 1e8`.

##### Setting the Settlement Price

The settlement price will be equal to `0` if the current [block timestamp](https://ethereum.stackexchange.com/questions/11060/what-is-block-timestamp) is prior to the [Series](glossary.md#series) [expiration date](glossary.md#expiration-date). It's also possible the settlement date will be equal to `0` for a small amount of time (1-15 minutes) after the expiration date. This is because the [EVM](https://ethereum.org/en/developers/docs/evm/) does not provide any functionality for automatically calling contract functions on a predefined schedule (such as [cron](https://en.wikipedia.org/wiki/Cron)), and so an offchain process must send a transaction calling [`PriceOracle.setSettlementPrice`](price-oracle.md#setSettlementPrice) and set the current underlying token's price manually. The EVM gives no guarantees on when a transaction will be included in the blockchain, so the protocol cannot guarantee the settlement price will be set exactly at the Series' expiration date.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_seriesId`  | uint64  | The ID of the Series                         |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `isSet`          | bool   | true if the Series price at expiration (the settlement date) has been set, false otherwise  |
| `settlementPrice` | uint256 | The price of the Series' underlying token at the time of the Series' expiration date |

#### getCurrentPrice

```solidity
  function getCurrentPrice(
    address underlyingToken,
    address priceToken
  ) public view override returns (uint256)
```

Returns the current price of the [underlying token](glossary.md#underlying-token) denominated in units of the [priceToken](glossary.md#price-token). For example, if the current price of `WBTC`[TODO glossary] was $34,000, then the value returned by `SeriesController.getCurrentPrice` for a `WBTC` Series would be `34_000 * 1e8`. This price is fetched via the [`PriceOracle`](price-oracle.md#overview), which in turn fetches it from an onchain oracle.

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

Creates one or more Series structs with the given parameters. This function has the `onlyOwner` modifier so it can only be called by the [protocol admins](glossary.md#owner).

The choice of [underlyingToken](glossary.md#underlying-token) determines the `ERC20` token the Series will be for (i.e. the token whose price movements will determine the [long](https://www.investopedia.com/terms/l/long.asp) and [short](https://www.investopedia.com/terms/s/short.asp) payouts). The [priceToken](glossary.md#price-token) will be the token used to denominate the price in (e.g. for a WBTC option denominated in USDC, the underlying token will be WBTC and the price token will be USDC, but for a WBTC option denominated in ETH, the underlying token will be WBTC and the price token will be ETH). The [collateralToken](glossary.md#collateral-token) determines the `ERC20` token used to underwrite the [covered](glossary.md#covered-options) options. For [Put](glossary.md#put-option) options the `collateralToken` must be equal to the `priceToken`, and for [Call](glossary.md#call-option) options the `collateralToken` must be equal to the `underlyingToken`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_tokens`  | ISeriesController.Tokens calldata  | The `underlyingToken`, `priceToken`, and `collateralToken` for these Series                        |
| `_strikePrices`  | uint256[]  | An array of the [strike prices](glossary.md#strike-price) at which the Series will become [in the money](glossary.md#in-the-money-option)                       |
| `_expirationDates`  | uint40[]  | An array of the block timestamps these Series will expire. These must be after the current block timestamp                       |
| `_restrictedMinters`  | address[]  | An array of [`MinterAmm`](minter-amm.md) addresses which will be the only AMM's permitted to mint [option tokens](glossary.md#option-tokens) for these Series                       |
| `_isPutOption`  | bool  | `true` if these Series should be [Put](glossary.md#put-option) options, and false if they should be [Call](glossary.md#call-option) options                       |

#### mintOptions

```solidity
  function mintOptions(
    uint64 _seriesId,
    uint256 _optionTokenAmount
  ) external override whenNotPaused
```

Creates new `_optionTokenAmount` amount of a given Series's `wTokens` and `bTokens` by locking collateral in that Series. The [option tokens](glossary.md#option-tokens) are transferred to the caller. One of the fundamental formulas for the SIREN protocol is:

`X collateralToken ==> X wToken and X bToken`

or, for every `X` amount of [collateral token](glossary.md#collateral-token) a minter locks in the protocol, the minter receives `X` amount of `wToken` and `X` amount of `bToken`.

The [`MinterAmm`](minter-amm.md) contracts are the only accounts permitted to call this function. They use this to mint additional option tokens using [LP](glossary.md#liquidity-providers) liquidity and sell them to traders.

The exact amount of collateral required depends on whether or not the given Series is a [Put](glossary.md#put-option) or a [Call](glossary.md#call-option) option. See [the function `SeriesController.getCollateralPerOptionToken`](#getCollateralPerOptionToken) for more details on computing the exact amount of collateral needed to mint a given amount of option tokens.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |
| `_optionTokenAmount`  | uint256 | The amount of `wTokens` or `bTokens` to mint                       |

#### exerciseOption

```solidity
  function exerciseOption(
    uint64 _seriesId,
    uint256 _bTokenAmount,
    bool _revertOtm
  ) external override whenNotPaused
```

Burns `_bTokenAmount` amount of [`bToken`](glossary.md#bToken) and transfers the `bToken`'s [payout](glossary.md#option-payouts) to the caller. The SIREN protocol are [cash-settled](glossary.md#cash-settled), so the payout for `bTokens` is calculated as some fraction of the [collateral token](glossary.md#collateral-token) locked in the Series. The payout of the `bToken` depends on the Series' [strike price](glossary.md#strike-price) and the price of the [underlying token](glossary.md#underlying-token). For [Puts](glossary.md#put-option), the payout increases as the underlying token's price _decreases_ and for [Calls](glossary.md#call-option) the payout increases as the underlying tokne's price _increases_. See the section [Protocol Math](TODO link) for the details on how the protocol calculates `bToken` payout.

Because SIREN's options are [European] style, this function can only be executed after the Series has [expired](glossary.md#expiration-date).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |
| `_bTokenAmount`  | uint256 | The amount of `bTokens` to exercise                       |
| `_revertOtm`  | bool | `true` if you want this function call to revert if the Series is [OTM](glossary.md#out-of-the-money), `false` if you want it to continue executing even if it's [OTM](glossary.md#out-of-the-money). Passing `true` will save gas but will require the calling context to handle the error (possibly resulting in the entire transaction reverting). Passing `false` will use more gas, but will effectively be a no-op and preclude the calling context from worrying about their transaction reverting                       |

#### claimCollateral

```solidity
  function claimCollateral(
    uint64 _seriesId,
    uint256 _wTokenAmount
  ) external override whenNotPaused
```

Burns `_wTokenAmount` amount of [`wToken`](glossary.md#wToken) and transfers the `wToken`'s [payout](glossary.md#option-payouts) to the caller. The SIREN protocol are [cash-settled](glossary.md#cash-settled), so the payout for `wTokens` is calculated as some fraction of the [collateral token](glossary.md#collateral-token) locked in the Series. The payout of the `wToken` depends on the Series' [strike price](glossary.md#strike-price) and the price of the [underlying token](glossary.md#underlying-token). For [Puts](glossary.md#put-option), the payout decreases as the underlying token's price _decreases_ and for [Calls](glossary.md#call-option) the payout decreases as the underlying token's price _increases_. See the section [Protocol Math](TODO link) for the details on how the protocol calculates `wToken` payout.

Because SIREN's options are [European] style, this function can only be executed after the Series has [expired](glossary.md#cash-settled).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |
| `_wTokenAmount`  | uint256 | The amount of `wTokens` to claim. It uses the same [decimals](glossary.md#decimals) as the Series' [underlying token](glossary.md#underlying-token)                       |

#### closePosition

```solidity
  function closePosition(
    uint64 _seriesId,
    uint256 _optionTokenAmount
  ) external override whenNotPaused
```

Burns `_optionTokenAmount` amount of [`wToken`](glossary.md#wToken) and [`bToken`](glossary.md#bToken), then transfers the equivalent amount of [collateral token](glossary.md#collateral-token) to the caller. For [Calls](glossary.md#call-option) the amount of collateral token received will be equal to `_optionTokenAmount`, and for [Puts](glossary.md#put-option) the amount of collateral token received will be equal to `_optionTokenAmount * strikePrice` (see the [getCollateralPerOptionToken function](#getCollateralPerOptionToken) for more details on this calculations).

This function can only be called while the Series' has not yet [expired](#glossary.md#expiration-date). If the Series has expired and you can't call `closePosition` but you still want your collateral, you can call [exerciseOption](#exerciseOption) and [claimCollateral](#claimCollateral) to convert your `bToken` and `wToken` to collateral token.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the Series                         |
| `_optionTokenAmount`  | uint256 | The amount of [option tokens](glossary.md#option-tokens) to burn in exchange for [collateral token](glossary.md#collateral-token). It uses the same [decimals](glossary.md#decimals) as the Series' [underlying token](glossary.md#underlying-token)                       |

#### updateImplementation

```solidity
  function updateImplementation(
    address _newImplementation
  ) external onlyOwner
```

Updates this `SeriesController`'s [logic contract](glossary.md#logic-contract). The SIREN protocol's contracts use the [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822) standard for implementing upgradeable contracts. This allows us to update vulnerable contracts and keep [LP's](glossary.md#liquidity-providers) liquidity and the collateral locked in Series safe. When the SIREN protocol has reached a certain level of stability, we can remove these safety guards and ensure no one on the Siren team can swap out the smart contract functionality.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newImplementation`  | address  | The address of the new logic contract to use for the `SeriesController`'s function implementations                      |

#### transferOwnership

```solidity
  function transferOwnership(address _newAdmin) external onlyOwner
```

Removes ownership from the current [owner](glossary.md#owner) and assigns ownership to the `_newAdmin` address. See [the `onlyOwner` modifier](#onlyOwner) for the permissions granted to the protocol admin.

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newAdmin`  | address  | The address of the new admin for this contract                      |

[^1]: See the article on [Protocol Math](TODO link to protocol math page)
