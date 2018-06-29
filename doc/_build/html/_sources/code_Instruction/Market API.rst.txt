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

Acquiring the current market quotations
Return value: Ticker structure
The Ticker structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
info                Object               the original structure returned by the exchange
high                Number               Highest price
low			        Number               lowest price
sell                Number               the latest selling price
buy                 Number               the latest buying price
last                Number	             last traded price
volume              Number               most recent trading volume
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

