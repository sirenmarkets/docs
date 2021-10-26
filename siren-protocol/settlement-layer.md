# Settlement Layer

Any on-chain options protocol must have a set of smart-contracts for the management of the lifecycle of options it trades.  SIREN’s Settlement Layer is responsible for minting options, maintaining in-use collateral, and the redemption of options post-expiry.

Key to this is the process of bilateral tokenization where both sides of the options contract are tokenized. The Settlement Layer mints bTokens representing the long side of the trade and wTokens representing the short side plus locked up collateral (wToken construction is equivalent to a covered call/cash-secured put). Bilateral Tokenization allows the Settlement Layer to keep track of both the long and short positions universally by counting bToken/wToken balances for a specific options series.

When a series expires, the Settlement Layer manages the exercise of bTokens, the claiming of wTokens, and the unwinding of collateral as both tokens are used in conjunction to close out the options contract.  SIREN supports European cash-settled options, meaning they are settled immediately after expiration based on pricing from a price oracle (here Chainlink).

![](../.gitbook/assets/image.png)

_NOTE: This is a living document that will continue to be updated as SIREN evolves. To contribute, please visit_ [_SIREN on GitHub_](https://github.com/sirenmarkets/core)_. Specific questions may be answered and technical guidance may also be provided from time to time in the_ [_SIREN Telegram_](https://t.me/sirenmarkets) _to those who are interested in building on top of the protocol._
