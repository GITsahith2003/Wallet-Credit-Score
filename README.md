# Wallet-Credit-Score


---

## 🎯Objective

This project develops a **credit scoring system (0-1000 scale)** to evaluate wallets interacting with the Aave V2 decentralized lending protocol. The goal is to quantify the creditworthiness of each wallet based on transactional behavior and financial responsibility, enabling more informed risk management and analytics in DeFi.

---

## 🛠️Setup Instructions

1. **Prerequisites**  
   Ensure you have Python 3.7+ installed with the following packages:  
   - pandas  
   - json (standard library)  
   - matplotlib (optional, for score distribution analysis)  

2. **Clone or Download**  
   Clone this repository or download the script files.

3. **Prepare Data**  
   Provide a JSON file containing Aave V2 wallet transaction records in the format:
   [
{
"userWallet": "0x...",
"timestamp": 1633072800,
"action": "deposit",
"actionData": {
"amount": "1000000",
"assetSymbol": "USDC",
"assetPriceUSD": "1.00"
}
},
...
] filename used in code: `user-wallet-transactions.json`

4. **Run Credit Score Calculation**  
Execute the main script `credit_score.py` after setting the correct data file path.  
The output will be a JSON file `wallet_scores.json` mapping wallet addresses to credit scores.


---

## 🚀Complete Method

The credit score calculation involves the following:

### Asset Normalization

- Transaction amounts use raw integer strings representing smallest units (e.g., wei for ETH).
- Using a predefined decimals mapping per asset (e.g., USDC = 6 decimals), convert raw amounts to human-readable decimal values.
- Multiply by the asset’s USD price at transaction time to get USD-equivalent values for consistent comparison.

### Feature Extraction and Aggregation

- Transactions are grouped by wallet address.
- For each wallet, aggregate:
- Total counts per action type (deposit, borrow, repay, redeemunderlying, liquidationcall).
- Total USD amounts for deposits, borrows, repayments, redemptions, and liquidations.
- First and last transaction timestamps to measure activity duration.
- Lists of timestamps for borrows and repays to assess repayment timeliness.
- Sets of unique deposited and borrowed asset symbols.
- Highest single deposit and borrow amounts (USD).

### Credit Scoring Logic

A base score of 500 points is adjusted using weighted features reflecting risk and positive behavior:

| Feature                                       | Effect                            | Scoring Range/Logic                           |
|-----------------------------------------------|---------------------------------|----------------------------------------------|
| **Activity Duration**                          | Positive                        | +0.2 points per day, max +100 points         |
| **Repay-to-Borrow Ratio**                      | Critical positive/negative risk | >=1.0: +350; 0.8–1.0: +250; 0.5–0.8: +100; partial: -100; none: -300 |
| **No Borrow + Deposit Only**                   | Positive low-risk indicator     | +200 points                                  |
| **Liquidation Events**                          | Strong negative                 | -150 points each, max -400                    |
| **Outstanding Net Borrowed Amount (Debt)**    | Negative                       | Max penalty -150, scaled by $100 USD increments |
| **Repayment Timeliness (Avg Days)**            | Positive/Negative               | <14 days: +70; <60 days: +30; longer: -50    |
| **Activity Level (Total Transactions > 5)**   | Positive                      | +1 point per tx beyond 5, max +50             |
| **Asset Diversification (Deposits & Borrows)**| Slight Positive                | Deposit assets: +5 pts each up to +20; Borrow assets: +5 pts each up to +10 |
| **Net Deposited Value (Deposit - Redeem)**    | Positive                      | 1 point per $1000 net deposit, max +50        |
| **Large Single Transaction Amounts**          | Slight Positive               | Deposit > $10k: +10; Borrow > $10k: +5        |

- Scores are clamped between 0 and 1000.
- Final scores are rounded to integers.

---

## 📈Process Chosen & Method Flow

1. **Input:** Raw transaction data JSON from Aave V2.
2. **Normalization:** Convert all transaction amounts to USD for uniform scale.
3. **Feature Aggregation:** Calculate transaction counts, totals, timestamps, and asset sets by wallet.
4. **Advanced Feature Derivation:** Compute repayment ratios, active duration in days, timeliness, and diversification metrics.
5. **Scoring:**  
Apply rule-based weighted scoring to encapsulate credit risk and positive financial behavior.
6. **Output:** Produce a wallet-to-score JSON mapping for downstream use or analysis.

---

---

## Notes

- The scoring emphasizes **repay-to-borrow behavior** and penalizes liquidations and outstanding debt heavily to reflect credit risk.
- Wallets providing liquidity without borrowing receive positive adjustments as they represent lower risk.
- Timely repayments and diversified portfolio interactions add to credibility.
- Large, consistent transactions hint at serious, engaged users and slightly boost the score.
- Parameters and thresholds are configurable to suit evolving risk assessment needs.

---





