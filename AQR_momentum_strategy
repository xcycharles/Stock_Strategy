# https://blog.csdn.net/xieyan0811/article/details/101283943
# https://stackoverflow.com/questions/38424673/python-import-a-module-directory-given-full-path
import sys
# print(sys.path)
sys.path.insert(0,'/Users/Charles/Desktop/git/pyalgotrade')
# print(sys.path)

# from importlib.machinery import SourceFileLoader
# problem_module = SourceFileLoader('pyalgotrade','/Users/Charles/Desktop/git/pyalgotrade/pyalgotrade/strategy/__init__.py').load_module()

import pyalgotrade
print(pyalgotrade.__file__)

from pyalgotrade import strategy  # 策略
from pyalgotrade import plotter  # 做图
from pyalgotrade.technical import ma  # 技术方法
from pyalgotrade.technical import cross  # 技术方法
from pyalgotrade.stratanalyzer import returns  # 评价
from pyalgotrade.stratanalyzer import sharpe
from pyalgotrade.stratanalyzer import drawdown
from pyalgotrade.stratanalyzer import trades
from pyalgotrade.barfeed import membf
from pyalgotrade import bar
import tushare as ts
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
# from samples import tushare_api_china_stock_pick
import datetime as dt
from dateutil.relativedelta import relativedelta


class Feed(membf.BarFeed):  # 做自己的数据源，从tushare中读取
    def __init__(self, frequency=bar.Frequency.MONTH, maxLen=None):
        super(Feed, self).__init__(frequency, maxLen)


    def rowParser(self, ds, frequency=bar.Frequency.MONTH):
        dt = pd.to_datetime(ds["date"])
        open = float(ds["open"])
        close = float(ds["close"])
        high = float(ds["high"])
        low = float(ds["low"])
        volume = float(ds["volume"])
        return bar.BasicBar(dt, open, high, low, close, volume, None, frequency)

    def barsHaveAdjClose(self):
        return False

    def addBarsFromCode(self, code, start, end, ktype="D", index=False):
        frequency = bar.Frequency.MONTH
        ds = ts.get_k_data(code=code, start=start, end=end,
                           ktype=ktype, index=index)
        bars_ = []
        for i in ds.index:
            bar_ = self.rowParser(ds.loc[i], frequency)
            bars_.append(bar_)
        self.addBarsFromSequence(code, bars_)  # 从数据流中组装数据
        # print(type(bar_))
        # print(bars.items())

def df_cut(data, cut, labels=None):
    min_num = data.min()
    max_num = data.max()
    b = [min_num] + cut + [max_num]
    if not labels:
        labels = range(len(cut) + 1)
    else:
        labels = [labels[i] for i in range(len(cut) + 1)]
        Bin = pd.cut(data, bins=b, labels=labels, include_lowest=True)
    return Bin

class MyStrategy(strategy.BacktestingStrategy):  # 继承策略的父类
    def __init__(self, feed, instrument):
        super(MyStrategy, self).__init__(feed) #this does loading of the feed
        self.__instrument = instrument

        # <class 'pyalgotrade.dataseries.SequenceDataSeries'>
        self.__closed = feed[instrument].getCloseDataSeries()

        # pyalgotrade.technical.ma.SMA object; must use all close prices here to calculate sma strategy

        self.__position = None

        # <class 'pyalgotrade.barfeed.resampled.ResampledBarFeed'>
        # **************important*****************
        self.__resampledBF = self.resampleBarFeed(bar.Frequency.MONTH, self.resampledOnBar)
        # **************important*****************
        # <class 'pyalgotrade.dataseries.bards.BarDataSeries'>
        self.__barMonth = self.__resampledBF.getDataSeries(self.__instrument)

        # <class 'pyalgotrade.dataseries.SequenceDataSeries'>
        self.__priceMonth = self.__resampledBF.getDataSeries(self.__instrument).getPriceDataSeries()

        # <class 'pyalgotrade.dataseries.SequenceDataSeries'>
        self.__closeMonth = self.__resampledBF.getDataSeries(self.__instrument).getCloseDataSeries()


    def onEnterCanceled(self, position):
        self.__position = None
        print("onEnterCanceled", position.getShares())

    def onExitOk(self, position):
        self.__position = None
        print("onExitOk", position.getShares())

    def onExitCanceled(self, position):
        self.__position.exitMarket()
        print("onExitCanceled", position.getShares())

    # just for my testings, close will be printed monthly if called in self.__resampledBF = self.resampleBarFeed(bar.Frequency.MONTH, self.onBars)
    def resampledOnBar(self, bars):
        bar = bars[self.__instrument] # pyalgotrade.bar.BasicBar object
        # print("Date: {} Close: {} PriceHigh: {}" .format(bar.getDateTime(), round(bar.getClose(),2), round(bar.getHigh(),2) ))
        shares = int(self.getBroker().getCash() * 0.5 / bar.getHigh()) # 80% of cash stock
        # print("this is current total cash {}" .format(round(self.getBroker().getCash(),2)))
        # print("shares is {}" .format(shares))
        print("Date: {} Open: {} Close: {} PriceLow: {} PriceHigh: {} ".format(bar.getDateTime(), round(bar.getOpen(), 2), round(bar.getClose(), 2),round(bar.getLow()), round(bar.getHigh())))
        if self.__position is None:
            self.__position = self.enterLong(self.__instrument, shares, False, True) # this to enter market order # pyalgotrade.strategy.position.LongPosition object
            print("enter long for {} shares at {} price".format(shares, bar.getPrice()))
            print("remaining cash is " + str(self.getBroker().getCash()))
            print("position is " + str(self.__position.getShares()))
        elif not self.__position.exitActive(): # Returns True if the exit order is active
            # if not exit orders being active
            self.__position.exitMarket(True) # this just submits the market order and self.__position becomes none
            print("exit for {} shares at {} price".format(self.__position.getShares(), bar.getPrice()))
            print("remaining cash is " + str(self.getBroker().getCash()))
            print("position is " + str(self.__position.getShares()))
    # 这个函数每天调一次!!! no matter the resampling
    def onBars(self, bars):
        pass
        # bar = bars[self.__instrument]  # bar是k线中的每个柱
        # self.info("onBars: Close: %s Frequency:%s" % (bar.getClose(), bar.getFrequency()))

    def getClose(self):
        return self.__closed

