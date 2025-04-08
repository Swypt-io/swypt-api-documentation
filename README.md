# SWYPT API DOCUMENTATION FOR ON-RAMPS & OFF-RAMPS
This repo serves as a documentation for integration with swypt APIs and how to integrate with our smart contract functions. 

Deployed Contract on POLYGON
[Swypt Contract](https://polygonscan.com/address/0x980b2f387bbecd67d94b2b6eebd4fd238946466a#code)

Deployed Contract on LISK
[Swypt Contract](https://blockscout.lisk.com/address/0x2816a02000B9845C464796b8c36B2D5D199525d5)

Deployed Contract on CELO
[Swypt Contract](https://celoscan.io/address/0x2816a02000B9845C464796b8c36B2D5D199525d5#code)



# Getting Quotes
Before performing any on-ramp or off-ramp operations, you should first get a quote to determine rates, fees, and expected output amounts.

## Get Quote Endpoint
`POST https://pool.swypt.io/api/swypt-quotes`

Get quote for converting between fiat and crypto currencies.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Request Parameters
| Parameter | Description | Example | Required |
| --- | --- | --- | --- |
| type | Type of operation ('onramp' or 'offramp') | "onramp" | Yes |
| amount | Amount to convert | "5000" | Yes |
| fiatCurrency | Fiat currency code | "KES" | Yes |
| cryptoCurrency | Cryptocurrency symbol | "USDT" | Yes |
| network | Blockchain network | "Celo" | Yes |
| category | Transaction category (for offramp only) | "B2C" | No |

### Example Requests
1. Onramp (Converting KES to USDT):
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-quotes', {
  type: "onramp",
  amount: "100",
  fiatCurrency: "KES",
  cryptoCurrency: "USDT", //cKes, USDC
  network: "Celo"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```

2. Offramp (Converting USDT to KES):
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-quotes', {
  type: "offramp",
  amount: "2",
  fiatCurrency: "KES",
  cryptoCurrency: "USDT",
  network: "Celo",
  category: "B2C"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```

### Response Format For Successful Quote Request for Off-Ramping
```json
{
  "statusCode": 200,
  "message": "Quote retrieved successfully",
  "data": {
    "inputAmount": "2",
    "outputAmount": 255.72,
    "inputCurrency": "USDT",
    "outputCurrency": "KES",
    "exchangeRate": 128.3556,
    "type": "offramp",
    "network": "Celo",
    "fee": {
      "amount": 0.05,
      "currency": "USDT",
      "details": {
        "feeInKES": 6,
        "estimatedOutputKES": 261.72
      }
    }
  }
}
```

### Response Format For Successful Quote Request for On-Ramping
```json
{
  "statusCode": 200,
  "message": "Quote retrieved successfully",
  "data": {
    "inputAmount": "200",
    "outputAmount": 1.4999,
    "inputCurrency": "KES",
    "outputCurrency": "USDT",
    "exchangeRate": 133.345489,
    "type": "onramp",
    "network": "Celo",
    "fee": {
      "amount": 0.03097,
      "currency": "USDT",
      "details": {
        "feeInKES": 4,
        "estimatedOutputKES": 104
      }
    }
  }
}
```

### Error Responses

#### Authentication Error
```json
{
  "status": "error",
  "message": "API key and secret are required"
}
```

```json
{
  "status": "error",
  "message": "Invalid API credentials"
}
```

#### Validation Error
```json
{
  "statusCode": 400,
  "message": "Invalid network",
  "error": "Unsupported network. Supported networks: Lisk, Celo, Base, Polygon"
}
```

#### Server Error
```json
{
  "error": "Internal server error"
}
```

## Get Supported Assets
`GET https://pool.swypt.io/api/swypt-supported-assets`

Retrieve all supported assets, networks, and currencies.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Example Request
```javascript
const response = await axios.get('https://pool.swypt.io/api/swypt-supported-assets', {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```

### Response Format
```json
{
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
    ],
    "Celo": [
      {
        "symbol": "cKES",
        "name": "Celo KES",
        "decimals": 18,
        "address": "0x3a0d9d7764FAE860A659eb96A500F1323b411e68"
      },
      {
        "symbol": "cUSD",
        "name": "Celo Dollar",
        "decimals": 18,
        "address": "0x765DE816845861e75A25fCA122bb6898B8B1282a"
      }
    ],
    "Base": [
      {
        "symbol": "USDbC",
        "name": "USD Base Coin",
        "decimals": 6,
        "address": "0xd9aAEc86B65D86f6A7B5B1b0c42FFA531710b6CA"
      }
    ],
    "Lisk": [
      {
        "symbol": "LSK",
        "name": "Lisk",
        "decimals": 8,
        "address": "0x0000000000000000000000000000000000000000"
      }
    ]
  }
}
```

### Error Responses

#### Authentication Error
```json
{
  "status": "error",
  "message": "API key and secret are required"
}
```

```json
{
  "status": "error",
  "message": "Invalid API credentials"
}
```

#### Server Error
```json
{
  "error": "Internal server error"
}
```


# OFF-RAMP FLOW 
- Swypt offramp involves the following steps 
1. Calling the Swypt smart contract `withdrawToEscrow` or `withdrawWithPermit` functions and perform a blockchain transaction
2. Call swypt offramp API  endpoint with a payload containing the required params as discussed below to initiate an offramp transaction

# withdrawWithPermit
Enables token withdrawal using EIP-2612 permit, eliminating the need for a separate approval transaction

`Parameters`

```javascript
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

```javascript
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
```javascript
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
`POST https://pool.swypt.io/api/swypt-order-offramp`
Process an offramp transaction after successful blockchain withdrawal.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Request Parameters
| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| chain | Blockchain network | Yes | "Celo" |
| hash | Transaction hash from blockchain | Yes | "0x80856f025..." |
| partyB | Recipient's phone number | Yes | "254703710518" |
| tokenAddress | Token contract address | Yes | "0x48065fbBE..." |
| args | Additional parameters (required for ICP chain) | Conditional | See ICP example |

#### ICP Chain Additional Parameters
When using the `chain: "icp"`, the following additional parameters are required in the `args` object:

| Parameter | Description | Required for ICP | Example |
| --- | --- | --- | --- |
| blockHeight | Block height of the transaction | Yes | "123456" |
| amountIn | Amount of tokens transferred | Yes | "100" |
| from | Sender address | Yes | "icp1234..." |
| to | Recipient address | Yes | "icp5678..." |
| exchangeRate | Token to fiat exchange rate | Yes | "1.5" |
| fee | Transaction fee | Yes | "0.1" |

### Example Request (Standard Chain)
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-order-offramp', {
  chain: "Celo",
  hash: "0x80856f025035da9387873410155c4868c1825101e2c06d580aea48e8179b5e0b",
  partyB: "254703710518",
  tokenAddress: "0x48065fbBE25f71C9282ddf5e1cD6D6A887483D5e"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});

```
### Example Request (ICP chain)
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-order-offramp', {
  chain: "icp",
  hash: "",  // For ICP, hash is generated server-side
  partyB: "254703710518",
  tokenAddress: "icp_token_address",
  args: {
    blockHeight: "123456",
    amountIn: "100",
    from: "icp1234...",
    to: "icp5678...",
    exchangeRate: "1.5",
    fee: "0.1"
  }
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});

````

### Successful Response

```json
{
  "status": "success",
  "message": "Withdrawal payment initiated successfully",
  "data": {
    "orderID": "WD-xsy6e-HO"
  }
}
```

### Error Responses
```json
  "status": "error",
  "message": "This blockchain transaction has already been processed",
  "data": {
    "orderID": "WD-xsy6e-HO"
  }
}
```

```json
{
  "status": "error",
  "message": "Missing required parameters: tokenAddress, hash, or phone number"
}
```

```json
{
  "status": "error",
  "message": "Missing required ICP parameters"
}
```
```json
{
  "status": "error",
  "message": "Unsupported blockchain: [chain name]"
}
```

## Check off-ramp Transaction Status
`GET https://pool.swypt.io/api/order-offramp-status/:orderID`
Check the status of an offramp transaction using its orderID.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Parameters
| Parameter | Location | Description | Required | Example |
| --- | --- | --- | --- | --- |
| orderID | URL | Transaction order ID | Yes | "WD-xsy6e-HO" |

