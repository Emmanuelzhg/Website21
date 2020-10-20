---
title: "About my undergarduate thesis : Volatility-Target Strategy"
output: html_document

categories:
- ""
- ""
date: "2019-10-31T21:28:43-05:00"
description: ""
draft: false
image: finance.jpg
keywords: ""
slug: blog2
title: About me
---

## Introduction
In the early 2000s, the ETF market was marked by a very rapid upward trend. While outstandings represented only some $100 billion under management, in 2018 they represented $4700 billion according to Blackrock, an increase 47 times higher. Year after year, ETFs are diversifying and there is something for everyone. Global assets under management according to Blackrock could still increase and reach some "$12 trillion by 2023 and $25 trillion by the end of 2027".

Passive management through ETFs is increasingly used by active managers as a way to diversify certain pockets of their portfolios for their clients. Their simplicity, transparency and cost make them perfect investment vehicles. 

On the other hand, an overinvestment in ETFs is particularly dangerous for financial markets, making their structure "sheep-like" and particularly exposed to the risks of different crises. This is why we will focus in our study on the alternative investment market as a means of diversification of asset portfolios for investors. More particularly, our study will focus on systematic strategies and in particular alternative Risk Premia strategies, but will also study the modeling of a simple systematic strategy of target volatility "Vol-Target".

Thus we will see that there is a democratization of systematic investment strategies. Our study will answer the question: How can quantitative investment strategies allow an investor to perform while reducing risk? 



## What is  alternative investment ?
An alternative investment is a financial asset that does not fall into one of the classic investment categories. Conventional categories include equities ("equity"), bonds ("fixed income") and cash ("cash"). Most alternative investment assets are held by accredited institutional or individual investors with high net worth due to their complex nature, lack of regulation and degree of risk.

The CFA Institute indicates that alternative investments include private equity or venture capital, hedge funds, managed futures, works of art and antiques, commodities and derivative contracts. Real estate is also often classified as an alternative investment.


## What is  Quantitative Investment Strategies (QIS) ?
Quantitative investment strategies have evolved into complex tools with the advent of modern computers, but the roots of these strategies go back more than 80 years. They are generally managed by highly skilled teams and use proprietary models to increase their ability to beat the market. Quantitative models still work well when "backtested", but their actual applications and success rate are questionable. While they seem to work well in bull markets, when markets go off track, quantitative strategies are subject to the same risks as any other strategy. 

The use of quantitative finance and calculation has given rise to many common tools, including the Black-Scholes formula for valuing options.

When quantitative theories are applied to portfolio management, the objective is the same as for any other investment strategy: to add value or also called alpha: excess returns over the risk-free rate of return ("Risk-free"). 
Quantitative developers ("Quants"), develop complex mathematical models to detect investment opportunities. The advantage of a quantitative investment strategy lies in the use of a model that makes the buy/sell decision, instead of the human being. This tends to suppress any emotional reaction a person may experience when buying or selling investments.

Quantum strategies are now accepted by the investment community and are managed by mutual funds, hedge funds and institutional investors.


## Methodology
Uncertainty about asset performance and the vagaries of the markets have prompted researchers in the various research departments of banks, other financial institutions and fintech to find ways to quantify and reduce these uncertainties. These anticipations have been the basis of research with the problem: "Do past performances influence future performances? 

This research process gave birth to various mathematical and computer models to validate this research. Our case study will focus on backtesting to validate a model for reducing volatility.

With the advent of new technologies and the accessibility of financial data, quantitative strategies are particularly easy to implement. In addition, financial institutions invest in IT (and fiber optics) to be able to execute trades at phenomenal speed.

In our study, we will backtest our Vol-Target strategy, i.e. the search for target volatility under Python. 

Our python code will give us the different daily levels of a benchmark index with its twin Vol-Target at a target level of 10%. The benchmark index of our study will be the CAC 40 index of the 40 largest market capitalizations in France as of June 2020. The code will work in a less sophisticated way than an optimal and efficient strategy that could be proposed by a bank or other financial institution at the forefront in terms of programming for reasons of simplification, illustration of the concept and implementation.

Our backtesting consists of several implementation steps. These steps include :