# this is not in the same strategy class
def tushares(startday, endday): # in the format like 20180101
    basics_data = ts.get_stock_basics()
    trade_data = ts.get_today_all()
    # print(basics_data.head(5))
    # data washing
    col = ['name', 'outstanding', 'pe', 'pb', 'esp',
           'reservedPerShare', 'rev', 'profit', 'gpr', 'npr']
    newcol = ['简称', '流通股', '市盈率', '市净率', '每股收益', '每股公积',
              '收入同比', '利润同比', '毛利率', '净利率']
    d = dict(zip(col, newcol))
    b_data = basics_data.loc[:, col]
    b_data.rename(columns=d, inplace=True)
    # 当前股价,如果停牌则设置当前价格为上一个交易日股价
    trade_data['trade'] = trade_data.apply(lambda x: x.settlement if x.trade == 0 else x.trade, axis=1)
    # 选取股票代码,名称,当前价格,总市值,流通市值
    t_data = trade_data.loc[:, ['code', 'trade', 'mktcap', 'nmc', 'volume', 'turnoverratio']]
    # 设置行情数据code为index列
    t_data = t_data.set_index('code')
    t_data.rename(columns={'trade': '收盘价', 'mktcap': '总市值', 'nmc': '流通市值','volume': '成交量', 'turnoverratio': '换手率'}, inplace=True)
    # 将总市值和流通值换成亿元单位
    t_data['总市值'] = t_data['总市值'] / 10000
    t_data['流通市值'] = t_data['流通市值'] / 10000

    data = b_data.merge(t_data, left_index=True, right_index=True)

    cut = [5, 20]
    labels = ['小盘股', '中盘股', '大盘股']
    # 调用函数df_cut,增加新列
    data_new = data.loc[:, ['简称', '收盘价', '流通股', '市盈率', '每股收益', '净利率', '收入同比', '利润同比']]
    data_new['股票类型'] = df_cut(data['流通股'], cut, labels)
    # 查看标签列，取值范围前面加上了序号，是便于后面生成表格时按顺序排列
    ############################## for pyalgotrade
    pro = ts.pro_api()
    performance = {}

    # 设置参数和过滤值(根据需要不断调整)
    # 市盈率>0
    mid = data_new['股票类型'] == '大盘股'
    # pe0 = data_new['市盈率'] > 0
    # pe1 = data_new['市盈率'] < 20
    # 流通股本<=20亿
    eps = data_new['每股收益'] >= 1
    # 收入同比正数
    # rev = data_new['收入同比'] > 15
    # 利润同比
    # profit = data_new['利润同比'] > 15
    # 净利率>5%
    # npr = data_new['净利率'] > 15
    # 取并集结果:
    select = mid & eps
    port = data[select]
    port.index

    import re
    for i in port.index:
        try:
            if re.match(r'60', i) != None:  # 60 is for shanghai stocks
                performance[i] = pro.monthly(ts_code=i + '.SH', start_date=startday, end_date=endday)['pct_chg'].values
            elif re.match(r'60', i) == None:  # 00 is for shenzhen stocks
                performance[i] = pro.monthly(ts_code=i + '.SZ', start_date=startday, end_date=endday)['pct_chg'].values
        except:
            pass

    performance_sorted = sorted(performance.items(), key=lambda x: x[1], reverse=True)  # this becomes a list
    print(performance_sorted)
    buy_stock = []
    for i in performance_sorted[:3]:
        buy_stock.append(i[0])
    return buy_stock