### Example Request
```javascript
const response = await axios.get('https://pool.swypt.io/api/order-offramp-status/WD-xsy6e-HO', {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```

### Successful Response

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
## Possible Status Values

`PENDING`: Transaction is being processed
`SUCCESS`: Transaction completed successfully
`FAILED`: Transaction failed

### Error Responses

1. Missing Order ID:
```json
{
  "status": "error",
  "message": "orderID ID is required"
}
```
2. Transaction not found
```json
{
  "status": "error",
  "message": "Transaction with the following WD-xsy6e-HO the not found"
}
```
3. Server Error
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
      "ReceiverPartyPublicName": "254703710518 - Henry Kariuki Nyagah",
      "transactionSize": "20.00",
      "transactionSide": "withdraw",
      "initiatedAt": "2025-02-02T12:45:21.859Z"
    }
  }
}
```
2. Failed Transaction
```json
{
  "status": "success",
  "data": {
    "status": "FAILED",
    "message": "Withdrawal failed",
    "details": {
      "phoneNumber": "254703710518",
      "ReceiverPartyPublicName": "254703710518 - Henry Kariuki Nyagah",
      "transactionSize": "20.00",
      "transactionSide": "withdraw",
      "initiatedAt": "2025-02-02T12:45:21.859Z",
      "failureReason": "Transaction timeout",
      "resultCode": "1234"
    }
  }
}
```

# Create Offramp Ticket
`POST https://pool.swypt.io/api/create-offramp-ticket`
Create a ticket for offramp transactions. This endpoint supports two methods:
1. Creating a ticket from a failed/pending transaction using orderID
2. Creating a new ticket directly with all required information

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Request Parameters
| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| orderID | ID of failed/pending transaction | No* | "WD-xsy6e-HO" |
| phone | User's phone number | Yes** | "254703710518" |
| amount | Crypto amount | Yes** | "100" |
| description | Ticket description | Yes** | "Failed withdrawal attempt" |
| side | Transaction side | Yes** | "off-ramp" |
| userAddress | User's blockchain address | Yes** | "0x742d35..." |
| symbol | Token symbol | Yes** | "USDT" |
| tokenAddress | Token contract address | Yes** | "0xc2132D05..." |
| chain | Blockchain network | Yes** | "Polygon" |

