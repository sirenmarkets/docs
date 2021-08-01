# Settlement Layer

## Overview

Any on-chain options protocol must have a set of contracts for managing the lifecycle of the options it trades. For SIREN we call this the Settlement Layer (SeriesController in the arthitecture). By calling the contracts in the Settlement Layer, a user can create covered calls/puts, close out existing positions, and redeem their option tokens.

The [SIREN AMMs](https://docs.sirenmarkets.com/siren-protocol/siren-amm) use the Settlement Layer to mint and sell options to traders. Without the Settlement Layer, the AMMs would have no method for tokenizing the long and short sides of an option, and there would be no market for options!

The most important concept in the Settlement Layer is the series. A series represents an option for a certain underlying asset ($UNI, $SUSHI, etc.), at a certain strike price, and with a certain expiration date. Using the Settlement Layer the SIREN Team creates new series based on community and market demand.

Each series is associated with an AMM that trades the series’ underlying asset. The AMM mints and sells [bTokens](https://docs.sirenmarkets.com/faq-options/what-are-options#what-are-btokens-and-wtokens) (representing the buyer's side of the trade) and holds [wTokens](https://docs.sirenmarkets.com/faq-options/what-are-options#what-are-btokens-and-wtokens) (representing the writer's side of the trade).

The next section explains the major flows of the Settlement Layer’s contracts.

## Functionality

### Case 1 - Closing of a position

1. The process is the opposite of [Minting](https://docs.sirenmarkets.com/siren-protocol/siren-amm#case-1---buying-options).
2. 1 **bToken** + 1 **wToken** always equals 1 unit of collateral ($UNI, $SUSHI, etc.).
3. A user selects a series and inputs the *# of contracts* to be closed.
4. The user pushes the *Close* button on the right panel on the *Portfolio* tab.
5. The AMM burns the equal amount of **bTokens** and **wTokens** (according to the *# of contracts*) for the respective series.
6. The SeriesController removes the unlocked collateral from the SeriesVault and sends it to a user's wallet linked (Metamask, etc.).

### Case 2 - Exercise of options

1. SIREN Protocol supports only European style of options so an exercise can be done only after expiry.
2. A trader selects a series to be exercised.
3. The trader pushes the *Exercise* button on the right panel of the *Portfolio* tab.
4. The AMM burns the expired **bTokens** (or the respective series) from a trader's wallet.
5. The SeriesController sends the unlocked collateral from the SeriesVault to the wallet.
> NOTE: Only ITM options exercise results in payoff

### Case 3 - Claim collateral from expired wTokens

1. An LP selects the series they wish to claim the collateral from.
2. The LP pushes the *Claim* button on the right panel on the *Portfolio* tab.
3. The AMM exercises any expired **bTokens** it holds.
4. The protocol claims collateral from all the expired **wTokens** for the respective series.
5. The AMM burns the expired **wTokens** from a LP's wallet.
6. The SeriesController sends the claimed collateral from the SeriesVault to the wallet.