###################################### below is main ##############################################
for x in [0 2 4 6 8]:
    dt_1 = dt.datetime(2019, 1, 1) + relativedelta(months=x)
    dt_1month = dt_1 + relativedelta(months=1)
    dt_3month = dt_1 + relativedelta(months=4)
    print(dt_1.strftime("%Y-%m-%d"))
    print(dt_1month.strftime("%Y-%m-%d"))
    print(dt_3month.strftime("%Y-%m-%d"))
    buy_stock = tushares(dt_1.strftime("%Y%m%d"),dt_1month.strftime("%Y%m%d")) #this must be one month and shows the 3 best performing stocks asof the enddate
    print("\n buy these stocks" + str(buy_stock))
    total = 0
    for i in buy_stock:
        print("===============this is for stock " + i + "====================")
        code = i
        feed = Feed()
        feed.addBarsFromCode(code, start=dt_1month.strftime("%Y-%m-%d"), end=dt_3month.strftime("%Y-%m-%d"))
        #########################
        myStrategy = MyStrategy(feed, code)  # 最重要的策略类
        myStrategy.run()  # 开始运行，然后事件驱动
        ###########################
        print("最终投资组合价值: $%.2f" % myStrategy.getResult())
        total += round(myStrategy.getResult(), 2)
myStrategy.info("最终投资组合价值: {}".format(round(total/len(buy_stock),0)))
    # plt = plotter.StrategyPlotter(myStrategy)  # 做图分析
    # retAnalyzer = returns.Returns()  # 评价
    # myStrategy.attachAnalyzer(retAnalyzer)
    # sharpeRatioAnalyzer = sharpe.SharpeRatio()
    # myStrategy.attachAnalyzer(sharpeRatioAnalyzer)
    # drawDownAnalyzer = drawdown.DrawDown()
    # myStrategy.attachAnalyzer(drawDownAnalyzer)
    # tradesAnalyzer = trades.Trades()
    # myStrategy.attachAnalyzer(tradesAnalyzer)
    # myStrategy.info("最终投资组合价值: {}".format(round(total/len(buy_stock),0)))
    # myStrategy.info("最终投资组合价值: $%.2f" % myStrategy.getResult())
    # print("累计回报率: %.2f %%" % (retAnalyzer.getCumulativeReturns()[-1] * 100))
    # print("夏普比率: %.2f" % (sharpeRatioAnalyzer.getSharpeRatio(0.05)))
    # print("最大回撤率: %.2f %%" % (drawDownAnalyzer.getMaxDrawDown() * 100))
    # print("最长回撤时间: %s" % (drawDownAnalyzer.getLongestDrawDownDuration()))
    # print("")
    # print("总交易 Total trades: %d" % (tradesAnalyzer.getCount()))
        # if tradesAnalyzer.getCount() > 0:
        #     profits = tradesAnalyzer.getAll()
        #     print("利润", "mean", round(profits.mean(), 2), "std", round(profits.std(), 2),
        #           "max", round(profits.max(), 2), "min", round(profits.min(), 2))
        #     returns = tradesAnalyzer.getAllReturns()
        #     print("收益率", "mean", round(returns.mean(), 2), "std", round(returns.std(), 2),
        #           "max", round(returns.max(), 2), "min", round(returns.min(), 2))
        # print("")
        # print("赢利交易 Profitable trades: %d" % (tradesAnalyzer.getProfitableCount()))
        # if tradesAnalyzer.getProfitableCount() > 0:
        #     profits = tradesAnalyzer.getProfits()
        #     print("利润", "mean", round(profits.mean(), 2), "std", round(profits.std(), 2),
        #           "max", round(profits.max(), 2), "min", round(profits.min(), 2))
        #     returns = tradesAnalyzer.getPositiveReturns()
        #     print("收益率", "mean", round(returns.mean(), 2), "std", round(returns.std(), 2),
        #           "max", round(returns.max(), 2), "min", round(returns.min(), 2))
        # print("")
        # print("亏损交易Unprofitable trades: %d" % (tradesAnalyzer.getUnprofitableCount()))
        # if tradesAnalyzer.getUnprofitableCount() > 0:
        #     losses = tradesAnalyzer.getLosses()
        #     print("利润", "mean", round(losses.mean(), 2), "std", round(losses.std(), 2),
        #           "max", round(losses.max(), 2), "min", round(losses.min(), 2))
        #     returns = tradesAnalyzer.getNegativeReturns()
        #     print("收益率", "mean", round(returns.mean(), 2), "std", round(returns.std(), 2),
        #           "max", round(returns.max(), 2), "min", round(returns.min(), 2))
        #
        # plt.plot()

