# PriceOracle

## Polygon Contract Address

[View on Polygonscan](https://polygonscan.com/address/0xcbcae36b425ce1a94d055602b3b514aed976c383)


## Source Code

[`PriceOracle.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/series/PriceOracle.sol)

## Overview

The `PriceOracle` performs two necessary functions for the SIREN protocol:

First and most importantly, the `PriceOracle` stores the settlement price at expiration for every [Series'](series.md#overview) underlying asset. On every Friday at 8am UTC, an offchain bot process is scheduled to fetch and set the current price for every asset traded on the SIREN protocol. Since all [`Series'`](series.md#overview) expiration dates are aligned to Friday 8am UTC, this allows any expired Series to fetch the settlement price of the underlying token at expiration and use it along with the `Series'` [strike price](glossary.md#strike-price) to calculate the payouts for [`bToken`](glossary.md#bToken) and [`wToken`](glossary.md#wToken) holders.

Second, the `PriceOracle` gives all other protocol contracts a single place to access onchain price data. Currently we use only Chainlink's oracles, but other's like Uniswap V3 could easily be used. Contracts like the [`MinterAmm`](minter-amm.md#overview) and [`SeriesController`](series-controller.md#overview) use the `PriceOracle` to fetch the current price for the tokens they use. It simplifies the protocol when all other contracts need only have a single address with a known interface for fetching onchain prices.

## Functions

### Modifiers

#### onlyOwner

```solidity
  modifier onlyOwner() {
    require(
        hasRole(DEFAULT_ADMIN_ROLE, msg.sender),
        "ERC1155Controller: Caller is not the owner"
    );

    _;
  }
```

Certain functions can only be called by the SIREN protocol owner multisig address. The function [`addTokenPair](#addTokenPair) exist so that as [`MinterAmm`s](minter-amm.md#overview) with new [underlying tokens](glossary.md#underlying-token) and [price tokens](glossary.md#price-token) are deployed, we can add the correct price oracles for these token pairs. If this function was callable by any address, than a malicious address could set their own malicious price oracle, and attack the protocol.

The functions [`updateImplementation`](#updateImplementation) and [`transferOwnership`](#transferOwnship) exist so that the owners can update the contracts in the event of a critical vulnerability that puts user's funds at risk.


### View/Pure Functions

#### getSettlementPrice

```solidity
  function getSettlementPrice(
    address underlyingToken,
    address priceToken,
    uint256 settlementDate
  ) external view override returns (bool, uint256)
```

Returns the [settlement price](glossary.md#settlement-price) of the [underlying token](glossary.md#underlying-token) denominated in units of the [priceToken](glossary.md#price-token) at the given `settlementDate`. The [`SeriesController](series-controller.md#overview) uses this price to compute the [payouts](glossary.md#option-payouts) for [option tokens](glossary.md#option-tokens). Only token pairs which have been previously added by an admin-only call to [`addTokenPair`](#addTokenPair) will execute successfully.

The settlement price is set by an offchain bot process every Friday 8am UTC, because all Series [expiration dates](glossary.md#expiration-date) are aligned to Friday 8am UTC. See the [setting the settlement price section](./series-controller.md#setting-the-settlement-price) for more details on how this price is set.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `underlyingToken`  | address  | An `ERC20` token address                        |
| `priceToken`  | address  | An `ERC20` token address                       |
| `settlementDate`  | uint256  | A block timestamp (i.e. seconds after epoch) that must be aligned to 8am Friday UTC                       |

##### Return Values

| Name                            | Type    | Description                                         |
| :------------------------------ | :------ | :-------------------------------------------------- |
| `isSet`          | bool   | `true` if a call to [`setSettlementPrice](#setSettlementPrice) has previously been executed and set the price to a non-zero value, otherwise it is `false` |
| `settlementPrice` | uint256 | The price of the given token pair at the given `settlementDate`  |

#### getCurrentPrice

```solidity
  function getCurrentPrice(
    address underlyingToken,
    address priceToken
  ) public view override returns (uint256)
```

Returns the current price of the [underlying token](glossary.md#underlying-token) denominated in units of the [priceToken](glossary.md#price-token). For example, if the current price of [`WBTC`](glossary.md#wbtc) was $34,000, then the value returned by `PriceOracle.getCurrentPrice` for a `WBTC` Series would be `34_000 * 1e8`. Only token pairs which have been previously added by an admin-only call to [`addTokenPair`](#addTokenPair) will execute successfully. The `PriceOracle` fetches this price from an onchain oracle, which for now our all [Chainlink Oracles](https://docs.chain.link/docs/reference-contracts/).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `underlyingToken`  | address  | An `ERC20` token address                        |
| `priceToken`  | address  | An `ERC20` token address                       |

#### getFriday8amAlignedDate

```solidity
  function getFriday8amAlignedDate(uint256 _timestamp)
    public
    pure
    override
    returns (uint256)
```

Returns the block timestamp of the first Friday 8am UTC date prior to `_timestamp`. The [`SeriesController`](./series-controller.md#overview) uses this function to ensure the [expiration dates](glossary.md#expiration-date) of each of its Series is aligned to Friday 8am UTC. The SIREN protocol uses a single date for each week's expiration dates in order to standardize the times at which its Series expire.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_timestamp`  | uint256  | The block timestamp which will be used to find the prior Friday 8am UTC aligned block timestamp                        |

### Mutating Functions

#### setSettlementPrice

```solidity
  function setSettlementPrice(address underlyingToken, address priceToken)
    external
    override
```

Sets all prior [Friday 8am UTC] settlement dates that have not yet been set by first fetching the current price of the `underlyingToken` denominated in units of `priceToken`, and setting all previous unset settlement dates. This function is intended to be called immediately after every Friday 8am UTC for each of the `PriceOracle`'s [token pairs](#addTokenPair), so that every Series that has their [expiration date](glossary.md#expiration-date) on that Frodau 8am UTC date will have the correct settlement date set. For now, an offchain bot process managed by the Siren team is scheduled to call this function for each token pair.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `underlyingToken`  | address  | An `ERC20` token address                        |
| `priceToken`  | address  | An `ERC20` token address                        |

#### addTokenPair

```solidity
  function addTokenPair(
    address underlyingToken,
    address priceToken,
    address oracle
  ) external onlyOwner
```

Sets the `oracle` as the price oracle to use for the given [token pair](#addTokenPair) of `underlyingToken` and `priceToken`. This function uses the `onlyOwner` modifier in order to prevent an malicious address from setting a malicious oracle for a given token pair. Once a token pair has been added with `addTokenPair`, the [`getCurrentPrice`](#getCurrentPrice), [`setSettlementPrice`](#setSettlementPrice) and  [`getSettlementPrice`](#getSettlementPrice) functions can be called with the given token pair.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `underlyingToken`  | address  | An `ERC20` token address                        |
| `priceToken`  | address  | An `ERC20` token address                        |
| `oracle`  | address  | The address of the onchain oracle to use when fetching current prices for this token pair                        |

#### updateImplementation

```solidity
  function updateImplementation(
    address _newImplementation
  ) external onlyOwner
```

Updates this `PriceOracle`'s [logic contract](glossary.md#logic-contract). The SIREN protocol's contracts use the [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822) standard for implementing upgradeable contracts. This allows us to update vulnerable contracts and keep users' [option tokens](glossary.md#option-tokens) safe. When the SIREN protocol has reached a certain level of stability, we can remove these safety guards and ensure no one on the Siren team can swap out the smart contract functionality.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newImplementation`  | address  | The address of the new logic contract to use for the `PriceOracle`'s function implementations                      |

#### transferOwnership

```solidity
  function transferOwnership(address newOwner) public virtual onlyOwner
```

Removes ownership from the current [owner](glossary.md#owner) and assigns ownership to the `newOwner` address. See [the `onlyOwner` modifier](#onlyOwner) for the permissions granted to the protocol admin.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `newOwner`  | address  | The address of the new admin for this contract                      |
