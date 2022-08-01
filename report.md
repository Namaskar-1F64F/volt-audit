# Always Be Growing
ABG is a talented group of engineers focused on growing the web3 ecosystem. Learn more at https://abg.garden

# Introduction

For this PR, a new "oracle" will be introduced as a stop-gap measure. This will replace the existing CPI based oracle.

This VoltSystemOracle has a pre-defined target set at deployment for interest to compound over an, also set at deployment, timeframe. While reviewing
the contract, there are only two functions, each with concerns.

1. Does these two functions work in the way they were intended?
2. Do the tests have coverage over the functionality?
3. Is the deployment strategy going to be successful?

As part of this PR, a new OraclePassThrough and this VoltSystemOracle will be deployed. Additionally, other contract calls will be made with an OptimisticTimelock.

*Disclaimer:* This security review does not guarantee against a hack. It is a snapshot in time of brink according to the specific commit by a single human. Any modifications to the code will require a new security review.

# Methodology
The following is steps I took to evaluate the change.

- Clone & Setup project
- Read Readme and related documentation
- Use Surya on VoltSystemOracle - Not super helpful except it shows the lack of inherent smart contract interaction 
  - Report
  - Contract interaction
  - Inheritance
- Line by line review
  - Contract
  - Interface
  - Unit Tests
  - Integration Tests
- Proposal Deployment

### Line by line review
Read through Interface and NatSpec. **Assumed** contract deployment would be as intended & values used to construct would be reasonable. Kept in mind Solcutiry guidelines. 

The tests and fuzz tests appear to cover every aspect of the state variables being correctly set.

#### getCurrentOraclePrice
Correctly a public view function, has no external calls,

Understood the time delta is capped on range of 1 - TIMEFRAME, the price percentage change is the total amount of monthly change, and price delta is capped on a range of nearly 0 - 1 times the total monthly change.

The tests and fuzz tests appear to cover every aspect of this.

#### compoundInterest
**Assumed** the time period will not be out of date due to the keepers, the function is correctly external and has no external calls.

Understood periodStartTime and TIMEFRAME are both in the same denomination, and even if block timestamp was manipulated after the `require`, the time would be in a valid state. A valid state being also means the period will not be more than one TIMEFRAME worth of compounding.

The tests and fuzz tests appear to cover every aspect of this.

### Integration Tests
I considered results of pointing the new oracle and oracle pass through. I did a line-by-line review of the integration tests.

1. Does the new oracle implement the correct methods?
2. Is the price returned correctly?
3. Is the configuration correct?

The integration tests for both the Arbitrum and Mainnet networks covered these questions and were all passing. The tests looked sound.

### Proposal Deployment
The deployment and proposal process was new to me and seemed like a particular area of interest as deployment, role changing, and setting configuration happens. I reviewed the flow and believe it to be valid. During deployment the starting oracle price must be set as close to the current price as possible to avoid potential arbitrage. I ran through the `checkProposal` script on the forked node which succesfully ran. 

After my initial review, a robust Solidity-based simulation environment was introduced. The simulation covers the proposal, scheduling, and execution of a timelock transaction. This simulated action, set out by VIP-2 is included in an integration test. 

# Findings 

## Critical Risk
No findings
## High Risk
No findings
## Medium Risk
No findings
## Low Risk
No findings
## Informational
1 finding

### Unnecessary / Inconsistent Override of Properties

**Severity:** Informational

**Context:** [`VoltSystemOracle.sol#L30`](https://github.com/volt-protocol/volt-protocol-core/pull/82/files#diff-885ec89719648151f14618577175e2577fb4b8a4599c5c4bc2ce5007308c0e91R30)

**Description:**

Currently, all of the state variables and methods (except `monthlyChangeRateBasisPoints`) are decorated with "override". This usually indicates that a base method has functionaly replaced.

```solidity
uint256 public immutable monthlyChangeRateBasisPoints;
```

**Recommendation:**
```diff
- uint256 public immutable monthlyChangeRateBasisPoints;
+ uint256 public immutable override monthlyChangeRateBasisPoints;
```

An alternative recommendation is to remove all `override` decoration as the interface is mearly being implemented, and `override` is not needed. See [the issue](https://github.com/ethereum/solidity/issues/8281) and [the PR](https://github.com/ethereum/solidity/pull/11628) for more discussion.


**Resolution**
VOLT removed the override modifier on state variables in commit \href{https://github.com/volt-protocol/volt-protocol-core/blob/05894cb683a4463cd6d504c063bec80793b4e550/contracts/oracle/VoltSystemOracle.sol}{05894cb}

# Additional Comments

The temporary oracle code is well-tested, well-documented, and appears to work as intended.  The changes to implement, by simulating the timelock, gives a higher confidence in the integration process of switching to the new oracle. 

