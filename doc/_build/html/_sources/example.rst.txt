3.1 Moving Average Strategy
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

3.2 Iceberg Buy Order
=======================

Check on https://www.fmz.com/strategy/103319

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


3.3 Dual Thrust OKEX Feature
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