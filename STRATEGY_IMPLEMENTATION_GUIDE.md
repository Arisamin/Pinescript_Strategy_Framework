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

### Strategy Backtesting
1. **Backtest**: Use TradingView's strategy tester
2. **Review Logs**: Check Pine Logs for detailed trade information
3. **Export CSV**: Enable CSV printing to analyze all trades
4. **Monitor Stats**: Win rate, P&L, drawdown, avg trade length

### Unit Testing Framework

The framework includes a comprehensive unit testing system for validating core functions:

#### Enabling Tests
Set the **"Test Code"** checkbox in the Debug group to run unit tests:
```pinescript
bool enableTestCode = input.bool(false, "Test Code", group = "Debug")
```

#### Test Features
- ✅ **One-time execution**: Tests run only once when enabled
- ✅ **Persistent tracking**: Test state is tracked across bars using `isTestRun` flag
- ✅ **Visual results**: Color-coded table displays test results on chart (top-right)
- ✅ **Automatic reset**: Tests reset when checkbox is disabled

#### Built-in Test Functions
The framework includes 9 test functions covering core utilities:

1. **test_r2()** - Tests rounding to 2 decimal places
2. **test_roundToDecimals()** - Tests custom decimal rounding
3. **test_endsWithString()** - Tests string suffix checking
4. **test_getPipValueForType()** - Tests pip value calculation for forex/crypto
5. **test_getPipValueForLot()** - Tests lot-based pip value calculation
6. **test_calculateCommission()** - Tests commission calculations
7. **test_getLotSize()** - Tests position sizing with leverage limits
8. **test_isTradeProfitable()** - Tests profitability validation (passing case)
9. **test_isTradeProfitableFail()** - Tests profitability validation (failing case)

#### Test Results Table
When tests are enabled, a table displays:
- **Summary**: "ALL TESTS PASSED ✓" (green) or "X TESTS FAILED ✗" (red)
- **Failure details**: Function name with expected vs actual values (red background)
- **Success message**: Confirmation when all tests pass

#### Adding Custom Tests
To test your strategy functions:

```pinescript
// Add test function after other test functions
test_yourFunction() =>
    testName = "yourFunction()"
    expected = 100.0
    actual = yourFunction(param1, param2)
    passed = testFloatEquals(actual, expected, 0.01)
    if not passed
        array.push(testFailures, testName + " - Expected " + str_(expected) + ", got " + str_(actual))
    log.info(testTag + testName + " - " + (passed ? "PASSED" : "FAILED"))
    passed

// Add call in test execution block
if enableTestCode and not isTestRun
    isTestRun := true
    log.info(testTag + "Starting unit tests...")
    
    // ... existing test calls ...
    test_yourFunction()  // Add your test here
    
    log.info(testTag + "All tests completed. Failures: " + str_(array.size(testFailures)))
```

#### Test Helper Functions
- **testFloatEquals(actual, expected, tolerance)** - Compares floats with tolerance
- **testFailures** - Array storing failure messages
- **testTag** - Log prefix "[TEST] " for filtering test output

## Notes

- The framework uses `strategy.entry()` and `strategy.exit()` for execution
- All position sizing is calculated automatically
- Trade IDs are generated with timestamps for tracking
- PineConnector alerts are sent automatically for live trading
- FTMO compliance features are built-in (daily limits, gap zones)
