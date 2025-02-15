# token-pay-swypt-documentation
This repo serves as a documentation for integration with swypt APIs and how to integrate with our smart contract functions. 

Deployed Contract on POLYGON
[Swypt Contract](https://polygonscan.com/address/0x980b2f387bbecd67d94b2b6eebd4fd238946466a#code)


# Getting Quotes
Before performing any on-ramp or off-ramp operations, you should first get a quote to determine rates, fees, and expected output amounts.

## Get Quote Endpoint
`POST https://pool.swypt.io/api/quotes`

Get quote for converting between fiat and crypto currencies.

### Request Parameters

| Parameter | Description | Example | Required |
| --- | --- | --- | --- |
| type | Type of operation ('onramp' or 'offramp') | "onramp" | Yes |
| amount | Amount to convert | "5000" | Yes |
| fiatCurrency | Fiat currency code | "KES" | Yes |
| cryptoCurrency | Cryptocurrency symbol | "USDT" | Yes |
| network | Blockchain network | "Polygon" | Yes |
| category | Transaction category (for offramp only) | "B2C" | No |

### Example Requests

1. Onramp (Converting KES to USDT):
```javascript
const response = await axios.post('https://pool.swypt.io/api/quotes', {
  type: "onramp",
  amount: "5000",
  fiatCurrency: "KES",
  cryptoCurrency: "USDT",
  network: "Polygon"
});
```

2. Offramp (Converting USDT to KES):
```javascript
const response = await axios.post('https://pool.swypt.io/api/quotes', {
  type: "offramp",
  amount: "100",
  fiatCurrency: "KES",
  cryptoCurrency: "USDT",
  network: "Polygon",
  category: "B2C"
});
```

### Response Format
```json
{
  "statusCode": 200,
  "message": "Quote retrieved successfully",
  "data": {
    "inputAmount": "2",
    "outputAmount": 255.72,
    "inputCurrency": "USDT",
    "outputCurrency": "KES",
    "exchangeRate": 0.99,
    "type": "offramp",
    "network": "Polygon",
    "fee": {
      "amount": 0.05,
      "currency": "USDT",
      "details": {
        "feeInKES": 6,
        "estimatedOutputKES": 258.3
      }
    }
  }
}

```

### Error Response
```json
{
  "statusCode": 400,
  "message": "Invalid network",
  "error": "Unsupported network. Supported networks: Lisk, Celo, Base, Polygon"
}
```

## Get Supported Assets
`GET https://pool.swypt.io/api/supported-assets`

Retrieve all supported assets, networks, and currencies.

### Response Format
```json
{
  "statusCode": 200,
  "message": "Assets retrieved successfully",
  "data": {
    "networks": ["Lisk", "Celo", "Base", "Polygon"],
    "fiat": ["KES"],
    "crypto": {
      "Polygon": [
        {
          "symbol": "USDT",
          "name": "Tether Polygon",
          "decimals": 6,
          "address": "0xc2132D05D31c914a87C6611C10748AEb04B58e8F"
        }
      ]
      // ... other networks
    }
  }
}
```

# Create Offramp Ticket
`POST /api/user-offramp-ticket`

Create a ticket for offramp transactions. This endpoint handles both new transactions and failed/pending transaction cases.

## Request Parameters

| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| orderID | ID of failed/pending transaction (for retries) | No | "ORD123456" |
| phone | Recipient's phone number | Yes | "254703710518" |
| amount | Transaction amount | Yes | "100" |
| description | Transaction description | Yes | "USDT withdrawal" |
| side | Transaction side | Yes | "off-ramp" |
| userAddress | User's blockchain address | Yes | "0x123..." |
| symbol | Token symbol | Yes | "USDT" |
| tokenAddress | Token contract address | Yes** | "0x55d398326f99..." |
| chain | Blockchain network | Yes | "Polygon" |


\* Required only for retrying failed/pending transactions
\** Required for new transactions (when orderID is not provided)

## Example Requests

1. Creating a new offramp ticket:
```javascript
const response = await axios.post('https://pool.swypt.io/api/user-offramp-ticket', {
  orderID: "ORD123456" //optional
  phone: "254703710518",
  amount: "100",
  description: "USDT withdrawal to M-Pesa",
  side: "off-ramp",
  userAddress: "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  symbol: "USDT",
  tokenAddress: "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
  chain: "Polygon"
});
```

2. Retrying a failed transaction:
```javascript
const response = await axios.post('https://pool.swypt.io/api/user-offramp-ticket', {
  orderID: "ORD123456",
  symbol: "USDT",  // Optional override
  chain: "Polygon" // Optional override
});
```

## Success Response
```json
{
  "status": "success",
  "data": {
    "refund": {
      "PhoneNumber": "254703710518",
      "Amount": "100",
      "Description": "USDT withdrawal to M-Pesa",
      "Side": "off-ramp",
      "userAddress": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
      "symbol": "USDT",
      "tokenAddress": "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
      "chain": "Polygon",
      "_id": "507f1f77bcf86cd799439011",
      "createdAt": "2024-02-14T12:00:00.000Z"
    }
  }
}
```

## Error Responses

1. Missing Required Fields:
```json
{
  "status": "error",
  "message": "Please provide all required inputs: phone, amount, description, side, userAddress, symbol, tokenAddress, and chain"
}
```

2. Invalid Order ID:
```json
{
  "status": "error",
  "message": "No failed or pending transaction found with this orderID"
}
```

3. Server Error:
```json
{
  "status": "error",
  "message": "Unable to process refund ticket"
}
```


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

```
- Returns 
nonce (uint256): Unique identifier for the withdrawal transaction



# withdrawToEscrow
Performs a token withdrawal to an escrow account. Requires prior token approval.
```
Parameters

_tokenAddress (address): Token contract address being withdrawn
_amountPlusfee (uint256): Total withdrawal amount including fees
_exchangeRate (uint256): Current exchange rate for the token
_feeAmount (uint256): Fee amount to be deducted

```
- Returns
nonce (uint256): Unique identifier for the withdrawal transaction

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


# Offramp Transaction Processing

## Initiate Offramp Transaction
`POST https://pool.swypt.io/api/swypt-offramp`

Process an offramp transaction after successful blockchain withdrawal.

### Request Parameters

| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| chain | Blockchain network | Yes | "Celo" |
| hash | Transaction hash from blockchain | Yes | "0x80856f025..." |
| partyB | Recipient's phone number | Yes | "254703710518" |
| tokenAddress | Token contract address | Yes | "0x48065fbBE..." |

### Example Request
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-offramp', {
  chain: "Celo",
  hash: "0x80856f025035da9387873410155c4868c1825101e2c06d580aea48e8179b5e0b",
  partyB: "254703710518",
  tokenAddress: "0x48065fbBE25f71C9282ddf5e1cD6D6A887483D5e"
});
```

### Success Response
```json
{
  "status": "success",
  "message": "Withdrawal payment initiated successfully",
  "data": {
    "orderID": "WD-xsy6e-HO"
  }
}
```

### Error Response
```json
{
  "status": "error",
  "message": "This blockchain transaction has already been processed",
  "data": {
    "orderID": "WD-xsy6e-HO"
  }
}
```

## Check off-ramp Transaction Status
`GET https://pool.swypt.io/api/swypt-offramp-status/:orderID`

