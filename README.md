# Finnhub Stock API Oracle Overview
![finnhub]

[finnhub]: ./pics/finnhub.png

The API is organized around REST. Our API has predictable resource-oriented URLs, accepts form-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, authentication, and verbs. This is 1 of the most comprehensive financial API available on the market.

This oracle's initial implementation will provide:
 - The Quote for a given stock with the endpoint: quote
 - The in-date Stock Candles with the endpoint: stock/candle

API responses are packed after executing our adapter and then pushed on-chain.

## Steps For Using This Oracle
### Write and deploy your Chainlink contract using the network details below
![metamask]

[metamask]: ./pics/metamask.png

Fund it with LINK.

Call your request method.
 
## Network Details
##### Ethereum Mainnet
 - Payment amount: 3 LINK
 - LINK Token address: 0x514910771af9ca656af840dff83e8264ecf986ca
 - Oracle Address: 0x2E5F9bA916558C7Dd6C8cEB5071b499218Fec436
 - JobID: 8097c0c8a09545b1a11593c3badda016

##### Ethereum Kovan Testnet
 - Payment amount: 1 LINK
 - LINK Token address: 0xa36085F69e2889c224210F603D836748e7dC0088
 - Oracle Address: 0x5D376D602BA795984c5430BD80401668c195451c
 - JobID: 81f2755374114f8aaa426c7224ac2cc2

##### Ethereum Rinkeby Testnet
 - Payment amount: 1 LINK
 - LINK Token address: 0x01BE23585060835E02B77ef475b0Cc51aA1e0709
 - Oracle Address: 0xE85d10D6440edcACc1478e28CEa9566704fe88Db
 - JobID: 061029413a4d48818df70086050bc60e

## Tasks
 - Finnhub
 - Copy
 - Multiply
 - ETH Uint256
 - ETH Transaction

## Request Parameters
### Endpoint: The real-time Quote and Stock Candles endpoint. Check [Finnhub APIs](https://finnhub.io/docs/api/) documment
Solidity Example

req.add("endpoint", "quote");

req.add("endpoint", "stock/candle");

### Symbol: The stock ticker from [Finnhub Stock Symbols](https://finnhub.io/docs/api/stock-symbols) api:
Solidity Example

req.add("ticker", "AAPL");

### Timestamp: The exact integer timestamp for candle check (stock/candle).
Solidity Example

req.add("timestamp", 1631627048);

### Field: The field selected from JSON response.
Solidity Example

req.add("field", "c");

#### quote
 - c: Current price
 - d: Change
 - dp: Percent change
 - h: High price of the day
 - l: Low price of the day
 - o: Open price of the day
 - pc: Previous close price

#### stock/candle:
 - c: List of close prices for returned candles.
 - h: List of high prices for returned candles.
 - l: List of low prices for returned candles.
 - o: List of open prices for returned candles.
 - t: List of timestamp for returned candles.
 - v: List of volume data for returned candles.


### Times: The amount to multiply the result to uint256.
Solidity Example

req.addInt("times", 100); (for close prise c)

### Note for Rinkeby and Kovan Test net Jobs:

 - Requests on a test net will return only static data for testing purposes. The returned result may not match Finnhub's API value. 

 - Entities that are working on Rinkeby or Kovan should contact us for service.

## Outputs
An unint256 value of JSON response from our API.

## Full Solidity contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

import '@chainlink/contracts/src/v0.8/ChainlinkClient.sol';
import '@chainlink/contracts/src/v0.8/ConfirmedOwner.sol';


contract APIConsumer is ChainlinkClient, ConfirmedOwner {
    using Chainlink for Chainlink.Request;

    uint256 public volume;
    bytes32 private jobId;
    uint256 private fee;

    event RequestVolume(bytes32 indexed requestId, uint256 volume);
    constructor() ConfirmedOwner(msg.sender) {
        setChainlinkToken(0x514910771af9ca656af840dff83e8264ecf986ca);
        setChainlinkOracle(0x2E5F9bA916558C7Dd6C8cEB5071b499218Fec436);
        jobId = '8097c0c8a09545b1a11593c3badda016';
        fee = (30 * LINK_DIVISIBILITY) / 10; // 3 * 10**18 (Varies by network and job)
    }

    function requestVolumeData() public returns (bytes32 requestId) {
        Chainlink.Request memory req = buildChainlinkRequest(jobId, address(this), this.fulfill.selector);

        req.add('endpoint', 'quote'); // or stock/candle
        req.add('symbol', 'AAPL');
        req.addInt('timestamp', 1631627048); // Random number for "quote" endpoint or 0000000000
        req.add('field', 'c');               // timestamp will be ignored as "quote" is real-time
        req.addInt('times', 100);            // All fields (endpoint, symbol, timestamp, etc.) are required

        // Sends the request
        return sendChainlinkRequest(req, fee);
    }

    /**
     * Receive the response in the form of uint256
     */
    function fulfill(bytes32 _requestId, uint256 _volume) public recordChainlinkFulfillment(_requestId) {
        emit RequestVolume(_requestId, _volume);
        volume = _volume;
    }

    /**
     * Allow withdraw of Link tokens from the contract
     */
    function withdrawLink() public onlyOwner {
        LinkTokenInterface link = LinkTokenInterface(chainlinkTokenAddress());
        require(link.transfer(msg.sender, link.balanceOf(address(this))), 'Unable to transfer');
    }
}
```