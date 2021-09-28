<!-- PROJECT LOGO -->
![portfolio](https://user-images.githubusercontent.com/86877457/135084460-58cfd65c-5482-4b98-90e1-65ca10c88431.jpg)
<br />
<p align="center">

  <h3 align="center">Portfolio Optimizing System</h3>

  <p align="center">
    In this project, I've tried to build a model to filter out specified number of bad stocks each month from our portfolio and add better performing stocks from a given wide range of stocks.
    <br />
  </p>
</p>




<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
    </li>
    <li>
      <a href="#problem-statement">Problem Statement</a>
    </li>
    <li>
	<a href="#data-overview">Data Overview</a>
	<ul>
          <li><a href="#data-attributes">Data Attributes</a></li>
          <li><a href="#data-snapshot">Data Snapshot</a></li>
        </ul>
    </li>
    <li><a href="#implementaion">Implementaion</a>
	<ul>
          <li><a href="#exploring-the-data">Exploring the Data</a></li>
          <li><a href="#kpis">KPIs</a></li>
	        <li><a href="#portfolio-rebalancing">Portfolio Rebalancing</a></li>
	        <li><a href="#result">Result</a></li>
        </ul>
    </li>
    <li><a href="#final-thoughts">Final Thoughts</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

If the thought of investing in the stock market scares you, you are not alone. Individuals with very limited experience in stock investing are either terrified by horror stories of the average investor losing 50% of their portfolio value. The reality is that investing in the stock market carries risk, but when approached in a disciplined manner, it is one of the most efficient ways to build up one's net worth.

Portfolio Managment is one of the crucial part of trading. Portfolio management consists of three main elements: investing time horizon, diversification of investments, and risk tolerance. Here, we take into account the diversification of investments and optimize our portfolio with the stocks giving best monthly returns.


<!-- PROBLEM STATEMENT -->
## Problem Statement

- Chose any universe of stocks (Large cap, mid cap, small cap, Industry specific, factor specific etc.) and stick to this group of stock as the source for your portfolio for the entire duration of backtesting
- Build fixed individual position sized long only portfolio by picking n number of stocks based on monthly returns (or any other suitable criterion)
- Rebalance the portfolio every month by removing worse x stocks and replacing them with top x stocks from the universe of stocks (can existing stock be picked again?)
- Backtest the strategy and compare the KPIs with that of simple buy and hold strategy of corresponding index.


<!-- DATA OVERVIEW -->
## Data Overview

Source: [Yahoo Finance](https://finance.yahoo.com/)

Data: The data set was extracted from Yahoo Finance with the help of [Pandas Data Reader](https://pandas-datareader.readthedocs.io/en/latest/#) and contains the Open, High, Low, Close, Adj Close and Volumes Data of the selected stocks.

### Data Attributes

**1. Open:** The daily Opening Price of the stock.

**2. High:** The highest price that the stock reaches during the day.

**3. Low:** The lowest price that the stock reaches during the day.

**4. Close:** The close price is the raw price, which is the cash value of the last transacted price before the market closes.

**5. Adj. Close:** The adjusted closing price takes into account divident on shares and factors that might affect the stock price after the market closes.

**6. Volumes:** The total units of stocks traded throughout the day.

### Data Snapshot

![image](https://user-images.githubusercontent.com/86877457/135076221-1005bbcc-fbfd-4c4b-981f-b22c0f5ffa59.png)

<!-- IMPLEMANTATION -->
## Implementaion

**Real-world/Business objectives and constraints**

- Interpretability is important.
- Errors can be costly.
- Model should be robust to deal with multiple finacial instruments.

### Exploring the Data

Data was processed and cleaned.

**Stock Selected**

In total, 30 companies listed on SENSEX were selected. The idea was to find whether carrying long position of the respective index over time was good or picking only the better performing stocks was.

``` sh
stocks = ['INFY.NS','TCS.NS','RELIANCE.NS','ICICIBANK.NS','HDFCBANK.NS','HCLTECH.NS','BHARTIARTL.NS','INDUSINDBK.NS','SBIN.NS',
         'LT.NS','TECHM.NS','AXISBANK.NS','ITC.NS','BAJAJ-AUTO.NS','ONGC.NS','TATASTEEL.BO','NTPC.NS','M&M.NS','ASIANPAINT.NS','POWERGRID.NS',
         'BAJAJFINSV.NS','TITAN.NS','NESTLEIND.NS','ULTRACEMCO.NS','SUNPHARMA.BO','BAJFINANCE.NS','MARUTI.NS','HDFC.NS','HINDUNILVR.NS','KOTAKBANK.NS']
```

Our portfolio was created:

It contained : **'TECHM.NS', 'POWERGRID.NS', 'NTPC.NS', 'HCLTECH.NS', 'TATASTEEL.BO', 'BHARTIARTL.NS'** i.e Tech Mahindra, Power Grid Corporation of India, NTPC Limited, TATA Steel and Bharti Airtel.

### KPIs
**Compound Annual Growth Rate (CAGR)**

The compound annual growth rate (CAGR) is the rate of return (RoR) that would be required for an investment to grow from its beginning balance to its ending balance, assuming the profits were reinvested at the end of each period of the investment’s life span.

![image](https://user-images.githubusercontent.com/86877457/135077931-a50320c4-6544-46d1-a505-87adbc84258f.png)

I wrote a function to calculate the CAGR value

```sh
  def CAGR(df):
    temp=df.copy()
    temp['cum_return']=(1+temp['monthly_return']).cumprod()
    year=round(((temp.index[len(temp)-1]-temp.index[0])).days/365)
    CAGR=((temp.cum_return[len(temp)-1]/1)**(1/year))-1
    return round(CAGR*100,3)
```

**Volatility**

Volatility is a statistical measure of the dispersion of returns for a given security or market index. In most cases, the higher the volatility, the riskier the security. Volatility is often measured as either the standard deviation or variance between returns from that same security or market index.

I wrote a function to calculate the Volatility

```sh
  def volatility(df):
    temp=df.copy()
    daily_vol=temp.monthly_return.std()
    ann_vol=(daily_vol*(12**0.5))
    return round(ann_vol*100,3)
```


**Sharpe Ratio**

Sharpe Ratio is the average return earned in excess of the risk-free rate per unit of volatility or total risk.

It adjusts a portfolio’s past performance—or expected future performance—for the excess risk that was taken by the investor. A high Sharpe ratio is good when compared to similar portfolios or funds with lower returns. The Sharpe ratio has several weaknesses, including an assumption that investment returns are normally distributed.

![image](https://user-images.githubusercontent.com/86877457/135083525-d8c9bf79-008f-4305-82a1-cd50e193f92b.png)

I wrote a function to calculate the sharpe ratio

```sh
  def sharpe(df,rf=6.2):
    temp=df.copy()
    ratio=(CAGR(df)-rf)/volatility(df)
    return round(ratio,3)
```

**Maximum Drawdown**

A maximum drawdown (MDD) is the maximum observed loss from a peak to a trough of a portfolio, before a new peak is attained. Maximum drawdown is an indicator of downside risk over a specified time period. In the price chart, Maximum Drawdown is the maximum fall in price in the specified time period.

It can be used both as a stand-alone measure or as an input into other metrics such as "Return over Maximum Drawdown" and the Calmar Ratio. Maximum Drawdown is expressed in percentage terms.

![image](https://user-images.githubusercontent.com/86877457/135079147-f9a1f2ea-10e0-4db2-a552-3d4d00555013.png)

I wrote a function to calculate the maximum drawdown

```sh
  def max_drawdown(df):
    temp=df.copy()
    temp['cum_return']=(1+temp['monthly_return']).cumprod()
    temp['cum_roll_max']=temp['cum_return'].cummax()
    temp['draw_down']=temp['cum_roll_max']-temp['cum_return']
    temp['draw_down_pct']=temp['draw_down']/temp['cum_roll_max']
    max_dd=temp['draw_down_pct'].max()
    return round(max_dd*100,3)
```

### Portfolio Rebalancing

Steps involved in Portfolio Rebalancing:

1. A dataframe with the monthly returns was created for all stocks.
2. Our portfolio at the beginning of the investment was taken as input.
3. For each month cycle, the best n performing stocks was listed out by comapring the monthly return.
4. This n stocks replaced our n worst perfroming stocks in our portfolio.  
5. We can have multiple units of the best perferfroming stock as per our wish.
6. This cycle was continued.
7. The KPIs were calculated for this strategy and compared with if we have had only taken a unit of SENSEX
8. The returns of both the methods were plotted

**INPUT**

 ```sh
  def portfolio(df,n,x,k=0):
    '''return cummulitive portfolio returns
    df = datframe with monthly return of all stock
    n = number of stock in the portfolio
    x = number of underperforming stock to be removed from the portfolio every month'''
 ```
The input chosen for this were: ```portfolio(return_df,6,3)```

**SENSEX**

The performance of the index over last five years.

![image](https://user-images.githubusercontent.com/86877457/135080357-28b9a9e1-b19a-4f18-9995-1ea78c760087.png)

**The Monthly returns DataFrame**

![image](https://user-images.githubusercontent.com/86877457/135080508-8faa6cb4-908d-4d6e-9a12-865d2e3fbf46.png)

**Overall Monthly Returns for the Optimized Portfolio**

![image](https://user-images.githubusercontent.com/86877457/135081021-cb7fd32c-431a-4a2a-8649-4688fff5f571.png)

### Result

For the given portfolio, stocks, the values of n and x, it perfermoed quite well. the returns obtained from this was 0.3 % better. 

![image](https://user-images.githubusercontent.com/86877457/135081662-e750ad7f-4546-44ba-8583-5ee461c8ebbf.png)

Our model efficiently showed the desired portfolio for each month.

![image](https://user-images.githubusercontent.com/86877457/135081559-7886c0db-9590-44b5-aa38-00f01b785a1a.png)

**KPIs**

![image](https://user-images.githubusercontent.com/86877457/135082967-7a3a35ed-bd94-4113-97eb-641d8c5ca5e5.png)


<!-- FINAL THOUGHTS -->
## Final Thoughts

From the above analysis, we can see that this method actually improves the returns from stocks significantly. May not work in every case but can show some splendid results for some specific portfolio and the values of n and x. For further improvements, we can apply a loop to find the best value of x and n. And also we can find the time after which we should change the x and n values.
