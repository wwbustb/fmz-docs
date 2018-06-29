2.2 Exchange Variable
=======================

Every API method is called by exchange object, such as ``exchange.GetTicker()``, ``exchange.Buy()``.

Exchange and trade pair is set when you start a robot.

**exchange**

It can be imaged as a real exchange house; default setting is the first exchange in your strategy’s parameter. 
all of data interactive with the exchange which achieve through this object function.

**exchanges**

The exchange’s array, it contains multiple exchange objects which are sorted by order of addition . used as ``exchanges[0]`` , ``exchange[1]``  etc...

.. note::

    If you use ``exchange`` directly, which equals to ``exchanges[0]``.

    When you add an exchange, it's always fllowed by a trade pair, such as ``BTC_USDT``, so
    ``exchanges[0]`` and ``exchanges[1]`` can be the same "actually  exchange" but with different trade pair.


**GetName**

``exchange.GetName()`` Return the name of the exchange, string type

**GetLabel**

``exchange.GetLabel()`` Return the exchange's custom label, string type




