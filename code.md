//Relative Price Strength (RS) Rating or Relative Strenght.
//This is a measure of a stock's price performance over the last
//twelve months, compared to all US stocks.
//The rating scale ranges frome 1 (lowest) to 99 (highest)
//Let's create an equivalent here for TradingView!
//
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Fred6724
// RaviYendru thank you for providing the intial script

//@version=5
indicator(title='RS Rating', shorttitle='RS Rating', overlay=true, max_bars_back = 253)
// Inputs
hideRSRat   = input(false, title='Hide Rating', group = 'RS Line', inline='0')
// seedetail   = input(false, title='Display the 3 results', group = 'Parameters', inline='0')
src         = close
comparativeTickerId =input('HOSE:VNINDEX', title='Symbol', group = 'RS Line', tooltip = 'Reference ticker used for calculation of the RS Line and RS Score.', inline = 'a')
colorRS     = input(color.rgb(0, 0, 255,0), title = 'Color', group = 'RS Line', inline = 'a')
ratingOnly  = input(false, title='Rating Only', group = 'RS Line')
SpxValue    = input(4400, title='Value of Comparative Symbol', group = 'Offset', tooltip = 'Used to display the RS Line under the price.')
offset      = input.int(20, minval = 0, maxval = 2000, title='Offset (%)', group = 'Offset', tooltip = 'Used to display the RS Line under the price.')
plotNewHigh = input(true, title = 'Plot RS New Highs', group = 'RS Line New High')
rsNewHigh   = input.string('RS New Highs', title = 'Type', options=['RS New Highs','RS New Highs Before Price', 'Historical RS New Highs', 'Historical RS New Highs  Before Price'], group = 'RS Line New High', inline = 'b')
blueDotCol  = input(color.rgb(121, 213, 242,62), title = 'Color', group = 'RS Line New High', inline = 'b')
lookback    = input.int(250, title = 'Look-back', minval = 1, maxval = 252, group = 'RS Line New High', tooltip = 'The lookback for calculation of price and RS New Highs.', inline = 'b')
sizeLabHigh = input.string('Tiny', title = 'Size', options = ['Tiny', 'Small', 'Normal', 'Large'], group = 'RS Line New High')
plotNewLow  = input(false, title = 'Plot RS New Lows', group = 'RS Line New High')
rsNewLow    = input.string('Historical RS New Lows', title = 'Type', options=['RS New Lows','RS New Lows Before Price', 'Historical RS New Lows', 'Historical RS New Lows  Before Price'], group = 'RS Line New High', inline = 'x')
redDotCol   = input(color.rgb(255, 82, 82, 62), title = 'Color', group = 'RS Line New High', inline = 'x')
lookback2   = input.int(250, title = 'Look-back', minval = 1, maxval = 252, group = 'RS Line New High', tooltip = 'The lookback for calculation of price and RS New Lows.', inline = 'x')
sizeLabLow  = input.string('Tiny', title = 'Size', options = ['Tiny', 'Small', 'Normal', 'Large'], group = 'RS Line New High')
boolMa      = input(false, title = 'Display MA', group = 'MA on RS Line')
lenMa       = input(21, title = 'Lenght Da', group = 'MA on RS Line', inline = 'c')
colMa       = input(color.orange, title = 'Color', group = 'MA on RS Line', inline = 'c')
typMa       = input.string('EMA', title = 'Type Da', options = ['SMA', 'EMA'], group = 'MA on RS Line', inline = 'c')
lenMaWe     = input(10, title = 'Lenght We', group = 'MA on RS Line', inline = 'c')
typMaWe     = input.string('SMA', title = 'Type We', options = ['SMA', 'EMA'], group = 'MA on RS Line', inline = 'c')
fillMa      = input(false, title = 'Area Color', group = 'MA on RS Line')
posCol      = input(color.rgb(0, 230, 119, 75), title = 'Positive Area', group = 'MA on RS Line', inline = 'd')
negCol      = input(color.rgb(255, 82, 82, 75),  title = 'Negative Area', group = 'MA on RS Line', inline = 'd')
allowReplay = input(false, title = 'Use fix values for replay mode', group = 'Replay mode (Approximate Method)', tooltip = 'Here we use constant values in order to provide the environment regardless of the date. See RSRATING ticker and report close values to have the last data.')
first2      = input(195.93, title='For 99 stocks' , group = 'Replay mode (Approximate Method)')
scnd2       = input(117.11, title='For 90+ stocks', group = 'Replay mode (Approximate Method)')
thrd2       = input(99.04, title='For 70+ stocks' , group = 'Replay mode (Approximate Method)')
frth2       = input(91.66, title='For 50+ stocks' , group = 'Replay mode (Approximate Method)')
ffth2       = input(80.96, title='For 30+ stocks' , group = 'Replay mode (Approximate Method)')
sxth2       = input(53.64, title='For 10+ stocks' , group = 'Replay mode (Approximate Method)')
svth2       = input(24.86, title='For 1- stocks'  , group = 'Replay mode (Approximate Method)')

