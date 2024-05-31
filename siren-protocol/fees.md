# Siren Protocol Fees

## Trading

### Margin Vault Deposit Fee

0.0% (no fee)

### Margin Vault Withdrawal Fee 

0.0% (no fee)

### Settlement Fee

0.0% (no fee)

### Trading Fee

First Siren Protocol calculates Leg Fee for each of the series traded, then a User pays Total Fee over all the legs:
- Leg Fee _(fee for each option series traded)_ = MIN {0.04% * Current Underlying Price; 12.5% * Premium} * number of contracts
- Total Fee = MAX over Leg Fees _(max fee among all legs)_

The formula above means that trading fee is capped at a maximum of 12.5% of the premium on options contracts.

All fees do not differ between buy and sell trade direction, and are applied in USDC.

#### Example with 1 Leg

An ETH option series is traded with the Premium of 400 USDC, and the Current Underlying Price is $3,000.

So, the Total Fee paid by a User will be:
- Leg Fee = MIN {0.04% * 3,000; 12.5% * 400} * 5 contracts = MIN {1.2; 50} * 5 = 1.2 * 5 = 6 USDC
- Total Fee = Leg Fee = 6 USDC

#### Example with 2 Legs

The first ETH option series is traded with the Premium of 400 USDC, the second one with the Premium of 500 USDC, and the Current Underlying Price is $3,000.

So, the Total Fee paid by a User will be:
- Leg Fee  = MIN {0.04% * 3,000; 12.5% * 400} * 10 contracts = MIN {1.2; 50} * 10 = 1.2 * 10 = 12 USDC
- Leg Fee  = MIN {0.04% * 3,000; 12.5% * 500} * 15 contracts = MIN {1.2; 62.5} * 15 = 1.2 * 15 = 18 USDC
- Total Fee = MAX {12; 18} = 18 USDC

## Earn

### USDC Pool Deposit Fee

0.0% (no fee)

### USDC Pool Withdrawal Fee

0.0% (no fee)
