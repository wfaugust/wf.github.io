When pricing **municipal callable bonds**, derived features play a crucial role in capturing market dynamics, liquidity, and callability effects. Below are **important feature categories** that can significantly improve your model's performance:

---

## **1. Trade-Level Features**

These features help capture transaction-specific information.

- **Log Trade Size (`log(quantity)`)** – Helps normalize trade size effects.
- **Trade Size Bucket (`small, medium, large`)** – To differentiate retail vs. institutional trades.
- **Trade Direction (`dealer buy/sell`)** – Helps understand trade pressure.
- **Trade Execution Type (`customer to dealer`, `dealer to dealer`)** – Dealer trades tend to be better priced.
- **Trade Time (`hour_of_day`, `day_of_week`)** – Liquidity varies throughout the day and week.
- **Recent Price Change (`trade_price_pct_change`)** – Measures short-term momentum.

---

## **2. Bond-Specific Features**

These features account for the bond's structural characteristics.

- **Coupon Rate (`coupon_rate`)** – Higher coupons tend to have different call behaviors.
- **Time to Maturity (`years_to_maturity`)** – Longer-dated bonds price differently.
- **Time to Next Call (`years_to_call`)** – Key feature for callable bonds.
- **Call Price (`call_price`)** – The callable price cap affects the upper bound of valuation.
- **Current Yield (`coupon / trade_price`)** – Simple yield approximation.
- **Yield to Worst (`ytw`)** – Accounts for the worst-case scenario given callability.
- **Option-Adjusted Spread (`oas`)** – A pricing spread incorporating the call option.

---

## **3. Liquidity & Market Features**

Municipal bonds trade in an OTC market with **varying liquidity**, so these features are critical.

- **Bid-Ask Spread (`bid_ask_spread`)** – Wider spreads indicate lower liquidity.
- **Trade Frequency (`trades_last_7d`, `trades_last_30d`)** – More frequent trades imply better liquidity.
- **Volume-Weighted Average Price (`vwap`)** – Helps benchmark fair trade price.
- **Implied Liquidity (`price impact per $100k trade`)** – Measures price slippage for large trades.

---

## **4. Interest Rate & Macro Factors**

Municipal bond prices are highly sensitive to interest rate changes.

- **Treasury Yield Curve (`10y_treasury`, `30y_treasury`)** – Key benchmarks.
- **Muni-Treasury Ratio (`muni_yield / treasury_yield`)** – Helps gauge relative attractiveness.
- **Fed Funds Rate (`ffr`)** – Impacts borrowing costs and liquidity.
- **Inflation Expectation (`5y_breakeven_inflation`)** – Affects long-term bond pricing.
- **Credit Spread (`muni_credit_spread`)** – Spread vs. high-grade municipals.

---

## **5. Callable Bond-Specific Features**

Callable bonds are path-dependent, meaning the call decision **must be modeled explicitly**.

- **Call Probability (`prob_call_in_1y`, `prob_call_in_2y`)** – Predict likelihood of being called.
- **Call Implied Volatility (`implied_vol_call`)** – Higher volatility increases call value.
- **Distance to Call Price (`call_price - trade_price`)** – If the bond is near the call price, it is more likely to be called.
- **Convexity Adjustment (`convexity`)** – Captures price sensitivity to interest rate changes.

---

## **6. Temporal Features (Rolling Windows)**

Since bond pricing depends on **historical behavior**, use rolling statistics:

- **Rolling Average Price (`avg_trade_price_30d`)** – Smooths out short-term noise.
- **Rolling Volatility (`volatility_30d`)** – Measures price stability.
- **Rolling Liquidity (`avg_spread_30d`)** – Captures liquidity fluctuations.
- **Momentum Signal (`price_momentum_10d`)** – Recent trend in bond pricing.

---

## **Which Features Should You Add First?**

1. **Trade-Level**: `log(quantity)`, `trade size bucket`, `trade_time`
2. **Bond-Specific**: `years_to_maturity`, `years_to_call`, `coupon_rate`
3. **Liquidity Features**: `vwap`, `bid_ask_spread`, `trades_last_30d`
4. **Interest Rate Factors**: `10y_treasury`, `muni-treasury ratio`
5. **Callable Bond Features**: `distance_to_call_price`, `call_probability`