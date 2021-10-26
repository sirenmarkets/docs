# bToken/wToken

Minted by the Settlement Layer in the process of Bilateral Tokenization, bTokens and wTokens together form the agreement called the options contract. bTokens represent the long side of the contract, and are distributed to the buyer of the option series. wTokens represent the short side plus locked collateral, and are held by the AMM. 

bTokens can be traded freely, even outside of SIREN’s platform. They are assigned a value pegging them to the particular option series by the Series Controller contract contained in the Settlement Layer, which will auto-exercise for the bToken holder once that series has expired (unless the series expires OTM).

wTokens are held by the AMM until their series expires, and can then be claimed by LP’s to unwind the collateral, closing the contract.  This freed collateral is then distributed to LP’s in a pro-rata fashion.