Check the status of an offramp transaction using its orderID.

### Parameters
| Parameter | Location | Description | Required | Example |
| --- | --- | --- | --- | --- |
| orderID | URL | Transaction order ID | Yes | "WD-xsy6e-HO" |

### Example Request
```javascript
const response = await axios.get('https://pool.swypt.io/api/swypt-offramp-status/WD-xsy6e-HO');
```

### Success Response
```json
{
  "status": "success",
  "data": {
    "status": "SUCCESS",
    "message": "Withdrawal completed successfully",
    "details": {
      "phoneNumber": "254703710518",
      "ReceiverPartyPublicName": "254703710518 - Henry Kariuki Nyagah",
      "transactionSize": "20.00",
      "transactionSide": "withdraw",
      "initiatedAt": "2025-02-02T12:45:21.859Z",
      "mpesaReceipt": "TB21GOZTI9",
      "completedAt": "2025-02-02T15:45:23.000Z"
    }
  }
}
```

### Possible Status Values
- `PENDING`: Transaction is being processed
- `SUCCESS`: Transaction completed successfully
- `FAILED`: Transaction failed

### Error Responses

1. Missing Order ID:
```json
{
  "status": "error",
  "message": "orderID ID is required"
}
```

2. Transaction Not Found:
```json
{
  "status": "error",
  "message": "Transaction with the following WD-xsy6e-HO the not found"
}
```

3. Server Error:
```json
{
  "status": "error",
  "message": "Failed to check withdrawal status"
}
```

### Response Details by Status

1. Pending Transaction:
```json
{
  "status": "success",
  "data": {
    "status": "PENDING",
    "message": "Your withdrawal is being processed",
    "details": {
      "phoneNumber": "254703710518",
      "transactionSize": "20.00",
      "transactionSide": "withdraw",
      "initiatedAt": "2025-02-02T12:45:21.859Z"
    }
  }
}
```

2. Failed Transaction:
```json
{
  "status": "success",
  "data": {
    "status": "FAILED",
    "message": "Withdrawal failed",
    "details": {
      "phoneNumber": "254703710518",,
      "transactionSize": "20.00",
      "transactionSide": "withdraw",
      "initiatedAt": "2025-02-02T12:45:21.859Z",
      "failureReason": "Transaction timeout",
      "resultCode": "1234"
    }
  }
}
```





