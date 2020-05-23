 #  A Whale off the Port(folio)

 In this assignment, you'll get to use what you've learned this week to evaluate the performance among various algorithmic, hedge, and mutual fund portfolios and compare them against the S&P 500.

import pandas as pd
import numpy as np
import datetime as dt
from pathlib import Path
%matplotlib inline

# Data Cleaning

In this section, you will need to read the CSV files into DataFrames and perform any necessary data cleaning steps. After cleaning, combine all DataFrames into a single DataFrame.

Files:
1. whale_returns.csv
2. algo_returns.csv
3. sp500_history.csv

## Whale Returns

Read the Whale Portfolio daily returns and clean the data

# Reading whale returns
whale_returns_csv = Path("/Users/connor/Desktop/Starter_Code/Resources/whale_returns.csv")
# YOUR CODE HERE
whale_returns_df = pd.read_csv(whale_returns_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
whale_returns_df.sort_index(inplace=True)
whale_returns_df.tail()

# Count nulls
# YOUR CODE HERE
whale_returns_df.isnull().sum()

# Drop nulls
# YOUR CODE HERE
whale_returns_df.dropna(inplace=True)

whale_returns_df.isnull().sum()

## Algorithmic Daily Returns

Read the algorithmic daily returns and clean the data

# Reading algorithmic returns
algo_returns_csv = Path("Resources/algo_returns.csv")
# YOUR CODE HERE
algo_returns_df = pd.read_csv(algo_returns_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
algo_returns_df.sort_index(inplace=True)
algo_returns_df.tail()

# Count nulls
# YOUR CODE HERE
algo_returns_df.isnull().sum()

# Drop nulls
# YOUR CODE HERE
algo_returns_df.dropna(inplace=True)

algo_returns_df.isnull().sum()

## S&P 500 Returns

Read the S&P500 Historic Closing Prices and create a new daily returns DataFrame from the data. 

# Reading S&P 500 Closing Prices
sp500_history_csv = Path("Resources/sp500_history.csv")
# YOUR CODE HERE
sp500_history_df = pd.read_csv(sp500_history_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
sp500_history_df.sort_index(inplace=True)
sp500_history_df.tail()

# Check Data Types
# YOUR CODE HERE
sp500_history_df.dtypes

# Fix Data Types -- Removing $ sign from Close value
# YOUR CODE HERE
sp500_history_df['Close'] = sp500_history_df['Close'].str.replace('$', '')
sp500_history_df['Close']

# Fix Data Types -- Converting 'Close' from 'object' to 'float'
sp500_history_df['Close'] = sp500_history_df['Close'].astype('float')
sp500_history_df.dtypes

# Calculate Daily Returns
# YOUR CODE HERE
sp500_returns = sp500_history_df.pct_change()

# Drop nulls
# YOUR CODE HERE
sp500_returns.dropna(inplace=True)
sp500_returns

# Rename Column
# YOUR CODE HERE
sp500_returns = sp500_returns.rename(columns={"Close": "S&P 500"})
sp500_returns.head()

## Combine Whale, Algorithmic, and S&P 500 Returns

# Concatenate all DataFrames into a single DataFrame
# YOUR CODE HERE
joined_returns_df = pd.concat([whale_returns_df, algo_returns_df, sp500_returns], axis="columns", join="inner")
joined_returns_df

---

# Portfolio Analysis

In this section, you will calculate and visualize performance and risk metrics for the portfolios.

## Performance

Calculate and Plot the daily returns and cumulative returns. Does any portfolio outperform the S&P 500? 

# Plot daily returns
# YOUR CODE HERE
daily_returns = joined_returns_df.plot(figsize=(20,10))
daily_returns.set_ylabel("Returns %")
daily_returns.set_title("Daily Returns")

# Plot cumulative returns
# YOUR CODE HERE
initial_investment = 1000
cum_ret = (1 + joined_returns_df).cumprod()
cumulative_returns_plot = (initial_investment * cum_ret).plot(figsize=(20,10))
cumulative_returns_plot.set_ylabel("Returns $")
cumulative_returns_plot.set_title("Cumulative Returns $")

The Algo 2 portfolio outperforms the S&P 500.

---

## Risk

Determine the _risk_ of each portfolio:

1. Create a box plot for each portfolio. 
2. Calculate the standard deviation for all portfolios
4. Determine which portfolios are riskier than the S&P 500
5. Calculate the Annualized Standard Deviation

# Box plot to visually show risk
# YOUR CODE HERE
box_plot = joined_returns_df.plot.box()
box_plot.set_xticklabels(labels=joined_returns_df, rotation=90)

Create a box plot for each of the returns. Which box has the largest spread? Which has the smallest spread?
Tiger Global Management has the largest spread. Paulson & Co. has the smallest spread.

# Daily Standard Deviations
# Calculate the standard deviation for each portfolio. 
# Which portfolios are riskier than the S&P 500?
# YOUR CODE HERE
daily_std = joined_returns_df.std()
print(daily_std)

daily_std_hl = daily_std.sort_values(ascending=False)
print(daily_std_hl)

# Determine which portfolios are riskier than the S&P 500
# YOUR CODE HERE
daily_std > daily_std_hl['S&P 500']

# Calculate the annualized standard deviation (252 trading days)
# YOUR CODE HERE
annualized_std = daily_std * np.sqrt(252) * 100
print(annualized_std)

Calculate the standard deviation for each portfolio. Which portfolios are riskier than the S&P 500?
Tiger Global Management and Berkshire Hathaway have riskier portfolios than the S&P 500

## Rolling Statistics

Risk changes over time. Analyze the rolling statistics for Risk and Beta. 

1. Plot the rolling standard deviation of the various portfolios along with the rolling standard deviation of the S&P 500 (consider a 21 day window). Does the risk increase for each of the portfolios at the same time risk increases in the S&P? The risk for each portfolio tracks with the general direction of the S&P 500.
2. Construct a correlation table for the algorithmic, whale, and S&P 500 returns. Which returns most closely mimic the S&P? The Algo 1 retuns closely mimic the S&P 500
2. Choose one portfolio and plot a rolling beta between that portfolio's returns and S&P 500 returns. Does the portfolio seem sensitive to movements in the S&P 500? The Soros Fund Management portfolio beta is lower than the S&P 500 beta.

# Calculate and plot the rolling standard deviation for
# the S&P 500 and whale portfolios using a 21 trading day window
# YOUR CODE HERE
rolling_std_dev = joined_returns_df.rolling(window=21).std().plot(figsize =(20,10))
rolling_std_dev.set_ylabel("Std Dev")
rolling_std_dev.set_title("Rolling Std Dev")

import seaborn as sns

# Construct a correlation table
# YOUR CODE HERE
portfolio_corr = joined_returns_df.corr()
sns.heatmap(portfolio_corr, vmin=-1, vmax=1)

# Calculate Beta for a single portfolio compared to the total market (S&P 500)
# (Your graph may differ, dependent upon which portfolio you are comparing)
# YOUR CODE HERE
soros_covariance = joined_returns_df['SOROS FUND MANAGEMENT LLC'].cov(joined_returns_df['S&P 500'])
variance_spy = joined_returns_df['S&P 500'].var()
soros_beta = soros_covariance / variance_spy
soros_beta

### Challenge: Exponentially Weighted Average 

An alternative way to calculate a rollwing window is to take the exponentially weighted moving average. This is like a moving window average, but it assigns greater importance to more recent observations. Try calculating the `ewm` with a 21 day half-life.

# (OPTIONAL) YOUR CODE HERE
exp_wgh_ma = joined_returns_df.ewm(halflife=21,adjust=False).mean().plot()

---

## Sharpe Ratios
In reality, investment managers and thier institutional investors look at the ratio of return-to-risk, and not just returns alone. (After all, if you could invest in one of two portfolios, each offered the same 10% return, yet one offered lower risk, you'd take that one, right?)

Calculate and plot the annualized Sharpe ratios for all portfolios to determine which portfolio has the best performance

# Annualized Sharpe Ratios
# YOUR CODE HERE
portfolio_sharpe_ratios = (joined_returns_df.mean() * 252) / (joined_returns_df.std() * np.sqrt(252))

 plot() these sharpe ratios using a barplot.
 On the basis of this performance metric, do our algo strategies outperform both 'the market' and the whales?

# Visualize the sharpe ratios as a bar plot
# YOUR CODE HERE
portfolio_sharpe_ratios.plot(kind="bar", title="Sharpe Ratios")

---

# Portfolio Returns

In this section, you will build your own portfolio of stocks, calculate the returns, and compare the results to the Whale Portfolios and the S&P 500. 

1. Choose 3-5 custom stocks with at last 1 year's worth of historic prices and create a DataFrame of the closing prices and dates for each stock.
2. Calculate the weighted returns for the portfolio assuming an equal number of shares for each stock
3. Join your portfolio returns to the DataFrame that contains all of the portfolio returns
4. Re-run the performance and risk analysis with your portfolio to see how it compares to the others
5. Include correlation analysis to determine which stocks (if any) are correlated

## Choose 3-5 custom stocks with at last 1 year's worth of historic prices and create a DataFrame of the closing prices and dates for each stock.

# Read the first stock
# YOUR CODE HERE
google_returns_csv = Path("../Starter_Code/googl_google_finance - Sheet1.csv")
# YOUR CODE HERE
google_returns_df = pd.read_csv(google_returns_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
google_returns_df.sort_index(inplace=True)
google_returns_df.tail()

# Read the second stock
# YOUR CODE HERE
aapl_returns_csv = Path("../Starter_Code/aapl_google_finance - Sheet1.csv")
# YOUR CODE HERE
aapl_returns_df = pd.read_csv(aapl_returns_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
aapl_returns_df.sort_index(inplace=True)
aapl_returns_df.head()

# Read the third stock
# YOUR CODE HERE
msft_returns_csv = Path("../Starter_Code/msft_google_finance - Sheet1.csv")
# YOUR CODE HERE
msft_returns_df = pd.read_csv(msft_returns_csv, index_col="Date", infer_datetime_format=True, parse_dates=True)
msft_returns_df.sort_index(inplace=True)
msft_returns_df.head()

# Concatenate all stocks into a single DataFrame
# YOUR CODE HERE
custom_portfolio_df = pd.concat([google_returns_df, aapl_returns_df, msft_returns_df], axis="columns", join="inner")
custom_portfolio_df

# Reset the index
# YOUR CODE HERE
custom_portfolio_df.describe()

custom_portfolio_df.reset_index("Date")

# Pivot so that each column of prices represents a unique symbol
# YOUR CODE HERE
custom_portfolio_df = custom_portfolio_df.pivot_table(values="Close", index="Date")

# Drop Nulls
# YOUR CODE HERE
custom_portfolio_df.dropna()

## Calculate the weighted returns for the portfolio assuming an equal number of shares for each stock

# YOUR CODE HERE
custom_portfolio_returns = custom_portfolio_df.pct_change()
custom_portfolio_returns

# Calculate weighted portfolio returns

weights = [.33, .33, .33]
wght_portfolio_returns = custom_portfolio_returns.dot(weights)
wght_port_ret = pd.DataFrame(data=wght_portfolio_returns)
wght_port_ret.reset_index("Date")
columns = ['Custom Portfolio']
wght_port_ret.columns = columns

## Join your custom portfolio returns to the DataFrame that contains all of the portfolio returns

# Add your "Custom" portfolio to the larger dataframe of fund returns
# YOUR CODE HERE
combined_df = pd.concat([joined_returns_df, wght_port_ret], axis="columns", join="outer", sort=True)
combined_df

## Re-run the performance and risk analysis with your portfolio to see how it compares to the others

# Risk
# YOUR CODE HERE
combined_df.loc[combined_df.index.date == '2019-04-30']

# Rolling
# YOUR CODE HERE

# Beta
# YOUR CODE HERE

# Annualized Sharpe Ratios
# YOUR CODE HERE

# Visualize the sharpe ratios as a bar plot
# YOUR CODE HERE

## Include correlation analysis to determine which stocks (if any) are correlated

# YOUR CODE HERE
