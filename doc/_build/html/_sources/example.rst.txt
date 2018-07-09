3.1 Use Websocket
=======================

For JavaScript:

.. code-block:: JavaScript

    function main() {
        LogStatus("connecting...");
        var client = Dial("wss://stream.binance.com:9443/ws/!ticker@arr");
        if (!client) {
            Log("Connection failed, program exited");
            return
        }
        Log("The connection is successful and the disconnected line is automatically reconnected")
        while (true) {
            var buf = client.read() // Read only returns data obtained after calling read
            if (!buf) {
                break;
            }
            var table = {
                type: 'table',
                title: 'Quotes chart',
                cols: ['Currency', 'highest', 'lowest', 'Bid', 'Ask', ' Last traded price', 'volume', 'Update time'],
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

For Python, you can use ``Dial()`` function  as well, or ``websocket_client`` library to access websocket.

3.2 Use C++ 
=======================

There are some difference between C++ and JavaScript in writing strategy.

.. code-block:: C++

    void main() {
        if (!Test("c++")) {
            Panic("please download the lastest docker");
        }
        // json: https://github.com/nlohmann/json
        // using Valid to judge return object is null or not. 
        LogProfitReset();
        LogReset();
        Log(_N(9.12345, 2));
        Log("use _C", _C(exchange.GetTicker), _C(exchange.GetAccount));

        // test async
        auto routineTicker = exchange.Go("GetTicker");
        auto routineDepth = exchange.Go("GetDepth");
        Ticker asyncTicker;
        Depth asyncDepth;
        
        // No timeout, wait until Ticker is returned
        if (routineTicker.wait(asyncTicker)) {
            Log("Wait Ticker OK", asyncTicker.Valid, asyncTicker, asyncTicker.Last);
        } else {
            Log("Wait Ticker fail");
        }
        // With 300ms tiemout, 0 means return immediately.
        if (routineDepth.wait(asyncDepth, 200)) {
            // not sure the depth returned is valid, use asyncDepth.Valid.
            Log("Wait Depth OK", asyncDepth.Valid, asyncDepth);
        } else {
            Log("Wait Depth timeout");
        }
        
        auto records = _C(exchange.GetRecords);
        Log("Last Kline", records[records.size()-1].Time, _D(records[records.size()-1].Time), records[records.size()-1]);
        // Test TA
        auto ema = TA.EMA(records, 20);
        Log("EMA", ema[ema.size()-1], ema[ema.size()-2], ema[ema.size()-3]);
        
        auto macd = talib.MACD(records);
        Log("MACD", macd[0][macd[0].size()-1], macd[1][macd[1].size()-1], macd[2][macd[2].size()-1]);
            
        //SetErrorFilter("timeout");
        Log(GetOS(), GetPid(), GetLastError(), "MD5", MD5("hello"));
        Log("Hash", Hash("md5", "hex", "hello"), Hash("sha512", "base64", "hello"));
        Log("HMAC", HMAC("sha512", "base64", "hello", "pass"));
        Log(Version(), Unix(), UnixNano());
        Log("Start Test Chart");
        Chart c = Chart(R"EOF({"chart":{"type":"line"},"title":{"text":"Simple Chart"},"xAxis":{"title":{"text":"Date"}},"yAxis":{"title":{"text":"Number"}},"series":[{"name":"number","data":[]}]})EOF");
        c.reset();
        for (size_t i = 0; i < 10; i++) {
            c.add(0, {(Unix() + i)*1000, rand() % 100});
        }
        Log(exchange.GetName(), exchange.GetLabel(), exchanges.size());
        auto acc = exchange.GetAccount();
        if (acc.Valid) {
            Log(acc);
        }
        
        // Using LogStatus and json draw a table, learn more about json libray: https://github.com/nlohmann/json
        json tbl = R"({"type" : "table", "title" : "AAA", "cols" : ["Head1", "Head2"], "rows": []})"_json;
        tbl["rows"].push_back({"111", "222"});
        tbl["rows"].push_back({"col2", "col22"});
        LogStatus("`"+tbl.dump()+"`");
        
        auto ticker = exchange.GetTicker();
        if (ticker.Valid) {
            Log(ticker);
            Log(ticker.Info); // Info Struct is json object
        }
        
        auto d = exchange.GetDepth();
        if (d.Valid) {
            Log(d.Asks[0], d.Bids[0]);
        }
        // Test features
        if (exchange.GetName() == "Futures_OKCoin") {
            exchange.SetContractType("this_week");
            exchange.SetMarginLevel(20);
            exchange.SetDirection("closebuy");
                
            auto positions = exchange.GetPosition();
            if (positions.Valid) {
                Log(positions);
            }
        }
        // test other function
        Log("HttpQuery", HttpQuery("http://www.baidu.com/404").size());
        auto obj = json::parse(HttpQuery("http://www.baidu.com/404", "", "", "", true));
        string body = obj["Body"];
        Log("HttpQuery", body.size(), obj["Header"].dump());
        Log(Mail("smtp://smtp.163.com", "test@163.com", "password", "admin@163.com", "title", "test c++ email"));
        // Test Dial
        auto client = Dial("tcp://www.baidu.com:80");
        if (client.Valid) {
            client.write("GET / HTTP/1.1\nHost: www.baidu.com\nConnection: Close\n\n");
            while (true) {
                string buf = client.read();
                if (buf == "") {
                    break;
                }
                Log("Dial receive", buf.size());
            }
            client.close();
        }

        _G("OK","xxx");
        Log(_G("OK"));
        _G("OK","yyyyyy");
        Log(_G("OK"));
        _G("OK",NULL);
        Log(_G("OK"));
    }

3.3 Sell ALL AltCoin to BTC in Binance
=======================

This strategy will sell all your AltCoin to BTC (or ETH, BNB, USDT), learn how to trade multiple trading pair,
and follow the price and amount filter of exchange.

.. code-block:: JavaScript

    function main() {
        var quoteCurrency = 'BTC'; //Can be 'ETH', 'BNB', 'USDT'
        Log("Quote Currency", quoteCurrency);
        var symbols = JSON.parse(_C(HttpQuery, "https://api.binance.com/api/v1/exchangeInfo")).symbols;
        _.each(_C(exchange.GetAccount).Info.balances, function(ele) {
            if (ele.asset == quoteCurrency) {
                return
            }
            var totalV = parseFloat(ele.free) + parseFloat(ele.locked);
            if (totalV == 0) {
                return;
            }
            var cfg = _.findWhere(symbols, {symbol: ele.asset+quoteCurrency});
            if (!cfg) {
                Log("Not found", ele.asset, "trading pair", ele);
                return;
            }
            var filter = _.findWhere(cfg.filters, {filterType: "LOT_SIZE"});
            if (!filter) {
                return;
            }
            
            var v = _N(parseInt(totalV/filter.stepSize)*filter.stepSize, cfg.baseAssetPrecision);
            if (v > 0) {
                Log(ele, "stepSize", filter.stepSize);
                exchange.IO("currency", ele.asset + "_"+quoteCurrency);
                while (true) {
                    var orders = _C(exchange.GetOrders);
                    _.each(orders, function(order) {
                        exchange.CancelOrder(order.Id);
                    });
                    if (orders.length == 0) {
                        break;
                    }
                }
                exchange.Sell(-1, v);
                Log(ele);
            }
        });
        Log("Done, Now", quoteCurrency, "Balance", _C(exchange.GetAccount).Balance);
    }



3.4 Moving Average Strategy
=======================

The moving average (MA) is a simple technical analysis tool that smooths out price data by creating a constantly updated average price. 

Check on https://www.fmz.com/strategy/103070 for parameters configs.The global variables ``FastPeriod``, ``SlowPeriod``,etc... are defined in configs.

Source code:

.. code-block:: JavaScript

    function main() {
    var initAccount = _C(exchange.GetAccount);//using _C() to retry
    var ticker = exchange.GetTicker();
    //calc InitValue to log profit
    var InitValue = (initAccount.Stocks + initAccount.FrozenStocks)*ticker.Last \
                    + initAccount.Balance + initAccount.FrozenBalance;
    while (true) {
        var records = _C(exchange.GetRecords);
        ticker =_C(exchange.GetTicker);
        var FastRecords = TA.MA(records,FastPeriod);// using MA of TA-Lib
        var SlowRecords = TA.MA(records,SlowPeriod);
        var NowAccount = _C(exchange.GetAccount);
        var n = _Cross(FastRecords, SlowRecords);// using _Cross(), check on global function
        if (n >= EnterPeriod && NowAccount.Balance > 0) {
            var Price = _N(ticker.Sell+Slippage, 2); // add Slippage to make sure order can be done
            var Amount = _N(0.99*NowAccount.Balance/Price, 3);
            if(Amount>0.1){
               var id = exchange.Buy(Price, Amount);
               //Cancel the pending order
               if(exchange.GetOrders(id).Status == ORDER_STATE_PENDING){exchange.CancelOrder(id);}
               LogProfit((NowAccount.Stocks + NowAccount.FrozenStocks)*ticker.Last\
                         + NowAccount.Balance + NowAccount.FrozenBalance - InitValue);
            }
        }
        if(n <= -EnterPeriod && NowAccount.Stocks > 0) {
            var Price = _N(ticker.Buy-Slippage, 2);
            var Amount = _N(NowAccount.Stocks, 3);
            if(Amount>0.1){
                var id = exchange.Sell(Price, Amount);
                if(exchange.GetOrders(id).Status == ORDER_STATE_PENDING){exchange.CancelOrder(id);}
                LogProfit((NowAccount.Stocks + NowAccount.FrozenStocks)*ticker.Last\
                         + NowAccount.Balance + NowAccount.FrozenBalance - InitValue);
            }
        }
        Sleep(Interval*1000);
    }
}

3.5 Iceberg Buy Order
=======================

Check on https://www.fmz.com/strategy/103319.

Source code:

.. code-block:: JavaScript

    function CancelPendingOrders() {
        while (true) {
            var orders = _C(exchange.GetOrders);
            if (orders.length == 0) {
                return;
            }

            for (var j = 0; j < orders.length; j++) {
                exchange.CancelOrder(orders[j].Id);
                if (j < (orders.length-1)) {
                    Sleep(Interval);
                }
            }
        }
    }

    var LastBuyPrice = 0;
    var InitAccount = null;

    function dispatch() {
        var account = null;
        var ticker = _C(exchange.GetTicker);
        if (LastBuyPrice > 0) {
            if (_C(exchange.GetOrders).length > 0) {
                if (ticker.Last > LastBuyPrice && ((ticker.Last - LastBuyPrice) / LastBuyPrice) > (2*(EntrustDepth/100))) {
                    Log('deviate to much, newest last price:', ticker.Last, 'order buy price', LastBuyPrice);
                    CancelPendingOrders();
                } else {
                    return true;
                }
            } else {
                account = _C(exchange.GetAccount);
                Log("order finised, total cost:", _N(InitAccount.Balance - account.Balance), "avg buy price:", _N((InitAccount.Balance - account.Balance) / (account.Stocks - InitAccount.Stocks)));
            }
            LastBuyPrice = 0;
        }
        
        var BuyPrice = _N(ticker.Buy * (1 - EntrustDepth/100),PricePerision);
        if (BuyPrice > MaxBuyPrice) {
            return true;
        }
        
        if (!account) {
            account = _C(exchange.GetAccount);
        }


        if ((InitAccount.Balance - account.Balance) >= TotalBuyNet) {
            return false;
        }
        
        var RandomAvgBuyOnce = (AvgBuyOnce * ((100 - FloatPoint) / 100)) + (((FloatPoint * 2) / 100) * AvgBuyOnce * Math.random());
        var UsedMoney = Math.min(account.Balance, RandomAvgBuyOnce, TotalBuyNet - (InitAccount.Balance - account.Balance));
        
        var BuyAmount = _N(UsedMoney / BuyPrice, 3);
        if (BuyAmount < MinStock) {
            return false;
        }
        LastBuyPrice = BuyPrice;
        exchange.Buy(BuyPrice, BuyAmount, 'Cost: ', _N(UsedMoney), 'last price', ticker.Last);
        return true;
    }

    function main() {
        CancelPendingOrders();
        InitAccount = _C(exchange.GetAccount);
        Log(InitAccount);
        if (InitAccount.Balance < TotalBuyNet) {
            throw "balance not enough";
        }
        LoopInterval = Math.max(LoopInterval, 1);
        while (dispatch()) {
            Sleep(LoopInterval * 1000);
        }
        Log("All Done", _C(exchange.GetAccount));
    }


3.6 Dual Thrust OKEX Feature
=======================

A classic breakout strategy, Check on https://www.fmz.com/strategy/103247 for configs.

You can learn how to trade features and draw charts from the source code.

learn more on https://www.quantconnect.com/tutorials/strategy-library/dual-thrust-trading-algorithm

Source code:

.. code-block:: JavaScript

    var ChartCfg = {
        __isStock: true,
        title: {
            text: 'Dual Thrust Up-Down Track'
        },
        yAxis: {
            plotLines: [{value: 0,
                color: 'red',
                width: 2,
                label: {
                    text: 'Up Track',
                    align: 'center'}
                    }, 
                {value: 0,
                color: 'green',
                width: 2,
                label: {
                    text: 'Down Track',
                    align: 'center'},
                }
            ]
        },
        series: [{type: 'candlestick',
            name: 'current cycle',
            id: 'primary',
            data: []
            },
            {type: 'flags',
            onSeries: 'primary',
            data: [],
            }
        ]
    };

    var STATE_IDLE = 0;
    var STATE_LONG = 1;
    var STATE_SHORT = 2;
    var State = STATE_IDLE;

    var LastBarTime = 0;
    var UpTrack = 0;
    var BottomTrack = 0;
    var chart = null;
    var InitAccount = null;
    var LastAccount = null;
    var Counter = {
        w: 0,
        l: 0
    };

    function _N(v) {
        return Decimal(v).toSD(4, 1).toNumber();
    }

    function GetPosition(posType) {
        var positions = exchange.GetPosition();
        for (var i = 0; i < positions.length; i++) {
            if (positions[i].Type === posType) {
                return [positions[i].Price, positions[i].Amount];
            }
        }
        return [0, 0];
    }

    function CancelPendingOrders() {
        while (true) {
            var orders = exchange.GetOrders();
            for (var i = 0; i < orders.length; i++) {
                exchange.CancelOrder(orders[i].Id);
                Sleep(Interval);
            }
            if (orders.length === 0) {
                break;
            }
        }
    }

    function Trade(currentState, nextState) {
        var pfn = nextState === STATE_LONG ? exchange.Buy : exchange.Sell;
        if (currentState !== STATE_IDLE) {
            exchange.SetDirection(currentState === STATE_LONG ? "closebuy" : "closesell");
            while (true) {
                var amount = GetPosition(currentState === STATE_LONG ? PD_LONG : PD_SHORT)[1];
                if (amount === 0) {
                    break;
                }
                // pfn(amount);
                pfn(nextState === STATE_LONG ? _C(exchange.GetTicker).Sell * 1.001 : _C(exchange.GetTicker).Buy * 0.999, amount);
                Sleep(Interval);
                CancelPendingOrders();
            }
            var account = exchange.GetAccount();

            if (account.Stocks > LastAccount.Stocks) {
                Counter.w++;
            } else {
                Counter.l++;
            }

            LogProfit(_N(account.Stocks - InitAccount.Stocks), "Profit rate:", _N((account.Stocks - InitAccount.Stocks) * 100 / InitAccount.Stocks) + '%');
            LastAccount = account;
        }
        exchange.SetDirection(nextState === STATE_LONG ? "buy" : "sell");
        while (true) {
            var pos = GetPosition(nextState === STATE_LONG ? PD_LONG : PD_SHORT);
            if (pos[1] >= AmountOP) {
                Log("Average Price", pos[0], "amount:", pos[1]);
                break;
            }
            // pfn(AmountOP-pos[1]);
            pfn(nextState === STATE_LONG ? _C(exchange.GetTicker).Sell * 1.001 : _C(exchange.GetTicker).Buy * 0.999, AmountOP-pos[1]);
            Sleep(Interval);
            CancelPendingOrders();
        }
    }

    function onTick(exchange) {
        var records = exchange.GetRecords();
        if (!records || records.length <= NPeriod) {
            return;
        }
        var Bar = records[records.length - 1];
        if (LastBarTime !== Bar.Time) {
            var HH = TA.Highest(records, NPeriod, 'High');
            var HC = TA.Highest(records, NPeriod, 'Close');
            var LL = TA.Lowest(records, NPeriod, 'Low');
            var LC = TA.Lowest(records, NPeriod, 'Close');

            var Range = Math.max(HH - LC, HC - LL);

            UpTrack = _N(Bar.Open + (Ks * Range));
            DownTrack = _N(Bar.Open - (Kx * Range));
            if (LastBarTime > 0) {
                var PreBar = records[records.length - 2];
                chart.add(0, [PreBar.Time, PreBar.Open, PreBar.High, PreBar.Low, PreBar.Close], -1);
            } else {
                for (var i = Math.min(records.length, NPeriod * 3); i > 1; i--) {
                    var b = records[records.length - i];
                    chart.add(0, [b.Time, b.Open, b.High, b.Low, b.Close]);
                }
            }
            chart.add(0, [Bar.Time, Bar.Open, Bar.High, Bar.Low, Bar.Close]);
            ChartCfg.yAxis.plotLines[0].value = UpTrack;
            ChartCfg.yAxis.plotLines[1].value = DownTrack;
            ChartCfg.subtitle = {
                text: 'Up Track: ' + UpTrack + '  Down Track: ' + DownTrack
            };
            chart.update(ChartCfg);
            chart.reset(PeriodShow);

            LastBarTime = Bar.Time;
        } else {
            chart.add(0, [Bar.Time, Bar.Open, Bar.High, Bar.Low, Bar.Close], -1);
        }

        LogStatus("Price:", Bar.Close, "Up:", UpTrack, "Down:", DownTrack, "Wins: ", Counter.w, "Losses:", Counter.l, "Date:", new Date());
        var msg;
        if (State === STATE_IDLE || State === STATE_SHORT) {
            if (Bar.Close >= UpTrack) {
                msg  = 'Long Price: ' + Bar.Close + ' Up Track:' + UpTrack;
                Log(msg);
                Trade(State, STATE_LONG);
                State = STATE_LONG;
                chart.add(1, {x:Bar.Time, color: 'red', shape: 'flag', title: 'Long', text: msg});
            }
        }

        if (State === STATE_IDLE || State === STATE_LONG) {
            if (Bar.Close <= DownTrack) {
                msg = 'Short Price: ' + Bar.Close + ' Down Track:' + DownTrack;
                Log(msg);
                Trade(State, STATE_SHORT);
                chart.add(1, {x:Bar.Time, color: 'green', shape: 'circlepin', title: 'Short', text: msg});
                State = STATE_SHORT;
            }
        }
    }

    function onexit() {
        var pos = exchange.GetPosition();
        if (pos.length > 0) {
            Log("Warning, has positions when exiting", pos);
        }
    }

    function main() {
        if (exchange.GetName() !== 'Futures_OKCoin') {
            throw "Only support OKEX features";
        }
        exchange.SetRate(1);
        exchange.SetContractType(["this_week", "next_week", "quarter"][ContractTypeIdx]);
        exchange.SetMarginLevel([10, 20][MarginLevelIdx]);

        if (exchange.GetPosition().length > 0) {
            throw "Can't have Positions when start.";}

        CancelPendingOrders();

        InitAccount = LastAccount = exchange.GetAccount();
        LoopInterval = Math.min(1, LoopInterval);
        Log('Exchange Name:', exchange.GetName(), InitAccount);
        LogStatus("Ready...");

        LogProfitReset();
        chart = Chart(ChartCfg);
        chart.reset();

        LoopInterval = Math.max(LoopInterval, 1);
        while (true) {
            onTick(exchange);
            Sleep(LoopInterval * 1000);
        }
    }


3.7 High Frequency Marketmaker
================================

This is a simple but powerful strategy that used to earn thousands of times in real BTC spot markets.
Can't run it on exchanges that have high trade fee.

Source code:

.. code-block:: JavaScript

    var floatAmountBuy = 20;
    var floatAmountSell = 20;
    var diffPrice = 3;
    var Interval = 3000;

    function CancelPendingOrders() {
        var orders = _C(exchange.GetOrders);
        for (var j = 0; j < orders.length; j++) {
            exchange.CancelOrder(orders[j].Id, orders[j]);}
    }

    function GetPrice(Type, depth) {
        var amountBids = 0;
        var amountAsks = 0;
        if(Type == "Buy"){
            for(var i=0;i<20;i++){
                amountBids += depth.Bids[i].Amount;
                if (amountBids > floatAmountBuy){
                    return depth.Bids[i].Price + 0.01;
                }
            }
        }
        if(Type == "Sell"){
            for(var j=0; j<20; j++){
                amountAsks += depth.Asks[j].Amount;
                if (amountAsks > floatAmountSell){
                    return depth.Asks[j].Price - 0.01;
                }
            }
        }
        return depth.Asks[0].Price
    }
    
    function onTick() {
        var depth = _C(exchange.GetDepth);
        var buyPrice = GetPrice("Buy", depth);
        var sellPrice = GetPrice("Sell", depth);
        if ((sellPrice - buyPrice) <= diffPrice){
            buyPrice -= 10;
            sellPrice += 10;
        }
        CancelPendingOrders();
        var account = _C(exchange.GetAccount);
        var amountBuy = _N((account.Balance / buyPrice-0.1), 2); 
        var amountSell = _N((account.Stocks), 2); 
        if (amountSell > 0.02) {
            exchange.Sell(sellPrice, amountSell);
        }
        if (amountBuy > 0.02) {
            exchange.Buy(buyPrice, amountBuy);
        }
    }
        
    function main() {
        while (true) {
            onTick();
            Sleep(Interval);
        }
    }