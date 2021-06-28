# SeriesController

## Overview

The `SeriesController` implements all of the logic involving the lifecycle of [option tokens](TODO glossary). It uses the `ISeriesController.Series` struct to represent all [option series](TODO glossary). And it uses the [Open Zeppelin ERC1155PresetMinterPauser contract](https://docs.openzeppelin.com/contracts/4.x/erc1155#Presets) to perform all [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) token functions.

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

Returns the value of `8`, which is the number of leading 0's in all of the oracle price values used by the `SeriesController`. More specifically, all of the current market prices returned by `SeriesController.getSettlementPrice` and `SeriesController.getCurrentPrice` use `8` leading decimals. For example, if the current price of `WBTC`[TODO glossary] was $35_000, then the value returned by `SeriesController.getCurrentPrice` for a `WBTC` Call `Series` would be `3500000000000`.

#### state

```solidity
  function state(
    uint64 seriesId,
  ) public view returns (ISeriesController.SeriesState)
```

Returns the state of given `Series`, which depends on the current `block.timestamp`. If `block.timestamp` is before the `Series`' [expiration date](TODO glossary), then it is in the state `OPEN`, otherwise it is in the state `EXPIRED`

##### Parameters:

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |

#### state

```solidity
  function state(
    uint64 seriesId,
  ) public view returns (ISeriesController.SeriesState)
```

Returns the state of given `Series`, which depends on the current `block.timestamp`. If `block.timestamp` is before the `Series`' [expiration date](TODO glossary), then it is in the state `OPEN`, otherwise it is in the state `EXPIRED`

##### Parameters:

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `seriesId`  | uint64  | The ID of the `Series`                         |


TODO: rest of functions


[^1]: See the article on [Protocol Math](TODO link to protocol math page)