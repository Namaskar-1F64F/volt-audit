# Always Be Growing
ABG is a talented group of engineers focused on growing the web3 ecosystem. Learn more at https://abg.garden

# Introduction

The Volt protocol aims to peg a stablecoin to inflation, but with this PR, a new "oracle" is going to be used as a stop-gap measure. 
This VoltSystemOracle has a pre-defined target set at deployment for interest to compound over an, also set at deployment, timeframe. While reviewing
the contract, there are only two functions, each with concerns.

1. Does these two functions work in the way they were intended?
2. Do the tests have coverage over the functionality?
3. Is the deployment strategy going to be successful?

*Disclaimer:* This security review does not guarantee against a hack. It is a snapshot in time of brink according to the specific commit by a single human. Any modifications to the code will require a new security review.

# Methodology
On a restricted time I was unable to deeply investigate all aspects of the system. The following is steps I took to evaluate the change.

- Clone & Setup project
- Read Readme and related documentation
- Use Surya on VoltSystemOracle - Not super helpful except it shows the lack of inherent smart contract interaction 
  - Report
  - Contract interaction
  - Inheritance
- Line by line review

### Line by line review
Read through Interface and NatSpec. **Assumed** contract deployment would be as intended & values used to construct would be reasonable. Kept in mind Solcutiry guidelines. 

The tests and fuzz tests appear to cover every aspect of the state variables being correctly set.

#### getCurrentOraclePrice
Correctly a public view function, has no external calls,

Understood the time delta is capped on range of 1 - TIMEFRAME, the price percentage change is the total amount of monthly change, and price delta is capped on a range of nearly 0 - 1 times the total monthly change.

The tests and fuzz tests appear to cover every aspect of this.
#### compoundInterest
Assume the time period will not be out of date due to the keepers, the function is correctly external and has no external calls.

Understood periodStartTime and TIMEFRAME are both in the same denomination, and even if block timestamp was manipulated after the `require`, the time would be in a valid state. A valid state being also means the period will not be more than one TIMEFRAME worth of compounding.

The tests and fuzz tests appear to cover every aspect of this.

#### Proposal
The proposal framework was new to me and seemed like a particular area of interest as deployment, role changing, and configuration set happens. Overall, I believe this will work.

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

# Additional Comments
Some of the NatSpecs are out of sync between the Interface & the Contract itself like `monthlyChangeRateBasisPoints` and `getCurrentOraclePrice`. I would prefer the information be the same so I do not have to check both places.

Some of the comments about timeframes are inaccurate from the change of a year to a month. Consider using something like timeframe and not being specific about 'month' or 'year'.

I did not fully understand the deployment process. I believe it can be in a state where the "validate" function is run, but without DO_DEPLOY on L34 of utils/checkProposal.ts the old version of the saved addresses are used to verify. Then, it might give a false sense of security that the deployment is "valid". Consider double checking this process.

