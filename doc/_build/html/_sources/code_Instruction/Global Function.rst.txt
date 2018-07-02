2.6 Global Founction
=======================

2.6.1 Log
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Log(message)

Save a message to the robot logs.

Parameter value: message can be any type.

.. code-block:: JavaScript

    Log(123);
    Log('hello', 'world', 123);
    Log("red color message", "#FF0000");
    Log("blue color message#FF0000");


Log supports pushing message to your WeChat account. 

adding the `@` character to the string, 
the message will go to the push queue and be pushed to the WeChat account that uses the binding (bind in account security) (50/hour, 1/5 second limit)

.. code-block:: JavaScript

    Log("hello Wechat!@");

Log supports printing base64-encoded pictures:

.. code-block:: JavaScript

    Log("`data:image/png;base64,AAAA`");

Log supports direct printing of Python matplotlib.pyplot object.
as long as the object contains ``savefig`` method can be directly Log print, such as:

.. code-block:: Python

    import matplotlib.pyplot as plt

    def main(): 
        plt.plot([3,6,2,4,7,1]) 
        Log(plt)

Log supports present message's character or background with custom color:

Changing message's character color, message need end with RGB color code such as ``##FF0000``, for more RGB color, check on https://www.w3schools.com/colors/colors_picker.asp.

Changing message's background color, message need end with like ``#FF0000CC3299`` ,the last six number ``CC3299`` represnt the RGB color code.

.. code-block:: JavaScript

    Log("red color message", "#FF0000");
    Log("blue color message#FF0000CC3299"); 

2.6.2 LogProfit
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogProfit(Profit)

Record profit value, draw a line chart in your robot page, will remain after you restart your robot.

Parameter value: profit , number type

A useful JavaScript example of Log Profit for a certain trading pair:

.. code-block:: JavaScript

    function GetValue(){
            var ticker = exchange.GetTicker();
            var account = exchange.GetAccount();
            var price = ticker.Buy;
            var stocks = account.Stocks + account.FrozenStocks;
            var balance = account.Balance + account.FrozenBalance;
            var value = stocks*price + balance;
            return value;
    }
    function main(){
        var initValue = GetValue();
        var profit = 0;
        while(true){
            profit = GetValue() - initValue;
            LogProfit(profit);
            Sleep(60000);//sleep one minute
        }    
    }

.. note::

    ``LogProfit`` doesn't has to be record your profit , it can be any number you like to present, such as total account value, free USDT value.
     ``Profit`` is calculated by your own.


