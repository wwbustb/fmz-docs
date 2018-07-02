2.4 Trade API
=======================

The following functions is used for trading, 
which are called from  ``exchange`` or ``exchanges[x]`` object, for example: ``exchange.Sell(100, 1)``; 
Send a buy order to the exchange with the price is 100, and the quantity is 1.

If you want change the trading pair before trade, check on ``exchange.IO("currency", symbol)``

2.4.1 GetAccount
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetAccount()

Acquiring exchange account information

**Return value: Account structure**

The Account structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Info                Object               The original data returned by the exchange 
Balance             Number               Balance (Pricing currency balance, BTC if your trading pair is ``ETH_BTC``)
FrozenBalance	    Number               Frozen balance in your pending buy orders
Stocks              Number               The available quantity of trading currency, ETH if your trading pair is ``ETH_BTC``
FrozenStocks        Number               Frozen Stocks in your pending sell orders
==================  ==================== ===============

Example of GetAccount from binance, trading pair is ``BTC_USDT``:

.. sourcecode:: http

    {   
        "Stocks":0.38594816,
        "FrozenStocks":0,
        "Balance":542.858308,
        "FrozenBalance":0
        "Info":{
                "takerCommission":10,
                "canTrade":true,
                "canDeposit":true,
                "updateTime":1530330645991,
                "balances":[
                            {"asset":"BTC","free":"0.38594816","locked":"0.00000000"},
                            {"asset":"LTC","free":"0.00736000","locked":"0.00000000"},
                            {"asset":"ETH","free":"2.44434439","locked":"0.00000000"},
                            {"asset":"BNC","free":"0.00000000","locked":"0.00000000"},
                            {"asset":"ICO","free":"0.00000000","locked":"0.00000000"},
                            ......
                            ]
                "makerCommission":10,
                "buyerCommission":0,
                "sellerCommission":0,
                "canWithdraw":true
            },
    }

A useful JavaScript example of Log your account value for a certain trading pair:

.. code-block:: JavaScript

    function main(){
        while(true){
            var ticker = exchange.GetTicker();
            var account = exchange.GetAccount();
            var price = ticker.Buy;
            var stocks = account.Stocks + account.FrozenStocks;
            var balance = account.Balance + account.FrozenBalance;
            var value = stocks*price + balance;
            Log('Account value is: ', value);
            Sleep(3000);
        }    
    }

2.4.2 Buy
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

2.4.3 Sell
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

2.4.4 CancelOrder
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

2.4.5 GetOrder
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

.. note::

    ``Buy``, ``Sell``, ``CancelOrder``, can be followed by some additional output parameters, such as: ``exchange.Buy(price, amount, 'BTC_USDT')``,
    ``exchange.CancelOrder(orderId, 'BTC_USDT')``, Which will give you extra information in robot Logs.


2.4.6 GetOrders
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetOrders()

Get all Current open orders for your trading pair.

**Return value: Order structure array**

Example of GetOrders from binance, the trading pair is ``YOYOETH``:

.. sourcecode:: http

    [
        {
            "Info":{
                    "executedQty":"0.00000000",
                    "type":"LIMIT",
                    "isWorking":true,
                    "price":"0.00012826",
                    "status":"NEW",
                    "timeInForce":"GTC",
                    "symbol":"YOYOETH",
                    "side":"SELL",
                    "stopPrice":"0.00000000",
                    "icebergQty":"0.00000000",
                    "time":1530331666316,
                    "orderId":16387538,
                    "origQty":"1123.00000000",
                    "clientOrderId":"TrKOsaHcqc667tjZQtg09b"
                    },
            "Id":16387538,
            "Amount":1123,
            "Price":0.00012826,
            "DealAmount":0,
            "AvgPrice":0,
            "Status":0,
            "Type":1,
            "ContractType":""
        }
    ]

A  JavaScript example of using this API, which will cancel all open orders for set trading pair:

.. code-block:: JavaScript

    fuction CancelAll(){
        var orders = exchange.GetOrders();
        for(var i=0;i<orders.length,i++){
            exchange.CancelOrder(orders[[i].Id);
        }
    }
    function main(){
        CancelAll();
        while(true){
            //do something
            Sleep(10000);
        }
    }

2.4.7 GetPosition
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetPosition()

Get the current position information, only for Futures trade. OKEX can pass a parameter, specify the type of contract to get.

Return value: position array

BTC Futures support: OKEX, BitMEX.

The position structure contains the following variables:

==================  ==================== ===============
Field               Type                 Description
==================  ==================== ===============
Info                Object               The original data returned by the exchange 
MarginLevel         Number               Leverage size, OKEX is 10 or 20, full amount of okex futures margin mode returns a fixed 10, because the original API does not support
Amount			    Number               ositions, OKEX indicates the number of contracts (integer and greater than 1)
FrozenAmount        Number               Position freeze
Price               Number	             Average price of positions
Profit              Number               Order Status
Type                Const                ``PD_LONG`` is a long position, ``PD_SHORT`` is a short position.
ContractType        String               Contract name
==================  ==================== ===============

A  JavaScript example:

.. code-block:: JavaScript

    // Note: GetPosition function obtains position information of all positions.
    function main(){
        exchange.SetContractType("this_week");
        exchange.SetMarginLevel(10);
        exchange.SetDirection("buy");
        exchange.Buy(10000, 2);
        position = exchange.GetPosition();
        Log("Amount:", position[0].Amount, "FrozenAmount:", position[0].FrozenAmount, "Price:",
            position[0].Price, "Profit:", position[0].Profit, "Type:", position[0].Type,
            "ContractType:", position[0].ContractType);
    }

2.4.8 SetMarginLevel
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetMarginLevel(MarginLevel)

Set the leverage size, only for Futures trade.

Parameter value: number integer

Set the leverage size of Buy or Sell. MarginLevel has 5, 10, 20 optional parameters. OKEX supports 10 times and 20 times. For example:
``exchange.SetMarginLevel(10)``

2.4.9 SetDirection
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetDirection(Direction)

Set Buy or Sell Order Types, only for Futures trade.

Parameter value: string type, can be ``buy``, ``closebuy``, ``sell``, ``closesell``.

A  JavaScript example:

.. code-block:: JavaScript

    function main(){
        exchange.SetMarginLevel(5); // Set the leverage to 5 times
        exchange.SetDirection("buy"); // Set the order type to buy long 
        exchange.Buy(1000, 2); //buy long at the price 1000, quantity of 2
        exchange.SetMarginLevel(5); 
        exchange.SetDirection("closebuy"); 
        exchange.Sell(1000, 2);
    }

2.4.10 SetContractType
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetContractType(ContractType)

Set contract type

Parameter value: string type

OKEX futures have “this_week”, “next_week”, “quarter” three parameters

.. code-block:: JavaScript

    exchange.SetContractType("this_week"); // Set to Weekly Contract
