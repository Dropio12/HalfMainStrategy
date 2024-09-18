// This Pine Script™ code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Kokoabe
//@version=5
strategy("Pyramiding Band Mean Reversion with Delayed TP Exit (9:30 to 16:00 NY Time)", overlay=true, pyramiding=10)  // Allow up to 10 simultaneous positions

// Input Parameters
emaPeriod = input(200, title='Lookback Period (EMA)')
bandMultiplier = input.float(1.8, title='Bands Multiplier')

// Calculate Fast and Slow ALMA
lengthFast = 20
lengthSlow = 50
src = close
fastALMA = ta.alma(src, lengthFast, offset=0.85, sigma=6)
slowALMA = ta.alma(src, lengthSlow, offset=0.85, sigma=6)

// Trading time range: 9:30 AM to 4:00 PM in NY timezone (auto-adjusts for daylight savings)
startTime = timestamp(syminfo.timezone, year, month, dayofmonth, 9, 30)  // 9:30 AM
endTime = timestamp(syminfo.timezone, year, month, dayofmonth, 10, 45)    // 4:00 PM
inSession = (time >= startTime and time <= endTime)

// Calculate Mean Reversion Bands
meanReversion = ta.ema(close, emaPeriod)
stdDev = ta.stdev(close, emaPeriod)
upperBand2 = meanReversion + stdDev * bandMultiplier
lowerBand2 = meanReversion - stdDev * bandMultiplier

// Boiling Bands for SL/TP Reference
atrPeriod = 10
multiplier = 3.0
hpf = ta.wma(high + low + close, 3) / 3
atr = ta.atr(atrPeriod)
upperBand = hpf + multiplier * atr
lowerBand = hpf - multiplier * atr

// Plot Bands
plot(upperBand2, title='Upper Band', color=color.new(color.fuchsia, 0), linewidth=1)
plot(meanReversion, title='Mean', color=color.new(color.gray, 0), linewidth=1)
plot(lowerBand2, title='Lower Band', color=color.new(color.blue, 0), linewidth=1)

plot(upperBand, title="Boiling Band Up", color=color.blue)
plot(lowerBand, title="Boiling Band Down", color=color.blue)

// Variables to store the stop loss at the time of entry and the bar count after TP hit
var float slShort = na
var float slLong = na
var int barsAfterTP = na  // Counter for bars after TP hit

// Conditions for Short Entry: Price crosses above the fuchsia Upper Band and Slow ALMA > Fast ALMA
shortEntry = ta.crossover(close, upperBand2) and inSession and slowALMA > fastALMA
if (shortEntry)
    slShort := upperBand // Store the upper Bollinger Band value at the moment of entry
    strategy.entry("Short", strategy.short, when=shortEntry)
    barsAfterTP := na  // Reset the counter when a new trade is entered

// Conditions for Long Entry: Price crosses below the fuchsia Lower Band and Fast ALMA > Slow ALMA
longEntry = ta.crossunder(close, lowerBand2) and inSession and fastALMA > slowALMA
if (longEntry)
    slLong := lowerBand // Store the lower Bollinger Band value at the moment of entry
    strategy.entry("Long", strategy.long, when=longEntry)
    barsAfterTP := na  // Reset the counter when a new trade is entered

// Track if Take Profit is hit, and start the bar count
if (strategy.opentrades > 0)
    // Check for TP hit for Short positions
    if strategy.position_size < 0 and close <= meanReversion
        if na(barsAfterTP)
            barsAfterTP := 0  // Start counting bars after TP is hit
        else
            barsAfterTP += 1
    
    // Check for TP hit for Long positions
    if strategy.position_size > 0 and close >= meanReversion
        if na(barsAfterTP)
            barsAfterTP := 0  // Start counting bars after TP is hit
        else
            barsAfterTP += 1

// Close the trade 3 bars after TP is hit
if not na(barsAfterTP) and barsAfterTP >= 3
    strategy.close("Short", when=strategy.position_size < 0)  // Close Short after 3 bars post-TP
    strategy.close("Long", when=strategy.position_size > 0)   // Close Long after 3 bars post-TP

// Plot ALMA lines
plot(fastALMA, title="Fast ALMA", color=color.orange)
plot(slowALMA, title="Slow ALMA", color=color.aqua)

// Plotting Entry Signals
plotshape(shortEntry, title='SELL', style=shape.triangledown, location=location.abovebar, color=color.red, size=size.small)
plotshape(longEntry, title='BUY', style=shape.triangleup, location=location.belowbar, color=color.green, size=size.small)

// Alerts for Entry Signals
alertcondition(shortEntry, title="Short Signal", message="Price crossed Upper Band. Short Entry!")
alertcondition(longEntry, title="Long Signal", message="Price crossed Lower Band. Long Entry!")