// Blue Dot
// If Blue Dot is ste to 250 Da, than we want it to be set on 52 We on the Weekly TimeFrame
if (lookback  == 250 and timeframe.isweekly)
    lookback  := 52
if (lookback2 == 250 and timeframe.isweekly)
    lookback2 := 52

// Switch Label Size
highLabel = switch sizeLabHigh
    'Normal'  => size.normal
    'Tiny'    => size.tiny
    'Small'   => size.small
    'Large'   => size.large

lowLabel  = switch sizeLabLow
    'Normal'  => size.normal
    'Tiny'    => size.tiny
    'Small'   => size.small
    'Large'   => size.large

// Using bar index in case of IPO to avoid NaN results
// Added max_bars_max = 253 to improve display speed
n63      = bar_index < 63  ? bar_index:63 
n126     = bar_index < 126 ? bar_index:126
n189     = bar_index < 189 ? bar_index:189
n252     = bar_index < 252 ? bar_index:252


// Comparative Ticker for RS Line
comparativeSymbol   = request.security(comparativeTickerId, timeframe.period, close)
// RS Line but multiplied by a little bit less than the constant value of the comparative ticker for correct display
rsCurve             = (src/comparativeSymbol)
// Adapt Offset for low ADR grapghs like indices and sectors
// Adapt Ratio for Sectors and Indices
if (syminfo.industry == 'Investment Trusts/Mutual Funds')
    offset := 90
// We use a wider offset for Weekly timeframe for a smoother display
rsRatio             = timeframe.isweekly ? SpxValue*(offset-10)/100:SpxValue*offset/100
rs                  = rsCurve*rsRatio
prevlookback  = lookback
prevlookback2 = lookback2 // For RS New Lows
lookback := math.min(lookback - 1, bar_index)
rsPlot = plot(rs, title='RS Line', style=plot.style_line, linewidth=1, color=colorRS)

// MA on RS Line
// SMA and EMA
rsMA      = ta.sma(rs, lenMa)
if (typMa == 'SMA' and not timeframe.isweekly)
    rsMA      := ta.sma(rs, lenMa)
if (typMa == 'EMA' and not timeframe.isweekly)  
    rsMA      := ta.ema(rs, lenMa)
if (typMaWe == 'SMA' and timeframe.isweekly)
    rsMA      := ta.sma(rs, lenMaWe)
if (typMaWe == 'EMA' and timeframe.isweekly)
    rsMA      := ta.ema(rs, lenMaWe)

maPlot    = plot(boolMa ? rsMA :na,    color = colMa, linewidth = 1)

// Color Filling
// I will use an invisible MA to be able to choose or not the display of the fill
maPlot2    = plot(boolMa and fillMa ? rsMA:na,    color = color.rgb(0,0,0,100), linewidth = 1)
// This variable gets the color that will be used for the fill
fillCol = rs > rsMA ? posCol:negCol
// Here if a MA is missing, there is no fill
fill(rsPlot, maPlot2 ,   color=fillCol)