- Selection of the investment universe: Here, for simplicity, we will directly take an index, the CAC 40, so that we do not have to create our own optimal portfolio and select our assets. This index will also serve as a benchmark to compare our two indices.
- An initial screening as a filter: This filter will make it possible to eliminate certain outliers that do not meet our performance or risk management expectations. 
- Modeling of results: In order to visualize our results, it is important to model them and to take into account (normally) transaction costs and various ancillary costs such as calculation agents, infrastructure, etc. In our case, for the sake of simplicity, we will not take into account ancillary costs to focus solely on the benefits and drawbacks of the strategy.
- Optimization of the results obtained: Our Vol-target strategy on the CAC 40 is exposed to various backtesting biases by the very fact of selecting a French benchmark index, particularly the survival bias. It would be important to visualize different possible scenarios and choose the behavior that the strategy will have globally. Here again, our optimization step will be done in a superficial way, notably by choosing an optimal target volatility level to illustrate the concept of "Vol-Target": here it will be 10%.

## Analytical results

We still take parameters with a target volatility of 10% but this time the maximum exposure goes from 100% to 500% and the volatility calculation window will be 7 days.


![Image](https://imgur.com/xUdFWKV.jpg)

![Image](https://imgur.com/WnE939b.jpg)

![Image](https://imgur.com/kjUSixJ.jpg)

![Image](https://imgur.com/nLhxkro.jpg)

![Image](https://imgur.com/uEdqNAG.jpg)

![Image](https://imgur.com/3ahaUA9.jpg)

With a volatility level calculation window reduced to 7 days to react very quickly to new market trends; our index performs extremely well against its benchmark, its underlying.

The excess return of our dynamic index compared to its benchmark allows us to say that it is in every way better and we have optimized the strategy compared to the previous backtest.


The analysis of the performance tables in figures 21 and 25 allows us to see that the volatility of our strategy being constant over time, as expected around 10%, only the performance obtained will modify the Sharpe ratio. The Sharpe ratio measures the excess return on the risk-free rate generated for an unité́ additional risk, volatility. 

The Sharpe ratio will reflect the psychology of an investor. Very logically one expects to get more return when one takes more risk. Here we can see that the Sharpe ratio of our strategy is higher than the benchmark during strong periods of decline. 


The choice of such an investment strategy is not profit maximization although the strategy outperforms its benchmark index on our backtesting but protection in case of a sharp decline and therefore high volatility.

The investor who will turn to this investment strategy will be looking for a product that replicates the variations of the CAC 40 without replicating the amplitude of these variations. Risk management through their investment choice allows them to better select the components of the assets they will integrate into their portfolio and guide them in their choice. 



## Conclusion
To conclude, it would be interesting to take stock of systematic strategies and in particular volatility control strategies in terms of risk management.

Systematic strategies allow investors to access a multitude of strategies in the form of an index. The advantage of these indices is that they operate according to a very precise strategy and therefore a well-defined execution by a set of programmed "rules".
This particularity makes it possible to remove the emotional variable that a human portfolio manager could have depending on the situation. 

All investors have a different risk tolerance. The risk in terms of volatility for each asset can be quantified and analyzed in order to allow everyone to decide on their asset allocation. With the rise of information technology, big data, and research, new models are appearing on the market giving rise to systematic strategies, (from computer-assisted active management). These new strategies allow investors to diversify their portfolios and we observe very interesting results!

Our volatility control strategy allows certain investors to meet a need, that of controlling risk. In our case, we have chosen to compare the dynamic index to its underlying to really appreciate the contribution of this kind of filter.
With a target volatility of 10% as a parameter, our optimized dynamic index proves to be more interesting than its underlying for various reasons: firstly, a target volatility that remains stable, and secondly, higher returns justified by overexposure during periods of rises while underexposing itself when the market is very bearish! We mitigate very large financial crises without using very expensive put options. The more we will reduce the target volatility compared to the benchmark/underlying index, the more difficult it will be for the dynamic strategy to achieve a significant performance. Simply, with a lower target volatility, the index will not follow when there is a rebound in the markets... This target volatility strategy is already relatively well marketed in large banks!

Introducing this type of strategy decorrelated from traditional asset classes allows to diversify the portfolio while reducing the risk to optimize the final Sharpe ratio of a portfolio. 

Finally, we observed during our study a point of improvement for this type of strategy, which is the capture of the rebound and in particular the inability of our strategy to "predict" the rise in volatility levels and therefore future rebounds. More and more hedge funds are developing strategies combined with a learning machine to better predict rebounds and minimize the time it takes their strategy to capture post-crisis returns.





## Contact details

Please feel free to me an e-mail at ezheng.mfa2021@london.edu if you want to connect. Alternatively you can connect with me on my social media accounts - you can see the links at the bottom of the page.