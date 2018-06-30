2.5 Extended API
=======================

The following functions is used for extending your API.
Most of the time, your code can be wrote by the API above. 

2.5.1 IO
>>>>>>>>>>>>>>>>>>

exchange.IO() has different usages depends on  pramaeters.

**Change trading pair**

.. code-block:: JavaScript

    exchange.IO("currency", symbol)

Parameter value: ``symbol`` , trading pair to switch.

Example:``exchange.IO("currency", "LTC_USDT")``, which will switch the default trading pairs configured by the robot when it was created to ``LTC_USDT``.

A JavaScript example of using IO to trade several trading pairs:

.. code-block:: JavaScript
    
    var symbols = ["BTC_USDT", "LTC_USDT", "EOS_USDT", "ETH_USDT", "BCC_USDT"];
    var buyValue = 1000;
    function main(){
        for(var i=0;i<symbols.length;i++){
            exchange.IO("currency", symbols[i]);
            var ticker = exchange.GetTicker();
            var amount = _N(buyValue/ticker.Sell, 3);
            exchange.Buy(ticker.Sell, amount);
            Sleep(1000);
        }
    }

**Change exchange base API address**

.. code-block:: JavaScript

    exchanges[0].IO("base", baseAddress)

Parameter value: ``baseAddress`` , exchange base API address to switch.

If an exchange change it's base API address,  you can use this function to change the base address temporarily till FMZ change it.

In some case, changing the address can acess another part of a exchange, such as huobipro and hadax,
their API is the same, just switching the base address you can trade on hadax.
``exchanges[0].IO("base", "https://api.hadax.com/v1")``

**Use exchange's orgin API**

.. code-block:: JavaScript

    exchange.IO("api", httpMethod, resource, params)




2.5.1 GetRawJSON
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetRawJSON()

Returning of the original content (string) that returned by the last REST API request, which can be used to resolve extension information on its own.

There are a lot of inforamtions in the raw data returned from exchange, part of them are in ``Info`` Field, if not, you can use this API.

**Return value : string type**

A JavaScript example of using GetRawJSON and parse the raw data:

.. code-block:: JavaScript

    function main(){
        exchange.GetAccount(); 
        var obj = JSON.parse(exchange.GetRawJSON());
        Log(obj);
    }

2.5.2 GetName
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetName()

Returns the name of the exchange.

Return value: string type.

2.5.3 GetLabel
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetLabel()

Return the exchange's custom label.

Return value: string type

2.5.4 GetCurrency
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetCurrency()

Returns the name of the currency pair operated by the exchange.

Return value: string type

2.5.4 GetQuoteCurrency
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetQuoteCurrency()

Returns the base currency name of the exchange operation, eg BTC_CNY returns CNY, ETH_BTC returns BTC.

Return value: string type

2.5.4 SetPrecision
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetPrecision(PricePrecision, AmountPrecision)

Set the decimal precision of price and type order quantity, and will automatically truncate after setting.

If you use ``exchange.Sell(7000.1225,1.223123)`` on Bianace directly, which will return error ``{"code":-1013,"msg":"Filter failure: PRICE_FILTER"}``.
that's why you should care about precision.

You can also use ``_N()`` function as  ``exchange.Sell(_N(price,2), _N(amount,5))``

You can find the demands of precision and others in exchange's docs, for example: https://api.binance.com/api/v1/exchangeInfo

**Parameter value:**

``PricePrecision`` , number type, used to control the decimal point of the price.

``AmountPrecision`` , number type, used to control the decimal point of the amount.

.. code-block:: JavaScript

    exchange.SetPrecision(2, 3);
    function main(){
        exchange.Sell(7000.1225,1.223123)   //which will be the same as exchange.Sell(7000.12,1.223)
    }

.. note::

    SetPrecision doesn't work in backtesting


2.5.5 GetRate
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetRate()

Returning of the exchange rate that between the exchange currency and the current display currency. Returning 1 means currency conversion is not allowed.

Return value: number type

.. note::

    If you do not call exchange.SetRate() to set the conversion rate, GetRate defaults to the exchange rate value of 1, ie, the current displayed denomination currency has not been converted. 


2.5.6 SetRate
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetRate(scale)

Parameter value: ``scale`` , number type

Return value: number type

If you use exchange.SetRate() to set an exchange rate value, such as 0.85(the rate of EUR and USD), 
then all exchange prices, depth, order price and all other price information in the current exchange currency represented by the exchange object will be multiplied by the setting.

2.5.5 Log
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.Log(logType, orderId, price, amount)

Doesn't actually sent the order, just record order information for testing your strategy.

**Parameter values: **

.. sourcecode:: http

    logType : LOG_TYPE_BUY, LOG_TYPE_SELL, LOG_TYPE_CANCEL
    orderId : order id, customizable an incremental value
    price   : price
    amount  : quantity


Return value: number type

.. note::

    This function is a member function of the exchange exchange object, which is different from the global function Log(). 






