# CLAUDE.md - TradingView Pine Script Repository

## Repository Overview

This repository contains a custom TradingView Pine Script trading strategy named "XXX" (SAIYAN OCC). The strategy is designed for automated trading on TradingView with support for webhooks, multiple take-profit levels, and advanced technical indicators.

**Version**: v6_1_23 (2023.10.13)
**Language**: Pine Script v5
**License**: Mozilla Public License 2.0
**Author**: @traderschatroom88 / @gu5tavo71 for Riccardo (@rikyfx)

## Repository Structure

```
/home/user/Tradingview/
├── .git/                    # Git repository data
└── XXX_MODDED.txt          # Main Pine Script strategy file (1,330 lines)
```

## Codebase Architecture

### File: XXX_MODDED.txt

This is a comprehensive Pine Script v5 strategy file organized into three main sections:

#### 1. G_SCRIPT01 (Lines 27-813): SAIYAN OCC Indicator Logic
**Purpose**: Core indicator calculations and visual elements

**Key Components**:
- **Moving Averages**: Configurable MA types (TEMA, HullMA, ALMA)
- **Supply/Demand Zones**: Automatic detection using swing highs/lows
  - Uses ATR-based box drawing
  - Tracks point of interest (POI) levels
  - Converts to Break of Structure (BOS) when zones are broken
- **Keltner Channels**: Multiple KC bands (80-period, various multipliers)
- **Support/Resistance Levels**: Dynamic S/R calculation with zones
  - Strength-based filtering (strengthSR parameter)
  - Percentage-based zone widths
- **Zig Zag Pattern**: Optional swing point visualization
- **Pivot Analysis**: Linear regression with deviation bands

**Key Functions**:
- `f_array_add_pop()`: Array management (lines 71-73)
- `f_sh_sl_labels()`: Swing high/low labeling (lines 76-105)
- `f_check_overlapping()`: Prevents overlapping zones (lines 108-126)
- `f_supply_demand()`: Draws supply/demand boxes (lines 130-163)
- `f_sd_to_bos()`: Converts zones to BOS when broken (lines 167-208)

**Critical Arrays**:
- `swing_high_values` / `swing_low_values`: Track recent swing points
- `current_supply_box` / `current_demand_box`: Active zone tracking
- `supply_bos` / `demand_bos`: Broken structure tracking

#### 2. G_RISK (Lines 821-1190): Risk Management System
**Purpose**: Entry/exit logic with multi-level take-profit and stop-loss

**Configuration**:
- **Entry Triggers**:
  - `leTrigger`: Long entry (closeSeriesAlt crosses over openSeriesAlt)
  - `seTrigger`: Short entry (closeSeriesAlt crosses under openSeriesAlt)
- **Risk Parameters**:
  - TP1: 1% level, 50% position exit
  - TP2: 1.5% level, 30% position exit
  - TP3: 2% level, 20% position exit
  - SL: 0.5% level
- **State Machine**: Uses `condition` variable to track trade state
  - 0.0: No position
  - ±1.0: Position opened
  - ±1.1: TP1 hit
  - ±1.2: TP2 hit
  - ±1.3: TP3 hit

**Key Functions**:
- `f_tp()`: Calculates take-profit lines (lines 888-895)
- `f_cross()`: Custom crossover/crossunder detection (lines 898-900)

**Strategy Calls**: Lines 957-1046
- Pyramiding disabled (no adding to positions)
- Percent-of-equity based position sizing (10% default)
- Processes orders on bar close (non-repainting)

#### 3. G_SCRIPT02 & G_SCRIPT03 (Lines 1192-1329): Visualization & Alerts
**Purpose**: Visual elements and alert management

**Features**:
- Dynamic TP/SL/Entry line plotting
- Price level labels on the right side
- Colored bars indicating trend
- Webhook message support for automated trading
- Multiple alert types (Entry, Exit, TP, SL)

## Technical Indicator Details

### Moving Average System
- **Base Type**: Configurable (TEMA/HullMA/ALMA)
- **Period**: 2 (default, highly responsive)
- **Timeframe**: 15-minute default with 8x multiplier for alternate signals
- **Non-Repainting**: Uses `delayOffset` and `request.security` with lookahead

### Supply/Demand Detection
- **Swing Length**: 10 bars (default)
- **History Kept**: 20 zones (default)
- **Box Width**: 2.5x ATR (default)
- **Overlap Prevention**: 2x ATR threshold to avoid cluttered zones

