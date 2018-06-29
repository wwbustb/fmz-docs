2.3 Market API
=======================

The following functions all can be called from object ``exchange`` or ``exchanges[x]``,for example：``exchange.GetTicker()`` or ``exchanges[0].GetTicker()`` means return the market quotations.

.. warning::

    When calling any API function that accesses the exchange house’s interface (such as ``GetTicker()``, ``Buy()``, ``CancelOrder()``, etc.), it may get access failure due to exchange server problem, the network transmission problem, and so on.
    In this case, ``GetTicker()`` will return ``null``, which may cause the stop of your programe.  a JS example to do fault tolerance as below.

    .. code-block:: JavaScript

        var ticker = exchange.GetTicker()
        if(ticker == null){
            // Retry, or other processing logic.
        }
    A deflaut retry function is ``_C()``

2.3.1 GetTicker
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetTicker()

Acquiring the current market quotations.

Return value: Ticker structure

The Ticker structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Info                Object               the original structure returned by the exchange
High                Number               Highest price
Low			        Number               lowest price
Sell                Number               the latest selling price
Buy                 Number               the latest buying price
Last                Number	             last traded price
Volume              Number               most recent trading volume
OpenInterest        Number               net position(only for features)
==================  ==================== ===============

Example ticker from binance:

.. sourcecode:: http

    {
        "Info":{
                "highPrice":"6173.01000000",
                "openTime":1530152780326,
                "firstId":53572675,
                "lastId":53735989,
                "count":163315,
                "priceChange":"-266.96000000",
                "weightedAvgPrice":"6035.81831943",
                "bidPrice":"5871.63000000",
                "openPrice":"6139.00000000",
                "closeTime":1530239180326,
                "lowPrice":"5827.00000000",
                "quoteVolume":"197096315.23211791",
                "priceChangePercent":"-4.349",
                "prevClosePrice":"6139.00000000",
                "lastQty":"0.25866000",
                "bidQty":"0.00300000",
                "askPrice":"5872.05000000",
                "symbol":"BTCUSDT",
                "lastPrice":"5872.04000000",
                "askQty":"0.07344000",
                "volume":"32654.44796400"
            },
        "High":6173.01,
        "Low":5827,
        "Sell":5872.05,
        "Buy":5871.63,
        "Last":5872.04,
        "Volume":32654.447964,
        "OpenInterest":0,
        "Time":1530239180443
    }

A JavaScript example using the variables in the Ticker structure:

.. code-block:: JavaScript

    function main(){
        var ticker = exchange.GetTicker();
        Log("High:", ticker.High, "Low:", ticker.Low, "Sell:", ticker.Sell, "Buy:", ticker.Buy, "Last:",
            ticker.Last, "Volume:", ticker.Volume);
    }

For Python the code is basically the same:

.. code-block:: Python

    def main():
        ticker = exchange.GetTicker()
        Log("High:", ticker.High, "Low:", ticker.Low, "Sell:", ticker.Sell, "Buy:", ticker.Buy, "Last:",
            ticker.Last, "Volume:", ticker.Volume)

.. note::

    If you use the a number in ``Info`` directly, make sure the data type is float.

    - For JavaScript: ``var priceChange = praseFloat(ticker.Info.priceChange);``
    - For Python: ``priceChange = float(ticker.Info["priceChange"])``.

2.3.2 GetDepth
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetDepth()

Acquiring the exchange order book.

Return value: Depth structure

The Depth structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Asks                Array                the array of asks,from low to high by price
Bids                Array                the array of bids,from high to low by price
Time                Number               the timestamp of request
==================  ==================== ===============

The Asks and Bids structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Price               Number               the pirce of ask or bid
Amount              Number               the amount of ask or bid
==================  ==================== ===============

Example depth from binance:

.. sourcecode:: http

    {
        "Info":null,
        "Asks":[
            {"Price":5866.38,"Amount":0.068644},
            {"Price":5866.39,"Amount":0.263985},
            {"Price":5866.73,"Amount":0.05},
            {"Price":5866.77,"Amount":0.05},
            {"Price":5867.01,"Amount":0.15},
            {"Price":5875.89,"Amount":0.05},
            ......
            ]
        "Bids":[
            {"Price":5865.13,"Amount":0.001898},
            {"Price":5865,"Amount":0.085575},
            {"Price":5864.15,"Amount":0.013053},
            {"Price":5863.65,"Amount":0.016727},
            {"Price":5863.51,"Amount":0.128906},
            {"Price":5863.15,"Amount":0.2}
            ......
            ],
        "Time":1530241857399
    }

A useful JavaScript example for using depth:

.. code-block:: JavaScript

    function main(){
        var depth = exchange.GetDepth();
        var price = depth.Asks[0].Price;
        var amount = depth.Asks[0].Amount;
        if(amount > 10){
            exchange.Buy(price, 10);
        }