// Historical New Highs & New Highs Before Price
var label newHigh = na
histNH = ta.highest(rs  , prevlookback)
histCl = ta.highest(high, prevlookback)
// Historical RS New High
if (rsNewHigh == 'Historical RS New Highs' and plotNewHigh and rs == histNH)
    newHigh := label.new(x = bar_index, y = rs, color = blueDotCol, style = label.style_circle, size = highLabel)
// Historical RS New High Before Price
if (rsNewHigh == 'Historical RS New Highs  Before Price' and plotNewHigh and rs == histNH and high < histCl)
    newHigh := label.new(x = bar_index, y = rs, color = blueDotCol, style = label.style_circle, size = highLabel)
// RS New High
if (barstate.islast and rsNewHigh == 'RS New Highs' and plotNewHigh and rs == histNH)
    label.delete(newHigh)
    newHigh := label.new(x = bar_index, y = rs, color = blueDotCol, style = label.style_circle, size = highLabel)
// RS New High Before Price
if (barstate.islast and rsNewHigh == 'RS New Highs Before Price' and plotNewHigh and rs == histNH and high < histCl)
    label.delete(newHigh)
    newHigh := label.new(x = bar_index, y = rs, color = blueDotCol, style = label.style_circle, size = highLabel)


// Historical New Lows & New Lows Before Price
var label newLow  = na
histNL  = ta.lowest(rs , prevlookback2)
histClL = ta.lowest(low, prevlookback2)
// Historical RS New Low
if (rsNewLow == 'Historical RS New Lows' and plotNewLow and rs == histNL)
    newLow := label.new(x = bar_index, y = rs, color = redDotCol, style = label.style_circle, size = lowLabel)
// Historical RS New Low Before Price
if (rsNewLow == 'Historical RS New Lows  Before Price' and plotNewLow and rs == histNL and low > histClL)
    newLow := label.new(x = bar_index, y = rs, color = redDotCol, style = label.style_circle, size = lowLabel)
// RS New Low
if (barstate.islast and rsNewLow == 'RS New Lows' and plotNewLow and rs == histNL)
    label.delete(newLow)
    newLow := label.new(x = bar_index, y = rs, color = redDotCol, style = label.style_circle, size = lowLabel)
// RS New Low Before Price
if (barstate.islast and rsNewLow == 'RS New Lows Before Price' and plotNewLow and rs == histNL and low > histClL)
    label.delete(newLow)
    newLow := label.new(x = bar_index, y = rs, color = redDotCol, style = label.style_circle, size = lowLabel)



// Calculation of the RS Rating
// Getting ticker and reference ticker daily data
closeDa    = request.security(syminfo.tickerid,    'D', close)
spxCloseDa = request.security(comparativeTickerId, 'D', close)

// Calculation of the performance from 1 to 4 last quarters
// Ticker
perfTicker63   = closeDa/closeDa[n63]
perfTicker126  = closeDa/closeDa[n126]
perfTicker189  = closeDa/closeDa[n189]
perfTicker252  = closeDa/closeDa[n252]

// SP500 of reference ticker
perfComp63     = spxCloseDa/spxCloseDa[n63]
perfComp126    = spxCloseDa/spxCloseDa[n126]
perfComp189    = spxCloseDa/spxCloseDa[n189]
perfComp252    = spxCloseDa/spxCloseDa[n252]

// Using Formula to calculate a relative score of the ticker and the SP500 with the last quarter weighted double
float rs_stock = 0.4*perfTicker63 + 0.2*perfTicker126 + 0.2*perfTicker189 + 0.2*perfTicker252
float rs_ref   = 0.4*perfComp63   + 0.2*perfComp126   + 0.2*perfComp189   + 0.2*perfComp252

// Calculation of the total relative score or rs performance
float totalRsScore  = (rs_stock) / (rs_ref) * 100
float totalRsRating = -1

