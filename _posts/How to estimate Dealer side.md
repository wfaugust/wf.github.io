In municipal bond trading data from **MSRB**, the trade direction (dealer buy or sell) is typically inferred using **execution price relative to recent market prices** and trade types.

### **Methods to Determine Dealer Side (Buy/Sell)**

---

## **1. MSRB Trade Data Fields (If Available)**

Some **MSRB datasets** may have direct trade indicators, but if they don’t, you can infer trade direction using:

- **`customer_buy` vs. `customer_sell`** (direct indicators when available)
- **`dealer-to-dealer` vs. `dealer-to-customer`** trade type

---

## **2. Price-Based Inference**

If trade direction isn’t explicitly provided, use **relative price movement**:

### **Rule-Based Approach:**

- **If the trade price is above recent market price (VWAP)** → Dealer **buys** from a customer (Bid Side).
- **If the trade price is below recent market price (VWAP)** → Dealer **sells** to a customer (Ask Side).

#### **Python Implementation:**


``def infer_dealer_side(df):
    """
    Infers whether the dealer is buying or selling based on trade price vs. VWAP.
    
    Parameters:
    df (pd.DataFrame): DataFrame containing 'trade_price' and 'vwap'.
    
    Returns:
    pd.Series: Column indicating 'Dealer Buy' or 'Dealer Sell'.
    """
    df = df.copy()
    df['dealer_side'] = df.apply(lambda row: 'Dealer Buy' if row['trade_price'] > row['vwap'] else 'Dealer Sell', axis=1)
    return df

###### Example Usage:
data = {'trade_price': [102, 101.5, 103, 102.8, 100.5],
        'vwap': [101.8, 101.5, 102.5, 102.7, 100.7]}
df = pd.DataFrame(data)

df = infer_dealer_side(df)
print(df[['trade_price', 'vwap', 'dealer_side']])
``

**Explanation:**

- If `trade_price > vwap`, it implies the dealer is **paying more than the market price** (likely a **buy**).
- If `trade_price < vwap`, it implies the dealer is **selling at a discount** (likely a **sell**).

---

## **3. Trade Size & Counterparty Rules (Additional Refinement)**

You can refine the logic by considering:

- **Dealer-to-Customer Trades:** Usually customer **sells** to a dealer at a discount.
- **Dealer-to-Dealer Trades:** More likely inter-dealer market making, bid-ask spreads apply.
- **Large trades (`> 100k`) are typically institutional:** Institutions tend to **sell to dealers** at competitive prices.

### **Enhanced Rule Logic**

``def infer_dealer_side_enhanced(df):
    """
    Infers dealer trade side using trade price, VWAP, and counterparty type.
    
    Parameters:
    df (pd.DataFrame): Contains 'trade_price', 'vwap', 'trade_size', 'trade_type'.
    
    Returns:
    pd.Series: Column indicating 'Dealer Buy' or 'Dealer Sell'.
    """
    df = df.copy()
    
    def classify_trade(row):
        if row['trade_price'] > row['vwap']:
            return 'Dealer Buy'
        elif row['trade_price'] < row['vwap']:
            return 'Dealer Sell'
        elif row['trade_type'] == 'Dealer-to-Dealer':
            return 'Market Making'
        else:
            return 'Unknown'

    df['dealer_side'] = df.apply(classify_trade, axis=1)
    return df

##### Sample Data:
data = {'trade_price': [102, 101.5, 103, 102.8, 100.5],
        'vwap': [101.8, 101.5, 102.5, 102.7, 100.7],
        'trade_type': ['Dealer-to-Customer', 'Dealer-to-Dealer', 'Dealer-to-Customer', 'Dealer-to-Customer', 'Dealer-to-Dealer']}
df = pd.DataFrame(data)

df = infer_dealer_side_enhanced(df)
print(df[['trade_price', 'vwap', 'trade_type', 'dealer_side']])
``

## **Best Approach for Your Model**

- If **trade type is available**, use it first.
- If not, use **trade price vs. VWAP** for estimation.
- Further refine using **trade size and counterparty**.