# token-pay-swypt-documentation
This repo serves as a documentation for integration with swypt APIs and how to integrate with our smart contract functions. 

Deployed Contract on BASE
[Swypt Contract](https://basescan.org/address/0x83c6a042f199588d20c1312C9826E90C420Bc9b7#code)

# off-ramp 
- Swypt offramp involves the following steps 
1. Calling the Swypt smart contract `withdrawToEscrow` or `withdrawWithPermit` functions and perform a blockchain transaction
2. Call swypt offramp API  endpoint with a payload containing the following data

# withdrawWithPermit
Enables token withdrawal using EIP-2612 permit, eliminating the need for a separate approval transaction

`Parameters`

```
_tokenAddress (address): Token contract address being withdrawn
_amountPlusfee (uint256): Total withdrawal amount including fees
_exchangeRate (uint256): Current exchange rate for the token
_feeAmount (uint256): Fee amount to be deducted
deadline (uint): Timestamp until which the signature is valid
v (uint8): Recovery byte of the signature
r (bytes32): First 32 bytes of the signature
s (bytes32): Second 32 bytes of the signature

```
Here is an example 

```
// Create permit signature (on client side)
const domain = {
    name: 'Token Name',
    version: '1',
    chainId: 1,
    verifyingContract: tokenAddress
};

const permit = {
    owner: userAddress,
    spender: contractAddress,
    value: amountPlusFee,
    nonce: await token.nonces(userAddress),
    deadline: Math.floor(Date.now() / 1000) + 3600 // 1 hour from now
};

const { v, r, s } = await signer._signTypedData(domain, types, permit);

// Call the withdraw function
const tx = await contract.withdrawWithPermit(
    tokenAddress,
    amountPlusFee,
    exchangeRate,
    feeAmount,
    permit.deadline,
    v,
    r,
    s
);

Returns 
nonce (uint256): Unique identifier for the withdrawal transaction

```


# withdrawToEscrow
Performs a token withdrawal to an escrow account. Requires prior token approval.
```
Parameters

_tokenAddress (address): Token contract address being withdrawn
_amountPlusfee (uint256): Total withdrawal amount including fees
_exchangeRate (uint256): Current exchange rate for the token
_feeAmount (uint256): Fee amount to be deducted

Returns

nonce (uint256): Unique identifier for the withdrawal transaction
```

- Example
```
// Approve contract first
await token.approve(contractAddress, amountPlusFee);

// Perform withdrawal
const tx = await contract.withdrawToEscrow(
    tokenAddress,
    amountPlusFee,
    exchangeRate,
    feeAmount
);
```

# Calling the swypt-offramp API endoint 
- After successful blockchain withdrawal transaction, now you can send a `POST` request to swypt backend API which will then process the offramp transaction, thereby sending the fiat amount to the user MPesa mobile money phone number. The following example below explains 

- `POST` request payload params

| Parameter | Description | Example | Required |
| --- | --- | --- | --- |
| partyB | Recipient's phone number (international format) | 254703710518 | Yes |
| tokenAddress | Token contract address | 0x55d398326f99059fF775485246999027B3197955 | Yes |
| hash | Withdrawal transaction hash | 0xe15dff0f39cdba04f2c0888935902552060a12474311bc6cb150ed0f012d665c | Yes |
| chain | Blockchain network identifier | base | Yes |


-  Here is how you can send the request using axios to the swypt-offramp api 
```
const axios = require('axios');

async function notifyWithdrawal(transactionHash) {
    try {
        const response = await axios.post('https://api.swypt.io/api/token-pay-offramp', {
            partyB: "254703710518",
            tokenAddress: "0x55d398326f99059fF775485246999027B3197955",
            hash: transactionHash,
            chain: "bsc"
        });
        return response.data;
    } catch (error) {
        console.error('Error notifying withdrawal:', error);
        throw error;
    }
}
```

