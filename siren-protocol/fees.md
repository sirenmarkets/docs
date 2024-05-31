# Siren Protocol Fees

## Trading

### Deposit Fee

0.0% (no fee)

### Withdrawal Fee 

0.0% (no fee)

### Settlement Fee

0.0% (no fee)

### Trading Fee

- Leg Fee (fee for each option series traded) = Minimum of {0.04% * Current Underlying Price; 12.5% * Premium} * number of contracts
- Total Fee = Maximum over Leg Fees (max fee among all legs)

The formula above means that trading fee is capped at a maximum of 12.5% of the premium on options contracts.
All fees do not differ between buy and sell trade direction, and are applied in USDC.

#### Example for 1 leg

An ETH option series is traded with the Premium of 400 USDC, and the Current Underlying Price is $3,000.
So, a Trading Fee paid by the User will be:
- Leg Fee = MIN{0.04% * 3,000; 12.5% * 400} * 5 contracts = MIN{1.2; 50} * 5 = 1.2 * 5 = 6 USDC
- Total Fee = MAX(over legFees) = 

#### Example for 2 legs

The first ETH option series is traded with the Premium of 400 USDC, the second one with the Premium of 500 USDC, and the Current Underlying Price is $3,000.
So, a Trading Fee paid by the User will be:
- Leg Fee  = MIN(0.04% * 3,000; 12.5% * 400) * 10 contracts = MIN((0.05% * 1 * $1000), (0.125 * $20)) = min($0.75, $2.5) = $0.75
- Leg Fee  = MIN(0.04% * 3,000; 12.5% * 400) * 15 contracts = MIN((0.05% * 1 * $1000), (0.125 * $20)) = min($0.75, $2.5) = $0.75
- Total Fee = MAX(over legFees) = 

## Earn

Pool deposit fee: 0.0%
Pool withdrawal fee: 0.0% 
