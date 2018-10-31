2.5 Extended API
=======================

The following functions is used for extending your API.
Most of the time, your code can be wrote by the API above. 

2.5.1 IO
>>>>>>>>>>>>>>>>>>

exchange.IO() has different usages depends on parameters.

**Change trading pair**

.. code-block:: JavaScript

    exchange.IO("currency", symbol)

Parameter value: ``symbol`` , trading pair to switch.

Example:``exchange.IO("currency", "LTC_USDT")``, which will switch the default trading pairs configured by the robot when it was created to ``LTC_USDT``.

A JavaScript example of using ``IO`` to trade several trading pairs:

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

    exchangs.IO("base", baseAddress)

Parameter value: ``baseAddress`` , exchange base API address to switch.

If an exchange change it's base API address,  you can use this function to change the base address temporarily till FMZ change it.

In some case, changing the address can acess another part of a exchange, such as huobipro and hadax,
their API is the same, just switching the base address you can trade on hadax.
``exchange.IO("base", "https://api.hadax.com")``

**Use exchange's orgin API**

.. code-block:: JavaScript

    exchange.IO("api", httpMethod, resource, params)

Acesss to exchange's other API.

.. note::

    Only used for those API methods needed secret key to sign, other methods can be acessed by ``HttpQuery(url)`` function.


Using this function requires the understanding of exchange’s orgin API, it extend the functionality that the FMZ does not add (to submit a POST request without having to worry about the parameter encryption process, 
the FMZ has completed the encryption already, just need fill in the corresponding parameters). 
For example, the FMZ platform does not currently support margin leverage trading on bitfinex exchanges. We can implement this function by using the IO function. 

- First find bitfinex API description webpage: bitfinex.
- Then we know that the order is interacting with a POST request, so we pass the parameter httpMethod to the order address of the "POST" margin transaction: ' https://api.bitfinex.com/v1/order/new '. Because the FMZ has internally specified the root address, we only need to pass the value of the parameter resource to "/v1/order/new".
- Then the params parameter is not filled in. The params variable represents the information to be exchanged. We can send all kinds of information with the "&" symbol to send them. We first go to bitfinex to see that the next buy or sell order requires 5 parameters., they are: symbol, amount, price, side, type. We assign these five parameters respectively. If we want to buy Litecoin LTC, the quantity is 1, the price is 10, and the margin trading mode, then we can construct such a string: "symbol=ltc&amount=1&price=10&side=buy&type= Limit".

The final JavaScript code:

.. code-block:: JavaScript

    function main(){
        exchange.IO("api","POST","/v1/order/new","symbol=ltc&amount=1&price=10&side=buy&type=limit");
    }

An OKEX Example：

.. code-block:: JavaScript

    function main(){
        var ret = exchange.IO("api", "POST", "/api/v1/future_position.do", "symbol=eth_usd&contract_type=this_week");
	    Log(ret);
    }

2.5.2 Go
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.Go(Method, Args)

Multi-threaded asynchronous support functions that can convert the operations of all supported functions into asynchronous concurrency. 

**Parameter value:**

.. sourcecode:: http

    Method : a function name.
    Args   : the args of method.

Supported Functions: ``GetTicker``, ``GetDepth``, ``GetTrades``, ``GetRecords``, ``GetAccount``, ``GetOrders``, ``GetOrder``, ``CancelOrder``, ``Buy``, ``Sell``, ``GetPosition``

robot thread must obtain the result from the wait function, the docker automatically releases the thread resource requested through the Go function.
If the return result of the  wait function is not obtained, the thread resource will not be automatically released, which will cause threads to accumulate, and more than 2000 will report an error.
"too many routine wait, max is 2000" 

A JavaScript example

