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


2.6.2 LogStatus
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogStatus(Msg)

This kind of Log information is displayed above the log and update every time when it is called.

Parameter value: Msg can be any type.

.. code-block:: JavaScript

    LogStatus(" This is a normal status prompt"); 
    LogStatus(" This is a red font status prompt #ff0000"); 
    LogStatus(" This is a multi-line status message\n I'm the second line"); 

Like ``Log()`` function, ``LogStatus`` supports base64-encoded images and Python matplotlib.pyplot object.

LogStatus can Log tables on your robot page.

Log a table example, add ````` characters to both sides and treat it as a complex message format (currently supported table)

.. code-block:: JavaScript

    var table = {type: 'table', title: ' Account information support color #ff0000', cols: ['BTC', 'ETH', 'USDT'], rows: [ ['free', 1, 2000], ['frozen', 0, 3000]]};     
    LogStatus('`' + JSON.stringify(table)+'`'); 

Another example, information can also appear in multiple lines:

.. code-block:: JavaScript

    LogStatus("First line message\n" + JSON.stringify(table)+"`\n third line message");`

Log multiple tables in a group, switching by TAB:

.. code-block:: JavaScript

    var table1 = {type: 'table', title: ' Account information 1', cols: ['BTC', 'ETH', 'USDT'], rows: [ ['free', 1, 2000], ['frozen', 0, 3000]]};
    var table2 = {type: 'table', title: ' Account information 2', cols: ['BTC', 'ETH', 'USDT'], rows: [ ['free', 1, 2000], ['frozen', 0, 3000]]}; 
    LogStatus('`' + JSON.stringify([table1, table2])+'`'); // Supports multiple tables to be displayed at the same time and will be displayed in a group with TAB 

Log multiple tables in one page:

.. code-block:: JavaScript

    function main(){
        var tab1 = {type : "table",title : "Table 1",cols : ["1", "2"],rows : []};
        var tab2 = {type : "table",title : "Table 2",cols : ["1", "2", "3"],rows : []};
        tab1.rows.push(["jack", "lucy"]);
        tab2.rows.push(["apple", "pen", "apple pen"]);
        LogStatus('`' + JSON.stringify(tab1) + '`\n' + '`' + JSON.stringify(tab2) + '`');
    }


Log table with a button in the table. The strategy uses ``GetCommand`` to receive the contents of the cmd property.

.. code-block:: JavaScript

    var table = { 
        type: 'table', 
        title: 'Positioning operations', 
        cols: ['Column 1', 'Column 2', 'Action'], 
        rows: [ 
            ['abc', 'def', {'type':'button', 'cmd': 'coverAll', 'name': 'Close the position'}], 
        ]
    }; 
    LogStatus('`' + JSON.stringify(table) + '`') 
    // Or construct a separate button
    LogStatus('`' + JSON.stringify({'type':'button', 'cmd': 'coverAll', 'name': ' Close the position'}) + '`') 
    // Can customize button styles (bootstrap button properties)
    LogStatus('`' + JSON.stringify({'type':'button', 'class': 'btn btn-xs btn-danger', 'cmd': 'coverAll', 'name': 'close the position'}) + '`')


2.6.3 LogProfit
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

    ``LogProfit`` doesn't have to be recording the profit , it can be any number you like to present, such as total account value, free USDT amount.
    ``Profit`` is calculated by your own.

2.6.4 LogReset
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogReset()

Clear the log, you can pass a parameter, specify the number of recent logs to keep, clear the rest of the log. 


2.6.5 LogProfitReset
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogProfitReset()

Clear all history logs, can take a number parameter, specify the number of reservations.


2.6.6 EnableLog
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    EnableLog(IsEnable)

Turn on or off logging of orders and error messages/

Parameter value: isEnable is bool type

2.6.7 LogVacuum
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogVacuum()

Reclaims the space occupied by SQLite when deleting data.

2.6.8 Sleep
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Sleep(Millisecond)

 Pause the robot program for a period of time.

 Parameter value: Millisecond is number type
 
 ``Sleep(1000)`` means sleep 1 second.

 .. warning::

    In almost all the situation, ``Sleep`` is necessary in ``while`` loops, otherwise you may exceed the exchange's API rate limits of REQUESTS. 


2.6.9 IsVirtual
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    IsVirtual()

Your robot is runing in a simulated backtest or not.

Return value: bool type, Simulate back test state return true, the real market returns false


