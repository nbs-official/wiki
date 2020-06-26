New and updated developer information can be found at [Bitshares Developer Portal](https://dev.bitshares.works/) .

* [[quick tips for devs]]

### Building
**BitShares requires a 64-bit operating system to build.**

**BitShares requires a [Boost](http://www.boost.org/) version in the range [1.58, 1.69].** Versions earlier than 1.58 are NOT supported. Newer versions may work, but have not been tested. If your system Boost version is not supported, then you will need to manually build a supported version of Boost and specify it to CMake using `-DBOOST_ROOT`, E.G.

```
cmake -DBOOST_ROOT=~/boost160 .
```

* [[Ubuntu (64-bit) Linux|BUILD_UBUNTU]]
* [[macOS|Building-on-OS-X]]
* [[Windows|BUILD_WIN32]]
* [[Reproducible builds with Gitian for Linux, macOS and Windows|https://github.com/bitshares/bitshares-gitian]]
* [[Web and light wallets|https://github.com/bitshares/bitshares-ui]]

### Nodes

* [[Guide to setup personal nodes|https://hive.blog/bitshares/@ihashfury/run-your-own-decentralised-exchange]]
* [[Guide to setup public API nodes|https://hive.blog/bitshares/@ihashfury/distributed-access-to-the-bitshares-decentralised-exchange]]
* [[Manage your nodes by using gnu screen]]
* [[Memory Reduction for Nodes]]
* [[HaProxy and BitShares]]
* [[ElasticSearch Plugin]]
* [[Delayed Node]]
* [[Debug Node|README-debug_node]]

### Wallet
* [[CLI Wallet Cookbook]]
* [[Wallet Login Protocol]]
* [[Wallet Merchant Protocol]]
* [[Wallet Argument Format]]
* [[Wallet 2-Factor Authentication Protocol]]
* [[Import account to cli wallet]]

### Architecture
* [[Wallet / Full Nodes / Witness Nodes|Wallet_Full Nodes_Witness_Nodes]]
* [[Blockchain Objects]]
* [[chain locked tx]]
* [[resolvable bitassets]]
* [[Stealth Transfers|StealthTransfers]]
* [[Hash Time Locked Contracts (HTLC)|HTLC]]
* [[SPV]]
* [[Node Initialization]]
* [[P2P network protocol]]
* [[Threading]]
* [[General API|API]]
* [[Websocket Subscriptions]]
* [[Scripting websockets easy]]
* [[Howto propose committee actions]]

### Contributing
* [[Contribution Guide]]
* [[Git Flow]]
* [[BitShares Coding Style / Guide|Style-Guide]]
* [[Testing]]

### Integration (exchanges and other businesses)
* [[Monitoring Accounts]]
* [[BitShares Integration Guide (Single Node Edition)|https://dev.bitshares.works/en/master/bts_guide/tutorials/exchange_single_node.html]]

### Witnesses
* [[How to become an active witness in BitShares 2.0]]
* [[How to setup your witness for test net (Ubuntu 14.04 LTS 64 bit)]]