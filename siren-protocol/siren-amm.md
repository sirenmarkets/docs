# SIREN’s Automated Market Maker

Liquidity: the pool of resources mandatory to the creation of a market and financial instruments, that pile of cash or gold or crypto assets traded against, exchanged with, leveraged off of and bet upon in a million ways that create risks crossed with rates and the entire financial network overlaid on the world.

Options require liquidity. DeFi liquidity definitionally comes from a decentralized source, also known as liquidity providers (LP’s).  In the prior discussion of use cases there was mention of differences in the role and implementation of options writers in SIREN protocol.  These differences are due to the use of SIREN’s Automated Market Maker (AMM) which simultaneously fulfills goals of decentralization while solving problems related to liquidity. 

The AMM is responsible for pricing options, allows LP’s to deposit and withdraw collateral, and replaces a centralized order book as the entity with which traders both buy and sell options.

In SIREN protocol, rather than options writers supplying specific contracts to sell that must then be matched to a buyer in a centralized order book, buyer demand draws on liquidity reserves held by the AMM to create option series.  This is the mechanism by which LP’s become options writers in SIREN protocol, choosing to provide towards a “call” or “put” pool and becoming broadly indexed across every option series created from the selected pool.  

The use of liquidity pools across multiple option series is a novel SIREN implementation that both bootstraps liquidity and frees LP’s from active management of their position. The SIREN AMM enables an automatic passive interaction on the writer’s side; once LP’s deposit funds to a pool, the AMM does the rest (writing options, rolling collateral into new series automatically once the options reach expiry).

Once this liquidity has been accepted by the AMM, the buyer-side cycle of interaction commences.  The buyer selects an option they would like to purchase from the AMM, which determines the premium price.  The buyer pays that premium, prompting the AMM to immediately distribute that premium pro-rata to LP’s that have contributed to that option series’ pool.   

The rest of this cycle is handled by SIREN’s Settlement Layer, which will be explained presently.
