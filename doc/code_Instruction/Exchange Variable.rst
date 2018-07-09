2.2 Exchange
=======================

Every API method is called by exchange object, such as ``exchange.GetTicker()``, ``exchange.Buy()``.

Exchange and trade pair is set when you start a robot. The following market api and trade api is based on this setting.
If you set your exchange is "Binance", trading pair is "BTC_USDT", then call ``exchange.GetTicker()``, you will get the ticker of "BTCUSDT" of Binance.
trading pair is also called ``symbol`` or ``currency`` in some exchenge's API docs,

2.2.1 Add Exchange
>>>>>>>>>>>>>>>>>>

Check on how to add exchange when start a robot:
:ref:`add exchange`


Now support:

.. sourcecode:: http

    AEX
    BigONE
    Binance
    BitFlyer
    Bitfinex
    Bithumb
    Bitpie
    Bitstamp
    Bittrex
    BotVS
    CoinEx
    CoinPlus
    Coincheck
    Coinone
    Exchange
    Futures_BitMEX
    Futures_CTP
    Futures_Deribit
    Futures_Esunny
    Futures_OKCoin
    GateIO
    HitBTC
    Huobi
    KEX
    Korbit
    Kraken
    LiveCoin
    OKCoin_EN
    OKEX
    Poloniex
    Quoine
    WEX
    ZB
    Zaif


2.2.2 Exchange Variable
>>>>>>>>>>>>>>>>>>

**exchange**

It can be imaged as a real exchange; default setting is the first exchange in your strategy’s parameter. 
all of data interactive with the exchange which achieve through this object function.

**exchanges**

The exchange’s array, it contains multiple exchange objects which are sorted by order of addition . used as ``exchanges[0]`` , ``exchange[1]``  etc...

.. note::

    If you use ``exchange`` directly, which equals to ``exchanges[0]``.

    When you add an exchange, it's always fllowed by a trade pair, such as ``BTC_USDT``, so
    ``exchanges[0]`` and ``exchanges[1]`` can be the same "actually  exchange" but with different trade pair.


2.2.3 Exchange Information
>>>>>>>>>>>>>>>>>>

**GetName**

``exchange.GetName()`` Return the name of the exchange, string type

**GetLabel**

``exchange.GetLabel()`` Return the exchange's custom label, string type

2.2.4 FMZ Simulation Exchange
>>>>>>>>>>>>>>>>>>

FMZ Simulation Exchange is basically the same as a real exchange, you can run your robot on FMZ Simulation Exchange for testing your strategy, which is totally free.
check it on https://www.fmz.com/m/sandbox, and deposit some money or bitcoin to start


