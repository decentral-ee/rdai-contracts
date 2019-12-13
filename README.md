<p align="center"><img src="https://rdai.money/images/logo.svg" width="160"/></p>

<p align="center">
    <img alt="GitHub Workflow Status" src="https://img.shields.io/github/workflow/status/rtoken-project/rtoken-contracts/Node CI">
    <img alt="GitHub" src="https://img.shields.io/github/license/rtoken-project/rtoken-contracts">
    <img alt="Coveralls github branch" src="https://img.shields.io/coveralls/github/rtoken-project/rtoken-contracts/master">
</p>

RToken Ethereum Contracts
=========================

`RToken`, or Reedemable Token, is an _ERC20_ token that is 1:1 redeemable to its
underlying _ERC20_ token. The underlying tokens are invested into interest
earning assets specified by the allocation strategy, for example into
[_Compound_](http://compound.finance). Owners of the _rTokens_ can use a
definition called _hat_ to configure who is the beneficiary of the accumulated
interest. _RToken_ can be used for community funds, charities, crowdfunding,
etc. It is also a building block for _DApps_ that need to lock underlying tokens
while not losing their earning potentials.

---

## How does it look like

```
            +------+
      +-----+ User +-------+
      |     +------+       |
      |                    |
      |                    |                      +-----------------------+
      |                    |                      |compound.finance cToken|
 +----+-----+  +-----------+-----------+          +-----------------------+
 |  Dapp    |  |ERC20 Compatible Wallet|                     ^
 +----+-----+  +-----------+-----------+                     |
      |                    |                     +--------------------------+
      |                    |                     |CompoundAllocationStrategy|
      |                    |                     +--------------------------+
      |                    |                                 ^
      |                    v                                 |
      |    +----------------------------------+              |
      |    |          RToken                  |              |
      |    |----------------------------------|              |
      |    | - ERC20 compatible               |              |
      +--->| - Mint/Redeem/PayInterst         |              |
           | - "Hat"/Beneficiary system       |              |
           | - Changeable allocation strategy +--------------+
           | - Configurable parameters        | IAllocationStrategy interface
           | - Admin role (human/DAO)         |
           +----------------------------------+
                       ^
                       |
                    +--+--+
                    |Admin|
                    +-----+
```

## What does it do

As an example, let's pick [_DAI_](https://dai.makerdao.com/) as our underlying
token contract. As a result, the _rToken_ instantiation is conveniently called
_rDAI_ in this example.

### 1. Hat Types

A hat defines who can keep the interest generated by the underlying _DAI_
deposited by users.
Every address can be configured with one and only one hat, but a hat can have
multiple beneficiaries.

There are three kinds of hats:

* `Zero Hat` - It is the default hat for all addresses (even before they have a
balance).
Any interest generated by the _DAI_ tokens locked by the owner are entitled to
the owner himself. This kind of hat is subjected to the [Hat Inheritance
Rules](#hat-inheritance-rules)

* `Self Hat` - Similar to the _Zero Hat_, as the owner keeps all _DAI_ tokens
and generated interest. However in this case it is a deliberate choice by the
owner, hence the [Hat Inheritance Rules](#hat-inheritance-rules) do not apply to
this address.

* `Other Hat` - This hat can be inherited or created by the user. The interest
generated by the _Hat_ can be withdrawn to the address of any recipient
indicated in the hat definition. _Hat Inheritance Rules_ do not apply to this
address.

### 2. Hat Definition

A _hat_ is defined by a list of recipients, and their relative proportions for
splitting the _rDAI_ loans from the owner.

For example:
```
{
    recipients: [A, B],
    proportions: [90, 10]
}
```
defines that the _DAI_ tokens will be loaned to address A and address B in the
relative proportions of 90:10, effectively A receives 90% and B receives 10% of
the generated interest.

### 3. Mint

The user first needs to approve the _rDAI_ contract to use its _DAI_ tokens,
then the user can mint as much _rDAI_ as they have _DAI_. One _rDAI_ is always
equal to one _DAI_.

As a result, the _DAI_ tokens transferred in order to mint new _rDAI_ tokens are
invested automatically into the _Saving Strategy_, and the recipients indicated
in the user's chosen hat can withdraw any generated interest.

### 4. Redeem

Users may redeem the _DAI_ tokens they deposited at any time by transferring
back the _rDAI_ tokens.

As a result, the invested _DAI_ tokens are recollected from the recipients, and
given back to the owner.

### 5. Transfer

_rDAI_ contract is _ERC20_ compliant, and one should use _ERC20_ _transfer_ or
_approve_ functions to transfer the _rDAI_ tokens between addresses.

As a result, the amount of _DAI_ tokens loaned out by the source relevant to the
transaction is recollected, and loaned to the new recipients according to the
hat of the destination.

### 6. Pay Interest

Recipients of loaned _DAI_ tokens are entitled to the full amount of interest
earned from them.

Anyone can call the _payInterest_ function, which converts the earned interest
to new _rDAI_ tokens for the recipient. This mechanism allows contract addresses
to also be recipients, despite not having implemented functions to call the
_payInterest_ function externally.

Unlike the mint processes, _rDAI_ generated in this process does not loan equal
amount of _DAI_ tokens to any recipient. The owner may choose to loan them by
using _loanInterest_, or transfer the _rDAI_ to another address and trigger the
hat switching process.

Interest payment rules may apply as per configuration (see _Governance
section_).

### 7. Hat Inheritance Rules

In order to maximize the cause the hat owners choose, the following rules are
stipulated in order to allow hats to spread to new users:

* All addresses have the _Zero Hat_ by default.

* During the transfer process, _DAI_ tokens are recollected and loaned to the
new recipients. If the recipient has the _Zero Hat_, and if the source hat is
not a _Self Hat_, the recipient will inherit the source's hat.

For example: Alice sets UNICEF France as recipients of her generated interest.
Bob has never used _rDAI_, and thus has a _Zero Hat_. When Alice sends Bob 100
_rDAI_, Bob inherits Alice's hat, and UNICEF France keep accruing interest. Bob
then sends the 100 _rDAI_ along to Charlie. But Charlie already has a hat, so
the underlying 100 _DAI_ are now loaned to Charlie's chosen recipients.

### 8. Hats for contract addresses

As most contract addresses can't execute arbitrary functions, they can generally
only change hat once, from inheritance by the first user to send _rDAI_ to the
contract address.
Because it is sometimes unclear who the owner of a contract is, the `rToken`
contract allows the admin to change the hat of any contract address.

##### Note that the addresses without code are assumed to be able to demonstrate
the ownership by indisputable ownership of the private key, so even _admin_ is
not allowed to change that for them.

In order to avoid needing to use the admin, we advise buidlers who are looking
to accept _rDAI_ to set up their contracts correctly by:
1. getting some _rDAI_ for themselves
2. selecting or creating a hat of their choosing
3. transferring any amount of _rDAI_ to their contracts

### 9. Allocation Strategy

The _IAllocationStrategy_ interface defines what _RToken_ can integrate for
investing the underlying assets in exchange for saving assets that earns
interest.

It is changeable by admin. Per request, the _rDai_ contract will redeem all
underlying assets at once from old allocation strategy, and invest all into new
allocation strategy.

_CompoundAllocationStrategy_ is one implementation. In case of _rDai_, it is
_cDai_.

While it is not possible to forbid admin from using risky strategy, and risk
strategy could cause redeemability to fail if the strategy has heavy losses,
it is up to the admin to make a sensible choice of what consists of a
proper allocation strategy.

### 10. Statistics

### 11. Admin & Governance

**(TODO NOTE! The list is not final and some are to be implemented!)**

The `RToken` contract has an admin role who can:

- Change allocation strategy
- Change hat for any contract address
- Upgrade code

It is up to the `rToken` instantiator to decide the degree of decentralization
of this admin. For maximum decentralization, the admin could be a DAO that is
implemented by a DAO framework such as [Aragon](https://aragon.org/), and the
hat change could be controlled by a arbitration process such as
(Kleros)(https://kleros.io/).

# How It Is Implemented

## Project Structure

The project uses truffle as development framework, and the contracts are written
in solidity.

The main contract is `RToken`, and the interface of it is in `IRToken` with more
comments aimed for users of the `RToken` contract.

The project also employs the Universal Upgradeable Proxy Standard, or [EIP-1822]
(https://eips.ethereum.org/EIPS/eip-1822]). `RTokenStorage` and `RTokenStructs`
are the storage contract as a result.

The allocation strategy is defined in the `IAllocationStrategy` contract.

The compound implementation of it is in the `CompoundAllocationStrategy`
contract. Compound V2 contracts are pulled from the _etherscan_ and stored under
the `compound/contracts` directory. `CErc20Interface` contract is used to
implement the `CompoundAllocationStrategy`.

## RToken Account

Each address has an _RToken Account_. The account data includes:

* `hatID` - the hat associated with the account,
* internal accounting information,
* account statistics.

## RToken Internal Accounting

There are three types of assets are changing hands during different processes:

* underlying tokens,
* `rToken`,
* saving assets (managed by allocation strategy).

The rules are:

* depositors of underlying token is given equivalent amount of `rToken`,
* underlying tokens are "loaned" to the hat recipients as debt,
* underlying tokens are transferred to the allocation strategy and becomes
saving assets that are owned by the hat recipients.

Another way to look at is that, hat recipients owes the donor the original
underlying assets as debt denominated in `rToken`, while the underlying tokens
are converted to saving assets and owned by the hat recipients.

The related account data properties are:

* `rAmount` - Redeemable token balance for the account.
* `rInterest` - Redeemable token balance portion that is from interest payment.
* `lRecipients` - Mapping of recipients and their amount of debt.
* `lDebt` - Loan debt amount for the account.
* `sInternalAmount` - Saving asset amount internal.

### Mint Process

When some underlying token is transferred to the `rToken` contract:

* an equivalent `rAmount` of `rToken` is minted,
* underlying tokens are distributed to the hat recipients,
* `lRecipients` records the amount of underlying tokens owed by each
  hat recipients,
* each hat recipients also adds those amount to their `lDebt` accordingly,
* the underlying tokens are transferred to the `AllocationStrategy`, and
  `sInternalAmount` of saving assets are created and owned by the hat
  recipients.

### Redeem Process

When user (aka. _redeemer_) wants to redeem `rToken` for underlying tokens:

* a portion of saving assets of each hat recipients are converted back to the
underlying token to cover exact same amount of `rToken` that is asked,
* underlying tokens are given back to the _redeemer_,
* _redeemer_ gets back its underlying tokens.

### Important Internal Functions

Ownership change logic is implemented by these two functions.

* `distributeLoans` - loan underlying tokens to hat recipients and convert them
to
  saving assets.
* `recollectLoans` - convert saving tokens back to underlying assets and pay
back
  debt owned by the hat recipients.

### Transfer Process

The `src` `recollectLoans` from its hat recipients, transfers the underlying
tokens recollected to the `dst`, then the `dst` `distributeLoans` to its hat
recipients

## Allocation Strategy

### Allocation Strategy Interface

To become a allocation strategy, one would need to implement:

* `exchangeRateStored` - Calculates the exchange rate from the underlying to the
saving assets
* `accrueInterest` - Applies accrued interest to all savings
* `investUnderlying` - Sender supplies underlying assets into the market and
receives saving assets in exchange
* `redeemUnderlying` - Sender redeems saving assets in exchange for a specified
amount of underlying asset

### Compound Allocation Strategy

Allocation strategy interface is largely derived from the compound v2 `CToken`
contract, hence the saving assets is directly denominated in `cToken` amount
and the compound allocation strategy is mostly a proxy call to the `cToken`
contract.

## RToken Allocation Strategy Switching

`RToken` has one saving strategy at a time, and all underlying assets are
transferred to and converted to saving assets that are expected to be liquid
and growing in value measured in underlying tokens.

`RToken` allows the admin to switch the allocation strategy during the contract
life time.

When the switching happens, all saving assets are converted to underlying tokens
and then reinvested in new saving assets immediately. There is inevitably a
difference in the exchange rate of different saving assets, a internal property
`savingAssetConversionRate` is used for the adjustment, in order to make this
equation true before and after the switching:

```
savingAssetOrignalAmount = sum(account.sInternalAmount /
savingAssetConversionRate
for all accounts)
```

# Deployed Contracts

For testnets, the logic contracts are continuously updated. To fetch logic contract address using web3, use:


```
web3.eth.getStorageAt(RDAI_PROXY_ADDRESS,"0xc5f16f0fcc639fa48a6947836d9850f504798523bf8c9a3a87d5876cf622bcf7")
```


## Rinkeby

* DAI: [0x5592EC0cfb4dbc12D3aB100b257153436a1f0FEa](https://rinkeby.etherscan.io/address/0x5592EC0cfb4dbc12D3aB100b257153436a1f0FEa)
* cDAI: [0x6d7f0754ffeb405d23c51ce938289d4835be3b14](https://rinkeby.etherscan.io/address/0x6d7f0754ffeb405d23c51ce938289d4835be3b14)
* rDai (proxy): [0x6AA5c6aB94403Bdbbf74f21607D46Be631E6CcC5](https://rinkeby.etherscan.io/address/0x6AA5c6aB94403Bdbbf74f21607D46Be631E6CcC5)

**v1**

* DaiCompoundAllocationStrategy: [0xB603cd1bdAcaBd5Ca9341d8a44f6430a5e371b76](https://rinkeby.etherscan.io/address/0xB603cd1bdAcaBd5Ca9341d8a44f6430a5e371b76)

## Kovan

* DAI: [0xbF7A7169562078c96f0eC1A8aFD6aE50f12e5A99](https://kovan.etherscan.io/address/0xbF7A7169562078c96f0eC1A8aFD6aE50f12e5A99)
* cDAI: [0x0A1e4D0B5c71B955c0a5993023fc48bA6E380496](https://kovan.etherscan.io/address/0x0A1e4D0B5c71B955c0a5993023fc48bA6E380496)
* rDAI (proxy): [0x3183683cEeAb01699722053A2cb6A945cE0D7CeC](https://kovan.etherscan.io/address/0x3183683cEeAb01699722053A2cb6A945cE0D7CeC)

**v1**

* DaiCompoundAllocationStrategy: [0xb4377efc05bd28be8e6510629538e54eba2d74e3](https://kovan.etherscan.io/address/0xb4377efc05bd28be8e6510629538e54eba2d74e3)

## Mainnet

* DAI: [0x89d24A6b4CcB1B6fAA2625fE562bDD9a23260359](https://etherscan.io/address/0x89d24a6b4ccb1b6faa2625fe562bdd9a23260359)
* cDAI: [0xf5dce57282a584d2746faf1593d3121fcac444dc](https://etherscan.io/address/0xf5dce57282a584d2746faf1593d3121fcac444dc)
* rDai (proxy): [0xea8b224eDD3e342DEb514C4176c2E72Bcce6fFF9](https://etherscan.io/address/0xea8b224eDD3e342DEb514C4176c2E72Bcce6fFF9)

**ethberlin edition**
* DaiCompoundAllocationStrategy: [0x21F090905D26073cb488440F98CcfbD8bF5aA9b3](https://etherscan.io/address/0x21F090905D26073cb488440F98CcfbD8bF5aA9b3)
* rDAI Logic: [0x09163bc9da7546ddA9D82Be98FE006a95C87E9B4](https://etherscan.io/address/0x09163bc9da7546ddA9D82Be98FE006a95C87E9B4)

**v1**
* DaiCompoundAllocationStrategy: [0x594e15580468d21D447299F2033Bd203036475FA](https://etherscan.io/address/0x594e15580468d21D447299F2033Bd203036475FA)
* rDAI Logic: [0x293908E6352b11a91bC08eeb335644C485D21170](https://etherscan.io/address/0x293908E6352b11a91bC08eeb335644C485D21170)

# Integration

Here are some documentations on how one can integrate rDai in applications.

## Mint/Redeem Flows

**!TODO! embed plantuml sequence diagrams**

**Some useful notes**

1. How does it really work?

Every address, including a contract address, is associated with one and only one hat.  Each hat specifies 1) a set of recipients and 2) the proportions of the interest generated from the allocation strategy that each recipient will receive.

When an account mints, the account's DAI is transferred to the rDAI contract and an equivalent amount of rDAI is minted. The account gets to keep that amount of rDAI always, and can redeem it at any time. Those DAI are invested into an allocation strategy, which at the moment is Compound. The interest generated by that allocation strategy is assigned to the recipients defined by the hat.

That interest is constantly accruing, but a transaction is necessary to realize it as rDAI. Any recipient can withdraw its proportional share of interest as rDAI whenever it wants. Note that the recipient can be the account itself, hence the special case we call zero hat(default hat for all accounts but subject to hat inheritance rules) or self hat (a deliberate choice that make the account immune of hat inheritance rules).

So to simplify, here is the flow into and out of rDAI:
1. DAI ---> rDAI (mint functions)
2. rDAI ---> Accrues Interest to Recipients (allocation strategy)
3. Recipient Realizes Interest as rDAI (payInterest function)
4. rDAI ---> DAI (redeem functions)

This is what's happening under the hood.

2. How do we really mint?

- First, find out the rDAI proxy contract down below this document,
- DAI.ERC20.approve (`spender` = rDAI proxy address, `amount` = max uint256 usually), allowing rDAI contract to use your DAI balances up to `amount`,
- use one of the mint functions: rDAI.mint|mintWithSelectedHat|mintWithNewHat, find out more at https://github.com/rtoken-project/rtoken-contracts/blob/master/contracts/IRToken.sol

## Stats

### On-chain stats:

* How many addresses are using the hat?

  `IRToken.getHatStats(hatID).useCount`

* how much loans distributed through the hat currently?

  `IRToken.getHatStats(hatID).totalLoans`

* how much interest has been accumulated under the hat?

  `IRToken.getHatStats(hatID).totalSavings - getHatStats(hatID).totalLoans`

* how much loans distributed to the account?

  `IRToken.receivedLoanOf(owner)`

* how much savings distributed to the account?

  `IRToken.receivedSavingsOf(owner)`

* how much is cumulative interest generated for the account?

  `IRToken.getAccountStats(owner).cumulativeInterest`

* total supply

  `ERC20.totalSuppl`

* total savings amount

  `IRToken.getGlobalStats().totalSavingsAmount`

### Off-chain stats, that require pre-processing:

These `IRToken` events will help for indexing stats per account/hat:

* Mint
* Redeem
* LoansTransferred
* InterestPaid
* HatCreated
* HatChanged

For example for these metrics:

* hats sorting by on-chain stats
* monthly/daily active hats
* top interest generating address
* top beneficiaries
* global volumes
* global interest earned by period
