# ERC1155Controller

## Mainnet Contract Address

TODO

## Source Code

[`ERC1155Controller.sol`](https://github.com/sirenmarkets/core/blob/master/contracts/series/ERC1155Controller.sol)

## Overview

The `ERC1155Controller` implements all of the functionality needed to mint, burn, approve, and transfer the [ERC1155](glossary.md#erc1155) [option tokens](glossary.md#option-tokens) (e.g. [`bTokens`](glossary.md#btoken) and [`wTokens`](glossary.md#wToken)). It inherits from the [Open Zeppelin ERC1155PresetMinterPauser contract](https://docs.openzeppelin.com/contracts/4.x/erc1155#Presets), which implements functions for minting, burning, approving, and transferring tokens, either as a single token or multiple tokens as a batch.

Any contract wishing to transfer SIREN [option tokens](glossary.md#option-tokens) will need the `ERC1155Controller` [contract address](#mainnet-address).

We chose to use the [ERC1155](glossary.md#erc1155) interface instead of the more popular [`ERC20`](https://eips.ethereum.org/EIPS/eip-20) standard because it provides us a more gas efficient way to create [option series](glossary.md#series). Rather than deploy a new `ERC20` **contract** with every new Series we create, we add an additional **id** to the same `ERC1155` contract. It also standardizes the [`ERC1155.setApprovalForAll`](https://eips.ethereum.org/EIPS/eip-1155#approval), which allows an owner address to make only a single onchain transaction in order to approve an operator account to transfer any/all of the [option tokens](glossary.md#option-tokens), which is more gas efficient than an onchain transaction for each option token.

## Functions

### Modifiers

#### onlySeriesController

```solidity
  modifier onlySeriesController() {
    require(
        msg.sender == controller,
        "ERC1155Controller: Sender must be the seriesController"
    );

    _;
  }
```

Certain functions can only be called by the associated [`SeriesController`](series-controller.md#overview) contract, such as the [`mint`](#mint) and [`burn`](#burn) functions. This is to ensure that no arbitrary address can inflate or deflate the supply of [option tokens](glossary.md#option-tokens).

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

Certain functions can only be called by the SIREN protocol owner multisig address. The functions [`updateImplementation`](#updateImplementation) and [`transferOwnership`](#transferOwnship) exist so that the owners can update the contracts in the event of a critical vulnerability that puts user's funds at risk.

### View/Pure Functions

#### uri

```solidity
  function uri(
    uint256 _id
  ) public view override returns (string memory)
```

This implementation returns the same URI for *all* token types. It relies on the token type ID substitution mechanism https://eips.ethereum.org/EIPS/eip-1155#metadata[defined in the EIP]. Clients calling this function must replace the `\{id\}` substring with the actual token type ID.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_id`  | uint256  | The ID of the ERC1155 token                        |

#### balanceOf

```solidity
  function balanceOf(
    address account,
    uint256 id
  ) public view override returns (uint256)
```

Returns the `account`'s balance on the ERC1155 token `id`. In the ERC1155 standard, a single contract tracks multiple tokens using different ID's. This is similar to the `ERC20` standard's `balanceOf` function, but the caller must also specify the ERC1155 token `id`.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `account`  | address  | The address of the user who's ERC1155 balance you wish to fetch                         |
| `id`  | uint256  | The ERC1155 token id                      |

#### balanceOfBatch

```solidity
  function balanceOfBatch(
    address[] memory accounts,
    uint256[] memory ids
  ) public view override returns (uint256[] memory)
```

Returns the balances of multiple `accounts` for multiple ERC1155 token `id`s. This is a more gas-efficient way to fetch multiple balances, compared to multiple calls to [balanceOf](#balanceOf). The length of the `accounts` array must equal the length of the `ids` array.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `accounts`  | address[] memory  | The addresses of the users who's ERC1155 balances you wish to fetch                         |
| `ids`  | uint256[] memory  | The ERC1155 token ids                     |

#### isApprovedForAll

```solidity
  function isApprovedForAll(
    address account,
    address operator
  ) public view override returns (bool)
```

Returns `true` if the `account` has approved `operator` to transfer all of `account`'s [`bTokens`](glossary.md#bToken) and [`wTokens`](glossary.md#wToken) on its behalf, and `false` if the `operator` is not approved to transfer any of `account`'s ERC1155 tokens. The ERC1155 standard does not expose a function for approving individual ERC1155 token ids, but does expose [`setApprovalForAll`](#setApprovalForAll) for approving the transfer of _all_ `account`'s ERC1155 tokens.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `account`  | address  | The address which owns the tokens and has approved the `operator`                         |
| `operator`  | address  | The address which has been delegated controll of `account`'s funds                    |

### Mutating Functions

#### setApprovalForAll

```solidity
  function setApprovalForAll(
    address operator,
    bool approved
  ) public override
```

If `approved` is `true`, allows the `operator` to transfer all of the caller's [`bTokens`](glossary.md#bToken) and [`wTokens`](glossary.md#wToken) on the caller's behalf. If `false`, the `operator` can no longer transfer tokens. This is similar to the [`ERC20`](glossary.md#erc20)'s `approve` function, except in the [ERC1155](glossary.md#erc1155) standard you only need to call the approval function once and forever after the `operator` can transfer any amount of tokens. However, this is more dangerous than single token approvals, because the `operator` has control over all of the caller's balance of ERC1155 tokens.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `account`  | address  | The address which owns the tokens and is approving/disapproving the `operator` to be able to transfer tokens                         |
| `approved`  | bool  | `true` if the `account` is allowing the `operator` to transfer the `account`'s tokens, and `false` if the `operator` is already approved and `account` no longer wants `operator` to be approved to transfer its tokens                    |

#### safeTransferFrom

```solidity
  function safeTransferFrom(
    address from,
    address to,
    uint256 id,
    uint256 amount,
    bytes memory data
  ) public override
```

Transfers `amount` tokens of token type `id` from `from` to `to`. `to` cannot be the zero address, and if the caller is not `from` then prior to calling this function the `from` address must have called [`setApprovalForAll`](#setApprovalForAll) with the caller as the `operator` argument, or else this call will fail. If `to` refers to a smart contract, it must implement ['onERC1155Received'](https://docs.openzeppelin.com/contracts/3.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `from`  | address  | The address from which ERC1155 tokens will be transferred out of                         |
| `to`  | address  | The address into which the ERC1155 tokens will be transferred into                    |
| `id`  | uint256  | The identifier of the [option token](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amount`  | uint256  | The amount of [option token](glossary.md#option-tokens) to transfer, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                     |
| `data`  | bytes memory  | Arbitrary data to be used by the individual ERC1155 token. For the SIREN protocol this can always unused, so it can be an empty array                    |

#### safeTransferFromBatch

```solidity
  function safeTransferFrom(
    address from,
    address to,
    uint256[] memory ids,
    uint256[] memory amounts,
    bytes memory data
  ) public override
```

Similar to [`safeTransferFrom`](#safeTransferFrom), but more gas efficient when multiple transactions for different amounts of different token ids must be sent to the same `to` address. Use this function rather than [`safeTransferFrom`](#safeTransferFrom) when the caller wants to save gas while sending multiple tokens to the same recipient address.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `from`  | address  | The address from which ERC1155 tokens will be transferred out of                         |
| `to`  | address  | The address into which the ERC1155 tokens will be transferred into                    |
| `ids`  | uint256[] memory  | The identifiers of the [option token](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amounts`  | uint256[] memory  | The amounts of [option token](glossary.md#option-tokens) to transfer, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                     |
| `data`  | bytes memory  | Arbitrary data to be used by the individual ERC1155 token. For the SIREN protocol this can always unused, so it can be an empty array                    |

#### mint

```solidity
  function mint(
    address to,
    uint256 id,
    uint256 amount,
    bytes memory data
  ) override(ERC1155PresetMinterPauserUpgradeable, IERC1155Controller)
    onlySeriesController
```

Creates `amount` tokens of token type `id` and sends them to address `to`. `to` cannot be the zero address, and if `to` refers to a smart contract, it must implement ['onERC1155Received'](https://docs.openzeppelin.com/contracts/3.x/api/token/erc1155#IERC1155Receiver-onERC1155Received-address-address-uint256-uint256-bytes-).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `to`  | address  | The address into which the newly minted ERC1155 tokens will be transferred into                         |
| `id`  | uint256  | The identifier of the [option token](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amount`  | uint256  | The amount of [option token](glossary.md#option-tokens) to mint, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                    |
| `data`  | bytes memory  | Arbitrary data to be used by the individual ERC1155 token. For the SIREN protocol this can always unused, so it can be an empty array                     |

#### mintBatch

```solidity
  function mintBatch(
    address to,
    uint256[] memory ids,
    uint256[] memory amounts,
    bytes memory data
  )
    public
    override(ERC1155PresetMinterPauserUpgradeable, IERC1155Controller)
    onlySeriesController
```

Similar to [`mint`](#mint), but more gas efficient when multiple transactions for different amounts of different token ids must be minted to the same `to` address. Use this function rather than [`mint`](#mint) when the caller wants to save gas while minting multiple tokens to the same recipient address.

The [`SeriesController`](series-controller.md#overview) calls this inside of [`SeriesController.mintOptions`](series-controller.md#mintOptions).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `to`  | address  | The address into which the newly minted ERC1155 tokens will be transferred into                         |
| `ids`  | uint256[] memory  | The identifiers of the [option tokens](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amounts`  | uint256[] memory  | The amounts of [option tokens](glossary.md#option-tokens) to mint, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                    |
| `data`  | bytes memory  | Arbitrary data to be used by the individual ERC1155 token. For the SIREN protocol this can always unused, so it can be an empty array                     |

#### burn

```solidity
  function burn(
    address account,
    uint256 id,
    uint256 amount
  )
    public
    override(ERC1155BurnableUpgradeable, IERC1155Controller)
    onlySeriesController
```

Destroys `amount` tokens of token type `id` owned by the address `account`. `account` cannot be the zero address, and `account` must have at least `amount` of the tokens for this call to succeed. If the caller is not `account`, then `account` must have approved the caller using [`setApprovalForAll`](#setApprovalForAll) prior to calling `burn`.

The [`SeriesController`](series-controller.md#overview) calls this inside of [`SeriesController.exerciseOption`](series-controller.md#exerciseOption) and [`SeriesController.claimCollateral`](series-controller.md#claimCollateral).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `to`  | account  | The address which will have the [option tokens](glossary.md#option-tokens) removed from it                         |
| `id`  | uint256  | The identifier of the [option token](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amount`  | uint256  | The amount of [option token](glossary.md#option-tokens) to burn, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                    |

#### burnBatch

```solidity
  function burnBatch(
    address account,
    uint256[] memory ids,
    uint256[] memory amounts
  )
    public
    override(ERC1155BurnableUpgradeable, IERC1155Controller)
    onlySeriesController
```

Similar to [`burn`](#mint), but more gas efficient when multiple transactions for different amounts of different token ids must be removed from the same `account` address. Use this function rather than [`burn`](#mint) when the caller wants to save gas while burning multiple tokens from the same address.

The [`SeriesController`](series-controller.md#overview) calls this inside of [`SeriesController.closePosition`](series-controller.md#closePosition).

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `to`  | account  | The address which will have the [option tokens](glossary.md#option-tokens) removed from it                         |
| `ids`  | uint256[] memory  | The identifiers of the [option token](glossary.md#option-tokens). Can be obtained by either `SeriesController.bTokenIndex(_seriesId)` or `SeriesController.wTokenIndex(_seriesId)`                    |
| `amounts`  | uint256[] memory  | The amounts of [option token](glossary.md#option-tokens) to burn, in base units. The [decimals](glossary.md#decimals) will always be the same as the Series' [underlying token](glossary.md#underlying-token)'                    |

#### updateImplementation

```solidity
  function updateImplementation(
    address _newImplementation
  ) external onlyOwner
```

Updates this `ERC1155Controller`'s [logic contract](glossary.md#logic-contract). The SIREN protocol's contracts use the [EIP-1822](https://eips.ethereum.org/EIPS/eip-1822) standard for implementing upgradeable contracts. This allows us to update vulnerable contracts and keep users' [option tokens](glossary.md#option-tokens) safe. When the SIREN protocol has reached a certain level of stability, we can remove these safety guards and ensure no one on the Siren team can swap out the smart contract functionality.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newImplementation`  | address  | The address of the new logic contract to use for the `ERC1155Controller`'s function implementations                      |

#### transferOwnership

```solidity
  function transferOwnership(address _newAdmin) external onlyOwner
```

Removes ownership from the current [owner](glossary.md#owner) and assigns ownership to the `_newAdmin` address. See [the `onlyOwner` modifier](#onlyOwner) for the permissions granted to the protocol admin.

##### Parameters

| Name        | Type    | Description                                                              |
| :---------- | :------ | :------------------------------------------------------------------------|
| `_newAdmin`  | address  | The address of the new admin for this contract                      |