// Here we calculated the relative score of the stock. The goal is now to assign the percentile correctly
// For this I took the curve given by my fork repo of Skyte on Rs Log and tried to calibrate the better possible
// the output curve of the relative performance of the 6,6xx stocks.
// Link: https://github.com/Fred6725/rs-log/blob/main/output/rs_stocks.csv
// Here is the curve in ASCII Art; on the x-axis, the Rs Rating and on the y-axis, the calculated performance.
      
//                      
//                                                                                        /                               
//                                                                                        /                               
//                                                                                        /                               
//                                                                                        /                               
//,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,/,,,,,,,,,,,,,,,,,              
//                                                                                        /                               
//                                                                                        /                               
//                                                                                        /                               
//                                                                                        /                               
//                                                                                       |                               
//                                                                                       /                               
//                                                                                      ‾                                 
//                                                                                     ‾                                   
//                                                                                   -‾                                    
//                                                                           _____, ‾                                     
//                                         _____----------------‾‾‾‾‾‾‾‾‾‾‾‾‾                                                   
//                        __ */‾‾‾‾‾‾‾‾‾‾‾‾                                                                                       
//             __ ,,----‾‾                                                                                                 
//          _/                                                                                                           
//        /                                                                                                               
//       |                                                                                                     
// ______|________________ _______________________________ _____________________________________            
//       |0               |20             |40             |60             |80              |100   
//
// I decided to cut this curve in 7 different levels that needs to be entered each day.
// These are relative strength scores corresponding to percentiles 98, 89, 69, 49, 29, 9 and 1.
// Finally I used the request.seed() function to auto update these levels automatically on a daily basis.
// Everything is managed in this repo if you're curious:
// https://github.com/Fred6725/relative-strength/tree/main    (Fork from Skyte)
// More precisly in rs_ranking.py for extracting what I needed and in workflows/output.yml for the auto update.
// The update is done in the private fork of the seed tradingview original repo, checked and synchronised automatically
// I tried to uplad the full 6,6xx list of relative strength score and rs rating but the display speed was too long.


// Use the request.seed() function to access the RS Score environment of all the market
curveRsPerf  = request.seed('seed_fred6725_rs_rating', 'RSRATING', close)

// To prevent loosing data because of week-ends and public holidays I decided to send the value 5 times in a row.
// Which gives 5*7 = 35 bars. (seed_fred6725_rs_rating:rsrating)
// Depending of the day we look at the graph we will have a variable amount of bars. 
// The goal is to get these 7 numbers anyway.

// In case the graph is not updated, we count the number of bars since we have the first data.
// Calculation of the number of bar since we have the first data
delta  = ta.barssince(na(curveRsPerf)!=true)

// Table to store the different values
var float[] different_values = array.new_float(7)

// Counter for stored values
var int counter = 0

// Variable for storage of the environment
float first = 0
float scnd  = 0
float thrd  = 0
float frth  = 0
float ffth  = 0
float sxth  = 0
float svth  = 0


// Browse seed's values and store the first 7 different values
if (not allowReplay)
    for i = delta to 34+delta
        close_value = nz(curveRsPerf[i])
        if (not array.includes(different_values, close_value) and counter < 7 and close_value!=0)
            array.set(different_values, counter, close_value)
            counter := counter + 1

    // Assign stored values to variables
    first := array.get(different_values, 0)
    scnd  := array.get(different_values, 1)
    thrd  := array.get(different_values, 2)
    frth  := array.get(different_values, 3)
    ffth  := array.get(different_values, 4)
    sxth  := array.get(different_values, 5)
    svth  := array.get(different_values, 6)

// Replay mode
if (allowReplay)
    first := first2
    scnd  := scnd2 
    thrd  := thrd2 
    frth  := frth2 
    ffth  := ffth2 
    sxth  := sxth2 
    svth  := svth2 

// Now that we've recovered the environment, we can assign a percentile using a simple linear approximation of the curve (+ adjustment).
if(totalRsScore >= first)
    totalRsRating := 99
