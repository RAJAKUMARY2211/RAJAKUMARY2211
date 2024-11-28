//+------------------------------------------------------------------+
//|                                            Moving Average EA.mq4 |
//|                                    Copyright 2024,Ankit Verma    |
//|                                                                  |
//+------------------------------------------------------------------+
#property copyright "Copyright 2024,Epic by Ankit"
#property link      "+917004763632"
#property version   "1.00"
#property strict

#property description "email: anki2020@gmail.com"
#property description "Iamankit"

#define BUYSIGNAL "Buy now"
#define SELLSIGNAL "Sell now"

#define EXITSELL "Exit sell"
#define EXITBUY "Exit Buy"

#include <stdlib.mqh>

input string sep1           = "========================================="; //=============
input string sep2           = " GENERAL SETTINGS";                         //*******
input string sep3           = "========================================="; //=============
input int    Magic          = 17339; //Magic Number
input string EA_Comment     = "CUSTOM EA"; //EA Comment
input string sep4           = "========================================="; //=============
input string sep5           = " TRADE SETTINGS";                           //*******
input string sep6           = "========================================="; //=============
input int    Slippage       = 10;
input double Lots           = 0.05; // Fixed Lots
input int    StopLoss       = 50; // SL High/Low (points)
input int    TakeProfit     = 100; // TP (Pips)

input string             sep10               = "========================================="; //=============
input string             sep11               = " MOVING AVERAGE SETTINGS";                           //*******
input string             sep12               = "========================================="; //=============

input string             fast                = "-- Fast MA --"; //Fast Low MA
input int                fast_ma_period      = 21; //Period
input int                fast_ma_shift       = 0; //Shift
input ENUM_MA_METHOD     fast_ma_method      = MODE_EMA; //MA method
input ENUM_APPLIED_PRICE fast_ma_price       = PRICE_CLOSE; //Apply to

input string             slow_MA             = "-- Slow MA --"; //Slow Low MA
input int                slow_ma_period      = 44; //Period
input int                slow_ma_shift       = 0; //Shift
input ENUM_MA_METHOD     slow_ma_method      = MODE_EMA; //MA method
input ENUM_APPLIED_PRICE slow_ma_price       = PRICE_CLOSE; //Apply to

int ErrCode;
datetime open_time;

struct trade_data
{
    int positions;
    bool position_active;
    int stoplevel;
};

trade_data data;  
//+------------------------------------------------------------------+
int OnInit()
{
    return(INIT_SUCCEEDED);
}
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
}
//+------------------------------------------------------------------+
void OnTick()
{
    data = GetData(data);
    if(data.positions == 0) CheckForOpen();
    else 
    {   
        CheckForClose();
    }
}
//+------------------------------------------------------------------+
void CheckForOpen()
{
    if(open_time >= Time[0]) return;
    string signal = GetSignal();
    if(signal == "") return;

    if(signal == BUYSIGNAL)
    {
        BUY();
        return;
    }
    if(signal == SELLSIGNAL)
    {
        SELL();
        return;
    }  
}
//+-------------------------------------------------------------------+
bool CheckForClose()
{
    string signal = GetExitSignal();
    if(signal == "") return false;

    bool closed = false;
    for(int i = OrdersTotal() - 1; i >= 0; i--)
    {
        if(!OrderSelect(i, SELECT_BY_POS, MODE_TRADES)) continue;
        if(OrderSymbol() != Symbol()) continue;
        if(OrderMagicNumber() != Magic) continue;
        if(OrderType() == OP_BUY)
        {
            if(signal == EXITBUY)
            {
                if(!OrderClose(OrderTicket(), OrderLots(), OrderClosePrice(), Slippage, clrWhite))
                {
                    ErrCode = GetLastError();
                    Print("Unable to close Buy order due to ", ErrorDescription(ErrCode));
                }
                else closed = true;
            }
        }
        if(OrderType() == OP_SELL)
        {
            if(signal == EXITSELL)
            {
                if(!OrderClose(OrderTicket(), OrderLots(), OrderClosePrice(), Slippage, clrWhite))
                {
                    ErrCode = GetLastError();
                    Print("Unable to close Sell order due to ", ErrorDescription(ErrCode));
                }
                else closed = true;
            }
        }
    }
    return closed;
} 
//+---------------------------------------------------------------+
string GetSignal()
{
    string signal = "";
    
    // Initialize moving averages
    double SlowMACurr = iMA(Symbol(), 0, slow_ma_period,
