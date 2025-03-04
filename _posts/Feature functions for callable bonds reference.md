import pandas as pd

def calculate_bid_ask_spread(df):
    """
    Computes bid-ask spread for each CUSIP on each trade date.
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', and 'trade_date'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'bid_price', 'ask_price', and 'bid_ask_spread'.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    
    # Group by CUSIP and trade date to find bid and ask prices
    bid_ask_df = df.groupby(['cusip', 'trade_date']).agg(
        bid_price=('price', 'min'),  # Dealers buy at lowest price
        ask_price=('price', 'max')   # Dealers sell at highest price
    ).reset_index()

    # Compute bid-ask spread
    bid_ask_df['bid_ask_spread'] = bid_ask_df['ask_price'] - bid_ask_df['bid_price']

    # Merge back with original data
    df = df.merge(bid_ask_df, on=['cusip', 'trade_date'], how='left')
    
    return df

def calculate_trade_frequency(df, window_days=[7, 30]):
    """
    Computes trade frequency in the past N days for each bond (CUSIP).
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip' and 'trade_date'.
    window_days (list): Time windows for trade frequency calculations.
    
    Returns:
    pd.DataFrame: Updated DataFrame with trade frequency columns.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df = df.sort_values(by=['cusip', 'trade_date'])

    for window in window_days:
        df[f'trades_last_{window}d'] = df.groupby('cusip')['trade_date'].transform(
            lambda x: x.rolling(f'{window}D', on=x).count()
        )

    return df

def calculate_implied_liquidity(df):
    """
    Computes price impact per $100k trade for each bond (CUSIP).
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', 'quantity', and 'trade_date'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'implied_liquidity'.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df = df.sort_values(by=['cusip', 'trade_date'])

    # Compute price change and trade size change per CUSIP
    df['price_change'] = df.groupby('cusip')['price'].diff()
    df['quantity_change'] = df.groupby('cusip')['quantity'].diff()

    # Avoid division by zero
    df['implied_liquidity'] = (df['price_change'] / df['quantity_change']) * 100000
    df['implied_liquidity'] = df['implied_liquidity'].fillna(0)

    return df

def calculate_trade_price_pct_change(df):
    """
    Computes trade price percentage change for each bond (CUSIP).
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', and 'trade_date'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'trade_price_pct_change'.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df = df.sort_values(by=['cusip', 'trade_date'])

    # Compute percentage change in price per CUSIP
    df['trade_price_pct_change'] = df.groupby('cusip')['price'].pct_change() * 100
    df['trade_price_pct_change'] = df['trade_price_pct_change'].fillna(0)

    return df

def process_municipal_bond_data(df):
    """
    Applies all feature engineering functions to the municipal bond trade dataset.
    
    Parameters:
    df (pd.DataFrame): Must contain 'cusip', 'price', 'quantity', and 'trade_date'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with new features.
    """
    df = calculate_bid_ask_spread(df)
    df = calculate_trade_frequency(df)
    df = calculate_implied_liquidity(df)
    df = calculate_trade_price_pct_change(df)
    
    return df

# Example Dataset
data = {
    'cusip': ['A1', 'A1', 'A1', 'B2', 'B2', 'B2'],
    'price': [101, 102, 101.5, 105, 106, 104.5],
    'quantity': [50000, 100000, 75000, 50000, 200000, 150000],
    'trade_date': ['2024-02-01', '2024-02-02', '2024-02-03', '2024-02-01', '2024-02-02', '2024-02-03']
}

df = pd.DataFrame(data)

# Process Data
df = process_municipal_bond_data(df)

# Display DataFrame
import ace_tools as tools
tools.display_dataframe_to_user(name="Municipal Bond Trade Data", dataframe=df)



import pandas as pd
import numpy as np

def calculate_call_probability(df: pd.DataFrame, years: list = [1, 2]) -> pd.DataFrame:
    """
    Estimates the probability of a callable bond being called in N years.
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'trade_date', 'maturity_date', 'call_date'.
    years (list): List of years for call probability estimation.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'prob_call_in_Xy' columns.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df['call_date'] = pd.to_datetime(df['call_date'])

    for y in years:
        df[f'prob_call_in_{y}y'] = np.where(
            (df['call_date'] - df['trade_date']).dt.days / 365 <= y, 1.0, 0.0
        )

    return df

def calculate_call_implied_volatility(df: pd.DataFrame) -> pd.DataFrame:
    """
    Estimates implied volatility of callable bond using price fluctuations.
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', 'trade_date'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'implied_vol_call'.
    """
    df = df.copy()
    df['trade_date'] = pd.to_datetime(df['trade_date'])
    df = df.sort_values(by=['cusip', 'trade_date'])

    # Compute rolling standard deviation of price changes as proxy for implied volatility
    df['implied_vol_call'] = df.groupby('cusip')['price'].pct_change().rolling(30).std()

    df['implied_vol_call'] = df['implied_vol_call'].fillna(0)
    return df

def calculate_distance_to_call_price(df: pd.DataFrame) -> pd.DataFrame:
    """
    Computes the distance between call price and current trade price.
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', 'call_price'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'distance_to_call_price'.
    """
    df = df.copy()
    df['distance_to_call_price'] = df['call_price'] - df['price']
    return df

``def calculate_convexity(df: pd.DataFrame) -> pd.DataFrame:
    """
    Estimates convexity adjustment for callable bonds.
    
    Parameters:
    df (pd.DataFrame): DataFrame with 'cusip', 'price', 'yield', 'duration'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with 'convexity'.
    """
    df = df.copy()
    df['convexity'] = (df['price'] / (1 + df['yield'])**2) * df['duration']
    return df

def process_callable_bond_features(df: pd.DataFrame) -> pd.DataFrame:
    """
    Applies all callable bond-specific feature engineering functions.
    
    Parameters:
    df (pd.DataFrame): Must contain 'cusip', 'price', 'call_price', 'trade_date', 'call_date', 'maturity_date', 'yield', and 'duration'.
    
    Returns:
    pd.DataFrame: Updated DataFrame with callable bond features.
    """
    df = calculate_call_probability(df)
    df = calculate_call_implied_volatility(df)
    df = calculate_distance_to_call_price(df)
    df = calculate_convexity(df)
    
    return df

# Example Dataset
data = {
    'cusip': ['A1', 'A1', 'A1', 'B2', 'B2', 'B2'],
    'price': [101, 102, 101.5, 105, 106, 104.5],
    'call_price': [102, 102, 102, 107, 107, 107],
    'trade_date': ['2024-02-01', '2024-02-02', '2024-02-03', '2024-02-01', '2024-02-02', '2024-02-03'],
    'call_date': ['2025-02-01', '2025-02-01', '2025-02-01', '2026-02-01', '2026-02-01', '2026-02-01'],
    'maturity_date': ['2030-02-01', '2030-02-01', '2030-02-01', '2035-02-01', '2035-02-01', '2035-02-01'],
    'yield': [0.03, 0.031, 0.029, 0.035, 0.036, 0.034],
    'duration': [5.2, 5.3, 5.1, 6.0, 6.1, 5.9]
}

df = pd.DataFrame(data)

# Process Data
``df = process_callable_bond_features(df)

# Display DataFrame
``import ace_tools as tools
``tools.display_dataframe_to_user(name="Callable Bond Features", dataframe=df)
