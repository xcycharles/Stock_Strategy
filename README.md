# My_Stock_Momemtum_Strategy
This trading strategy is based on the momentum paper published by AQR in the 1990's. The strategy file needs to run with modified pyalgotrade source code. Request to xcy9999@hotmail.com if needed. I used "Tushare" as my datasource and pyalgotrade as the backtesting platform. After washing the raw financial data, I grouped daily k bars to monthly k bars for this strategy, which was devised to pick the three best performing stocks in one month with Outstanding_Share and EPS requirements to filter the entire stock population. Then hold the position from trading at market price in maximum volume for a month to measure the portfolio performance. Finally the PnL result turned out to be randomly distributed when running against year 2018-2019.

# Notes:
- Daily k bars are combined into monthly k bars
- All orders are sent using the market price, but some may not be filled due to volumes are messed up in monthly k bars
- The strategy picks up 3 best pct_chg stocks in one month with some outstanding share and EPS requirements to filter the population, and hold for a month to measure the portfolio performance

# Portfolio performance in 2019

Month	Best performing SH/SZ stocks with specified OS > 20 million and EPS > 1	win pct change	Return after holding for 1 month
01/2019	601155	23.9%	110.7%
	000858	18.5%	
	000001	18.3%	
2019 March	600352	58.4%	99.4%
	601155	33.5%	
	000858	32.9%	
2019 May	603288	12.2%	96.0%
	002142	1.3%	
	600036	-0.7%	
2019 July	300498	12.9%	101.4%
	600048	11.4%	
	600031	8.0%	
2019 Sep	601155	14.5%	99.6%
	600188	11.3%	
	002142	10.4%	
		Total return:	101.4%
