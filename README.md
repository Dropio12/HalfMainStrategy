# Pyramiding Band Mean Reversion Strategy

## Overview
This Pine Script implements a mean reversion trading strategy using a pyramiding approach. It allows for up to 10 simultaneous positions and operates within the time frame of 9:30 AM to 4:00 PM (NY Time). The strategy employs Exponential Moving Averages (EMA) and Arnaud Legoux Moving Averages (ALMA) to identify entry points based on price crossings of calculated bands.

## Features
- **Pyramiding**: Allows up to 10 simultaneous positions.
- **Time-based trading**: Activates trades only during market hours (9:30 AM - 4:00 PM NY time).
- **Mean Reversion Bands**: Utilizes standard deviation bands around a moving average to determine potential reversal points.
- **ALMA Indicators**: Fast and slow ALMA indicators help to confirm trend direction for entries.
- **Take Profit Management**: Implements a delayed exit strategy that waits for three bars after a take profit condition is met.

## Input Parameters
- **Lookback Period (EMA)**: `emaPeriod` - Determines the period for the EMA used in mean reversion calculations (default: 200).
- **Bands Multiplier**: `bandMultiplier` - Sets the multiplier for the standard deviation to create the upper and lower mean reversion bands (default: 1.8).

## Strategy Logic
1. **Entry Conditions**:
   - **Short Entry**: Triggered when the price crosses above the upper mean reversion band and the slow ALMA is greater than the fast ALMA.
   - **Long Entry**: Triggered when the price crosses below the lower mean reversion band and the fast ALMA is greater than the slow ALMA.

2. **Exit Conditions**:
   - Trades are closed three bars after a take profit condition is met, allowing for potential further profit.

3. **Plotting**:
   - Displays the mean reversion bands, ALMA indicators, and entry signals on the chart for visualization.

## Alerts
The script includes alerts for both long and short entry signals, notifying traders when the price conditions for entries are met.

## Usage
1. Open TradingView.
2. Copy the Pine Script code into the Pine Script editor.
3. Adjust the input parameters as needed.
4. Add the script to your chart to visualize the strategy and its signals.

## License
This code is licensed under the Mozilla Public License 2.0. See [Mozilla Public License 2.0](https://mozilla.org/MPL/2.0/) for details.

## Author
- **Kokoabe**
