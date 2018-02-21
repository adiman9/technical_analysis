# Technical Analysis

This project aims to provide robust and extensible tools for better understanding market behavior and gaining insights on future trends. It has three pillars of focus:

1. Collecting data (historic and real-time) relevant to market behavior (e.g. GDAX candles, tweets, etc.)
2. Training plugins using historic data
3. Inferring future market behavior using live data and knowledge gained through training


## Installing Dependencies

First, ensure that [Mongo](https://www.mongodb.com/), [Node](https://nodejs.org/en/) and [Yarn](https://github.com/yarnpkg/yarn) are installed. Then, run the following from the project's root:

```shell
yarn install
```


## Configuring the Environment

This repository's scripts use [dotenv](https://www.npmjs.com/package/dotenv) to manage environment variables. All variables documented in [`.env.template`](./.env.template) must be defined either in a `.env` file (located at the project root) or configured on the system.

__Note: Currently there are no required environment variables, but if future plugins require API keys, this is where they should live.__

In addition to any environment variables, there are a handful of required shared settings defined in [`./settings.js`](./settings.js), notably `mongoUri`, `mongoDatabaseName`, and `priceHistoryDir`. The default values are sensible, but should be reviewed.


## Collecting Historic Data

Most/all plugins will require historic data for training. Listed below are the strategies currently available for collecting data:

### Historic GDAX Candles

To collect GDAX's historic candle data for the product, timeframe, and candle size indicated in [`./settings.js`](./settings.js), run the following from the project root:

```shell
yarn run collect-gdax-price-history
```

### Historic Tweets

_Coming Soon..._


## Training With Historic Data

Once the necessary data has been collected, configure the desired training plugins (described below) in [`./settings.js`](./settings.js), then run the `train` script from the project root:

```shell
yarn run train
```

### Available Training Plugins

#### Bullish Engulfing Trainer

The Bullish Engulfing Trainer identifies historic instances of bullish engulfing candles and calculates the probability of various percent-price increases in the near-term future. For this exercise, a bullish engulfing candle is defined as having:

1. a higher closing price than opening price
2. a taller candle body than all recent candles
3. a closing price that is at or near its highest price
4. larger volume than all recent candles

The plugin requires the following to be configured:

- __product__: The trading pair to be analyzed, e.g. `BTC-USD`
- __priceHistoryFile__: The absolute path to a historic GDAX candle file generated by the script mentioned in the "Historic GDAX Candles" section
- __dbCollection__: The mongo collection into which the results will be inserted
- __lookbackCandles__: The number of previous candles that the current candle needs to be larger than in order to be considered "engulfing"
- __lookaheadCandles__: The maximum number of candles to look ahead when determining the highs following a bullish engulfing candle
- __allowedWickToBodyRatio__: The maximum body-to-upper-wick ratio that is allowed for a candle to be considered engulfing
- __groupSizeForPctPriceIncreaseProbability__: The grouping size applied when calculating the probability of the price increasing by a particular percentage


#### Twitter Sentiment Trainer

_Coming Soon..._


## Inferring Based On Live Data

Once the necessary training has been completed (see "Training With Historic Data" section), configure the desired inference plugins (described below) in [`./settings.js`](./settings.js), then run the `infer` script from the project root:

```shell
yarn run infer
```

### Available Inference Plugins

#### Bulllish Engulfing Inferrer

When the current candle meets the criteria defined in the "Bullish Engulfing Trainer" section, the Bullish Engulfing Inferrer emits a `BullishEngulfingInferrer.CURRENT_CANDLE_IS_BULLISH_ENGULFING` event, which contains the plugin settings, data about the most recent candle, along with probabilities of various percentage-price increases over the next `lookaheadCandles` candles. The plugin requires the following to be configured:

- __product__: The trading pair to be analyzed, e.g. `BTC-USD`
- __dbCollection__: The mongo collection where Bullish Engulfing Training data is stored
- __lookbackCandles__: The number of previous candles that the current candle needs to be larger than in order to be considered "engulfing"
- __lookaheadCandles__: The maximum number of candles to look ahead when determining the highs following a bullish engulfing candle (used to identify the appropriate training data)
- __allowedWickToBodyRatio__: The maximum body-to-upper-wick ratio that is allowed for a candle to be considered engulfing


##### Example Usage (see [./bin/infer.js](./bin/infer.js))

```javascript
const inferrer = new QuantInferrer(mongoUri, mongoDatabaseName, collectorConfigs, pluginConfigs);

inferrer.on(BullishEngulfingInferrer.events.CURRENT_CANDLE_IS_BULLISH_ENGULFING, (pluginConfig, candle, probabilities) => {
  console.log(`CURRENT_CANDLE_IS_BULLISH_ENGULFING at ${ formatTime() }`);
  console.log(`...${ pluginConfig.product }: ${ formatUSD(candle.close) }`);
  probabilities.forEach(({ probability, pctPriceChange }) => {
    if (probability >= 0.8 && pctPriceChange > 0) {
      console.log(`......${ formatPercent(probability) } likelihood of increasing by ${ formatPercent(pctPriceChange) } over the next ${ pluginConfig.lookaheadCandles } candles`);
    }
  });
});

inferrer.run();
```

#### Twitter Sentiment Inferrer

_Coming Soon..._


## Registering A New Collector

_Coming Soon..._

## Registering A New Plugin Set

_Coming Soon..._
