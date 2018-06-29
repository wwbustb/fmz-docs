2.4 Trade API
=======================

The following functions can all be called via an ``exchange`` or ``exchanges[x]`` object, for example: ``exchange.Sell(100, 1)``; 
Send a buy order to the exchange with the price is 100, and the quantity is 1.

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



    