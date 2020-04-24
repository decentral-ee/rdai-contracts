<p align="center"><img src="https://rdai.money/images/logo.svg" width="160"/></p>

<p align="center">
    <a href="https://www.npmjs.com/package/@rtoken/contracts">
        <img alt="npm" src="https://img.shields.io/npm/v/@rtoken/contracts">
    </a>
    <img alt="GitHub" src="https://img.shields.io/github/license/rtoken-project/rtoken-contracts">
</p>

# @rtoken/utils

This library provides tools for getting rDAI and rToken data into your dapp.

> NOTE: This package was formerly named `rtoken-analytics`. See the archived repo [here](https://github.com/rtoken-project/rtoken-analytics).

> :warning: Warning: the code in this package is under active development. Please contact the team if you have questions via twitter/discord.

# What does it do?

This library wraps the rToken subgraph, so you can easily gain insight about rToken users, without needing to learn or use GraphQL.

```js
const userDetails = userA.details();
const allUsersOfHat = myHat.allUsers();
```

# Getting started

#### 1. Connect to the Data :raised_hands: :rainbow:

This will be your connection to the rToken subgraph (`../packages/subgraph`), which provides the blockchain data. Use the helper function to set it up.

```js
import { getClient } from '@rtoken/utils';

const apolloInstance = getClient();
```

Optionally, configure your client by passing an object to `getClient()` with the following:

| option | default                     | description                      |
| ------ | --------------------------- | -------------------------------- |
| uri    | (mainnet rDAI subgraph URL) | Location of your rToken subgraph |
| debug  | `false`                     | Display logs on Apollo errors    |

You can also bring your own client if you want more control (see [Using your own Apollo client](#Using-your-own-Apollo-client)).

#### 2. Instantiate the @rtoken/utils library

Pass the `apolloInstance` to create the `RTokenUtils` object.

```js
import RTokenUtils, { getClient } from '@rtoken/utils';

const apolloInstance = getClient();
const rutils = new RTokenUtils(apolloInstance);

// Alternatively with options
const rutils = new RTokenUtils(apolloInstance, options);
```

Options are an object type with the following properties:

| option | default | description  |
| ------ | ------- | ------------ |
| debug  | `false` | print errors |

#### 3. Create and use your entities

Users, Hats, and Global entities are now available for inspecting.

```js
// Users
const user = rutils.user({ address: '0xabc...' });
const userDetails = user.details();

// Hats
const myHat = rutils.hat({ id: 11 });
const allUsers = myHat.allUsers();

// Global
// (under development)
```

If you have any questions, please contact us via Discord.

## Methods

### :bust_in_silhouette: User

#### Available

- `details`

#### Planned

- `interestSentTo`
- `totalInterestReceived`
- `totalInterestSent`

### :tophat: Hat

#### Available

- `allUsers`

#### Planned

- `interestEarned`

# Additional options

## Using your own Apollo client

This might be helpful if you want more control over the apollo-client, such as custom caching options or authentication of a private client. See `/src/utils/client` for how we instantiate the client.

```js
const { ApolloClient } = require('apollo-client');
const { InMemoryCache } = require('apollo-cache-inmemory');
const { HttpLink } = require('apollo-link-http');
const fetch = require('cross-fetch');

const cache = new InMemoryCache();
const link = new HttpLink({
  uri: 'http://localhost:4000/',
  fetch,
});

const apolloInstance = new ApolloClient({
  link,
  cache,
  onError: (e) => {
    console.log(e);
  },
  defaultOptions: {
    query: {
      fetchPolicy: 'network-only',
    },
  },
});

const rutils = new RTokenUtils(apolloInstance);
```

# Contributing

Contributions, suggestions, and issues are welcome. At the moment, there are no strict guidelines to follow.
