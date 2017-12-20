# Oraclize-Tutorial
## Introduction
Oracle service builds the bridge between blockchain and the real world, and we are going to introduce an oracle service called [Oraclize](http://www.oraclize.it/). 

The following image describes the underlying mechanism of the Oraclize.

<p align=center>
<img src="https://docs.oraclize.it/images/flowchart.png">
</p>

It uses the TLSNotary to make its serivce trust-worthy, and pre-fetch the real world data for all the nodes within the network to achieve consensus, which cannot be easiliy done with Ethereum smart contract.

## Example explanation
The following example shows a simple contract using the orcalize serivce

```javascript
//this import should be replaced by oraclizeAPI.sol
import "dev.oraclize.it/api.sol";
contract YoutubeViews is usingOraclize {

    uint public viewsCount;

    function YoutubeViews() {
        //OAR = OraclizeAddrResolverI(resolve_addr);  //add this line if you are using Oraclize in private chain environment
        update(0);
    }

    function __callback(bytes32 myid, string result) {
        if (msg.sender != oraclize_cbAddress()) throw;
        viewsCount = parseInt(result, 0);
        // do something with viewsCount

        update(60*10); // update viewsCount every 10 minutes
    }

    function update(uint delay) {
        oraclize_query(delay, 'URL', 'html(https://www.youtube.com/watch?v=9bZkp7q19f0).xpath(//*[contains(@class, "watch-view-count")]/text())');
    }

}
```
The example shows three steps to utilize the oraclize service:
+ Constructor will first call the `update` function, which will trigger the orclize_query transaction and commence data fetching process. You also have to specify the delay (in seconds) from the current time or the timestamp in the future as first argument.
+ Once the data query returns the result, it will call the `__callback function` with the result passed into the function.
+ We can now do some operation using the code defined in `__callback function`, and you can also call `update function` again if you want to execute the query once in a while.

``Caution! Please note that in order for the future timestamp to be accepted by Oraclize it must be within 60 days from the current time. ``

## Private chain scenarios
If you are currently using private chain, you need to use the Ethereum-Bridge API, which is a log listener listens to the oraclize query from private chain and connect to oraclize service. The following list the steps of using Ethereum-Bridge:
+ Download the repository of Ethereum-Bridge by using `git clone https://github.com/oraclize/ethereum-bridge`, and execute `npm install` in the nodejs folder.
+ Unlock certain account (in this example we will use acoounts[0]) by using `geth --unclock 0`, and execute `node bridge -H localhost:8545 -a 0`. (make sure the account you unlock matches the account input to the bridge)
+ The bridge will automatically deploys the Oraclize Address Resolver (OAR) and the Oraclize Connector in the private chain, which interface your contract with the Oraclize service. Please note: your contract shouldn't be deployed with the same address used by the Ethereum Bridge (in this example the address 0).

## Scheduler
Since smart contract can only be triggered by transaction, it is hard to have code executed in a specified timestamp. However, there are already some solutions to this issue, for instance, the Ethereum Alarm Clock service.
This part we are gonna use Oraclize as a scheduler instead of an oracle. How ? The following example will show you
```javascript
import "dev.oraclize.it/api.sol";
contract YoutubeViews is usingOraclize {

    uint public viewsCount;

    function YoutubeViews() {
        update(0);
    }

    function __callback(bytes32 myid, string result) {
        if (msg.sender != oraclize_cbAddress()) throw;
        viewsCount = parseInt(result, 0);
        // do something 

        update(10); // do something every 10 seconds
    }

    function update(uint delay) {
        oraclize_query(delay, 'URL', '');   // This query can still work without the URL parameters
    }

}
```
As you can see, the oraclize service can be a convenient scheduler if no URL parameters are inputted. 
