# TrustRelay Specification
 
TrustRelay is an open financial system enabling access to access financial opportunities. 

TrustRelay consists of three parts:
 
- **TrueUSD**
A compliant ERC-20 stablecoin backed 1:1 by US Dollars.
 
- **TrueReward**
A smart contract layer allowing TrueUSD holders to access financial opportunities.
 
- **TrustToken Assurance**
Protection for TrueReward users against vulnerabilities and risk in financial opportunities.
 
## Glossary of Terms

**TUSD** - TrueUSD, a compliant stablecoin backed 1:1 by US Dollars.

**TRU** - TrustToken, an ecosystem token for staking on opportunities to provide assurance.

**finOp** - A third party financial opportunity to earn rewards for pooled TUSD codified by a smart contract.

**rToken** - Reward Token, a non-transferable token which represents an account's stake in a financial opportunity.

**deFi** - Decentralized Finance: opportunities which require no centralized party.

**ceFi** - Centralized Finance: traditional opportunities which rely on centralized parties (banks, companies, governments, etc.)

# TrueUSD

TrueUSD is a compliant stablecoin backed 1:1 by US Dollars. TrueUSD is minted in exchange for US dollar transfers (currently through wire transfer) and redeemed for US dollars (also via wire transfer). TrueUSD is partnered with Armanino to provide real time attestations for USD balances backing TrueUSD.

### TokenController

The TokenController contract is used to manage TrueUSD, separating roles by permission.

### Mints

### Redemptions

### Supply

There are multiple functions to get the supply of TrueUSD. The three functions are:

- **depositBackedSupply()** - supply of TUSD backed by US Dollars
- **rewardBackedSupply()** - supply of TUSD backed by TrueReward
- **totalSupply()** - supply of TUSD backed by TrueReward and US Dollars

Only deposit-backed TrueUSD can ever be exchanged for US Dollars. Please read the TrueReward section to learn more about the difference between reward-backed TUSD and deposit-backed TUSD.

# TrueReward

TrueReward allows TUSD holders to earn rewards by loaning TUSD to financial opportunities in return for block-by block rewards. Accounts can enable TrueReward directly from an Ethereum wallet and are able to view their total balance update block-by block.

Accounts opted into TrueReward can NEVER exchange their balances for US Dollars. Rather, an account MUST opt out before making a redemption.

### Enabling and Disabling TrueReward

TrueReward is enabled and disabled through the TrueUSD smart contract directly. When TrueReward is enabled for an account, the smart contract overrides the default logic for ERC-20 functions to allow for normal mints, transfers, and redemptions for TrueReward accounts.

Enabling TrueReward will trigger an event signifying

#### Rewards Distribution

When TrueReward is enabled, a distribution is set mapping basis values to financial opportunity addresses. This distribution represents what percentage of an account's balance is opted into what opportunity. Currently, we only support a single opportunity, therefore we simply map the max basis amount to the default financial opportunity. In the future, we will be able to opt accounts into multiple financial opportunities, allowing an account to diversity their reward sources.

### rewardTokens (rTokens)

rewardTokens (rTokens) are an accounting mechanism used to keep track of user balances opted into trueReward. Internally, the TrueReward contract keeps track of each account's stake in a financial opportunity.

The easiest way to understand rTokens is to think of them as non-transferable ERC-20 tokens which simply track the amount owed to a TrueReward enabled account.

### Transfers with TrueReward

Transfers when the sender or receiver TrueReward incorporate the following algorithm:

1. If sender TrueReward is enabled, exchange rTokens for TUSD
2. Transfer TUSD from sender to receiver
3. If receiver is enabled, exchange TUSD for rTokens

This way, rewardTokens can never be transferred directly, rather they are always exchanged for TUSD before being transferred.

### Reserve Mechanism

Interacting with defi smart contracts can become very expensive. Therefore, a gas saving mechanism was implemented to allow cheap opt-ins, opt-outs, and transfers for TrueReward enabled accounts. This mechanism is part of the TrueUSD smart contract, and is intended to facilitate low gas transfers, opt-ins, and opt-outs.

#### How the Reserve Works

The RESERVE address is a special address which nobody holds the private key to. Rather than using transfer(), the TrueUSD contract simply updates the balance of the reserve TUSD and rToken balances. The rToken balances of this contract are managed by the TokenController contract

The reserve works by using a RESERVE address can holds balances of both rewardTokens and TrueUSD. This is a unique address which is managed internally by TrustToken to keep transfers cheap by sponsoring internal contract swaps between rewardToken and TUSD balances. There must always be enough liquidity in the reserve to sponsor swaps, otherwise rTokens must directly call the financial opportunity to exchange rTokens and TUSD.

## Financial Opportunity

A financial opportunity is a pool which TrueReward users can enter and exit. Users that are opted into this pool earn compounding rewards over time on a block-by-block basis.

A financial opportunity is a smart contract which interfaces between TrueRewards and a deFi or ceFi opportunity. We can think of a financial opportunity like a non-transferable ERC20 token contract, which mints tokens given TUSD and redeems tokens for TUSD. Currently only the.

A financial opportunity has 4 functions:
- deposit()
- redeem()
- value()
- supply()

#### deposit()

*paramaters:*
@param **account** account to transfer TUSD from
@param **amount** TUSD amount to exchange for rTokens
@return tokens rTokens created

*algorithm:*
1. Call transferFrom from the account to the underlying opportunity for deposit amount
2. Calculate the amount of stake (rToken) to mint for TUSD deposit amount
3. Increase rToken supply
4. Return the amount of rToken minted

#### redeem():

*parameters:*
@param **account** account to transfer TUSD to
@param **amount** rTUSD amount to exchange for TUSD
@return TUSD amount

*algorithm:*
1. Calculate the TUSD value of rTUSD
2. TUSD is transferred to account
3. Update total rTUSD supply
4. Return the TUSD value of redeemed rTUSD

#### value()

value() is used to calculate the value of rTokens in TUSD

*parameters:*
@return value of 1 rToken in TUSD

#### supply()

@return uint256 supply
total supply of rTokens in this contract.

# TrustToken Assurance

TrustToken Assurance is a system to protect TrueReward users from vulnerabilities in financial opportunities. Assurance uses a staking mechanism where stakers are financially incentivised to take on the risk of financial opportunities becoming insolvent. When an opportunity is unable to return a user's TrueUSD, stake is liquidated to cover the redemption, and the stakers take on the responsibility of redeeming reward tokens.

1. TRU holders stake on a financial opportunity.
2. As rewards are earned through an opportunity, a portion of the rewards are directed to the staking pool.
3. When an opportunity is unable to return funds to an account exiting TrueRewards, TRU is sold at market price for TUSD, and the exiting account is sent this TUSD.
4. The pool is able to attempt to redeem reward tokens at a later date if the opportunity becomes solvent again.

## TrustToken

TrustToken (TRU) is a standard ERC-20 token used to stake on financial opportunities. TrustTokens have a total supply of 1.45 billion TRU.

## StakedToken Contract

StakedToken is the name of the staking pool contract

## Liquidator Contract

The Liquidator sells TRU for TUSD when a financial opportunity becomes insolvent.