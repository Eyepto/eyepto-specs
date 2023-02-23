# Generate QR codes for Eyepto wallet

Generate your QR codes using the following JSON templates:

## Simple transaction

A simple ETH transaction with a target address:

```
{
  "n": "1",
  "t": "v",
  "a": "123456",
  "ta": "0x1230000000000000000000000000000000000123"
}
```

### Explanation

What every param means.

* **"n"** is the network ID, **1** for mainnet, 137 for polygon etc.
* **"t"** is the type, **v** for value transfer
* **"a"** is the amount, in this example **123456** wei
* **"ta"** is the target address, the address we are sending the network's native token to

### Result QR code

![QR code for value](./assets/qrcode-value-transfer.png)

## Smart contract call

A call to a smart contract using the called function ABI's and data to be sent:

```
{
  "n": "1",
  "t": "f",
  "a": "123321",
  "ta": "0x1112223330000000000000000000000000000000",
  "d": {
    "a": {
      "constant": false,
      "inputs": [
        {
          "name": "inputOne",
          "type": "address"
        },
        {
          "name": "otherInput",
          "type": "uint256"
        }
      ],
      "name": "methodName",
      "outputs": [
        {
          "name": "",
          "type": "uint256"
        }
      ],
      "payable": true,
      "stateMutability": "payable",
      "type": "function"
    },
    "i": {
      "inputOne": "0x1230000000000000000000000000000000000321",
      "otherInput": "123"
    }
  }
}
```

### Explanation

What every param means.

* **"n"** is the network ID, **1** for mainnet, 137 for polygon etc.
* **"t"** is the type, **f** for function call
* **"a"** is the amount, in this example **123321** wei
* **"ta"** is the target address, the smart contract to call
* **"d"** is the data to be used for the smart contract call, contains **"a"** and **"i"**
* **"a"** is the abi of the method to call, you only need to specify the method to call
* **"i"** is the input of the method to call, it has to match the inputs list in the ABI of the function

### Result QR code

![QR code for smart contract call](./assets/qrcode-function-call.png)

## Sequence of transactions

Eyepto wallet users can send multiple, sequenced calls if you display a QR code in this format (they'll be called one after the other, and the user will only need to confirm them once):

```
{
  "t": "ts",
  "tl": [
    {
      "n": "1",
      "t": "f",
      "a": "123321",
      "ta": "0x1112223330000000000000000000000000000000",
      "d": {
        "a": {
          "constant": false,
          "inputs": [
            {
              "name": "inputOne",
              "type": "address"
            },
            {
              "name": "otherInput",
              "type": "uint256"
            }
          ],
          "name": "methodName",
          "outputs": [
            {
              "name": "",
              "type": "uint256"
            }
          ],
          "payable": true,
          "stateMutability": "payable",
          "type": "function"
        },
        "i": {
          "inputOne": "0x1230000000000000000000000000000000000321",
          "otherInput": "123"
        }
      }
    },
    ...
  ]
}
```

As you can see, you can mix networks and types. The formatting of the transactions inside of the array is the same as we've seen before. For now, only function and value types are accepted in transactions sequence.

### Explanation

What every param means.

* **"t"** is the type, **st** for sequenced transactions.
* **"tl"** contains an array of all the transactions.

### QR code too large?

If you need to call several transactions and the QR code is too big, you can use the following endpoint to store the transaction data and show only a QR code with the link to the user.

## Storing a transaction

You can store the transaction by making a POST call to /transactions, and showing the user the link as a QR code. 

#### Example:

**1) Generate an ID to use for the transaction, UUIDv4 are recommended, but any string shorter than 64 with url safe characters will do. This step can be done directly on the frontend of the DApp.**

```
import { uuid } from 'uuidv4';
const transactionID = uuid();
```

**2) You can create a transaction as follow:**

```
const url = "https://api.eyepto.com/transactions";
const transactionData = {
  // Your transaction information here. Function, value, or lists are accepted ("t" = "f" or "v" or "tl")
}

const data = {
	"transactionID": transactionID,
  "transaction": transactionData
}

const response = await fetch(url, {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
});

// const res = response.json(); // if you need to read the response
```

**3) Display a QR code to the user containing the transaction url you generated.**

```
// import { uuid } from 'uuidv4';

import QRCode from "react-qr-code";

// const transactionID = ..

const urlToQRify = `https://api.eyepto.com/transactions?transactionID=${transactionID}`

function SuperDAppComponent() {
  return (
    <QRCode style={{border: "white 10px solid"}} value={urlToQRify} />
  )
}
```
### Note:

For now only eyepto.com urls are recognized by the mobile app, but other, potentially decentralized, options will be made available when all risks related to them are assessed.

## Verify wallet

Get a user wallet address:

```
{
  "t": "ve",
  "id": "00000a00-aaa0-00a0-0a00-00000a000000",
  "pn": "Super DApp"
}
```

### Explanation

What every param means.
* **"t"** is the type, **ve** for verification
* **id** is the request ID, preferably a UUIDv4 string.
* **pn** dapp/project name to show in the DApp.

### Result QR code

![QR code for wallet verification](./assets/qr-code-basic-verification.png)

### How to read the user address?
Make a GET call to /verify-wallet on the Eyepto backend API with the param requestID.

#### Example:

**1) Generate an ID to use for the verification, UUIDv4 are recommended, but any string shorter than 64 characters will do. This step can be done directly on the frontend of the DApp.**

```
import { uuid } from 'uuidv4';
const requestID = uuid();
```

**(*Optional*) You can create a request for verification, this step while possible, is unnecessary unless you need it for some reason:**

```
// Optional step, pre-populate request for verification
const url = "https://api.eyepto.com/verify-wallet";
const data = {
	"requestID": requestID
}

const response = await fetch(url, {
    method: "POST",
    headers: {
        "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
});

// const res = response.json(); // if you need to read the response
```

**2) Display a QR code to the user containing the requestID you generated.**

```
// import { uuid } from 'uuidv4';

import QRCode from "react-qr-code";

// const requestID = uuid();

const dataToQRify = {
  "t": "ve",
  "id": requestID,
  "pn": "Name of your DApp"
}

function SuperDAppComponent() {
  return (
    <QRCode style={{border: "white 10px solid"}} value={JSON.stringify(dataToQRify, null, 0)} />
  )
}
```

**3) Requesting verification result (recommended interval, every 5 seconds):**

```
const url = `https://api.eyepto.com/verify-wallet?requestID=${requestID}`;

const response = await fetch(url, {
    method: "GET",
});

const verifRes = response.json(); // will contain the address
```

The user address (if verified), should be inside of the "data" object. 

This call is meant to be made at a certain interval from the DApp frontend until a result address is found.

From a UX perspective, if the user hasn't verified their wallet address after certain amount of time, you could just ask them to re-verify their address, perhaps with a newly generated requestID.

### Note

UUIDv4 are recommended for the request ID, altough any string under 64 characters is supported.[uuid](https://www.npmjs.com/package/uuid) is recommended for generating UUIDs.


### Signing a message for verification

Signing a message for verifying ownership of the wallet address is not supported yet and will be added later.

---

This document is a WIP. Other types of transactions like legacy ones (pre-compiled ones with web3) and bundled transfers are yet to be added.
