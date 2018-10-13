2.6 Global Function
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

Log supports base64-encoded pictures print:

.. code-block:: JavaScript

    Log("`data:image/png;base64,AAAA`");

Log supports Python matplotlib.pyplot object print directly.
as long as the object contains ``savefig`` method, such as:

.. code-block:: Python

    import matplotlib.pyplot as plt

    def main(): 
        plt.plot([3,6,2,4,7,1]) 
        Log(plt)

Log supports present message's character or background with custom color:

Change message's character color: message need end with RGB color code such as ``##FF0000``, for more RGB color, check on https://www.w3schools.com/colors/colors_picker.asp.

Change message's background color: message need end with like ``#FF0000CC3299`` ,the last six number ``CC3299`` represnt the RGB color code.

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

Log a table example, add ````` characters to both sides and treat it as a complex message format (currently supported table).

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

2.6.4 SetErrorFilter
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    SetErrorFilter(RegEx)

Error message filtering

Parameter value: string type

Errors that are matched by this regular expression will not be uploaded to the log system. Multiple set (filtered logs, database files corresponding to robot IDs in the logs/robot under the docker directory can be called multiple times to prevent frequent errors Causes database file expansion.)

.. code-block:: JavaScript

    SetErrorFilter("502:|503:|tcp|character|unexpected|network|timeout|WSARecv|Connect|GetAddr|no such|reset|http|received|EOF|reused");



2.6.5 LogReset
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogReset()

Clear the log, you can pass a parameter, specify the number of recent logs to keep, clear the rest of the log. 


2.6.6 LogProfitReset
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogProfitReset()

Clear all history logs, can take a number parameter, specify the number of reservations.


2.6.7 EnableLog
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    EnableLog(IsEnable)

Turn on or off logging of orders and error messages/

Parameter value: isEnable is bool type

2.6.8 LogVacuum
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    LogVacuum()

Reclaims the space occupied by SQLite when deleting data.


2.6.9 GetLastError
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    GetLastError()

Get the latest error message, generally do not need to use, because the program will automatically upload the error message to the log system.

Return value: string type


2.6.10 GetCommand
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    GetCommand()

Get Interactive Command (utf-8).

Get the command sent from the strategy interactive interface and clear it. If there is no command, 
it will return ``null``. The returned command format is "Button name: parameter". If there is no parameter, the command is the button name.

A JavaScript example

.. code-block:: JavaScript

    function main(){
        while(true) { 
            var cmd = GetCommand(); 
            if (cmd) { 
                Log(cmd);
            } 
            Sleep(1000); 
        }
    }

2.6.11 Sleep
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Sleep(Millisecond)

Pause the robot program for a period of time.

Parameter value: Millisecond is number type
 
``Sleep(1000)`` means sleep 1 second.

.. warning::

    In almost all the situation, ``Sleep`` is necessary in ``while`` loops, otherwise you may exceed the exchange's API rate limits of REQUESTS. 


2.6.12 IsVirtual
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    IsVirtual()

Your robot is runing in a simulated backtest or not.

Return value: bool type, Simulate back test state return true, the real market returns false

2.6.13 GetOS
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    GetOS()

Returns the information of the docker's system.

2.6.14 GetPid
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    GetPid()

Return robot process ID

Return value: string type


2.6.15 Mail
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Mail(smtpServer, smtpUsername, smtpPassword, mailTo, title, body)

Send a e-mail.

Parameter values: all are string types

Return value: bool type, return true if successful

.. code-block:: JavaScript

    function main(){
        Mail("smtp.163.com", "test@163.com", "password", "usr@163.com", "title", "body");
    }


2.6.16 Dial
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Dial(Address, Timeout)

Get Original Socket access, support tcp, udp, tls, unix protocol.

Parameter value: Address is string type, fill in the address, TimeOut is the timeout

A JavaScript example:

.. code-block:: JavaScript

    function main(){
        var client = Dial("tls://www.baidu.com:443"); // Dial supports tcp://, udp://, tls://, unix:// protocol, plus one parameter to specify the number of seconds to timeout
        if (client) {
            client.write("GET / HTTP/1.1\nConnection: Closed\n\n"); // Write can be followed by a number parameter to specify the timeout, write to return the number of bytes successfully sent
            while (true) {
                var buf = client.read();// Read can be followed by a number parameter to specify a timeout, return null to indicate an error or timeout, or the socket is already closed
                if (!buf) {
                    break;
                }
                Log(buf);
            }
        client.close();
        }
    }

Support websocket.

A JavaScript example of connecting to binance websocket ticker.

.. code-block:: JavaScript

    function main() {
        LogStatus("connecting...");
        var client = Dial("wss://stream.binance.com:9443/ws/!ticker@arr");
        if (!client) {
            Log("Connection failed, program exited");
            return
        }
        Log("The connection is successful and will automatically reconnected")
        while (true) {
            var buf = client.read() // Read only returns data obtained after calling read
            if (!buf) {
                break;
            }
            var table = {
                type: 'table',
                title: 'Quotes chart',
                cols: ['Currency', 'highest', 'lowest', 'buy one', 'sell one', ' Last traded price', 'volume', 'Update time'],
                rows: [],
            };
            var obj = JSON.parse(buf);
            _.each(obj, function(ticker) {
                table.rows.push([ticker.s, ticker.h, ticker.l, ticker.b, ticker.a, ticker.c, ticker.q, _D(ticker.E)])
            });
            LogStatus('`' + JSON.stringify(table) + '`')
        }
        client.close();
    }



2.6.16 HttpQuery
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    HttpQuery(Url, PostData, Cookies, Headers, IsReturnHeader)

Web URL access, support ``PUT``, ``GET``, ``POST``, ``DELETE`` ,etc.

Parameter values: all are string types

Get the content of a Url. If the second parameter PostData is a string, submit it as POST.
The second parameter PostData can be a custom method such as:

.. code-block:: JavaScript

    HttpQuery("http://www.abc.com", {method:'PUT', data:'parameter1=value1&parameter2=value2'});//PUT method
    HttpQuery("http://www.abc.com", {method:'PUT', data:'parameter1=value1&parameter2=value2', timeout:1000});

Passing the cookie string requires a third parameter, but does not require POST. Please set the second parameter to ``null``
When runing in the backtes, the function returns the fixed string Dummy Data because the URL cannot be simulated.

You can use this interface to send text messages or interact with other APIs

.. code-block:: JavaScript

    HttpQuery("http://www.google.com"); // Get 
    HttpQuery("http://www.google.com", "parameter1=value1&parameter2=value2"); // Post 
    HttpQuery("http://www.google.com", null, "a=10; b=20", "User-Agent: Mobile\nContent-Type: text/html", true);  

Example Accessing BIANACE APIs that do not require signatures:

.. code-block:: JavaScript

    var exchangeInfo = JSON.parse(HttpQuery('https://api.binance.com/api/v1/exchangeInfo'));
    Log(exchangeInfo);
    var ticker = JSON.parse(HttpQuery('https://api.binance.com/api/v1/ticker/24hr'));
    Log(ticker);

.. note::

    The ``HttpQuery`` function only supports JavaScript, for Python, using the ``urlib2`` or ``request`` library to send http requests directly.


2.6.17 MD5
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    MD5(string)

Parameter value: string type

.. code-block:: JavaScript

    Log("MD5", MD5("hello world"))


2.6.18 Hash
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Hash(Algo, OutputAlgo, Data)

Support hash calculation for md5/sha256/sha512/sha1, only supports in the real maket.

Parameter values: all are string types

The second parameter can be set to raw/hex/base64, which means output encrypted original content/hex encoded/base64 encoded.

.. code-block:: JavaScript

    function main(){
        Log(Hash("md5", "hex", "hello")); 
        Log(Hash("sha512", "base64", "hello"));
    }


2.6.19 HMAC
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    HMAC(Algo, OutputAlgo, Data, Key)

HMAC encryption calculation of md5/sha256/sha512/sha1 is supported, only supported in the real market.

Parameter values: all are string types.

The second parameter can be set to raw/hex/base64, which means output encrypted original content/hex encoded/base64 encoded.


2.6.20 UnixNano
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    UnixNano()

Return the nanosecond timestamp. If you need to obtain the millisecond timestamp, you can use the following code:

.. code-block:: JavaScript

    var time = UnixNano() / 1000000;
    Log(_N(time, 0));

2.6.21 Unix
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Unix()

Returns the second-level timestamp, which is only supported in a strategy written in ``C++``.

.. code-block:: C++

    uint64_t t = Unix();
    Log(t);


2.6.21 _C
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    _C(function, args…)

Retry function

Will always call the specified function to return successfully (function returns null or false will retry), such as ``_C (exchange.GetTicker)``, the 
default retry interval is 3 seconds, you can call ``_CDelay`` function to control the retry interval, such as ``_CDelay (1000)`` means change the ``_C`` function retry interval to 1 second, suggesting

Support Function:

- ``exchange.GetTicker()``
- ``exchange.GetDepth()``
- ``exchange.GetTrade()```
- ``exchange.GetRecords()``
- ``exchange.GetAccount()``
- ``exchange.GetOrders()``
- ``exchange.GetOrder()``

