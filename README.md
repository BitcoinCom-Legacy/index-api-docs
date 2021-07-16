
## Overview

The Bitcoin.com Composite Price Index (BCX) is a daily historical price index that tracks the value of Bitcoin in United States Dollars. A real-time spot rate is published as well. The price displayed in the top menu of Bitcoin.com is the BCX Spot Rate. Historical values are charted here.

The BCX is a composite of multiple Bitcoin indices, providing a robust measurement of Bitcoin's value. Downtime or API changes on any one exchange or constituent index will not drastically alter the quality of the BCX. Constituent indices include:

- [Brave New Coin Bitcoin Liquid Index](https://bravenewcoin.com/bitcoin)
- [CoinDesk Bitcoin Price Index](http://www.coindesk.com/price/bitcoin-price-index/)
- [TradeBlock XBX](https://tradeblock.com/markets/index/)

When the constituent indices publish multiple timeframes, the daily value is chosen for daily BCX construction. The Spot Rate uses the latest data available for each index.

## Methodology

The value of the BCX is a quadratic time-weighted average of the most recent data point of each constituent index. For the daily BCX, the weight is as follows:

```
weight[i] = ((timestamp[i] - previous_midnight_timestamp) / 86400)^2
```

Quadratic time-weighting places more emphasis on recent data points. For instance, a data point at 1200 hours would only receive `0.25` weight.

## Index Value Format

All price values are denominated in US Dollar cents, so divide by `100` to convert to dollars. A return value of `price: 100000` corresponds to an index value of \$1,000.00.

## API Endpoints

The base URI for all API calls is `https://index-api.bitcoin.com/api`

| Endpoint                                             |              Resource |
| ---------------------------------------------------- | --------------------: |
| [Bitcoin Cash (BCH)](#bitcoin-cash-bch-endpoints)    |            `/v0/cash` |
| [Bitcoin (BTC)](#bitcoin-core-btc-endpoints)         |            `/v0/core` |
| [Server Time](#server-time)                          |            `/v0/time` |
| [BCX Historic Value](#historic-index-value)          |         `/v0/history` |
| [BCX Date/Time Value Lookup](#historic-price-lookup) |          `/v0/lookup` |
| [BCX Spot Rate](#spot-rate)                          | `/v0/price/:currency` |

<br>

## Bitcoin Cash (BCH) endpoints

`/v0/history`, `/v0/lookup`, and `/v0/price` endpoints for Bitcoin Cash
Prices are volume weighted and sourced from the [cryptowat.ch](https://cryptowat.ch/docs/api) API

## Bitcoin (BTC) endpoints

`/v0/history`, `/v0/lookup`, and `/v0/price` endpoints for Bitcoin
Prices are volume weighted and sourced from the [cryptowat.ch](https://cryptowat.ch/docs/api) API

## Server Time

The current time on the server:

```
GET https://index-api.bitcoin.com/api/v0/time

{
    "unix": 1483228800,
    "iso": "2017-01-01T00:00:00.000Z"
}
```

## Historic Index Values

The daily values of the price index. The return value is an array of `[time, price]` entries.

```
GET https://index-api.bitcoin.com/api/v0/history

[
    [
        "2017-01-01T00:00:00.000Z",
        96760
    ],
    [
        "2016-12-31T00:00:00.000Z",
        95917
    ],
    [
        "2016-12-30T00:00:00.000Z",
        96664
    ],
    ...
]
```

### HISTORIC TIMEFRAME

By default, the `/v0/history` endpoint returns the past 6 months of data. Include query parameter `?span=all` to return data back to the first day of Bitcoin trading, July 18, 2010.

```
GET https://index-api.bitcoin.com/api/v0/history?span=all

[
    [
        "2017-01-01T00:00:00.000Z",
        96760
    ],
    [
        "2016-12-31T00:00:00.000Z",
        95917
    ],
    ...
    [
        "2010-07-19T00:00:00.000Z",
        8
    ],
    [
        "2010-07-18T00:00:00.000Z",
        5
    ]
]
```

### TIMESTAMP FORMAT

The default time format is `ISO 8601`, but the server will return unix timestamps in seconds [since the epoch](https://en.wikipedia.org/wiki/Unix_time) with the query parameter `?unix=1`.

```
GET https://index-api.bitcoin.com/api/v0/history?unix=1

[
    [
        1483228800,
        96760
    ],
    [
        1483142400,
        95917
    ],
    [
        1483056000,
        96664
    ],
    ...
]
```

## Historic Price Lookup

To return the value of the index on a given date & time, use `/v0/lookup`. If a non-midnight time is specified, the value is interpolated between the open and close price for the day. Specify the `?time=<unix or ISO 8601>` query parameter. See price lookup in action on the [Tools Page](https://tools.bitcoin.com/).

```
GET https://index-api.bitcoin.com/api/v0/lookup?time=2017-01-01T06:00:00Z
- or -
GET https://index-api.bitcoin.com/api/v0/lookup?time=1483250400

{
    "open": {
        "price": 96760,
        "time": {
            "unix": 1483228800,
            "iso": "2017-01-01T00:00:00.000Z"
        }
    },
    "close": {
        "price": 99847,
        "time": {
            "unix": 1483315200,
            "iso": "2017-01-02T00:00:00.000Z"
        }
    },
    "lookup": {
        "price": 97532,
        "k": 0.25,
        "time": {
            "unix": 1483250400,
            "iso": "2017-01-01T06:00:00.000Z"
        }
    }
}
```

The `open` and `close` prices will be for the UTC midnights straddling the query time stamp. The `lookup` object will contain the timestamp of the query, the fraction `k` of one day elapsed between `open.time` and `close.time`, and the price as linearly interpolated between those endpoints:

```
lookup.price = open.price + lookup.k \* (close.price - open.price)
```

## Spot Rate

The current Bitcoin price, updated in real-time, is available at `/v0/price/:currency`. Simple conversion from US Dollars to world currencies takes place behind the scenes with Open Exchange Rates FX values. **Actual Bitcoin exchange rates in non-USD international currencies may vary significantly relative to the reported values from the BCX Spot Rate.**

```
GET https://index-api.bitcoin.com/api/v0/price/usd

{
    "price": 96760,
    "time": {
        "unix": 1483228800,
        "iso": "2017-01-01T00:00:00.000Z"
    }
}
```

The `:currency` URI path should contain a supported [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) currency code:

`AED` `AFN` `ALL` `AMD` `ANG` `AOA` `ARS` `AUD` `AWG` `AZN` `BAM` `BBD` `BDT` `BGN` `BHD` `BIF` `BMD` `BND` `BOB` `BRL` `BSD` `BTN` `BWP` `BYN` `BZD` `CAD` `CDF` `CHF` `CLF` `CLP` `CNH` `CNY` `COP` `CRC` `CUC` `CUP` `CVE` `CZK` `DJF` `DKK` `DOP` `DZD` `EGP` `ERN` `ETB` `EUR` `FJD` `FKP` `GBP` `GEL` `GGP` `GHS` `GIP` `GMD` `GNF` `GTQ` `GYD` `HKD` `HNL` `HRK` `HTG` `HUF` `IDR` `ILS` `IMP` `INR` `IQD` `IRR` `ISK` `JEP` `JMD` `JOD` `JPY` `KES` `KGS` `KHR` `KMF` `KPW` `KRW` `KWD` `KYD` `KZT` `LAK` `LBP` `LKR` `LRD` `LSL` `LYD` `MAD` `MDL` `MGA` `MKD` `MMK` `MNT` `MOP` `MRO` `MUR` `MVR` `MWK` `MXN` `MYR` `MZN` `NAD` `NGN` `NIO` `NOK` `NPR` `NZD` `OMR` `PAB` `PEN` `PGK` `PHP` `PKR` `PLN` `PYG` `QAR` `RON` `RSD` `RUB` `RWF` `SAR` `SBD` `SCR` `SDG` `SEK` `SGD` `SHP` `SLL` `SOS` `SRD` `SSP` `STD` `SVC` `SYP` `SZL` `THB` `TJS` `TMT` `TND` `TOP` `TRY` `TTD` `TWD` `TZS` `UAH` `UGX` `USD` `UYU` `UZS` `VEF` `VND` `VUV` `WST` `XAF` `XAG` `XAU` `XCD` `XDR` `XOF` `XPD` `XPF` `XPT` `YER` `ZAR` `ZMW` `ZWL`

## Revision History

| Date       | Notes                                                       |
| ---------- | ----------------------------------------------------------- |
| 2016/12/23 | Initial launch of BCX daily price index                     |
| 2017/01/06 | Adds BCX Spot Rate API endpoint and server-side computation |
| 2017/02/10 | Adds world currency conversion via Open Exchange Rates      |
| 2017/09/01 | Adds Bitcoin Cash endpoints                                 |