.. code-block:: JavaScript

    function main(){
        var a = exchange.Go("GetTicker"); //GetTicker Asynchronous multithreaded execution 
        var b = exchange.Go("GetDepth"); 
        var c = exchange.Go("Buy", 1000, 0.1); 
        var d = exchange.Go("GetRecords", PERIOD_H1);
        // The above four operations are concurrent multi-threaded asynchronous execution, will not be time-consuming and immediately return
        var ticker = a.wait(); // Call wait method wait for return to asynchronous get ticker result
        var depth = b.wait(); // Return depth, it is also possible to return null if it fails 
        var orderId = c.wait(1000); // Return the order number, limit 1 second timeout, timeout returns undefined, this object can continue to call wait until the last wait timeout
        var records = d.wait(); // Wait for K-line result
        var ret = d.wait();  // Here waits for an asynchronous operation that has waited and ended, returns null, and logs an error message.
    }

The difference between Python and JavaScript, Python's wait returns two parameters, the first is the result of the asynchronous API, and the second is whether the asynchronous call is completed.

.. code-block:: Python

    ret, ok = d.wait(); // Ok is bound to return true unless the strategy is stopped
    ret, ok = d.wait(100); // Ok returns False, waits for a timeout, or waits for an instance that has ended

.. note::

    This function only creates multi-threaded execution tasks when it runs on a real market. Backtesting does not support multithreaded concurrent execution of tasks (backtesting is available, but it is also performed sequentially).

2.5.2 GetRawJSON
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

2.5.3 GetName
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetName()

Returns the name of the exchange.

Return value: string type.

2.5.4 GetLabel
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

2.5.5 GetQuoteCurrency
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetQuoteCurrency()

Returns the base currency name of the exchange operation, eg BTC_CNY returns CNY, ETH_BTC returns BTC.

Return value: string type

2.5.6 SetPrecision
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


2.5.7 GetRate
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.GetRate()

Returning of the exchange rate that between the exchange currency and the current display currency. Returning 1 means currency conversion is not allowed.

Return value: number type

.. note::

    If you do not call exchange.SetRate() to set the conversion rate, GetRate defaults to the exchange rate value of 1, ie, the current displayed denomination currency has not been converted. 


2.5.8 SetRate
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.SetRate(scale)

Parameter value: ``scale`` , number type

Return value: number type

If you use exchange.SetRate() to set an exchange rate value, such as 0.85(the rate of EUR and USD), 
then all exchange prices, depth, order price and all other price information in the current exchange currency represented by the exchange object will be multiplied by the setting.

2.5.8 SetProxy
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript
     
     exchange.SetProxy()

Switch to Proxy Server to Access Exchange 

Each exchange object (exchanges[n]) can set up an agent. After setting up the agent, the access exchange API will be accessed through the agent.

.. code-block:: JavaScript

    // Take the exchange of the main exchange object as the first added exchange object, ie: exchanges[0] as an example.
    exchange.SetProxy("socks5://127.0.0.1:8889")                      // Set proxy, no username, no password.
    exchange.SetProxy("socks5://username:password@127.0.0.1:8889")    // Set up the proxy, enter the username and password 
    exchange.SetProxy("")                                            // Switch to normal mode without using a proxy.

2.5.9 SetTimeout
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript
     
     exchange.SetTimeout(time)

Set timeout for exchange's rest request.

Only the REST request is used to set the timeout time.

For example: ``exchange.SetTimeout(3000)``, set the exchange exchange object, send a rest request starts timing, exceeds 3 seconds, timeout returns null.


.. code-block:: JavaScript

    exchange.SetTimeout()

2.5.10 Log
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    exchange.Log(logType, orderId, price, amount)

Doesn't actually sent the order, just record order information for testing your strategy.

**Parameter values:**

.. sourcecode:: http

    logType : LOG_TYPE_BUY, LOG_TYPE_SELL, LOG_TYPE_CANCEL
    orderId : order id, customizable an incremental value
    price   : price
    amount  : quantity


Return value: number type

.. note::

    This function is a function of the exchange exchange object, which is different from the global function Log(). 