A JavaScript example:

.. code-block:: JavaScript

    function main(){
        var ticker = _C(exchange.GetTicker);
        var depth = _C(exchange.GetDepth);
        Log(ticker);
        Log(depth);
    }

.. note::
    
    ``_C()`` can often be misused as ``_C(exchange.GetRecords(PERIOD_H1))``, should be ``_C(exchange.GetRecords,PERIOD_H1)``


2.6.22 _G
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript
    _G(K,V)

Global dictionary that can be saved after restart robot.

The KV dict is permanently stored in the local file. Each robot has a separate database. After the restart or the escrow withdrawal, K must be a number or a string. It is case-insensitive and V can be any JSON serialized content. 

A JavaScript example:

.. code-block:: JavaScript

    _G('initValue', 1000); // set value
    var initValue = _G('initValue'); // get value
    _G("initValue", null); // remove global variable "initValue"
    _G(null); //Remove all global variables
    _G(); // Returns the ID of the current robot


2.6.23 _D
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    _D(timestamp, fmt="yyyy-MM-dd hh:mm:ss")

Returns the specified timestamp

Returns the specified timestamp (ms) string, returns the current time without any parameter, such as _D(), or _D(1478570053241), The default format is yyyy-MM-dd hh:mm:ss.

.. code-block:: JavaScript

    function main(){
        while(true){
            var time = _D();
            LogStatus('Last update time: ', time);
            //do some thing
        }
    }


