New and updated developer information can be found at [Bitshares Developer Portal](https://dev.bitshares.works/) 

### Building
**BitShares requires a 64-bit operating system to build**

**BitShares requires a [Boost](http://www.boost.org/) version in the range [1.58, 1.69].** Versions earlier than 1.58 are NOT supported. Newer versions may work, but have not been tested. If your system Boost version is not supported, then you will need to manually build a supported version of Boost and specify it to CMake using `-DBOOST_ROOT`, E.G.:

```
cmake -DBOOST_ROOT=~/boost160 .
```

* [[Ubuntu (64-bit) Linux|BUILD_UBUNTU]]
* [[macOS|Building-on-OS-X]]
* [[Windows|BUILD_WIN32]]
* [[Reproducible builds with Gitian for Linux, macOS and Windows|https://github.com/bitshares/bitshares-gitian]]
* [[Web and light wallets|https://github.com/bitshares/bitshares-ui]]

### Architecture
* [[Blockchain Objects|Blockchain Objects]]
* [[Wallet / Full Nodes / Witness Nodes|Wallet_Full Nodes_Witness_Nodes]]
* [[Stealth Transfers|StealthTransfers]]
* [[Hash Time Locked Contracts (HTLC)|HTLC]]

### Wallet
* [[CLI Wallet Cookbook|CLI-Wallet-Cookbook]]
* [[Wallet Login Protocol|Wallet Login Protocol]]
* [[Wallet Merchant Protocol|Wallet Merchant Protocol]]
* [[Wallet Argument Format|Wallet Argument Format]]
* [[Wallet 2-Factor Authentication Protocol|Wallet 2-Factor Authentication Protocol]]

### Contributing
* [[General API|API]]
* [[Websocket Subscriptions|Websocket Subscriptions]]
* [[Testing|Testing]]

### Exchanges
* [[Monitoring Accounts|Monitoring accounts]]

### Witnesses
* [[How to become an active witness in BitShares 2.0|How to become an active witness in BitShares 2.0]]
* [[How to setup your witness for test net (Ubuntu 14.04 LTS 64 bit)|How to setup your witness for test net (Ubuntu 14.04 LTS 64 bit)]]