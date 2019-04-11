The documents for www.fmz.com. used to be www.botvs.com.

Read on http://fmz-docs.readthedocs.io/en/latest/.

## Start with FMZ platform

Thanks for using our platform, this is a basic instruction for beginers, for full version of our API documentation, check on [FMZ API](https://fmz-docs.readthedocs.io/en/latest/). There are many obvious specs that aren't covered in this tutorial left for you to discover.

>**What is FMZ platform?**

FMZ is an automated trading platform for cryptocurrency traders with support for many bitcoin/eth/altcoin exchange markets.


>**What can FMZ do for you?**

You can learn how to write your bots(strategies) from our strategies’ square which contains lots of open source code, share your strategy’s code with others, ask for professional help any time, run your strategy on many exchanges, contorl your bot on website with computer or cellphone, sell your strategies if you want, communicate with many other auto-trading lovers in our [group](https://t.me/fmzquant) . In a word, FMZ is a perfect platform for those who want to automated trading.

>**Which Cryptocurrency exchanges does FMZ support?**

FMZ supports almost all exchanges that are popular, such as ``Binance``, ``Bitfinex``, ``Bitstamp``, ``OKEX``, ``Huobi``, ``Poloniex``,etc. you can also trade futures on ``OKEX`` and ``BitMEX``. Check the full support list on [API](https://fmz-docs.readthedocs.io/en/latest/code_Instruction/Exchange%20Variable.html). You only need to write one strategy and run it on all exchanges without any changes.

>**What programming languages does FMZ support?**

FMZ supports JavaScript, Python, C++ (JavaScript and Python are recommended) for coding your strategies. Benefiting from the completed languages supporting(not a custom language only can be used for one platform), you can improve your programming skills as well as learn to write strategies.

>**Is your API KEY safe?**

Indeed. your API-KEYs are saved after encryption.Here is how it works.
   - You need to  input the FMZ password When add the key.Web-browser will encrypt your api key and pass the encrypted key to FMZ server using ``https``.
   - You need to input the FMZ password when you run the docker on your server or computer.
   - FMZ will send the encrypted key to your docker using https when you start a bot. The docker will decrypt to get the key.
   - FMZ don't save your password(only password Hash).So even we don't know your api key.
   - You need to pretect your own password and docker server.It is best to use a second verification login FMZ.
  
>**Current feature list:**

  - 1.Cross-platform, supports for all major cryptocurrency exchanges — more coming soon
  - 2.Supports for the simulated exchange. https://wex.app/
  - 3.Has an effective simulate backtesting system. 
  - 4.Supports for sending e-mails, pushing messages to your telegram account.
  - 5.Web-based control, can be acessed through your phone.
  - 6.Supports for Python\C++\JavaScript programming.
  - 7.The cost is extremely low. 0.125 RMB per hour, about 0.018 dollar.
  - 8.No API-KEY or passwords are saved on our server. FMZ has been running for more than four years without any security issues.