if(totalRsScore <= svth)
    totalRsRating := 1

// Function to attribute the percentile with a simple linear approximation
f_attributePercentile(totalRsScore, tallerPerf, smallerPerf, rangeUp, rangeDn, weight) =>
    sum = totalRsScore + (totalRsScore-smallerPerf)*weight // weight is used for manual calibration
    if(sum > tallerPerf - 1)
        sum := tallerPerf - 1
    k1 = smallerPerf/rangeDn
    k2 = (tallerPerf-1)/rangeUp
    k3 = (k1-k2)/(tallerPerf-1-smallerPerf)
    RsRating = sum/(k1-k3*(totalRsScore-smallerPerf))
    if (RsRating > rangeUp)
        RsRating := rangeUp
    if (RsRating < rangeDn)
        RsRating := rangeDn
    RsRating

// Between 199 & 120 the score where approx 98 to 90.
if (totalRsScore < first and totalRsScore >= scnd)
    totalRsRating := f_attributePercentile(totalRsScore, first, scnd, 98, 90, 0.33)
// Between 119 and 100 we have scores between 89 and 70.
if (totalRsScore < scnd and totalRsScore >= thrd)
    totalRsRating := f_attributePercentile(totalRsScore, scnd, thrd, 89, 70, 2.1)
// Between 100 and 91 we have scores between 69 and 50.
if (totalRsScore < thrd and totalRsScore >= frth)
    totalRsRating := f_attributePercentile(totalRsScore, thrd, frth, 69, 50, 0)
// Between 90 and 82 we have scores between 49 and 30.
if (totalRsScore < frth and totalRsScore >= ffth)
    totalRsRating := f_attributePercentile(totalRsScore, frth, ffth, 49, 30, 0)
// Between 81 and 56 we have scores between 29 and 10.
if (totalRsScore < ffth and totalRsScore >= sxth)
    totalRsRating := f_attributePercentile(totalRsScore, ffth, sxth, 29, 10, 0)
// Between 55 and 28 we have scores between 9 and 2.
if (totalRsScore < sxth and totalRsScore >= svth)
    totalRsRating := f_attributePercentile(totalRsScore, sxth, svth, 9, 2, 0)

// Check if one of this value is empty for replay mode
for i = 0 to 6
    if (nz(array.get(different_values, i)) == 0 and not allowReplay)
        totalRsRating := -1

// Display the RS Rating
// The results can only be used in Daily TimeFrame
isDaily = timeframe.isdaily
labelText1 = '                RS Rating'
labelText2 = ''
// Here we want to display 'RS' without value if one of the constants is missing
if (isDaily and totalRsRating != -1)
    labelText2 := '\n\n       '+str.tostring(totalRsRating,'#0')
// Rating Only
if (ratingOnly)
    labelText1 := ''
    labelText2 := '\n    '+str.tostring(totalRsRating,'#0')
// Display the labels
label1 = (hideRSRat == false) and barstate.islast ? label.new(bar_index, rs, text=labelText1 , color = color.rgb(0,0,0,100), size=size.normal, textcolor=colorRS, style=label.style_label_center, textalign=text.align_left, yloc=yloc.price) : na
label2 = (hideRSRat == false) and barstate.islast ? label.new(bar_index, rs, text=labelText2 , color = color.rgb(0,0,0,100), size=size.large,  textcolor=colorRS, style=label.style_label_center, textalign=text.align_left, yloc=yloc.price) : na

// Delete previous Labels (When new candle opens or when replay mode, the labels were piling on)
label.delete(label1[1])
label.delete(label2[1])




//SMA
out1 = ta.sma(close, 20)
plot(out1, title="SMA #1", color =color.aqua)

out2 = ta.sma(close, 50)
plot(out2, title="SMA #2", color= color.orange)

out3 = ta.sma(close, 150)
plot(out3, title="SMA #4", color= #d400ff)

out4 = ta.sma(close, 200)
plot(out4, title="SMA #5", color= color.red)
