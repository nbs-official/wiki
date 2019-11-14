# Preface

This document is intended to assist with the use of custom authorities (CA) per [BSIP 40 Specifications](https://github.com/bitshares/bsips/blob/master/bsip-0040.md) in the [4.0.0 Consensus Release](https://github.com/bitshares/bitshares-core/milestone/17?closed=1).

|[Custom Authority Templates](#templates)|
|-|
|[Authorized Restricted Transfers](#template-authorized-restricted-transfers)|
|[Authorized Unrestricted Trading](#template-authorized-unrestrictred-trading)|
|[Authorized Feed Publishing by an Account](#template-authorized-feed-publishing-by-an-account)|
|[Authorized Feed Publishing by a Key](#template-authorized-feed-publishing-by-a-key)|
|[Authorized Account Registration](#template-authorized-account-registration)|

# <div id="templates"/> Custom Authority JSON Templates

_One_ way for an account to _create_ a custom authority is with the CLI Wallet's `add_operation_to_builder_transaction` command.  This command is used as one step of the of "builder transaction" sequence.

```
begin_builder_transaction
add_operation_to_builder_transaction <builder_handle> [54, <JSON_template>]
set_fees_on_builder_transaction <builder_handle> 1.3.0
preview_builder_transaction <builder_handle>
sign_builder_transaction <builder_handle> true
```

where the `<builder_handle>` is the integer "handle" output (e.g. 0, 1, 2, etc.) from the first command in the sequence `begin_builder_transaction`.  <JSON_template> is the JSON encoding of the `custom_authority_create_operation` (Operation 54) that can be broadcast to the network.

Different authorizations require different templates.  This section contains the JSON-encoded templates for various authorizations.  Each of the templates have validity period from `valid_from` through `valid_to` that should be tailored for your use case **and** which must be compatible with the _existing_ limitations on custom authorites (`custom_authority_options`) that may be queried by invoking the `get_global_properties` command in the CLI Wallet or on an RPC-API node.


## <div id="template-authorized-restricted-transfers"/> CA Template: Authorized Transfers

Alice (1.2.19) authorizes Bob (1.2.20) to transfer any amount of any asset from her account to Charlie's account (1.2.21).  `transfer_operation` is Operation 0.

```json
{"account":"1.2.19","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2020-01-31T00:00:00","operation_type":0,"auth":{"weight_threshold":1,"account_auths":[["1.2.20",1]],"key_auths":[],"address_auths":[]},"restrictions":[{"member_index":2,"restriction_type":0,"argument":[7,"1.2.21"]}]}
```

## <div id="template-authorized-unrestrictred-trading"/> CA Template: Authorized Unrestricted Trading

Alice (1.2.17) authorizes Bob (1.2.18) to _create_ limit orders for her account without any restrictions.  `limit_order_create_operation` is Operation 1.

```json
{"account":"1.2.17","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2030-01-01T00:17:25","operation_type":1,"auth":{"weight_threshold":1,"account_auths":[["1.2.18",1]],"key_auths":[],"address_auths":[]},"restrictions":[]}
```

Alice (1.2.17) authorizes Bob (1.2.18) to _cancel_ limit orders for her account without any restrictions.  `limit_order_cancel_operation` is Operation 2.

```json
{"account":"1.2.17","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2030-01-01T00:17:25","operation_type":2,"auth":{"weight_threshold":1,"account_auths":[["1.2.18",1]],"key_auths":[],"address_auths":[]},"restrictions":[]}
```

## <div id="template-authorized-feed-publishing-by-an-account"/> CA Template: Authorized Feed Publishing by an Account

A feed publisher (1.2.16) authorizes/delegates Bob (1.2.17) to publish feeds for an asset.  `asset_publish_feed_operation` is Operation 19.

```json
{"account":"1.2.16","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2030-01-01T00:17:30","operation_type":19,"auth":{"weight_threshold":1,"account_auths":[["1.2.17",1]],"key_auths":[],"address_auths":[]},"restrictions":[]}
```

## <div id="template-authorized-feed-publishing-by-a-key"/> CA Template: Authorized Feed Publishing by a Key

A feed publisher (1.2.16) authorizes/delegates a public key (BTS74YKubbAGUpihj1BP9cCNfdtUbiAhathRs92Ai5EvEQegbpTm8) to publish feeds for an asset.  `asset_publish_feed_operation` is Operation 19.

```json
{"account":"1.2.16","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2030-01-01T00:17:25","operation_type":19,"auth":{"weight_threshold":1,"account_auths":[],"key_auths":[["BTS74YKubbAGUpihj1BP9cCNfdtUbiAhathRs92Ai5EvEQegbpTm8",1]],"address_auths":[]},"restrictions":[]}
```

## <div id="template-authorized-account-registration"/> CA Template: Authorized Account Registration

A faucet account (1.2.16) authorizes a public key (BTS74YKubbAGUpihj1BP9cCNfdtUbiAhathRs92Ai5EvEQegbpTm8) to register accounts on its behalf.  `account_create_operation` is Operation 5.

```json
{"account":"1.2.16","enabled":true,"valid_from":"1970-01-01T00:00:00","valid_to":"2030-01-01T00:17:20","operation_type":5,"auth":{"weight_threshold":1,"account_auths":[],"key_auths":[["BTS74YKubbAGUpihj1BP9cCNfdtUbiAhathRs92Ai5EvEQegbpTm8",1]],"address_auths":[]},"restrictions":[]}
```