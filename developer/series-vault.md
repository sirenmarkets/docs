# SeriesVault

## Mainnet Contract Address

TODO

## Source Code

[`SeriesVault.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/series/SeriesVault.sol)

## Overview

The `SeriesVault` acts as a simple storage contract for the [`SeriesController's`](series-controller.md#overview) [`ERC20`](glossary.md#erc20) and [`ERC1155`](glossary.md#erc1155) tokens. Whenever the `SeriesController` receives collateral from minting options, it transfers the collateral to be stored in the `SeriesVault`. Currently the `SeriesVault` does not store any [option tokens](glossary.md#option-tokens), but a future upgrade to the protocol might enable this so that we can implement [spreads](https://www.investopedia.com/articles/active-trading/032614/which-vertical-option-spread-should-you-use.asp).

The `SeriesVault` and `SeriesController` are separate contracts because we wanted to separate the [Series](glossary.md#series) settlement layer logic (the `SeriesController`) from the Series collateral storage (the `SeriesVault`). This results in higher gas usage, but gives more flexibility for future changes to either contract.

## Functions

### Modifiers

#### onlySeriesController

```solidity
  modifier onlySeriesController() {
    require(
        msg.sender == controller,
        "SeriesVault: Sender must be the seriesController"
    );

    _;
  }
```

Certain functions can only be called by the associated [`SeriesController`](series-controller.md#overview) contract, such as the [`setERC20ApprovalForController`](#setERC20ApprovalForController) and [`setERC1155ApprovalForController`](#setERC1155ApprovalForController) functions. This is to ensure that no arbitrary address can approve themselves to transfer the tokens held in the `SirenVault`

#### onlyOwner

```solidity
  modifier onlyOwner() {
    require(owner() == _msgSender(), "Ownable: caller is not the owner");
    _;
  }
```

Certain functions can only be called by the SIREN protocol owner multisig address. The functions [`updateImplementation`](#updateImplementation) and [`transferOwnership`](#transferOwnship) exist so that the owners can update the contracts in the event of a critical vulnerability that puts user's funds at risk.

### Mutating Functions

#### setERC20ApprovalForController

```solidity
  function setERC20ApprovalForController(address erc20Token)
    external
    override
    onlySeriesController
```

Approves the caller to transfer `Math.pow(2, 256) - 1` amount of the [`ERC20`](https://eips.ethereum.org/EIPS/eip-20) token from the `SirenVault`. This is called by the [`SeriesController`](series-controller.md#overview) upon initialization.

The `SeriesController` uses this function to store the `ERC20` Series collateral tokens in the `SeriesVault`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `erc20Token`  | address  | The [`ERC20`](https://eips.ethereum.org/EIPS/eip-20) token address.                         |

#### setERC1155ApprovalForController

```solidity
  function setERC1155ApprovalForController(address erc1155Contract)
    external
    override
    onlySeriesController
    returns (bool)
```

Approves the caller to transfer any amount of the [`ERC1155`](https://eips.ethereum.org/EIPS/eip-1155) tokens from the `SirenVault`. This is called by the [`SeriesController`](series-controller.md#overview) upon initialization.

Currently this function is not used for anything, as the `SeriesController` does not store [option tokens](glossary.md#option-tokens) in the `SeriesVault`. However, a future extension to the protocol may make use of Vault-stored option tokens, so we expose this functionality for the future.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `erc1155Contract`  | address  | The [`ERC1155`](https://eips.ethereum.org/EIPS/eip-1155) token address.                         |

#### updateImplementation

```solidity
  function updateImplementation(
    address _newImplementation
  ) external onlyOwner
```

Updates this `SeriesVault`'s [logic contract](glossary.md#logic-contract). The SIREN protocol's contracts use the [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822) standard for implementing upgradeable contracts. This allows us to update vulnerable contracts and keep users' [option tokens](glossary.md#option-tokens) safe. When the SIREN protocol has reached a certain level of stability, we can remove these safety guards and ensure no one on the Siren team can swap out the smart contract functionality.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newImplementation`  | address  | The address of the new logic contract to use for the `SeriesVault`'s function implementations                      |

#### transferOwnership

```solidity
  function transferOwnership(address newOwner) public virtual onlyOwner
```

Removes ownership from the current [owner](glossary.md#owner) and assigns ownership to the `newOwner` address. See [the `onlyOwner` modifier](#onlyOwner) for the permissions granted to the protocol admin.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `newOwner`  | address  | The address of the new admin for this contract                      |
