# WeatherBot dashboard

Dashboard: https://corentinlaudicina.github.io/weatherbot-dashboard/

This is a paper trading account. The money is simulated and no orders reach a real exchange. The page is
rebuilt every hour, and also whenever a new weather model run comes in and the bot trades on it.

## The markets

Polymarket runs a market each day on the highest temperature in a number of cities. The bot trades eight
European ones: Amsterdam, London, Paris, Milan, Madrid, Warsaw, Munich and Moscow.

A market is split into one-degree brackets (for example "14°C", "15°C", "16°C or higher"). A Yes share of a
bracket pays $1 if the day's official high falls in it and $0 if not, so its price works as the market's
probability for that bracket. The official high comes from the routine METAR reports of the city's airport
weather station.

## How the bot decides

**1. Its own forecast.** A gradient-boosted model gives a probability for each whole degree of the day's high.
Its inputs are the latest runs of about ten weather models (ECMWF IFS and AIFS, NOAA GFS and GEFS, DWD ICON,
Météo-France AROME and others) that were published at that moment, how wrong each model has been recently,
the highest temperature already observed that day, the time of year and the city. One model is trained on
all eight cities together, which works better than one model per city.

**2. Blended with the market.** The market price carries information the model doesn't have, so the bot
combines both:

    p = p_market^a × p_model^b, rescaled so the brackets sum to 1

`a` and `b` are fitted on past markets, separately for 0-12, 12-24 and 24-48 hours before 15:00 local time on
the target day. With the current fit the market gets the larger weight (a ≈ 0.9, b ≈ 0.3).

**3. A bet only with an edge.** For every bracket the bot looks at both sides, Yes and No. It buys a side when
the blended probability is at least 5 cents above the price after Polymarket's taker fee. The order is a
limit order at the highest price that still leaves that 5 cent edge, and expires after an hour if not filled.
It also needs:

- the target day's 15:00 local to be 12 to 48 hours away;
- the market to give that side at least a 15% chance (long shots are skipped).

**4. Size.** The bets on one day's market are sized together with the Kelly criterion, scaled down to 2% of a
full Kelly bet on the account value.

**5. Hold.** Positions are held until the market settles. There is no early exit.

## Reading the dashboard

- **Two pages:** [daily high](index.html) and [daily low](low.html) markets, toggled at the top. Both show
  the same paper account; each lists only the positions in its own markets.
- **Top line:** account value, profit or loss with open positions valued at the mid price, and cash.
- **Open positions:** one panel per market held. The curves are the market (orange), the GBM forecast alone
  (green) and the model that was traded, the blend of the two (blue), across temperatures. Solid lines are
  now, dashed lines are at entry. The shaded bracket is the one held. On a day that has started, a vertical
  line marks the highest temperature reported so far by the station (the lowest, on the daily-low page).
- **Price move after entry (CLV):** the price of the side bought, 6 and 24 hours after the fill, minus the
  price paid. If it is positive on average, the market tends to move toward the bot's view after it buys.
- **Closed positions:** settled trades, with results by city and by how far ahead they were placed.

## How this page is updated

The bot runs on a laptop. After each update it copies the two pages to `index.html` and `low.html` here and pushes them.
Only the latest version is kept in the history of this repository.