2.6.24 _N
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    _N(num, precision)

Format a float

Parameter value, num is number type, precision is integer number

For example ``_N (3.1415, 2)`` will delete the value after the two decimal points, return "3.14". ``_N(1321,-2)`` will return "1300".


2.6.25 _Cross
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    _Cross(arr1, arr2)

Returns the number of cross periods of the arrays arr1 and arr2.

A positive number is the upper pass period, a negative number indicates the period of the wear pass, and a 0 indicates the current price.
Can be used in MACD strategy.

Parameter value: array of numbers

Specific instructions for use: built-in function _Cross analysis and instructions

.. code-block:: JavaScript

    var arr1 = [1,2,3,4,5,6,8,8,9]
    var arr2 = [2,3,4,5,6,7,7,7,7]
    function main(){
        Log("_Cross(arr1, arr2) : ", _Cross(arr1, arr2))
        Log("_Cross(arr2, arr1) : ", _Cross(arr2, arr1))
    }

2.6.26 TA Indicator function
>>>>>>>>>>>>>>>>>>

TA-Lib Indicator Library. support ``MACD``, ``EMA``, ``KDJ``, ``ATR``, ``RSI``, etc...

Need to add TA. or talib. prefix when calling indicator function.

For more details about TA-Lib functions, check on  http://mrjbq7.github.io/ta-lib/ 

You can also install TA-lib library of Python by yourself.

JavaScript example:

.. code-block:: JavaScript

    function main(){
        var records = exchange.GetRecords();
        var macd = TA.MACD(records);
        Log("DIF:", macd[0], "DEA:", macd[1], "MACD:", macd[2]);
        var atr = TA.ATR(records, 14);
        // Print out the last row of values
        Log(macd[0][records.length-1], macd[1][records.length-1],
        macd[2][records.length-1]);
        Log(atr[atr.length-1]);
        Log(talib.MACD(records));
        Log(talib.MACD(records, 12, 26, 9));
        Log(talib.OBV(records));
        // Talib can also pass in an array of numbers, which can be passed in successively
        // Such as: OBV(Records[Close], Records[Volume]), need Close, Volume two array parameters
        Log(talib.OBV([1,2,3], [7.1, 6.2, 3,3]));
        // You can also directly pass in an array of records containing the Close, Volume property
        Log(talib.OBV(records));
        Log(TA.Highest(records, 30, 'High'));
        Log(TA.Highest([1,2,3,4], 0));
        // For Python, the system extends the properties of the array returned by GetRecords, adding Open, High, Low, Close, Volume, to facilitate talib calls, such as
        talib.MACD(records.Close); 
        /*For Python, the system expands the properties of the array returned by GetRecords, adds Open, High, Low, Close, Volume, 
        and facilitates talib calls. For example, the Close property returns the Close property of all records members as a numpy array passed to talib. 
        The same as other properties*/
    }