\* Either `orderID` OR all other fields are required  
\** Required only when `orderID` is not provided

### Example Requests
1. Creating ticket from failed/pending transaction:
```javascript
const response = await axios.post('https://pool.swypt.io/api/create-offramp-ticket', {
  orderID: "WD-xsy6e-HO",
  description: "Refund for failed withdrawal",
  symbol: "USDT",  // Optional override
  chain: "Polygon" // Optional override
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```
2. Creating new ticket direclty 
```javascript
const response = await axios.post('https://pool.swypt.io/api/create-offramp-ticket', {
  phone: "254703710518",
  amount: "100",
  description: "Failed withdrawal attempt",
  side: "off-ramp",
  userAddress: "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  symbol: "USDT",
  tokenAddress: "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
  chain: "Polygon"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});

```
### Successful Response
```json
{
  "status": "success",
  "data": {
    "refund": {
      "PhoneNumber": "254703710518",
      "Amount": "100",
      "Description": "Failed withdrawal attempt",
      "Side": "off-ramp",
      "userAddress": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
      "symbol": "USDT",
      "tokenAddress": "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
      "chain": "Polygon",
      "_id": "507f1f77bcf86cd799439011",
      "createdAt": "2025-02-15T12:00:00.000Z"
    }
  }
}
```
### Error Responses

1.Missing Required Fields:

```json
{
  "status": "error",
  "message": "Please provide all required inputs: phone, amount, description, side, userAddress, symbol, tokenAddress, and chain"
}
```
2.Invalid orderID
```json
{
  "status": "error",
  "message": "No failed or pending transaction found with this orderID"
}
```
3.Validation Error
```json
{
  "status": "error",
  "message": "ValidationError: [specific validation message]"
}
```
4.Server Error
```json
{
  "status": "error",
  "message": "Unable to process refund ticket"
}
```
### Important Notes

- Either provide orderID OR all other required fields
- When using orderID, the ticket will be created using data from the failed or pending transaction
- You can override specific fields (symbol, chain) even when using orderID
- Both failed AND pending transactions can be used with orderID
- All fields are required when creating a ticket directly without orderID
- Phone numbers should be in international format (e.g., "254703710518")
- The system automatically sets side to 'off-ramp' if not provided



### ONRAMP PROCESS FLOW
## 1. Initiate STK Push
`POST https://pool.swypt.io/api/swypt-onramp`
Initiates the M-Pesa STK push process for fiat deposit.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Request Parameters
| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| partyA | User's phone number | Yes | "254703710518" |
| amount | Amount in KES | Yes | "5000" |
| side | Transaction side | Yes | "onramp" |
| userAddress | User's blockchain address | Yes | "0x742d35Cc6634C0532925a3b844Bc454e4438f44e" |
| tokenAddress | Token contract address | Yes | "0xc2132D05D31c914a87C6611C10748AEb04B58e8F" |

