# QLI Pool Connect
Do you run your own Qubic miner?
Now you can connect your own miner to the QLI Pools.

The Distribution of your solutions is handled automatically.

Pool Connect uses a lightweight `Stratum-like` protocol.
The Pool Connect is `PPS+` based.

> [!NOTE]  
> Connect your miner to the pool through a secure web socket `wss://wps.qubic.li/stratum`

## The Protocol
The protocol is JSON based and runs through secure websockets.

### Base Packet
Every packet exchanged between client and server uses same base structure:
```json
{
  "Method": "StratumLogin", // the method you want to call / or received by the server
  "MessageVersion": 0 // default 0; defines the version of the message; reserved for future use

  // followed by specific fields per request/response
}
```

## Known Message Types
`StratumLogin` => Authenticate/Login

`StratumSubmit` => Submit a Solution

`StratumJob` => A job description

## General Flow

1. Connect to the RPC Service `wss://wps.qubic.li/stratum`
2. Authenticate yourself
3. Submit Solutions

## Connection and Authentication
Send a `StratumLogin`. Make sure you use either `AccessToken` **OR** `Wallet`. Both are not allowed!

** Request **
```json
{
    "Method": "StratumLogin", // the method you call
    "AccessToken": "", // the QLI Access Token (your user account)
    "Wallet": "", // a qubic wallet address 
    "Worker":"test", // the name of your worker
    "Os": "Linux" // OPTIONAL: the OS you run your miner
}
```

example:

```json
{
    "Method": "StratumLogin",
  "AccessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJJZCI6ImQzNzMyODc2LTY5ZDctNGI1OC1hNmUzLWM2MzZkMGQ4ZDE0NiIsIk1pbmluZyI6IiIsIm5iZiI6MTcyNTM3NjkyOSwiZXhwIjoxNzU2OTEyOTI5LCJpYXQiOjE3MjUzNzY5MjksImlzcyI6Imh0dHBzOi8vcXViaWMubGkvIiwiYXVkIjoiaHR0cHM6Ly9xdWJpYy5saS8ifQ.sregOyk2PEyXv8ssdQDBtTps1JFBLghcJCzDFvaD6hWoVA_T-crfQZbiV0E_atqd6sxNHYKGmeVCOoU9crLU4mnojZdF1vyp3VttB3ZIqo3qIgr0R4jWnwZ95bGN1c6NE3zb9y7ZWor5-4ttLkR_5moxiZZvaKG2WWSxFJ-7kk6SVSw7z8iaYyVpPX1Tdu6pBWxDStYYaoVvgNzx6RShU_r2AVCB1JGfv16vKvAIGmPcluvS-ayKwfgOpY1uEbsH6Lswd_KGbB1aJC7g8AI1CUoYiUUl_CJUBZfG0FbBgtGDRhfPUcYM5z8BEyIrm6bfKhMHuJmIF86NJYydRUHgow",
    "Worker":"test",
    "Os": "Linux"
}
```
if your login was successful you receive a job message and the connection stays alive. If there was an error the connection is closed.

## Receive a Job
When your miner is connected it will receive jobs automatically as `StraumJob` packet.
All binary data is HEX encoded.

> [!NOTE]  
> Jobs are broadcasted after authenticateion, solution submit or in an interval of ~30 seconds.

**Response**
```json
{
    "Epoch": 170, // active epoch 
    "RandomSeed": "", // HEX encoded random seed
    "PublicKey": "", // HEX encoded publicKey (target to mine to)
    "Difficulty": 316, // the actual active difficulty for the pool
    "DifficultyAddition": 74300, // the actual active difficulty for the pool
    "Method": "StratumJob", // defines that this is a job
    "MessageType": 501 // id of the message type = StratumJob
}
```

example job to work on:

```json
{
    "Epoch": 170,
    "RandomSeed": "A030176C55672400F4C6472076C1C15AB2BC0EFC16B6B6481578C3F3FF54D561",
    "PublicKey": "B43D85DF69E4E9C92938A214C5C20634D90768C0E98A94575FEB9AE53D3C60F8",
    "Difficulty": 316,
    "DifficultyAddition": 74300,
    "Method": "StratumJob",
    "MessageType": 501
}
```

Qubic allows to run multiple algorithms in parallel. `Difficulty` is always for `algo0` while `DifficultyAddition` is for `algo1`.


> [!IMPORTANT]  
> During the Qubic idle phase. Your miner will receive empty/idle jobs and should stop mining.

example job during idle (your miner can do other work but not working on Qubic):

```json
{
    "Epoch": 170,
    "RandomSeed": "0000000000000000000000000000000000000000000000000000000000000000",
    "PublicKey": "0000000000000000000000000000000000000000000000000000000000000000",
    "Difficulty": 316,
    "DifficultyAddition": 74300,
    "Method": "StratumJob",
    "MessageType": 501
}
```


## Submit A Solution
If your miner has found a solution, you can use `StratumSubmit`.
All binary data is HEX encoded.

**Request**
```json
{
    "Method": "StratumSubmit", // the method you call
    "Epoch":170, // the epoch; derived from StratumJob
    "RandomSeed": "A030176C55672400A4C6472076C1C15AB2BC0EFC16B6B6481578C3F3FF54D561", // the random seed; derived from StratumJob
    "PublicKey": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271", // the public key; derived from StratumJob
    "Nonce": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271" // the nonce; derived from StratumJob
}
```

example:

```json
{
    "Method": "StratumSubmit", 
    "Epoch":170,
    "RandomSeed": "A030176C55672400A4C6472076C1C15AB2BC0EFC16B6B6481578C3F3FF54D561",
    "PublicKey": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271",
    "Nonce": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271"
}
```
> [!IMPORTANT]
> The algorithm of the submitted Solutions/Shares is identified by `nonce[0]`.
> 
> nonce[0]      algo byte      (bit 0 selects Hyperidentity/Addition)
> 
> e.g. 0x00 -> Hyperidentity (algo 0, bit 0 = 0)
> 
> 0x01 -> Addition       (algo 1, bit 0 = 1)
> 
> (only bit 0 is read: any even byte = Hyperidentity, any odd byte = Addition)


> [!IMPORTANT]  
> Solutions/Shares are validated async. When your miner sends a solution you will get a message which indicates if the solution was accepted for validation or it was rejected. An accepted solution may be not valid and not count toward your account.

**Response**
```json
{
    "Success": false, // indicates if your solution was accepted for validation or not
    "Message": "Obsolete MiningSeed", // message from the server;
    "RandomSeed": "", // HEX encoded randomSeed for your reference
    "PublicKey": "", // HEX encoded publicKey for your reference
    "Nonce": "", // HEX encoded nonce for your reference
    "Method": "StratumSubmit",
    "MessageType": 502
}
```
example:
```json
{
    "Success": false,
    "Message": "Obsolete MiningSeed",
    "RandomSeed": "A030176C55672400A4C6472076C1C15AB2BC0EFC16B6B6481578C3F3FF54D561",
    "PublicKey": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271",
    "Nonce": "C83634426BC99DAA83D16EF2BAF7CA3116C196559EA3CB888CE63CDFE910A271",
    "Method": "StratumSubmit",
    "MessageType": 502
}
```

