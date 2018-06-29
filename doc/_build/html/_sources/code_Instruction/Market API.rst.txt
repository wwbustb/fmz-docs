2.3 Market API
=======================

The following functions all can be called from object ``exchange`` or ``exchanges[x]``,for example：``exchange.GetTicker()`` or ``exchanges[0].GetTicker()`` means return the market quotations.

.. warning::

    When calling any API function that accesses the exchange house’s interface (such as ``GetTicker()``, ``Buy()``, ``CancelOrder()``, etc.), it may get access failure due to exchange server problem, the network transmission problem, and so on.
    In this case, ``GetTicker()`` will return ``null``, which may cause the stop of your programe.  a JavaScript example to do fault tolerance as below.

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

A useful JavaScript example by using depth:

.. code-block:: JavaScript

    function main(){
        var depth = exchange.GetDepth();
        var price = depth.Asks[0].Price;
        var amount = depth.Asks[0].Amount;
        if(amount > 10){
            exchange.Buy(price, 10);
        }    
    }

2.3.2 GetTrades
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetTrades()

Acquiring Exchange Trading History.(not your trading history)

Return value: Array of Trade Structure 

.. note::

    Some exchanges do not support this method, the number of return data depends on exchanges.

The Trade structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Time                Number               Unix timestamp of the trade time
Price               Number               price of the trade
Amount              Number               amount of the trade
Type                Order Type           Order Type Constant
==================  ==================== ===============

Order Type is global constant, you can take ``ORDER_TYPE_BUY`` as ``0`` :

==================  ==================== ===============
Global constant     Meaning                 Value
==================  ==================== ===============
ORDER_TYPE_BUY      buy order            0
ORDER_TYPE_SELL     sell order           1
==================  ==================== ===============

Example trades from binance:

.. sourcecode:: http

    [
        {"Id":47317269,"Time":1530244709886,"Amount":0.002902,"Price":5884.38,"Type":1},
        {"Id":47317270,"Time":1530244709886,"Amount":0.082102,"Price":5884.78,"Type":1},
        {"Id":47317271,"Time":1530244713111,"Amount":0.122439,"Price":5884,"Type":0},
        .....
        {"Id":47317278,"Time":1530244717131,"Amount":0.000029,"Price":5884,"Type":0},
    ]

A useful JavaScript example by using trades:

.. code-block:: JavaScript

    function main(){
        while(true){
            var trades = exchange.GetTrades();
            for(var i=0;i<trades.length;i++){
                if(trades[i].Type == ORDER_TYPE_BUY && trades[i].Amount > 100){
                    Log("Big amount buy order","time:", trades[0].Time, "Price:", trades[0].Price, "Amount:", trades[0].Amount);
                }
            }
            Sleep(3000)//sleep 3 seconds
        }
    }

.. warning::

    The trades in simulation backtesting is empty.