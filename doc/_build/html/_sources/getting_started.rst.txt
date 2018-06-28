1.1 Getting Started
=======================

What if I don’t have any program developing or financial skills? Can I still make quantitative trading on this platform?

Absolutely, our platform has a complete tutorial for beginners. Only need studying a couple of days, you are good to go.   Botvs for rookies

**What kinds of programming languages can I use to achieve my strategies?**

FMZ supports `JavaScript`, `Python`, `C++` for coding your strategies.	JavaScript Tutorial		Python Tutorial. (One of these two main programming tools will meet the most requirement)

**What is The docker?**

The docker is a program that runs on your own Internet Server, which in charge of the Data request, Data reception, Network link, Log review etc. 
You can treat it like your strategy’s executor. Even if the FMZ server offline (breakdown, etc.), 
it will have no influence on your manager for function properly. 

The Manager can run on variety of operating system. 
Such as `Windows`, `Linux`, `Mac OS`, `Android`, `Raspbian`, etc.		The manager Installation

1.2	The Backtest System
=======================

**What is backtest system? What it uses for?**

When you completed a quantitative strategy, how do you know that the logic, amount and direction of profit of this strategy goes? Does it work? Apparently, we wouldn’t use the real money to test our strategies on the real market. But we could use the historical data to test it, see how it works out on the time, profit, and asset management in the past. It is winning or losing?		How to use the Backtest System?

**Does the number from the backtest system accurate? The results can be trusted?**

FMZ divides the backtest system into real market level and simulation level. 
The real market level contains the whole completed historical data back testing. 
The simulation level contains the candle bar graph data at regular intervals. 
Both level are	based on the real market historical data. Only the real market level is more accurate and the results are more reliable.

**The simulation level Backtest System explanation.**

Notice that backtesting is the strategy only preform in the past. Historical data doesn’t represent the future. History may replay, it also may lead to the black swan. Please treat the backtest results reasonable and objectively.