### Support/Resistance System
- **Lookback Period**: 284 bars
- **Rebound Detection**: 10 bars each side
- **Strength Filter**: Minimum 2 touches (configurable)
- **Zone Width**: 2% of 300-bar high/low range

## Development Workflows

### For AI Assistants Working on This Code

#### 1. Understanding Trade Flow
```
Entry Signal → State 1.0 → TP1 Hit → State 1.1 → TP2 Hit → State 1.2 → TP3 Hit → State 1.3
                     ↓         ↓                    ↓                    ↓
                     └─────────┴────────────────────┴─── SL Hit → State 0.0
```

#### 2. Modifying Entry Logic
- **Location**: Lines 818-819 (triggers)
- **Base Logic**: Lines 748-754 (MA calculation)
- **Important**: Maintain `barstate.isconfirmed` checks to prevent repainting

#### 3. Adjusting Risk Management
- **TP/SL Levels**: Lines 844-864 (input definitions)
- **Position Sizing**: Lines 967-995 (qty_percent parameters)
- **Exit Logic**: Lines 958-1046 (strategy calls)

#### 4. Adding New Indicators
- **Best Location**: After line 813 (end of G_SCRIPT01)
- **Naming Convention**: Use descriptive variable names with camelCase
- **Performance**: Avoid excessive historical calculations (use `var` for state)

#### 5. Webhook Integration
- **Message Templates**: Lines 867-878 (input definitions)
- **Format**: Plain text or JSON for bot integration
- **Trigger Points**:
  - Entry: `alert_message` in `strategy.entry()`
  - Exit: `alert_profit` and `alert_loss` in `strategy.exit()`

### Code Style Conventions

1. **Indentation**: Use 4 spaces (consistent throughout)
2. **Variable Naming**:
   - Inputs: `i_` prefix (e.g., `i_lxLvlTP1`)
   - Constants: UPPERCASE (e.g., `G_RISK`, `O_LEMSG`)
   - Private variables: lowercase with underscore (e.g., `swing_length`)
3. **Comments**:
   - Section headers use `// ————————— <section_name>`
   - Regions marked with `#region` and `#endregion`
4. **Color Handling**:
   - Use `#00000000` for transparent (currently set for many visual elements)
   - Define color variables before plotting
5. **Line Breaks**: Use for readability in long parameter lists

### Testing Strategy Changes

1. **Backtest Settings**:
   - Default: 10% of equity per trade
   - No pyramiding
   - Process orders on close (realistic execution)

2. **Visual Verification**:
   - Enable alert labels (`i_alertOn`)
   - Enable bar coloring (`i_barColOn`)
   - Check TP/SL lines appear correctly