### Example Request
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-onramp', {
  partyA: "254703710518",
  amount: "5000",
  side: "onramp",
  userAddress: "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  tokenAddress: "0xc2132D05D31c914a87C6611C10748AEb04B58e8F"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```
### Successful Response
```json
{
  "status": "success",
  "message": "STK Push initiated successfully",
  "data": {
    "orderID": "D-rclsg-VL",
    "message": "Success. Request accepted for processing"
  }
}
```
### Error Responses

1.Missing Required Parameters:
```json
{
  "status": "error",
  "message": "Failed to record the onramp transaction order"
}
```
2. STK Push Failure
```json
{
  "status": "error",
  "message": "Failed to initiate STK Push payment"
}
```
3.Server Error
```json
{
  "status": "error",
  "message": "An error occurred during the STK Push transaction"
}
```

## 2. Check Onramp Status
`GET https://pool.swypt.io/api/order-onramp-status/:orderID`
Check the status of an STK push transaction.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### URL Parameters
| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| orderID | Transaction order ID | Yes | "D-rclsg-VL" |

### Example Request
```javascript
const response = await axios.get('https://pool.swypt.io/api/order-onramp-status/D-rclsg-VL', {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```
## Response Scenarios

1.Successful Transaction:
```json
{
  "status": "success",
  "data": {
    "status": "SUCCESS",
    "message": "Deposit completed successfully",
    "orderID": "D-ri3b1-7H",
    "details": {
      "phoneNumber": "254703710518",
      "mpesaReceipt": "TBF842GPCO",
      "transactionDate": "2025-02-15T08:33:38.000Z",
      "cryptoAmount": "100",
      "symbol": "USDT",
      "transactionHash": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
      "resultDescription": "Transaction initiated",
      "chain": "Ethereum"
    }
  }
}
```
2.Failed Transaction:
```json
{
  "status": "success",
  "data": {
    "status": "FAILED",
    "message": "Insufficient balance",
    "orderID": "D-rm3qn-3Q",
    "details": {
      "phoneNumber": "254703710518",
      "mpesaReceipt": "ws_CO_15022025083640743703710518",
      "transactionDate": "2025-02-15T08:36:40.000Z",
      "cryptoAmount": "",
      "symbol": "",
      "transactionHash": "",
      "resultDescription": "Insufficient balance",
      "chain": ""
    }
  }
}
```
3.Pending Transaction:
```json
{
  "status": "success",
  "data": {
    "status": "PENDING",
    "message": "Your deposit is being processed",
    "orderID": "D-roug7-UT",
    "details": {
      "phoneNumber": "254703710518",
      "mpesaReceipt": "ws_CO_15022025083848183703710518",
      "transactionDate": "2025-02-15T08:38:48.000Z",
      "cryptoAmount": "",
      "symbol": "",
      "transactionHash": "",
      "resultDescription": "Transaction initiated",
      "chain": ""
    }
  }
}
```
## Error Responses

1. Missing Order ID:
```json
{
  "status": "error",
  "message": "orderID is required"
}
```
2. Transaction Not Found:
```json
{
  "status": "error",
  "message": "Transaction D-rclsg-VL not found"
}
```
3.Server Error:
```json
{
  "status": "error",
  "message": "Failed to check deposit status"
}
```


## 3. Process Crypto Transfer To User
`POST https://pool.swypt.io/api/swypt-deposit`
Process the crypto transfer after successful M-Pesa payment.

### Authentication Required
This endpoint requires API key authentication. Include the following headers with your request:
- `x-api-key`: Your API key
- `x-api-secret`: Your API secret

### Request Parameters
| Parameter | Description | Required | Example |
| --- | --- | --- | --- |
| chain | Blockchain network | Yes | "Celo" |
| address | Recipient address | Yes | "0x742d35..." |
| orderID | Original transaction order ID | Yes | "D-ri3b1-7H" |
| project | Project identifier | No | "onramp" |

### Example Request
```javascript
const response = await axios.post('https://pool.swypt.io/api/swypt-deposit', {
  chain: "Celo",
  address: "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  orderID: "D-ri3b1-7H",
  project: "onramp"
}, {
  headers: {
    'x-api-key': 'YOUR_API_KEY',
    'x-api-secret': 'YOUR_API_SECRET'
  }
});
```
## Success Response
```json
{
  "status": 200,
  "message": "Transaction processed successfully",
  "createdAt": "2025-02-15T08:33:38.000Z",
  "updatedAt": "2025-02-15T08:33:45.000Z",
  "hash": "0x80856f025035da9387873410155c4868c1825101e2c06d580aea48e8179b5e0b"
}
```
## Error Responses

