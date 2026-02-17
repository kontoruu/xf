# Indodax Market Analyzer - Script Documentation

## Overview
This is a **Cloudflare Workers** application that functions as a **real-time crypto market analyzer** for Indodax (Indonesian cryptocurrency exchange). The script fetches market data, analyzes trading signals using technical indicators, and displays results in a professional stock exchange-style web interface.

## Main Components

### 1. **Cloudflare Workers Event Handler**
```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});
```
- Runs on Cloudflare edge (not traditional server)
- Handles every incoming HTTP request
- Responds in real-time with low latency

### 2. **Request Router (`handleRequest`)**
The script has 3 main endpoints:

#### a. `/api/market` - Complete Market Data
- Fetches data for all trading pairs from Indodax
- Calculates trading signals for 10 different timeframes
- Returns top gainers, top losers, and highest volume
- Response format: JSON

#### b. `/api/ticker/{pair}` - Trading History
- Fetches last 100 trades for a specific pair
- Example: `/api/ticker/btc_idr`
- Response format: JSON

#### c. `/` (root) - Web Interface
- Displays interactive HTML dashboard
- Real-time market data with auto-refresh
- Professional stock exchange-style display

### 3. **Market Analysis Function (`getMarketData`)**

#### Process:
1. **Fetch Data from Indodax API**
   ```
   https://indodax.com/api/summaries
   ```

2. **Data Normalization**
   - Convert all volumes to IDR
   - Handle USDT pairs with USDT/IDR rate
   - Calculate price change percentage

3. **Trading Signal Analysis**
   - Analysis for 10 timeframes: 15m, 30m, 1h, 2h, 4h, 1d, 3d, 1w, 2w, 1m
   - Each ticker gets signals for all timeframes
   - Uses `analyzeSignalAdvanced()` function

4. **Categorization**
   - **Top Gainers**: Coins with highest gains
   - **Top Losers**: Coins with biggest drops
   - **Top Volume**: Coins with highest trading volume

5. **Response Data Structure**
   ```json
   {
     "success": true,
     "timestamp": "2026-02-02T12:28:53.310Z",
     "usdtIdrRate": 15000,
     "stats": {
       "totalPairs": 150,
       "totalVolume": 1000000000,
       "activeMarkets": 120,
       "totalGainers": 80,
       "totalLosers": 40,
       "totalVolumeAssets": 150
     },
     "tickers": [...],
     "topGainers": [...],
     "topLosers": [...],
     "topVolume": [...]
   }
   ```

### 4. **Technical Signal Analysis (`analyzeSignalAdvanced`)**

This function is the heart of the analysis system. It uses several technical indicators:

#### A. **Timeframe Configuration**
Each timeframe has different sensitivity:
- **15m-30m**: High sensitivity (1.6-1.8) - for scalpers
- **1h-4h**: Medium sensitivity (1.0-1.4) - for swing trading
- **1d-1m**: Low sensitivity (0.3-0.8) - for long-term investors

#### B. **Technical Indicators**

1. **RSI Proxy (Relative Strength Index)**
   - Calculated from price momentum
   - Range: 0-100
   - Oversold: < 30 (BUY signal)
   - Overbought: > 70 (SELL signal)
   
2. **Stochastic RSI**
   - Based on price position relative to high-low range
   - Range: 0-100
   - Oversold: < 20
   - Overbought: > 80

3. **Bollinger Bands Position**
   - Calculates price position relative to upper/lower bands
   - Signals:
     - `OVERSOLD`: Near lower band (buy signal)
     - `OVERBOUGHT`: Near upper band (sell signal)
     - `ABOVE_MID`: Above mid band
     - `BELOW_MID`: Below mid band

4. **Volume Ratio**
   - Compares current volume to average
   - Ratio > 3: Volume spike 🔥
   - Ratio > 2: High volume 📈
   - Ratio > 1.5: Above average ✅

5. **Momentum**
   - Calculated from price change % * sensitivity
   - High momentum: strong price movement
   - Negative momentum: selling pressure

6. **Order Book Pressure**
   - Analyzes bid-ask spread
   - `BUY_PRESSURE`: Negative spread (buy > sell)
   - `SELL_PRESSURE`: Positive spread (sell > buy)

#### C. **Scoring System**
Script assigns scores 0-100 based on:
- RSI oversold/overbought: +25 points
- Extreme StochRSI: +20 points
- Bollinger bands position: +25 points
- Volume spike: +15 points
- Strong momentum: +15 points

#### D. **Trading Recommendations**
Based on score and type:
- **BUY NOW**: Score ≥ 90, very strong buy signal
- **BUY STRONG**: Score ≥ 70, strong buy signal
- **SELL NOW**: Score ≥ 90, very strong sell signal
- **SELL STRONG**: Score ≥ 70, strong sell signal
- **HOLD**: Score 40-70, wait for confirmation
- **HOLD SIDEWAY**: Score < 40, sideways market

#### E. **Signal Output**
```json
{
  "type": "BUY",
  "timeframe": "1 Hour",
  "score": 85,
  "recommendation": "BUY STRONG",
  "signals": [
    "📊 RSI Oversold",
    "🎯 StochRSI Oversold",
    "📉 BB: Near Lower",
    "🔥 Vol Spike",
    "✨ STRONG BUY"
  ],
  "indicators": {
    "rsi": "28.5",
    "stochRSI": "15.2",
    "bbPosition": "OVERSOLD",
    "momentum": "5.43",
    "volumeRatio": "3.25",
    "volatility": "8.76",
    "pricePosition": "15",
    "orderPressure": "BUY_PRESSURE"
  }
}
```

### 5. **Web Interface (UI)**