2.6.27 Chart
>>>>>>>>>>>>>>>>>>

.. code-block:: JavaScript

    Chart({…})

Will draw a figure in you robot management page.

Chart is based on HighStocks. check on http://api.highcharts.com/highstock for more details.

The parameter is a HighCharts.StockChart parameter that can be JSON-serialized and has a ``__isStock`` attribute that is not exist in the original one.
If ``__isStock`` is ``false``, chart will displayed as an ordinary chart.

The return object can call ``add([series index(like 0), data])`` to add data to the specified index series, 
call ``reset()`` to clear the chart data, reset can take a number parameter, specify the number of reservations.

You can call ``add([series index, data, index of data in the series])`` to change the data,
Can be negative, -1 refers to the last, -2 is the second to last, such as:
``Chart.add([0, 13.5, -1])``, change the data of the first-to-last point of series[0].data.

Supports the display of multiple charts. You only need to pass in array parameters like: ``var chart = Chart([{...}, {...}, {...}])``. 

A JavaScript example of using Chart to draw the prices of two coins.two exchanges need to be added before run the robot.

.. code-block:: JavaScript

    // This chart is an object in the JS language. Before using the Chart function, we need to declare an object variable chart that configures the chart.
    var chart = {
        // Whether the mark is a general chart, if you are interested, you can change it to false and run it.
        __isStock: true,                                    
        tooltip: {xDateFormat: '%Y-%m-%d %H:%M:%S, %A'},    // Zoom tool
        title : { text : 'Spread Analysis Chart'},          // title
        rangeSelector: {                                    // Selection range
            buttons:  [{type: 'hour',count: 1, text: '1h'}, {type: 'hour',count: 3, text: '3h'}, {type: 'hour', count: 8, text: '8h'}, {type: 'all',text: 'All'}],
            selected: 0,
            inputEnabled: false
        },
        xAxis: { type: 'datetime'},                         // The horizontal axis of the coordinate axis is the x axis and the current setting type is :time
        yAxis : {                                           // The vertical axis of the axis is the y axis, and the default value is adjusted with the data size.
            title: {text: 'Spread'},                        // title
            opposite: false,                                // Whether to enable the right vertical axis
        },
        series : [                                          // Data series, this attribute is saved for each data series (line, K-line graph, label, etc...)
            {name : "line1", id : "Line 1,buy1Price", data : []},  // The index is 0, the data array is stored in the index series of data
            {name : "line2", id : "Line 2,lastPrice", dashStyle : 'shortdash', data : []}, 
            // The index is 1, dashStyle is set: 'shortdash' ie: Set the dotted line.
        ]
    };
    function main(){
        var ObjChart = Chart(chart);                      // Call the Chart function to initialize the chart.
        ObjChart.reset();                                 // Empty the chart
        while(true){
            var nowTime = new Date().getTime();           // Get the timestamp of this poll, which is a millisecond timestamp. Used to determine the position of the X axis written to the chart.
            var tickerOne = _C(exchanges[0].GetTicker);   // Get market data
            var tickerTwo = _C(exchanges[1].GetTicker);
            ObjChart.add([0, [nowTime, tickerOne.Last]]); // Use the timestamp as the X value and buy the price as the Y value to pass the index 0 data sequence.
            ObjChart.add([1, [nowTime, tickerTwo.Last]]); // Same as above
            ObjChart.update(chart);                       // Update the chart to show it.
            Sleep(2000);
        }
    }





2.6.28 Third-party Library
>>>>>>>>>>>>>>>>>>

JavaScript:

 - http://mathjs.org/
 - http://mikemcl.github.io/decimal.js/
 - http://underscorejs.org/
 - http://ta-lib.org/

C++:

 - https://nlohmann.github.io/json/

Python:

 - install any libray you want. 