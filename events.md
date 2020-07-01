# Events for TrueReward

The events emitted when using TrueReward can be a bit confusing at first, so this document is meant to break down each possible case when enabling, transferring, and disabling.

## Transfer Logic:

1. Check if sender is enabled 
    - If YES -> redeem or swap reward tokens for deposit tokens
2. Make Transfer of deposit-backed tokens
    - Update balances
    - Emit Normal Transfer Event
    - If receiver is autosweep address, forward funds to final address
3. If receiver is enabled
    - If YES -> deposit or swap deposit tokens for reward tokens
4. Execute hook

## How the reserve works

The reserve is an internal mechanism to save gas costs for TrueReward users with small balances. A reserve address is used to cheaply swap deposit and reward tokens. The reserve is an ethereum address that nobody has the private key to, located at 

## Important Events

#### TrueRewardEnabled

```
event TrueRewardEnabled(
    address indexed _account, 
    uint256 _amount, 
    address indexed _finOp
);
```

#### TrueRewardDisabled

```
event TrueRewardDisabled(
    address indexed _account, 
    uint256 _amount, 
    address indexed _finOp
);
```

#### MintRewardToken

```
MintRewardToken(
    address indexed account, 
    uint256 tokensDeposited, 
    uint256 rewardTokensMinted, 
    address indexed finOp
);
```

#### RedeemRewardToken

```
RedeemRewardToken(
    address indexed account,
    uint256 tokensWithdrawn,
    uint256 rewardTokensRedeemed,
    address indexed finOp
);
```


#### BurnRewardToken

```
BurnRewardToken(
    address indexed account, 
    uint256 rewardTokenAmount, 
    address indexed finOp
);
```

#### MintRewardBackedToken 

```
MintRewardBackedToken(
    address indexed account, 
    uint256 indexed amount
);
```
#### BurnRewardBackedToken

```
BurnRewardBackedToken(
    address indexed account, 
    uint256 indexed amount
);
```

## All TrueReward Transfer Cases

#### Enable TrueReward

*reserve:    no*

1. TrueRewardEnabled(account, balance, finOp)
2. Transfer(account, finOp, balance) // TUSD
3. Transfer(0x0, account, balance) // Debt
4. Mint(account, balance) // Debt

----------------------------------------------------------------

#### Disable TrueReward

*reserve:    no*

1. TrueRewardDisabled(account, balance, finOp)
2. Transfer(finOp, account, balance) // TUSD
3. Transfer(account, 0x0, balance) // Debt
4. Burn(account, balance) // Debt

------------------------------------------------------------

#### Enable TrueReward

*reserve:    yes*

1. TrueRewardEnabled(account, balance, finOp)
2. Transfer(account, reserve, balance)
3. Transfer(reserve, account, balance)

------------------------------------------------------------

#### Disable TrueReward

*reserve:    yes*

1. TrueRewardDisabled(account, balance, finOp)
2. Transfer(reserve, account, balance)
3. Transfer(account, reserve, balance)

------------------------------------------------------------

#### Transfer

*from:       disabled*  
*to:         disabled*  
*reserve:    no*

1. Transfer(from, to)

------------------------------------------------------------

#### Transfer

*from:       enabled*  
*to:         disabled*  
*reserve:    no*

1. Transfer(finOp, from, amount)
2. Transfer(from, 0x0, amount)
3. Burn(from, amount)
4. Transfer(from, to, amount)

------------------------------------------------------------

#### Transfer

*from:       disabled*  
*to:         enabled*  
*reserve:    no*

1. Transfer(from, to, amount)
2. Transfer(to, finOp, amount)
3. Transfer(0x0, to, balance)
4. Mint(to, amount)

------------------------------------------------------------

#### Transfer

*from:       enabled*  
*to:         enabled*  
*reserve:    no*

// opt out amount -> transfer amount -> opt in amount
1. Transfer(finOp, from, amount)
2. Transfer(from, 0x0, amount)
3. Burn(from, amount)
4. Transfer(from, to, amount)
5. Transfer(to, finOp, amount)
6. Transfer(0x0, to, amount)
7. Mint(to, amount)

------------------------------------------------------------

#### Transfer

*from:       enabled*  
*to:         enabled*  
*reserve:    yes*

1. Transfer(sender, RESERVE, amount) // send TUSD to Reserve
2. Transfer(RESERVE, sender, amount) // receive reward from Reserve
3. Transfer(sender, receiver, amount)
4. Transfer(receiver, RESERVE, amount)
5. Transfer(RESERVE, receiver, amount)

------------------------------------------------------------

#### Transfer

*from:       enabled*  
*to:         disabled*  
*reserve:    yes*

1. Transfer(sender, RESERVE, amount)
2. Transfer(RESERVE, sender, amount)
3. Transfer(sender, reciever, amount)

------------------------------------------------------------

#### Transfer

*from:       disabled*  
*to:         enabled*  
*reserve:    yes*

1. Transfer(sender, reciever, amount)
2. Transfer(RECIEVER, reserve, amount)
3. Transfer(RESERVE, reciever, amount)

------------------------------------------------------------

