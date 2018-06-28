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

exchange.GetTicker();
Acquiring the current market quotations
Return value: Ticker structure
The Ticker structure contains the following variables: