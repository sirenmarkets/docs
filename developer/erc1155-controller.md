# ERC1155Controller

## Mainnet Contract Address

TODO

## Overview

The `ERC1155Controller` implements all of the functionality needed to mint, burn, approve, and transfer the [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) [option tokens](TODO glossary) (e.g. [`bTokens'](TODO glossary) and [`wTokens`](TODO glossary)). It inherits from the [Open Zeppelin ERC1155PresetMinterPauser contract](https://docs.openzeppelin.com/contracts/4.x/erc1155#Presets), which implements functions for minting, burning, approving, and transferring tokens, either as a single token or multiple tokens as a batch.

Any contract wishing to transfer SIREN [option tokens](TODO glossary) will need the `ERC1155Controller` [contract address](#mainnet-address).

We chose to use the [ERC1155](https://eips.ethereum.org/EIPS/eip-1155) interface instead of the more popular [`ERC20`](https://eips.ethereum.org/EIPS/eip-20) standard because it provides us a more gas efficient way to create [option series](TODO glossary). Rather than deploy a new ERC20 **contract** with every new `Series` we create, we add an additional **id** to the same `ERC1155` contract. It also standardizes the [`ERC1155.setApprovalForAll`](https://eips.ethereum.org/EIPS/eip-1155#approval), which allows an owner address to make only a single onchain transaction in order to approve an operator account to transfer any/all of the [option tokens](TODO glossary), which is more gas efficient than an onchain transaction for each option token.

## Functions