1. Missing Parameters:
```json
{
  "status": "error",
  "message": "Missing required parameters: address or orderID"
}
```
2.Transaction Not Found:
```
{
  "status": "error",
  "message": "No transaction found for orderID: D-ri3b1-7H"
}
```
3. Transaction Pending:
```json
{
  "status": "error",
  "message": "Transaction D-ri3b1-7H: STK push payment is being processed"
}
```
4. Transaction Failed:
```json
{
  "status": "error",
  "message": "Transaction D-ri3b1-7H: Payment failed"
}
```
5.Transaction Already Processed:
```json
{
  "status": "error",
  "message": "Transaction D-ri3b1-7H has already been processed"
}
```
6. Invalid Chain:
```json
{
  "status": "error",
  "message": "Invalid chain: XYZ"
}
```
7. Unsupported Chain:
```json
{
  "status": "error",
  "message": "XYZ chain is not supported"
}
```
8.Processing Error:
```json
{
  "status": "error",
  "message": "Failed to process [Chain] transaction: [specific error message]"
}
```
### Supported Chains

- Celo
- Base
- Lisk
- icp (Internet Computer Protocol)

## Notes

- This endpoint should only be called after a successful STK push payment (status: SUCCESS)
- The system automatically calculates exchange rates and fees based on the payment amount
- For ICP chain, the system uses a predefined identity to process transactions


# Create Onramp Ticket
`POST /api/user-onramp-ticket`

Create a ticket for onramp transactions. This endpoint supports two methods:
1. Creating a ticket from a failed/cancelled transaction using orderID
2. Creating a new ticket directly with all required information

## Request Parameters

| Parameter | Description | Required* | Example |
| --- | --- | --- | --- |
| orderID | ID of failed/cancelled transaction | No** | "D-rm3qn-3Q" |
| phone | User's phone number | Yes | "254703710518" |
| amount | Transaction amount | Yes | "5000" |
| description | Ticket description | Yes | "Failed STK push attempt" |
| side | Transaction side | Yes | "on-ramp" |
| userAddress | User's blockchain address | Yes | "0x742d35..." |
| symbol | Token symbol | Yes | "USDT" |
| tokenAddress | Token contract address | Yes | "0xc2132D05..." |
| chain | Blockchain network | Yes | "Polygon" |

\* Required for direct ticket creation
\** If orderID is provided, other fields become optional and will be populated from the failed transaction

## Example Requests

1. Creating ticket from failed transaction:
```javascript
const response = await axios.post('https://pool.swypt.io/api/user-onramp-ticket', {
  orderID: "D-rm3qn-3Q",
  description: "Refund for failed STK push",
  symbol: "USDT",  // Optional override
  chain: "Polygon" // Optional override
});
```

2. Creating new ticket directly:
```javascript
const response = await axios.post('https://pool.swypt.io/api/user-onramp-ticket', {
  phone: "254703710518",
  amount: "5000",
  description: "Failed deposit attempt",
  side: "on-ramp",
  userAddress: "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
  symbol: "USDT",
  tokenAddress: "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
  chain: "Polygon"
});
```

## Success Response
```json
{
  "status": "success",
  "data": {
    "refund": {
      "PhoneNumber": "254703710518",
      "Amount": "5000",
      "Description": "Failed deposit attempt",
      "Side": "on-ramp",
      "userAddress": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e",
      "symbol": "USDT",
      "tokenAddress": "0xc2132D05D31c914a87C6611C10748AEb04B58e8F",
      "chain": "Polygon",
      "_id": "507f1f77bcf86cd799439011",
      "createdAt": "2025-02-15T12:00:00.000Z"
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
  "message": "No failed transaction found with this orderID"
}
```

3. Validation Error:
```json
{
  "status": "error",
  "message": "ValidationError: [specific validation message]"
}
```

4. Server Error:
```json
{
  "status": "error",
  "message": "Unable to process refund ticket"
}
```

## Important Notes
- When using `orderID`, the ticket will be created using data from the failed transaction
- You can override specific fields (symbol, chain) even when using `orderID`
- Only failed or cancelled transactions can be used with `orderID`
- All fields are required when creating a ticket directly without `orderID`
- Phone numbers should be in international format (e.g., "254703710518")


### Supported Chains
- Lisk
- Celo
- Base
- ICP
- Polygon

Each chain has specific processing logic and may have different requirements or response formats.


