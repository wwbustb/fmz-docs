2.4 Trade API
=======================

The following functions can all be called via an ``exchange`` or ``exchanges[x]`` object, for example: ``exchange.Sell(100, 1)``; 
Send a buy order to the exchange with the price is 100, and the quantity is 1.

If you want change the trading pair before trade, check on ``exchange.IO("currency", symbol)``

2.4.1 Buy
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.Buy(Price, Amount)

Send a buy order, return an order ID

**Parameter**

.. sourcecode:: http

    Price  : order price, number type
    Amount : order quantity, number type

**Return**

.. sourcecode:: http

    order id, number type

.. tip::

    If the exchange's order API support market orders, use ``exchange.Buy(-1, 0.1)``, if your trading pair is ``ETH_BTC``,
    which means buy 0.1 ETH at market price. Be carefully when using market order.

A useful JavaScript example of Buy for buy certain amount of bitcoin at a certain price:

.. code-block:: JavaScript

    function main(){
        while(true){
            var ticker = exchange.GetTicker();
            var price = ticker.Buy;
            if(price >= 7000){
                exchange.Buy(price, 10);
            }
            Sleep(3000);
        }    
    }

2.4.2 Sell
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.Sell(Price, Amount)

Send a sell order, return an order ID

**Parameter**

.. sourcecode:: http

    Price  : order price, number type
    Amount : order quantity, number type

**Return**

.. sourcecode:: http

    order id, number type

.. tip::

    If the exchange's order API support market orders, use ``exchange.Sell(-1, 0.1)``, if your trading pair is ``ETH_BTC``,
    which means sell 0.1 ETH at market price.

2.4.3 CancelOrder
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.CancelOrder(orderId)

Cancel an order by order id.

**Parameter**

.. sourcecode:: http

    orderId  : order id, returned by Buy or Sell API.

Return value: bool type

``true`` means that the cancellation of the order request was successful. 
``false`` means cancellation of the order request failed. (It is only a successful request. 
Whether the exchange cancels the order, it is best to call ``exchange.GetOrders()``.)

A  JavaScript example of cancel an order after some time:

.. code-block:: JavaScript

    function main(){
        var id = exchange.Sell(99999, 1);
        Sleep(3000);
        exchange.CancelOrder(id);
    }

2.4.4 GetOrder
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetOrder(orderId)

Get order details by order id.

**Parameter**

.. sourcecode:: http

    orderId  : order id, returned by Buy or Sell API.

**Return value: Order structure**

The Order structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Info                Object               The original data returned by the exchange 
Id                  Number               Unique ticket identifier
Price			    Number               Order price
Amount              Number               Order quantity
DealAmount          Number               The deal amount of this order
AvgPrice            Number	             Average transaction price (0 means the exchange do not return this field)
Status              Const                Order Status
Type                Const                Order Type, ``ORDER_TYPE_BUY`` : Buy Order, ``ORDER_TYPE_SELL`` : Sell Order
==================  ==================== ===============

Order Status is global constant:

=====================  ====================  ==================== 
Global constant        Meaning               value
=====================  ====================  ====================
ORDER_STATE_PENDING    Incomplete            0
ORDER_STATE_CLOSED     Completed             1
ORDER_STATE_CANCELED   Canceled              2   
=====================  ====================  ====================

Example of GetOrder from binance:

.. sourcecode:: http

    {
        "Id":125723661,
        "Amount":0.01,
        "Price":7000,
        "DealAmount":0,
        "AvgPrice":0,
        "Status":0,
        "Type":1,
        "ContractType":"",
        "Info":{
            "side":"SELL",
            "stopPrice":"0.00000000",
            "timeInForce":"GTC",
            "type":"LIMIT",
            "time":1530325939498,
            "orderId":125723661,
            "clientOrderId":"H3R333f47MsFrahQUsa8egU",
            "origQty":"0.01000000",
            "status":"NEW",
            "executedQty":"0.00000000",
            "isWorking":true,
            "symbol":"BTCUSDT",
            "price":"7000.00000000",
            "icebergQty":"0.00000000"
        }
    }

A  JavaScript example of using this API, which will buy until your account has 10 coins:

.. code-block:: JavaScript

    function main(){
        while(true){
            var amount = exchange.GetAccount().Stocks;
            var ticker = exchange.GetTicker();
            if(10-amount>0.01){
                var id = exchange.Buy(ticker.Sell, Math.min(10-amount,1));
            }else{
                return;
            }
            var status = exchange.GetOrder(id).Status;
            if(Status == ORDER_STATE_PENDING){
                exchange.CancelOrder(id);
            }
            Sleep(3000);
        }
    }