3. **Common Issues**:
   - **Repainting**: Always use `barstate.isconfirmed` for entries
   - **Lookahead Bias**: Use `barmerge.lookahead_on` correctly
   - **Array Bounds**: Check array sizes before access
   - **Color Visibility**: Many colors set to transparent (#00000000)

## Key Conventions for AI Assistants

### DO's:
- ✅ Preserve the state machine logic (condition variable)
- ✅ Maintain non-repainting behavior
- ✅ Keep webhook message structure intact
- ✅ Test array bounds before operations
- ✅ Use `var` for variables that maintain state
- ✅ Comment significant logic changes
- ✅ Preserve the three-section structure (G_SCRIPT01/G_RISK/G_SCRIPT02-03)

### DON'Ts:
- ❌ Don't enable pyramiding without explicit user request
- ❌ Don't remove `barstate.isconfirmed` checks
- ❌ Don't change default risk parameters without discussion
- ❌ Don't introduce lookahead bias in signals
- ❌ Don't modify array sizes without checking all references
- ❌ Don't add CPU-intensive calculations in loops
- ❌ Don't break the webhook message format

### Performance Considerations:
- **Heavy Loops**: Lines 586-629 (S/R calculation) - avoid adding more nested loops
- **Array Operations**: Use built-in functions when possible
- **Security Calls**: Minimize `request.security()` calls (expensive)
- **Visual Elements**: Too many boxes/lines can cause performance issues

## Git Workflow

### Branch Strategy:
- **Working Branch**: `claude/claude-md-mi0nm101ma2xn6wv-01UTwrGf7UK4wFnjqtaAD8HH`
- **Main Branch**: Not specified (this appears to be initial development)

### Commit Guidelines:
1. Use descriptive commit messages
2. Reference specific line numbers when fixing bugs
3. Test in TradingView before committing
4. Include version number updates if significant changes

### File Naming:
- Current: `XXX_MODDED.txt`
- Keep `.txt` extension for easy viewing/editing
- Can be imported directly into TradingView

## TradingView Integration

### Importing the Strategy:
1. Open TradingView Pine Editor
2. Create new indicator/strategy
3. Copy contents of `XXX_MODDED.txt`
4. Save and add to chart

### Configuring for Live Trading:
1. **Alerts**: Set up with specific conditions (lines 1318-1327)
2. **Webhooks**: Configure webhook URLs with message templates
3. **Risk Settings**: Adjust TP/SL levels in inputs (lines 844-864)
4. **Timeframe**: Set chart timeframe and `res` parameter
5. **Visual Elements**: Toggle S/R, zones, labels as needed

## Common Modifications

### 1. Change Take-Profit Levels
```pinescript
// Lines 844-864
i_lxLvlTP1 = input.float(1,    'Level TP1', group = G_RISK)  // Change 1 to desired %
i_lxLvlTP2 = input.float(1.5,  'Level TP2', group = G_RISK)  // Change 1.5 to desired %
i_lxLvlTP3 = input.float(2,    'Level TP3', group = G_RISK)  // Change 2 to desired %
```

### 2. Modify Position Sizing
```pinescript
// Lines 14-15
default_qty_type  = strategy.percent_of_equity,
default_qty_value = 10,  // Change from 10% to desired percentage
```

### 3. Adjust Entry Signal Sensitivity
```pinescript
// Lines 35-36
basisLen = input.int(2, 'MA Period', minval=1)  // Increase for less sensitivity
intRes   = input(8, 'Multiplier for Alernate Signals')  // Adjust timeframe multiplier
```

### 4. Enable/Disable Visual Elements
```pinescript
// Lines 541-551 (S/R)
enableSR = input(false, "SR On/Off", group="SR")  // Change to true

// Lines 51-52 (Supply/Demand)
show_zigzag = input.bool(false, 'Show Zig Zag', group = 'Visual Settings')
show_price_action_labels = input.bool(false, 'Show Price Action Labels')
```

## Advanced Topics

### State Machine Details
The `condition` variable tracks the position lifecycle:
- **Entry**: 0.0 → ±1.0
- **TP1 Hit**: ±1.0 → ±1.1
- **TP2 Hit**: ±1.1 → ±1.2
- **TP3 Hit**: ±1.2 → ±1.3
- **Exit (SL/Manual)**: Any → 0.0

This allows for dynamic TP/SL updates as the trade progresses.

### Non-Repainting Implementation
```pinescript
// Line 480-481: Security function
rp_security(_symbol, _res, _src) =>
    request.security(_symbol, _res, _src[barstate.isrealtime ? 1 : 0])
```
Uses historical bar in backtesting, previous bar in real-time.

### Supply/Demand Algorithm
1. Detect swing highs/lows (line 239-240)
2. Calculate ATR-based box dimensions (line 131)
3. Check for overlapping zones (lines 108-126)
4. Draw boxes with POI (Point of Interest) markers (lines 148-163)
5. Monitor for breakouts → convert to BOS (lines 167-208)

## Troubleshooting

### Common Errors:
1. **"Cannot call 'array.get' with index X"**
   - Check array initialization sizes (lines 243-259)
   - Verify history_of_demand_to_keep parameter

2. **"Script has too many drawings"**
   - Reduce history_of_demand_to_keep (line 47)
   - Disable unused visual elements

3. **"Repainting detected"**
   - Verify `barstate.isconfirmed` in entry logic (lines 958, 1003)
   - Check security function usage (lines 480-484)

4. **"Alerts not firing"**
   - Verify trade conditions in lines 1318-1327
   - Check `alert.freq_once_per_bar_close` setting

## Version History

- **v6_1_23 (2023.10.13)**: Current version
  - Full supply/demand zone implementation
  - Multi-level TP system
  - Webhook support
  - S/R dynamic levels

## Additional Resources

- **Pine Script Documentation**: https://www.tradingview.com/pine-script-docs/
- **License**: https://mozilla.org/MPL/2.0/
- **Original Project**: Project #827 by @gu5tavo71

## Disclaimer

⚠️ **Educational Purpose Only**: This script is for educational purposes. Not financial advice. Use at your own risk. The author is not a financial advisor.

---

*Last Updated: 2025-11-15*
*For Claude AI Assistant and Human Developers*
