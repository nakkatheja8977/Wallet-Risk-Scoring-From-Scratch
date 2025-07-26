# Wallet-Risk-Scoring-From-Scratch

This project assigns a **risk score (0–1000)** to Ethereum wallets based on their activity on the **Compound V2/V3 lending protocol**. Using data from The Graph, it analyzes key on-chain events—supply, borrow, repay, and liquidation—to evaluate user behavior. Features such as repayment ratio, borrow/supply ratio, liquidation count, and activity duration are extracted and scored using a weighted formula. The output is a CSV file listing each wallet with its risk score. This model helps identify reliable vs. risky wallets and serves as a scalable foundation for **on-chain credit scoring and DeFi risk analytics**.

---

## Objective

Given a list of 100 Ethereum wallet addresses, the goal is to:
1. Retrieve transaction data related to the Compound protocol.
2. Engineer meaningful features that capture user risk.
3. Score each wallet from **0 (high risk)** to **1000 (low risk)**.
4. Export results in a clean CSV format.

---

##  Data Collection Method

>  Data Source: [The Graph Protocol](https://thegraph.com/)

We use **Compound V2 Subgraph**:  
`https://api.thegraph.com/subgraphs/name/graphprotocol/compound-v2`

For each wallet, the following events are queried:
- `mint` – Supplied assets
- `redeem` – Withdrawn assets
- `borrow` – Borrowed assets
- `repayBorrow` – Loan repayments
- `liquidation` – Liquidation incidents

🛠️ Queries are done using GraphQL via Python (`gql`, `requests`, or `graphclient`).

---

##  Feature Engineering

Each wallet is evaluated on these risk-relevant features:

| Feature              | Description                                                 |
|----------------------|-------------------------------------------------------------|
| `total_supplied`     | Total assets supplied to Compound                           |
| `total_borrowed`     | Total assets borrowed                                       |
| `borrow/supply ratio`| Proxy for leverage risk                                     |
| `repayment_ratio`    | Repaid amount / borrowed amount                             |
| `liquidation_count`  | Number of times liquidated (high = risky)                   |
| `active_duration`    | Number of days between first and last protocol interaction  |
| `recent_activity`    | Days since last transaction (low = risky or inactive)       |

---

##  Scoring Logic

All features are normalized to a `[0, 1]` range. Then, the score is computed using a weighted sum:

```python
score = 1000 * (
    0.25 * normalized_supply_ratio +
    0.25 * normalized_repayment_ratio +
    0.20 * (1 - normalized_borrow_supply_ratio) +
    0.15 * (1 - normalized_liquidation_count) +
    0.15 * normalized_activity_duration
)