The script generates complete HTML with features:

#### A. **Styling**
- Dark theme for comfortable viewing
- Gradient backgrounds
- Responsive design
- Consistent font sizes (CSS variables)

#### B. **Real-Time Animations**
- **Flash animations**: Green for up, red for down
- **Pulse effects**: For volume spikes
- **Live badge**: Blinking animation for live status

#### C. **Dashboard Components**
1. **Header**
   - Application title
   - Live badge
   - Market statistics
   - USDT/IDR rate

2. **Controls**
   - Timeframe selector (15m - 1m)
   - Auto-refresh toggle
   - Refresh interval (5-60 seconds)
   - Search/filter
   - Manual refresh button

3. **Tabs Navigation**
   - All Markets: All pairs
   - Top Gainers: Highest gains
   - Top Losers: Biggest drops
   - Top Volume: Highest volume

4. **Market Table**
   Columns:
   - Pair (trading pair name)
   - Last Price
   - 24h Change
   - High/Low
   - Volume (trading volume in IDR)
   - Signal (trading signals with emojis)
   - Score (analysis score)
   - Action (detail button)

#### D. **Interactivity**
- Auto-refresh data every X seconds
- Click for ticker details
- Modal popup for charts and deep analysis
- Real-time price updates with flash animations
- Sorting and filtering

### 6. **Front-End JavaScript Logic**

The HTML script includes JavaScript for:

1. **Data Fetching**
   ```javascript
   async function loadMarketData() {
     const response = await fetch('/api/market');
     const data = await response.json();
     // Process and render
   }
   ```

2. **Auto-Refresh**
   - SetInterval for automatic updates
   - Detects price changes for flash animations
   - Caches previous data for comparison

3. **Rendering**
   - Dynamic table rendering
   - Color coding (green for up, red for down)
   - Number formatting (IDR currency, percentages)

4. **Modal Details**
   - Shows complete analysis per ticker
   - Chart history (optional)
   - All timeframe signals
   - Detailed technical indicators

## Technologies Used

1. **Cloudflare Workers**
   - Serverless edge computing
   - Global distribution
   - Auto-scaling

2. **Indodax API**
   - Public REST API
   - Real-time market data
   - No authentication required

3. **Vanilla JavaScript**
   - No framework dependencies
   - Pure DOM manipulation
   - Fetch API for AJAX

4. **CSS3**
   - Modern layouts (Flexbox, Grid)
   - Animations and transitions
   - CSS variables for theming

## Use Cases

### 1. **Day Traders**
- Monitor real-time prices
- Buy/sell signals for short timeframes (15m, 30m, 1h)
- Volume spike alerts

### 2. **Swing Traders**
- Medium timeframe analysis (4h, 1d)
- Multi-timeframe analysis
- Trend confirmation

### 3. **Long-term Investors**
- Long timeframes (1w, 2w, 1m)
- Fundamental trends
- Accumulation zones

### 4. **Market Researchers**
- Market statistics
- Volume analysis
- Gainers/losers tracking

## Script Advantages

1. ✅ **Real-time**: Always fresh data from Indodax
2. ✅ **Multi-timeframe**: 10 different timeframes
3. ✅ **Comprehensive Signals**: Multiple technical indicators
4. ✅ **Professional UI**: Stock exchange style
5. ✅ **Fast**: Cloudflare edge network
6. ✅ **Free**: No API key required
7. ✅ **Responsive**: Works on mobile & desktop
8. ✅ **No Database**: Stateless, fully real-time

## Limitations

1. ⚠️ **Data Source**: Depends on Indodax API availability
2. ⚠️ **Historical Data**: No stored historical data
3. ⚠️ **Technical Analysis Only**: No fundamental analysis included
4. ⚠️ **Proxy Indicators**: RSI and other indicators are approximations, not standard calculations
5. ⚠️ **No Trading**: Analysis only, cannot execute orders

## Deployment

This script is designed for Cloudflare Workers:

1. Create a Cloudflare Workers account
2. Copy the `gg` script to Workers editor
3. Deploy
4. Access via your Workers URL

## Security & Performance

1. **CORS Enabled**: API accessible from other domains
2. **No Cache**: Data always fresh
3. **Error Handling**: Try-catch for all API calls
4. **Rate Limiting**: Should be implemented for high traffic
5. **No Secrets**: No sensitive data in script

## File Structure

```
/xf
├── gg              # Main Cloudflare Workers script (JavaScript)
├── PENJELASAN.md   # Indonesian documentation
└── README.md       # English documentation (this file)
```

## API Endpoints Reference

### GET `/api/market`
Returns complete market analysis with signals for all pairs and timeframes.

**Response:**
```json
{
  "success": true,
  "timestamp": "ISO8601",
  "usdtIdrRate": number,
  "stats": { ... },
  "tickers": [ ... ],
  "topGainers": [ ... ],
  "topLosers": [ ... ],
  "topVolume": [ ... ]
}
```

### GET `/api/ticker/{pair}`
Returns last 100 trades for a specific pair.

**Parameters:**
- `pair`: Trading pair (e.g., "btc_idr", "eth_idr")

**Response:**
```json
{
  "success": true,
  "pair": "btc_idr",
  "trades": [ ... ]
}
```

### GET `/`
Returns the HTML dashboard interface.

## Conclusion

This script is a **complete market analyzer** that:
- Fetches real-time data from Indodax
- Analyzes with multiple technical indicators
- Provides buy/sell signals for 10 timeframes
- Displays in a professional stock exchange-style UI
- Runs on Cloudflare edge network for optimal performance

Perfect for crypto traders who want to monitor Indodax market with automated technical analysis and professional display.
