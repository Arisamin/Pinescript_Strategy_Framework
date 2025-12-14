# Strategy Implementation Guide

## Overview
This framework separates strategy logic from execution infrastructure. To implement a new strategy, you only need to create a custom library with your entry/exit logic.

## Implementation Steps

### 1. Create Your Strategy Library

Create a new PineScript library with these required functions:

```pinescript
// @version=6
library("YourStrategyName")

// Initialize strategy context with configuration parameters
export getContext(int param1, string param2, float param3) =>
    // Store your strategy parameters
    // Return a context object or tuple

// Main strategy logic - called on every bar
// Returns: [direction, stopLossPrice]
// - direction: "Long", "Short", or "None"
// - stopLossPrice: The stop loss price level
export tryEnterPosition(context) =>
    string direction = "None"
    float stopLossPrice = na
    
    // Your entry logic here
    // Analyze price action, indicators, patterns, etc.
    
    [direction, stopLossPrice]
```

### 2. Configure the Framework

In the main framework file, update these sections:

#### Import Your Library
```pinescript
import YourUsername/YourStrategyName/1 as myStrategy
```

#### Initialize Strategy Context
```pinescript
// Add input parameters for your strategy
int yourParam1 = input.int(9, "Your Parameter 1")
string yourParam2 = input.string("Both", "Your Parameter 2")

// Initialize your strategy
var strategyContext = myStrategy.getContext(yourParam1, yourParam2, fvgbBiggerRatio)
```

#### Call Strategy Logic
```pinescript
[longOrShort, stopLossDistance_] = myStrategy.tryEnterPosition(strategyContext)
```

### 3. Strategy Requirements

Your strategy must:
- **Return "Long"** when entry conditions for long position are met
- **Return "Short"** when entry conditions for short position are met  
- **Return "None"** when no entry conditions are met
- **Provide stop loss price** - the exact price level for stop loss

### 4. Available Framework Features

The framework automatically handles:
- ✅ Position sizing based on risk percentage
- ✅ Multiple position pyramiding (2 positions with different R targets)
- ✅ Trade cost calculations (commissions, swap fees)
- ✅ FTMO-compliant daily drawdown limits
- ✅ Gap zone blocking (market session filters)
- ✅ Trade tracking and statistics
- ✅ CSV export of trade history
- ✅ PineConnector alert integration
- ✅ Profitability validation before execution

### 5. Example Strategy Structure

```pinescript
// @version=6
library("BreakoutStrategy")

export getContext(int accumulationCandles, string tradeDirection, float fvgbRatio) =>
    [accumulationCandles, tradeDirection, fvgbRatio]

export tryEnterPosition(context) =>
    [accumulationCandles, tradeDirection, fvgbRatio] = context
    
    string signal = "None"
    float slPrice = na
    
    // Example: Simple breakout logic
    highestHigh = ta.highest(high, accumulationCandles)
    lowestLow = ta.lowest(low, accumulationCandles)
    
    if close > highestHigh and (tradeDirection == "Long" or tradeDirection == "Both")
        signal := "Long"
        slPrice := lowestLow
    
    if close < lowestLow and (tradeDirection == "Short" or tradeDirection == "Both")
        signal := "Short"
        slPrice := highestHigh
    
    [signal, slPrice]
```

## Configuration Parameters

### Risk Management (Already Configured)
- Account size per pair
- Total risk percentage per trade
- Position portions and R multiples
- Daily drawdown threshold

### Asset Settings (Pre-configured for FTMO)
- Leverage limits per instrument
- Commission rates
- Swap rates (long/short)
- Minimum trade sizes

### Execution Settings
- Gap zone hours (market session filters)
- Trade direction filter (Long/Short/Both)
- Profitability threshold
- Backtesting date ranges

## Testing Your Strategy

1. **Backtest**: Use TradingView's strategy tester
2. **Review Logs**: Check Pine Logs for detailed trade information
3. **Export CSV**: Enable CSV printing to analyze all trades
4. **Monitor Stats**: Win rate, P&L, drawdown, avg trade length

## Notes

- The framework uses `strategy.entry()` and `strategy.exit()` for execution
- All position sizing is calculated automatically
- Trade IDs are generated with timestamps for tracking
- PineConnector alerts are sent automatically for live trading
- FTMO compliance features are built-in (daily limits, gap zones)
