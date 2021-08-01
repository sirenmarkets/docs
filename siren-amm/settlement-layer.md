# Settlement Layer

## Overview

Any on-chain options protocol must have a set of contracts for managing the lifecycle of the options it trades. For SIREN we call this the Settlement Layer (SeriesController in the arthitecture). By calling the contracts in the Settlement Layer, a user can create covered calls/puts, close out existing positions, and redeem their option tokens.

The [SIREN AMMs](https://docs.sirenmarkets.com/siren-protocol/siren-amm) use the Settlement Layer to mint and sell options to traders. Without the Settlement Layer, the AMMs would have no method for tokenizing the long and short sides of an option, and there would be no market for options!

The most important concept in the Settlement Layer is the series. A series represents an option for a certain underlying asset ($UNI, $SUSHI, etc. for Calls or $USDC for Puts), at a certain strike price, and with a certain expiration date. Using the Settlement Layer the SIREN Team creates new series based on community and market demand.

Each series is associated with an AMM that trades the series’ underlying asset. The AMM mints and sells [bTokens](https://docs.sirenmarkets.com/faq-options/what-are-options#what-are-btokens-and-wtokens) (representing the buyer's side of the trade) and holds [wTokens](https://docs.sirenmarkets.com/faq-options/what-are-options#what-are-btokens-and-wtokens) (representing the writer's side of the trade).

The next section explains the major flows of the Settlement Layer’s contracts.

## Functionality

### Case 1 - Closing of a position (as a part of your Portfolio)

1. The process is the opposite of [Minting](https://docs.sirenmarkets.com/siren-protocol/siren-amm#case-1---buying-options) **b/wTokens**.
2. 1 bToken + 1 wTokens always equals 1 unit of collateral.
3. The User selects the series they wish to close the position on and inputs the # of contracts.
4. The User pushes the Close button on the right panel of the Portfolio tab.
5. Burn equal amount of bTokens and wTokens.
6. The SeriesController removes the collateral from the **Series_Vault** and sends it to the caller's address.

### Case 2 - Exercise of expired bTokens

1. We support only European so we exercise only after expiry
2. The User selects the series they wishes to exercise
3. The User pushes the Exercise button on the right panel of the Portfolio tab
4. The User’s bToken gets burned from the User’s address
5. The SeriesController sends the unlocked collateral from the seriesVault to the User’s wallet (note: only ITM exercise results in payoff)

### Case 3 - Claim collateral from expired wTokens

The User selects the series they wish to claim the collateral from.
The User pushes the Claim button on the right panel on the Portfolio tab
The AMM exercises any expired bTokens it holds
Claiming collateral from all the expired wTokens for  the respective series:
Burn the expired wTokens from the User’s address 
Sending the claimed collateral from the seriesVault to the User’s wallet (Metamask, etc.).
