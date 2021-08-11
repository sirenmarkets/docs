# AmmFactory

## Polygon Contract Address

[View on Polygonscan](https://polygonscan.com/address/0x0cdaa64b47474e02cdfbd811ec9fd2d265cd3a0a)

## Source Code

[`AmmFactory.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/amm/AmmFactory.sol)

## Overview

The AmmFactory is a relatively simple contract that allows an admin to deploy new [MinterAmm](glossary.md#minteramm) contracts. It emits the `AmmCreated` event upon every successful call to `MinterAmm.createAmm`, which can be used in the subgraph to query for all existing AMMs.

It is not very interesting for users of the SIREN protocol, because most of its functions contain the `onlyOwner` modifier and thus can only be called by protocol admins.

## Functions


### seriesController

```solidity
  function seriesController(
  ) public view returns (ISeriesController)
```

Returns the `AMMFactory`'s associated `SeriesController`. This is set on all AMM's the `AmmFactory` creates.

### amms

```solidity
  function amms(
    bytes32 input,
  ) public view returns (address)
```

Returns the address of an AMM earlier created by the `AmmFactory`

#### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `input`     | bytes32 | The keccak encoding of the AMM's underlying, price, and colalteral tokens |

### createAmm

```solidity
  function createAmm(
    address _sirenPriceOracle,
    IERC20 _underlyingToken,
    IERC20 _priceToken,
    IERC20 _collateralToken,
    uint16 _tradeFeeBasisPoints
  ) external onlyOwner
```

Deploys a new `MinterAmm` contract with the given parameters. Can only be called by the owner of the `AmmFactory`.
#### Parameters

| Name                  | Type    | Description                                                             |
| :-------------------- | :------ | :---------------------------------------------------------------------- |
| `_sirenPriceOracle`   | address | The address of the `PriceOracle` to use for fetching market prices      |
| `_underlyingToken`    | IERC20  | The `ERC20` token whose price movements determine the option's [moneyness](glossary.md#moneyness)  |
| `_priceToken`         | IERC20  | The `ERC20` token whose units are used for all prices (often USDC)        |
| `_collateralToken`    | IERC20  | The `ERC20` token used as the Series collateral (one of `_underlyingToken` or `_priceToken`) |
| `_tradeFeeBasisPoints`| uint16  | The fees to charge on option token trades (100 = 1%)                    |
