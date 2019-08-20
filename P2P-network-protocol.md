*This page is work in progress.*

## Overview

BitShares nodes communicate with each other through a peer-to-peer (P2P) network protocol.

Each node accepts connections on a (not necessarily public) TCP socket. Immediately after the connection has been established, nodes exchange cryptographic keys that are subsequently used for encrypting the traffic within that connection.

The protocol consists of messages that are exchanged through the encrypted connection. The protocol supports various message types for requesting information or transmitting blockchain items.

## Communication layers

### Encryption layer

All network traffic after the initial key exchange is encrypted with AES-256.

For the key exchange, each node creates a random private key on the secp256k1 curve, computes the corresponding public key and transmits that in clear over the connection.

Upon receiving the remote public key, it is multiplied with the own private key. The resulting curve point is hashed with SHA-512 to produce a 512 bit shared secret.

From this shared secret, a 256 bit key is created by hashing it with SHA-256. Similarly, a 128 bit IV is created by hashing the secret with city_hash_128. The 256-bit key and the 128-bit IV are then used to set up AES-256-CBC en/decryption streams for sending and receiving data.

### Messaging layer

Messages consist of an 8 bytes header (4 bytes little-endian integer size, 4 bytes little-endian integer type) plus actual message content. The content is a binary serialized representation of a data structure indicated by the type field.

For transmission, messages are padded to a multiple of 16 bytes. (16 bytes is the block size handled by the underlying AES streams. Thus, an messages can always be en/decrypted without having to wait for further data.)

### Messages

Message types are defined in and data structures https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L68-L87 . Data structures are defined in the same file.

## Connection lifecycle

P2P connections are typically long-lived. A node will try to connect to a certain minimum number of peers and can accept additional connections up to a certain maximum number. Peers are only disconnected when they are misbehaving in some sense.

### Handshake

After setting up encryption, both peers send a [hello_message](https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L68-L87) to each other. Both sides perform some sanity checking on the hello_messages they receive, and reply with either [connection_accepted_message](https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L225-L230) or [connection_rejected_message](https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L225-L230). In the latter case, they also initiate disconnecting the peer.

During the handshake, the node keeps the connection in its "handshaking" list. After a successful handshake, the node moves the connection into its "active" list. The handshake can also time out, which leads to the connection being closed.

### Disconnecting

A node that wants to disconnect and has already received a [closing_connection_message](https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L305-L311) simply closes the connection. If it hasn't, it sends a closing_connection_message and waits for the peer either sending a closing_connection_message or closing it.

Note: before sending a closing_connection_message, the node moves the connection from its "active" list to its "closing" list. It is likely that the peer will subsequently timeout due to this.

Note: a node will close the connection forcibly at most `GRAPHENE_NET_PEER_DISCONNECT_TIMEOUT` seconds after sending a closing_connection_message.

## Normal operation

The node implements several ["loops"](https://github.com/bitshares/bitshares-core/wiki/Threading#p2p-thread) for handling connections. Depending on connection state (i. e. the list containing the connection), the connection may be used by one or more such loops.

Active connections are used by
* fetch_sync_items_loop
* fetch_items_loop
* advertise_inventory_loop
* terminate_inactive_connections_loop (disconnect if last message was received too long ago)
* fetch_updated_peer_lists_loop

Note: a known problem from testnet is that active connections will be terminated if the connection is too slow to transmit a large message (such as a huge block) within the timeout, because terminate_inactive_connections_loop only considers *complete* messages.

### address_request_message

An [address_request_message](https://github.com/bitshares/bitshares-core/blob/3.2.1/libraries/net/include/graphene/net/core_messages.hpp#L263-L265) is sent every 15 minutes by `fetch_updated_peer_lists_loop` to all active peers.

Upon receiving it, the node compiles a list of all its active peers into an address_message (see below) and sends it back to the requesting peer.

### address_message

