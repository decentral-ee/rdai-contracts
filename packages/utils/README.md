# @rtoken/utils

> NOTE: This package was formerly named "rtoken-analytics". There is A LOT going on in this one package, so we may break this apart into smaller individual pieces for each.

This library provides tools for getting rDAI and rToken data into your dapp.

## Install

```bash
yarn add rtoken-analytics
```

## Usage

```js
import RTokenAnalytics from 'rtoken-analytics';

const MyComponent = () => {
  const from = "0x9492510bbcb93b6992d8b7bb67888558e12dcac4"
  const to = "0x8605e554111d8ea3295e69addaf8b2abf60d68a3"

  const rTokenAnalytics = new RTokenAnalytics();
  const interestSent = await rTokenAnalytics.getInterestSent(from, to);
}
```

If you are using your own rToken subgraph, provide the information in the constructor.

|    Arguments     | Default value                                    |
| :--------------: | ------------------------------------------------ |
|  `subgraphURL`   | `https://api.thegraph.com/subgraphs/id/`         |
| `rdaiSubgraphId` | `QmfUZ16H2GBxQ4eULAELDJjjVZcZ36TcDkwhoZ9cjF2WNc` |

```js
const options = {
  subgraphURL: 'some other url',
  rdaiSubgraphId: 'some other id'
};
const rTokenAnalytics = new RTokenAnalytics(options);
```

## API

### `getAllOutgoing(address)`

Get all loans where interest is being sent to another address

Returns array of active loans. Example:

```js
[
  {
    amount: '0.50000000058207661',
    hat: {id: '11'},
    recipient: {id: '0x358f6260f1f90cd11a10e251ce16ea526f131b02'}
  },
  {
    amount: '24.49999999941792339'
    // ...
  }
];
```

### `getAllIncoming(address)`

Get all loans where interest is being received from another address

Returns array of active loans (same schema as above)

### `getInterestSent(fromAddress, toAddress)`

Get the total amount of interest sent

Returns: value in DAI

> What other features do you want? Let us know by making an issue.

# Misc. tools

## Get the Compound Interest Rate

This is one method for obtaining the Compound interest rate in your dapp.

```js
import axios from 'axios';

const COMPOUND_URL = 'https://api.compound.finance/api/v2/ctoken?addresses[]=';
const daiCompoundAddress = '0x5d3a536E4D6DbD6114cc1Ead35777bAB948E3643';

const getCompoundRate = async () => {
  const res = await axios.get(`${COMPOUND_URL}${daiCompoundAddress}`);
  const compoundRate = res.data.cToken[0].supply_rate.value;
  const compoundRateFormatted = Math.round(compoundRate * 10000) / 100;

  return {
    compoundRate,
    compoundRateFormatted
  };
};
```

Then use it like this

```js
const {compoundRate, compoundRateFormatted} = await getCompoundRate();

console.log(`Compound Rate: ${compoundRateFormatted}%`);
// > Compound Rate: 4.56%

// Recommend you save the rate for quick reference, as the API can be slow.
if (typeof window !== 'undefined') {
  localStorage.setItem('compoundRate', compoundRate);
}
```

# Contributing

Contributions, suggestions, and issues are welcome. At the moment, there are no strict guidelines to follow.
