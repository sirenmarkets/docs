# Contract Specifications

## Bitcoin Options

| **Contract Type**           | **BTC Call and Put Options**                 |
|-----------------------------|----------------------------------------------|
| Underlying Asset            | Bitcoin (BTC)                                |
| Style                       | European                                     |
| Method of Exercise          | Cash Settlement in USDC                      |
| Settlement Asset            | USD Coin (USDC)                              |
| Trading Hours               | 24/7                                         |
| Expiration Date & Time      | D0/D+1*, or Friday at 8am UTC                |
| Oracle Negotiation Period** | 8am – 20pm UTC                               |
| Tick Size                   | $0.01                                        |
| Multiplier                  | 1 (1 contract = 1 BTC)                       |
| Minimum Order Value         | 0.000001 contract                            |
| Strike Price Step           | $1,000                                       |
| Contract Expirations*       | D0/D+1, W+1, W+2, M+1, M+2, M+3, Q1, and Q+2 |

## Ethereum Options

| **Contract Type**           | **ETH Call and Put Options**                 |
|-----------------------------|----------------------------------------------|
| Underlying Asset            | Ethereum (ETH)                               |
| Style                       | European                                     |
| Method of Exercise          | Cash Settlement in USDC                      |
| Settlement Asset            | USD Coin (USDC)                              |
| Trading Hours               | 24/7                                         |
| Expiration Date* & Time     | D0/D+1, or Friday at 8am UTC                 |
| Oracle Negotiation Period** | 8am – 20pm UTC                               |
| Tick Size                   | $0.01                                        |
| Multiplier                  | 1 (1 contract = 1 ETH)                       |
| Minimum Order Value         | 0.000001 contract                            |
| Strike Price Step           | $100                                         |
| Contract Expirations*       | D0/D+1, W+1, W+2, M+1, M+2, M+3, Q1, and Q+2 |

## Arbitrum Options

| **Contract Type**           | **ARB Call and Put Options**                 |
|-----------------------------|----------------------------------------------|
| Underlying Asset            | Arbitrum (ARB)                               |
| Style                       | European                                     |
| Method of Exercise          | Cash Settlement in USDC                      |
| Settlement Asset            | USD Coin (USDC)                              |
| Trading Hours               | 24/7                                         |
| Expiration Date* & Time     | D0/D+1, or Friday at 8am UTC                 |
| Oracle Negotiation Period** | 8am – 20pm UTC                               |
| Tick Size                   | $0.001                                       |
| Multiplier                  | 1 (1 contract = 1 ARB)                       |
| Minimum Order Value         | 0.000001 contract                            |
| Strike Price Step           | $0.04                                        |
| Contract Expirations*       | D0/D+1, W+1, W+2, M+1, M+2, M+3, Q1, and Q+2 |

*Expiration Dates Logics:

- D0/D1: current day or next day (if current time is > 8 a.m. UTC)
- W+1: the nearest Friday from the current or next day
- W+2: next Friday from the current or next day
- M+1: last Friday in +1 month from the current or next day
- M+2: last Friday in +2 months from the current or next day
- M+3: last Friday in +3 months from the current or next day
- Q+1: last Friday in +1 quarter from the current or next day
- Q+2: last Friday in +2 quarter from the current or next day

**Oracle Negotiation Period

