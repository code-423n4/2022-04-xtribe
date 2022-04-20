
---

# xTRIBE contest details
- $71,250 main award pot
- $3,750 gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-04-xtribe-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts April 21, 2022 00:00 UTC
- Ends April 27, 2022 23:59 UTC

This repo will be made public before the start of the contest. (C4 delete this line when made public)

# Contest Scope
This contest is open for one week. Representatives from Tribe will be available in the Code Arena Discord to answer any questions during the contest period. 

The focus for the contest is to try and find:
* any logic errors or ways to drain funds from the protocol in a way that is advantageous for an attacker at the expense of users with funds invested in the protocol.
* Ways to corrupt state through improper state transitions that brick functionality.


 Wardens should assume that governance variables are set sensibly (unless they can find a way to change the value of a governance variable, and not counting social engineering approaches for this).

## Overview
The xTRIBE tokenomics upgrade combines a few features into one new governance token:
* autocompounding single sided staking rewards
* multi-delegation capabilities
* reward delegation

https://tribe.fei.money/t/xtribe-tokenomics-upgrade/4038

Additionally, this contest will review "Flywheel v2" which handles the reward delegation component.
---
Pulling in changes:
1. clone the repo
2. download foundry (https://github.com/foundry-rs/foundry)
3. forge install

This will clone and pull in all the libs.

---
Libraries used:
* solmate
* OpenZeppelin EnumerableSet

---
# Contracts in scope
## xTRIBE

source: `lib/xTRIBE/src/xTRIBE.sol`
LoC: 150

Description: The xTRIBE token is essentially a combination of ERC20Gauges, ERC20MultiVotes, and xERC4626 with a Multicall. An important security consideration is that it combines the TRIBE voting balance with xTRIBE balance. It should not be possible to double count votes. 

## Flywheel v2

source: `lib/flywheel-v2/src/FlywheelCore.sol`
LoC: 270

Description: FlywheelCore is an accounting layer for a rewards distribution of a single token to multiple reward strategies. See the [Flywheel V2 README](https://github.com/fei-protocol/flywheel-v2) for more details.

---

source: `lib/flywheel-v2/src/rewards/FlywheelGaugeRewards.sol`
LoC: 250

Description: FlywheelGaugeRewards is a "rewards module" for flywheel core which reads in rewards balances from the ERC20Gauges contract. The rewards are batched into cycles, which can be queued all at once or paginated to support a large number of reward tokens.

This contract is the most complex piece of the system, as it integrates FlywheelCore and ERC20Gauges with state being held in all 3 locations.

If a gauge is deprecated, it should return 0 for all future cycles until re-added. If it is currently a part of a cycle, those rewards should distribute normally.

---

source: `lib/flywheel-v2/src/token/ERC20Gauges.sol`
LoC: 600

Description: ERC20Gauges allows liquid voting on a set of gauges. The weights are allocated pro-rata and can be used for any kind of resource allocation. The main use case is reward direction, such as in FlywheelGaugeRewards and similar to Curve finance or Tokemak.

---

source: `lib/flywheel-v2/src/token/ERC20MultiVotes.sol`
LoC: 400

Description: Similar to OpenZeppelin [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes), but uses a multi-delegation algorithm where users can partially delegate to multiple addresses.

## ERC4626

source: `lib/ERC4626/src/xERC4626.sol`
LoC: 100

Description: an "xToken" which autocompounds single sided rewards. Should be completely price manipulation resistant due to internal balances being used.