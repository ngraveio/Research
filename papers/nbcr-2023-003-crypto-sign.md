# Signing Protocol

## NBCR-2023-003

© 2023 NGRAVE
Authors: Mathieu Da Silva, Irfan Bilaloglu <br/>
Date: April 26, 2023

---

# I - Introduction

The goal of this document is to propose a QR protocol standardizing the communication between an offline wallet signing the transactions and a watch-only wallet preparing and broadcasting the transactions to the blockchain.

This protocol specifies the QR formats for the signing request and the signature. The signing request is created by the watch-only wallet including the transaction to sign and the signer account to be sent via QR code to the offline signer. The transaction is signed by the offline signer that generates a QR code to send back the signed transaction. 

For some blockchains, standards already exist, as proposed in [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md) for Bitcoin, in [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) for Ethereum blockchain, in [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) for Tezos blockchain and in Keystone repository [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer) for Solana blockchain. 

This document defines a generic QR protocol to sign on any blockchain and extends at the same time the QR code-based protocol to blockchains where a standard has not been defined yet.

## Specification

**Offline signer**: An offline signer is a device or application which holds the user’s private keys and does not have network access.

**Watch-only wallet**: A watch-only wallet is a wallet that has network access and can interact with the blockchain.

## Motivation to use Uniform Ressources (UR) types

The BlockchainCommons (BC) have published a series of data transmission protocol called Uniform Resources (UR). It provides a basic method to encode data into animated QR Codes. The UR have been used for different proposals of QR-code based data transmission (e.g. [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527),  [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) etc…). The UR types form the basis of the data transmission protocol for the following reasons:

- **Standardization**: a standard already commonly used by other hardware wallets (Keystone, AirGap, CoolWallet, D’Cent).
- **Data compression**: Byte savings using CBOR compared to text encoding.
- **Human-friendly text encoding**: Format is defined as `ur:<type>/<message>` (details can be found [here](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md))
- **Efficiency of QR code encoding**: data is encoded as CBOR (Concise Binary Object Representation) supporting sizes up to 2^32-1 bytes (~ 4 GB) compared to the largest QR code ("version 40") consisting of 2953 bytes (details can be found in [[BCR-2020-005]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md))
- **Time-sequence QR code**: ability to break up messages too large to fit into a QR code into a sequence of QR codes using sophisticated "fountain codes."
- **Error detection**: CRC32 checksum on the entire message in each part to tie them together and ensure the transmitted message has been reconstructed.

## Technical specifications

This document specifies the following limits on the QR code format in order to be able to scan with any devices without requiring the best quality camera.

- The fragment size is fixed at **90 characters** per QR code**.**
- The time-sequence of QR code is displayed at a fps of **8**.
- The QR code correction level is set to **Medium**.

## Limitation

Despite the generic purpose of this document to standardize a signing protocol via QR codes, the different blockchains are various and present each their own specificity. We are focusing the signing protocol to support the following blockchains:

- Bitcoin
- Ethereum and other EVM blockchain
- Solana
- Stellar
- Elrond
- Tezos

The integration of a new blockchain should reuse as much as possible the generic communication protocol introduced in this document.

This document is limited to the detailed description of the signing process between the offline signer and the watch-only wallet. The sync communication protocol via QR codes is discussed in [[NBCR-2023-002]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md) specification.

---

# II - Signing protocol

As preliminary step before signing any transaction, the offline signer needs to provide via QR code the necessary accounts’ information to the watch-only wallet. 

The user can initiate the signing protocol on a watch-only wallet synchronized with an offline signer by following the described process:

1. The watch-only wallet creates a QR code including the unsigned data to be signed based on the user inputs (sender and receiver addresses, amount, transaction fee, etc…)
2. The offline signer reads the QR code including the signature request, then decodes the transaction to display the transaction information in order to ensure the transaction integrity.
3. After user confirmation, the offline signer signs the transaction and provides the signature to the watch-only wallet via a QR code.
4. The watch-only wallet receiving the signature is able to broadcast the transaction to the blockchain.

The existing communication protocol for signing are based on UR types specific to each blockchain. Except for Bitcoin with `crypto-psbt` UR type, the signing request prepared by the watch-only wallet is embedded in a UR type with the extension `sign-request`. Once the transaction is signed, the offline signer sends the signature back to the watch-only wallet by encoding the data in a UR type with the extension `signature`.

This document proposes to standardize into one UR type `crypto-sign-request` for the signing request and another UR type `crypto-signature` for the signed message.

The following table listed the existing UR types depending on the blockchain and introduced the new UR types intended to be blockchain agnostic.

| Blockchain | Sign request  | Signature  | Owner | Definition | Implementation |
| --- | --- | --- | --- | --- | --- |
| Bitcoin | crypto-psbt <br> Tag: 310 | crypto-psbt <br> Tag: 310 | Blockchain Commons (BC) | [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md) | [[keytool-cli]](https://github.com/BlockchainCommons/keytool-cli/tree/master) |
| Ethereum/EVM | eth-sign-request <br> Tag: 401 | eth-signature <br> Tag: 402 | Keystone | [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) | [[ur-registry-eth]](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth) |
| Solana| sol-sign-request <br> Tag: 1101 | sol-signature <br> Tag: 1102 | Keystone | [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer)  | [[ur-registry-sol]](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-sol) |
| Tezos | xtz-sign-request <br> Tag: 501 | xtz-signature <br> Tag: 502 | Airgap  | [[tzip-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) | https://socket.dev/npm/package/@airgap/ur-registry-xtz/overview/0.0.1-beta.0 |
| Generic | crypto-sign-request <br> Tag: 1411 | crypto-signature <br> Tag: 1412 | Ngrave | This document | Not available |

The specification for each UR type contains CBOR structure, expressed thereafter in Concise Data Definition Language [[CDDL]](https://datatracker.ietf.org/doc/html/rfc8610).

## II - 1. Blockchain-specific UR types registry

The UR types designed to be used for specific blockchain are based on existing specification referenced in each Section. 

### Definition of common embedded CBOR tags

The UR types for signing embed common tags, described in the following list:

- Universal Unique IDentifier, noted as UUID, are CBOR binary strings tagged with #6.37, per the IANA [[CBOR Tag]](https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml).
- `crypto-keypath` UR type specifies the key derivation path of the signer account. This structure is embedded with the tag #6.304 as defined in [[BCR-2020-007]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-007-hdkey.md).

### Bitcoin UR type

For Bitcoin blockchain, the UR types are based on the Partially Signed Bitcoin Transaction (PSBT) format defined in [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki). 

- **CDDL for BTC PSBT** `crypto-psbt`

Both BTC sign request and signature embeds `crypto-psbt` UR type tagged with #6.310 and belonging to the [[BCR-2020-006]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-006-urtypes.md). It contains a single, deterministic length byte string of variable length up to 2^32-1 bytes. Semantically, this byte string MUST be a valid Partially Signed Bitcoin Transaction encoded in the binary format specified by [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki).

```
crypto-psbt = bytes
```

This UR type is used for both signing request and signature response.

### Ethereum UR types

For Ethereum and other EVM blockchains, the UR types are based on the [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposal.

- **CDDL for ETH signature request** `eth-sign-request`

This UR type belongs to the [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposal and is tagged with #6.401.

```
; Metadata for the signing request for Ethereum.
; 
sign-data-type = {
    type: int .default 1 transaction data; the unsigned data type
}

eth-transaction-data = 1; legacy transaction rlp encoding of unsigned transaction data
eth-typed-data = 2; EIP-712 typed signing data
eth-raw-bytes=3;   for signing message usage, like EIP-191 personal_sign data
eth-typed-transaction=4; EIP-2718 typed transaction of unsigned transaction data

; Metadata for the signing request for Ethereum.
; request-id: the identifier for this signing request.
; sign-data: the unsigned data
; data-type: see sign-data-type definition
; chain-id: chain id definition see https://github.com/ethereum-lists/chains for detail
; derivation-path: the key path of the private key to sign the data
; address: Ethereum address of the signing type for verification purposes which is optional

eth-sign-request = (
    sign-data: sign-data-bytes, ; sign-data is the data to be signed by offline signer, currently it can be unsigned transaction or typed data
    data-type: sign-data-type,
    chain-id: int .default 1,
    derivation-path: #6.304(crypto-keypath), ;the key path for signing this request
    ?request-id: uuid, ; the uuid for this signing request
    ?address: eth-address-bytes,            ;verification purpose for the address of the signing key
    ?origin: text  ;the origin of this sign request, like wallet name
)
request-id = 1
sign-data = 2
data-type = 3
chain-id = 4 ;it will be the chain id of ethereum related blockchain
derivation-path = 5
address = 6
origin = 7
eth-address-bytes = bytes .size 20
sign-data-bytes = bytes ; for unsigned transactions it will be the rlp encoding for unsigned transaction data and ERC 712 typed data it will be the bytes of json string.
```

- **CDDL for ETH signature response** `eth-signature`

This UR type belongs to the [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposal and is tagged with #6.402.

```
eth-signature  = (
    request-id: uuid,
    signature: eth-signature-bytes,
    ? origin: text, ; The device info providing this signature
)

request-id = 1
signature = 2
origin = 3

eth-signature-bytes = bytes .size 65; the signature of the signing request (r,s,v)
```

### Solana UR types

For Solana blockchain, the UR types are based on the proposed implementation by Keystone [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer).

- **CDDL for SOL signature request** `sol-sign-request`

This UR type extends the existing protocol for Ethereum to Solana blockchain and is tagged with #6.1101.

```
; Metadata for the signing request for Solana.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `derivation-path` is the path of the private key to sign the data
; `origin` is the origin of this sign request. like watch-only wallet name.
; `address` is the Solana address of the signing type for verification purpose which is optional 

sol-sign-request = (
    ?request-id: uuid,
    sign-data: bytes,
    derivation-path: #6.304(crypto-keypath), ;the key path for signing this request
    ?origin: text,
    ?address: bytes
    ?type: int .default sign-type-transaction, ;sign type identifier
)

request-id = 1
sign-data = 2
derivation-path = 3
address = 4
origin = 5
type = 6

sign-type-transaction = 1
sign-type-message = 2
```

- **CDDL for SOL signature response** `sol-signature`

This UR type extends the existing protocol for Ethereum to Solana blockchain and is tagged with #6.1102.

```
sol-signature  = (
    request-id: uuid,
    signature: bytes
)

request-id = 1
signature = 2
```

### Tezos UR types

For Tezos blockchain, the UR types are based on the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal.

- **CDDL for XTZ signature request** `xtz-sign-request`

This UR type belongs to the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal and is tagged with #6.1411. 

Two different CDDL expressions have been proposed regarding how to specify the signer account. The [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal opted for specifying the signer account through the derivation path, while the implementation [[ur-registry-xtz]](https://socket.dev/npm/package/@airgap/ur-registry-xtz/overview/0.0.1-beta.0) proposed to specify only the associated public key. 

The CDDL expression described in this document follows the TZIP proposal by requesting the derivation path tagged with #6.304 as `crypto-keypath` UR type, as defined in [[BCR-2020-007]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-007-hdkey.md).

```
; Metadata for the signing request for Tezos.
; `request-id` is the identifier for this signing request.
; `sign-data` is the transaction data to be signed.
; `data-type` is the type of data to be signed (by default, operation type for forged operation)
; `public-key` is the public key associated to the signer account
; `key-type` is the elliptic curves used to sign the data
; Tezos supports three types of keys associated with 3 kinds of elliptic curves
; tz1 for Ed25519 keys,
; tz2 for Secp256k1 keys (same as Bitcoin/Ethereum)
; tz3 for NIST P256 keys which may have led to some amount of controversy but is one of the most used elliptic curve
; zet for sapling
; 'master-fingerprint' is fingerprint for the master public key as per BIP32

xtz-sign-request = (
    ?request-id: uuid,
    sign-data: bytes,
		data-type: int .default data-type-operation,
    derivation-path: #6.304(crypto-keypath), ;the key path for signing this request
		key-type: int .default public-key-type-ed25519, 
		?master-fingerprint: uint32, ; Master fingerprint (fingerprint for the master public key as per BIP32)
)

request-id = 1
sign-data = 2
data-type = 3
derivation-path = 4
key-type = 5
master-fingerprint = 6

data-type-operation = 1 ; A forged operation

public-key-type-ed25519 = 1
public-key-type-sep256k1 = 2
public-key-type-nistp256r1 = 3
public-key-type-sapling = 4
```

- **CDDL for XTZ signature response** `xtz-signature`

This UR type belongs to the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal and is tagged with #6.1412.

```
xtz-signature  = (
    ?request-id: uuid, ; Optional only if payload is specified
    signature: bytes,
		?payload: bytes, ; The payload is optional. If the payload is not set, 
the watch-only wallet needs to match the signature with the original payload 
using the ID.
)

request-id = 1
signature = 2
payload = 3
```

## II - 2. Blockchain-agnostic UR types registry

This document aims to propose common UR types for signing on every blockchain. 

- **CDDL for generic signature request** `crypto-sign-request`

This UR type is tagged with #6.1411 and embeds:

- The `crypto-coin-identity` UR type tagged with #6.140 and introduced in [[NBCR-2023-001]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md). This UR type identifies the elliptic curve and the blockchain on which the sign request is initiated.
- An optional UR type to specify metadata related to a specific coin.

```
; Metadata specific for each coin
; The list is not exhaustive and includes only the metadata for ETH, SOL and XTZ
coin-meta = #6.1420(eth-meta) / #6.1421(sol-meta) / #6.1422(xtz-meta) 

crypto-sign-request = {
  ?request-id: uuid,                        ; Identifier of the signing request
	coin-id: #6.1401(crypto-coin-identity),   ; Provides information on the elliptic curve and the blockchain/coin
  ?derivation-path: #6.304(crypto-keypath), ; Key path for signing this request
  sign-data: bytes,                         ; Transaction to be decoded by the offline signer 
  ?master-fingerprint: uint32,              ; Fingerprint for the master public key
  ?origin: text,                            ; Origin of this sign request, e.g. wallet name
	?metadata: coin-meta                      ; Specify metadata for some coins
}

request-id = 1
coin-id = 2
derivation-path = 3
sign-data = 4
master-fingerprint = 5
origin = 6
metadata = 7
```

We are listing thereafter the metadata UR types for the blockchains listed in this document. This list is not exhaustive and needs to be updated for each coin needing additional data to the `crypto-sign-request` UR type.

| Blockchain | Metadata required | coin-meta UR type |
| --- | --- | --- |
| Ethereum/EVM chains | Yes | - CDDL description: <br> `eth-meta = { data-type: sign-data-type, ?address: eth-address-bytes}` <br> `sign-data-type` and `eth-address-bytes` definition are inherited from eth-sign-request. <br> - Tag: 1420 |
| Solana | No | - CDDL description: <br> `sol-meta = { ?type: int .default sign-type-transaction, ?address: bytes}`  <br> `type` definition is inherited from sol-sign-request. The default value `sign-type-transaction` should be used in case of missing metadata in crypto-sign-request for a Solana transaction. <br> - Tag: 1421 |
| Tezos | No | - CDDL description: <br> `xtz-meta = { ?data-type: int .default data-type-operation}` <br> `data-type definition` is inherited from xtz-sign-request.
The default value of data-type should be used in case of missing metadata in crypto-sign-request for Tezos transaction.
- Tag: 1422 |
| Bitcoin <br> MultiversX <br> Stellar | No | No metadata needed |

- **CDDL for generic signature response** `crypto-signature`

This UR type is tagged with #6.1412. 

```
crypto-signature = {
  ?request-id: uuid,     ; Identifier of the signing request 
  signature: bytes,      ; Signature result
  ?origin: text,         ; The device info providing this signature
}

; request-id must be present in case of response to a crypto-sign-request where 
; the request-id is specified

request-id = 1
signature = 2
origin = 3
```

---

# III - Use cases on several blockchains

This Section gives more details on the use case for each blockchain, the signing process with examples and provides guidance to enable user verification of the transaction on the offline signer.

## III - 1. Bitcoin transactions

Since Bitcoin transactions follow [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki) defining PSBT, the privileged UR type for signing is `crypto-psbt`. In this document, we present its integration into the common UR types `crypto-sign-request` and `crypto-signature`.

### Integration with the generic UR types registry

- **Bitcoin sign request**

A Bitcoin transaction is uniquely identified through the URI format `bc-coin://secp256k1/0`, information shared in `crypto-coin-identity` UR type.

The only other required field for requesting the signature of a Bitcoin transaction is the bytes composing the unsigned PSBT transaction, corresponding to the `sign-data` field of `crypto-sign-request` UR type.

The following table indicates the corresponding fields between ****the UR types `crypto-psbt` and `crypto-sign-request`.

| crypto-sign-request fields
Associated number corresponds to the order in UR type | Corresponding crypto-psbt fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | No corresponding field, adds potential verification step  |
| 2. coin-id | No corresponding field, serves as identifier for BTC transactions |
| 3. derivation-path (optional) | No corresponding field, adds potential verification step  |
| 4. sign-data | 1. bytes |
| 5. master-fingerprint (optional) | No corresponding field, adds potential verification step  |
| 6. origin (optional) | No corresponding field, provides an additional description |

`crypto-sign-request` UR type has the advantage to provide additional optional fields compared to `crypto-psbt` used as signing request: an unique identifier for the request, the derivation path of the account required to sign (not mandatory since this information is specified in the PSBT), the master fingerprint as verification and a text describing the origin (e.g. the wallet name). 

- **Bitcoin signature**

A Bitcoin sign request results in a signed transaction after user verification. A Bitcoin signature is contained in the same format as the request, a PSBT transaction.

The only required field for a Bitcoin signature is the bytes composing the signed PSBT transaction, corresponding to the `signature` fields of `crypto-signature` UR type.

The following table indicates the corresponding fields between the UR types `crypto-psbt` and `crypto-signature`.

| crypto-signature fields
Associated number corresponds to the order in UR type | Corresponding crypto-psbt fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id | No corresponding field, adds potential verification step  |
| 2. signature | 1. bytes |
| 3. origin | No corresponding field, adds a description |

`crypto-signature` UR type has the advantage to provide additional optional fields compared to `crypto-psbt` used as signature: an unique identifier to link to the request and a text describing the origin (e.g. the device name). 

### Use case

A typical use case follows the following process between a watch-only wallet and an offline signer:

- The watch-only wallet creates a PSBT following the Creator role in [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki):
    1. The Creator creates an unsigned transaction and places it in the PSBT.
    2. The Creator adds empty input and output fields.
- The watch-only adds then information to the PSBT that it has access to, following the Updater role in [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki):
    1. If the Updater has the UTXO for an input, it should add it to the PSBT. 
    2. The Updater should also add redeemScripts, witnessScripts, and BIP 32 derivation paths to the input and output data if it knows them.
- The watch-only wallet generates a QR code containing the `crypto-sign-request` UR type with the unsigned transaction and BTC as coin identity. Additionally, some recommended information is provided to the offline signer for verification purposes: request identifier, derivation path of the signer account, master fingerprint of the master public key and the watch-only wallet name.
- The offline signer accepts the transmitted PSBT in `crypto-sign-request` UR type and provides the signature, following the Signer role in [[BIP174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki):
    1. The Signer must only use the UTXOs provided in the PSBT to produce signatures for inputs.
    2. Before signing a non-witness input, the Signer must verify that the TXID of the non-witness UTXO matches the TXID specified in the unsigned transaction. 
    3. Before signing a witness input, the Signer must verify that the witnessScript (if provided) matches the hash specified in the UTXO or the redeemScript, and the redeemScript (if provided) matches the hash in the UTXO. 
    4. The Signer may choose to fail to sign a segwit input if a non-witness UTXO is not provided. 
    5. The Signer should not need any additional data sources, as all necessary information is provided in the PSBT format. 
    6. The Signer must only add data to a PSBT. Any signatures created by the Signer must be added as a "Partial Signature" key-value pair for the respective input it relates to. 
    7. The Signer can additionally compute the addresses and values being sent, and the transaction fee, showing this data to the user as a confirmation of intent and the consequences of signing the PSBT.
    8. Signers do not need to sign for all possible input types. For example, a signer may choose to only sign Segwit inputs.
- The offline signer responds with `crypto-signature` containing the output signatures in the PSBT and if provided replied with the same identifier used for the request.

### Example

An example illustrates how the signature request and response for Bitcoin PSBT are encoded.

ToDo update example

<details>

<summary>Example BTC signature request</summary>

- BTC PSBT
    
    ```
    0x70736274ff01009a020000000258e87a21b56daf0c23be8e7070456c336f7cbaa5c8757924f545887bb2abdd750000000000ffffffff838d0427d0ec650a68aa46bb0b098aea4422c071b2ca78352a077959d07cea1d0100000000ffffffff0270aaf00800000000160014d85c2b71d0060b09c9886aeb815e50991dda124d00e1f5050000000016001400aea9a2e5f0f876a588df5546e8742d1d87008f000000000000000000
    ```
    
- CBOR diagnosis format:
    
    ```
    {
    	1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; transaction-id: 3B541437-5E3A-450B-8FE1-251CBC2B3FB5
    	2: 502({ ; body: request-psbt-signature
    		1: 310(h'70736274ff01009a020000000258e87a21b56daf0c23be8e7070456c336f7cbaa5c8757924f545887bb2abdd750000000000ffffffff838d0427d0ec650a68aa46bb0b098aea4422c071b2ca78352a077959d07cea1d0100000000ffffffff0270aaf00800000000160014d85c2b71d0060b09c9886aeb815e50991dda124d00e1f5050000000016001400aea9a2e5f0f876a588df5546e8742d1d87008f000000000000000000')
    	})
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```
    A2                                      # map(2)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             3B5414375E3A450B8FE1251CBC2B3FB5 
       02                                   # unsigned(2)
       D9 01F6                              # tag(502)
          A1                                # map(1)
             01                             # unsigned(1)
             D9 0136                        # tag(310)
                58 A7                       # bytes(167)
                   70736274FF01009A020000000258E87A21B56DAF0C23BE8E7070456C336F7CBAA5C8757924F545887BB2ABDD750000000000FFFFFFFF838D0427D0EC650A68AA46BB0B098AEA4422C071B2CA78352A077959D07CEA1D0100000000FFFFFFFF0270AAF00800000000160014D85C2B71D0060B09C9886AEB815E50991DDA124D00E1F5050000000016001400AEA9A2E5F0F876A588DF5546E8742D1D87008F000000000000000000
    ```
    
    - UR encoding in `ur:crypto-request/<message>` format

</details>

<details>

<summary>Example BTC signature response</summary>

- CBOR diagnosis format:
    
    ```
    {
    	1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; transaction-id: 3B541437-5E3A-450B-8FE1-251CBC2B3FB5
    	2: 310( h'70736274ff01009a020000000258e87a21b56daf0c23be8e7070456c336f7cbaa5c8757924f545887bb2abdd750000000000ffffffff838d0427d0ec650a68aa46bb0b098aea4422c071b2ca78352a077959d07cea1d0100000000ffffffff0270aaf00800000000160014d85c2b71d0060b09c9886aeb815e50991dda124d00e1f5050000000016001400aea9a2e5f0f876a588df5546e8742d1d87008f000000000000000000') ; payload
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A2                                      # map(2)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             3B5414375E3A450B8FE1251CBC2B3FB5 
       02                                   # unsigned(2)
       D9 0136                              # tag(310)
          58 A7                             # bytes(167)
             70736274FF01009A020000000258E87A21B56DAF0C23BE8E7070456C336F7CBAA5C8757924F545887BB2ABDD750000000000FFFFFFFF838D0427D0EC650A68AA46BB0B098AEA4422C071B2CA78352A077959D07CEA1D0100000000FFFFFFFF0270AAF00800000000160014D85C2B71D0060B09C9886AEB815E50991DDA124D00E1F5050000000016001400AEA9A2E5F0F876A588DF5546E8742D1D87008F000000000000000000
    ```
    
- UR encoding in `ur:crypto-response/<message>` format

</details>

### Transaction verification guidance

Once the request is received by the offline signer, the user should be able to verify the transaction content including optional `description` text field. However, only the decoded PSBT can be considered authoritative. 

The received PSBT should be decoded by the offline signer to know each input and output contained in the transaction before signing. The following information should be displayed to the user:

- Types
- BIP32 derivation paths
- Input addresses
- Output addresses
- Fee
- Amount

## II - 2. Ethereum transactions

[[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposes the specific UR types `eth-sign-request` and `eth-signature` for signing Ethereum transactions and in the same way transactions on every EVM blockchains. In this document, we present their integration into the common UR types `crypto-sign-request` and `crypto-signature`.

### Integration with the generic UR types registry

- **Ethereum sign request**

An Ethereum transaction is uniquely identified through the URI format `bc-coin://secp256k1/60`, and every EVM blockchains by adding their chain ID as subtype in the URI format (e.g. `bc-coin://137.secp256k1/60` for Polygon). This information is shared in `crypto-coin-identity` UR type.

Several fields are required for requesting the signature of an Ethereum transaction:

- Unsigned transaction
- Type of data to sign
- Chain ID
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `eth-sign-request` and `crypto-sign-request`.

| crypto-sign-request fields
Associated number corresponds to the order in UR type | Corresponding eth-sign-request fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id specifying the chain ID for EVM blockchains | 4. chain-id  |
| 3. derivation-path | 5. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. master-fingerprint (optional) | No corresponding field, adds potential verification step  |
| 6. origin (optional) | 7. origin |
| 7.1. eth-meta.data-type | 3. data-type |
| 7.2. eth-meta.address (optional) | 6. address (optional, redundant with derivation-path where the public key is already included) |

`crypto-sign-request` UR type provides the exact same information as `eth-sign-request` and has even the advantage to provide an additional optional field: the master fingerprint for verification purposes.

- **Ethereum signature**

An Ethereum sign request results in a signed transaction after user verification. 

The required fields for an Ethereum signature are the signature itself and potentially the request identifier to identify uniquely the response. Additionally, a description can be provided to specify the device name for example.

The following table indicates the corresponding fields between the UR types `eth-signature` and `crypto-signature`.

| crypto-signature fields
Associated number corresponds to the order in UR type | Corresponding eth-signature fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | 3. origin (optional) |

`crypto-signature` and `eth-signature` provides identical information, with the advantage of `crypto-signature` to be blockchain-agnostic.

### Use case

The watch-only wallet generates a QR code containing the `crypto-sign-request` UR type with the following information:

1. (Recommended) Request ID, an universally unique identifier (UUID) of the transaction
2. Ethereum coin identity and additionally the chain ID to specify an EVM blockchain
3. Derivation path of the key requested to sign the request
4. Transaction to sign
5. (Recommended) Master fingerprint of the master public key
6. (Recommended) Origin to indicate the wallet name
7. Data type (legacy RLP encoding, typed data, raw bytes or typed transaction)
8. (Optional) Sender address for verification, redundant however with derivation path where the public key is already included

The offline signer decodes the transaction depending on the data type and responds to the request with the signature once the transaction has been verified and approved by the user. The offline signer generates a QR code containing the `crypto-signature` UR type with the following information:

1. (Mandatory if provided in the request) Request ID indicating the UUID of the transaction
2. Signature 
3. (Recommended) Origin to indicate the name of the offline signer

### Example

An example illustrates how the signing protocol works on EVM blockchains:

ToDo update example

<details>

<summary>Example Polygon signature request</summary>

- CBOR diagnosis format:
    
    ```
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'f849808609184e72a00082271094000000000000000000000000000000000000000080a47f7465737432000000000000000000000000000000000000000000000000000000600057808080', ; sign-data
    3: 1, ; data-type = RLP transaction
    4: 137, ; chain-id = Polygon
    5: 303( ; #6.303(crypto-hdkey)
         {3: h'0337A5619FBC2E951B426D23E9736C7576840AE3139AAE5D0B9D0D81736D5706D9', ; key-data
    		  6: 304({1: [44, true, 60, true, 0, true]}), ; origin m/44'/60'/0'
    		  7: 304({1: [0, false, 0, false]}) ; children m/44'/60'/0'/0/0
         }),
    7: "Trust Wallet" ; wallet name
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A6                                      # map(6)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 4B                                # bytes(75)
          F849808609184E72A00082271094000000000000000000000000000000000000000080A47F7465737432000000000000000000000000000000000000000000000000000000600057808080 
       03                                   # unsigned(3)
       01                                   # unsigned(1)
       04                                   # unsigned(4)
       18 89                                # unsigned(137)
       05                                   # unsigned(5)
       D9 012F                              # tag(303)
          A3                                # map(3)
             03                             # unsigned(3)
             58 21                          # bytes(33)
                0337A5619FBC2E951B426D23E9736C7576840AE3139AAE5D0B9D0D81736D5706D9 
             06                             # unsigned(6)
             D9 0130                        # tag(304)
                A1                          # map(1)
                   01                       # unsigned(1)
                   86                       # array(6)
                      18 2C                 # unsigned(44)
                      F5                    # primitive(21)
                      18 3C                 # unsigned(60)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
             07                             # unsigned(7)
             D9 0130                        # tag(304)
                A1                          # map(1)
                   01                       # unsigned(1)
                   84                       # array(4)
                      00                    # unsigned(0)
                      F4                    # primitive(20)
                      00                    # unsigned(0)
                      F4                    # primitive(20)
       07                                   # unsigned(7)
       6C                                   # text(12)
          54727573742057616C6C6574          # "Trust Wallet"
    ```
    
- UR encoding in `ur:eth-sign-request/<message>` format

</details>

<details>

<summary>Example Polygon signature response</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'd4f0a7bcd95bba1fbb1051885054730e3f47064288575aacc102fbbf6a9a14daa066991e360d3e3406c20c00a40973eff37c7d641e5b351ec4a99bfe86f335f713', ; signature
    3: "NGRAVE Zero" ; device name
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A3                                      # map(3)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 41                                # bytes(65)
          D4F0A7BCD95BBA1FBB1051885054730E3F47064288575AACC102FBBF6A9A14DAA066991E360D3E3406C20C00A40973EFF37C7D641E5B351EC4A99BFE86F335F713 
       03                                   # unsigned(3)
       6B                                   # text(11)
          4E4752415645205A65726F            # "NGRAVE Zero"
    ```
    
- UR encoding in `ur:eth-signature/<message>` format

</details>

### Transaction verification guidance

The watch-only wallet should indicate the transaction information when the signature request is generated. Once the request is received by the offline signer, the user should be able to verify the correspondence of the transaction content he is about to sign.

The transaction needs to be decoded by the offline signer depending on the data type, e.g. RLP decoding for the first type. Once decoded, the following generic information can be displayed by the offline signer:

- Sender address
- Receiver address
- Nonce
- Max fee
- Amount

The EVM blockchain is identified using the chain ID and should be clearly notified to the user, as well as the specific coin used on the blockchain and associated to the amount and the max fee.

In case of more complex transactions, the offline signer should be able to recognise the transaction and displays additional required information:

1. Specific message: the message associated to the transaction should be decoded from the data to sign and displayed to the user.
2. ERC20 token or ERC721 NFT transfers: specific information are included in the data, such as contract address, receiver address, amount of token or NFT (see [[goethereumbook/transfer-tokens]](https://goethereumbook.org/en/transfer-tokens/)).
3. Smart contracts: they are numerous and diverse and needs to implement Application Binary Interface [[ABI]](https://docs.soliditylang.org/en/develop/abi-spec.html) in order to decode the interface functions of the contract and to display the necessary information to the user. 

## II - 3. Solana transactions

Keystone has proposed and implemented the specific UR types `sol-sign-request` and `sol-signature` for signing Solana transactions in [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer). In this document, we present their integration into the common UR types `crypto-sign-request` and `crypto-signature`.

### Integration with the generic UR types registry

- **Solana sign request**

A Solana transaction is uniquely identified through the URI format `bc-coin://ed25519/508`, information shared in `crypto-coin-identity` UR type.

Several fields are required for requesting the signature of a Solana transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Type of data to sign, if not specified, transaction type is used by default
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `sol-sign-request` and `crypto-sign-request`.

| crypto-sign-request fields
Associated number corresponds to the order in UR type | Corresponding sol-sign-request fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id | No corresponding field, serves as identifier for SOL transactions |
| 3. derivation-path | 3. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. master-fingerprint (optional) | No corresponding field, adds potential verification step |
| 6. origin (optional) | 5. origin (optional) |
| 7.1. sol-meta.data-type (optional, transaction type by default) | 6. type (optional, transaction type by default) |
| 7.2. sol-meta.address (optional) | 4. address (optional, redundant with derivation-path where the public key is already included) |

`crypto-sign-request` UR type provides the exact same information as `sol-sign-request` and has even the advantage to provide an additional optional field: the master fingerprint for verification purposes.

- **Solana signature**

A Solana sign request results in a signed transaction after user verification. 

The required fields for a Solana signature are the signature itself and potentially the request identifier to identify uniquely the response. 

The following table indicates the corresponding fields between the UR types `sol-signature` and `crypto-signature`.

| crypto-signature fields
Associated number corresponds to the order in UR type | Corresponding sol-signature fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | No corresponding field, provides an additional description |

`crypto-signature` UR type provides the exact same information as `sol-signature` and has even the advantage to provide an additional optional field to describe the origin with the device name for example.

### Use case

The watch-only wallet generates a QR code containing the `crypto-sign-request` UR type with the following information:

1. (Recommended) Request ID, an universally unique identifier (UUID) of the transaction
2. Solana coin identity
3. Derivation path of the key requested to sign the request
4. Transaction to sign
5. (Recommended) Master fingerprint of the master public key
6. (Recommended) Origin to indicate the wallet name
7. (Optional) Data type with either transaction or message type (if not specified, transaction type is defined by default)
8. (Optional) Sender address for verification, redundant however with derivation path where the public key is already included

The offline signer decodes the transaction depending on the data type and responds to the request with the signature once the transaction has been verified and approved by the user. The offline signer generates a QR code containing the `crypto-signature` UR type with the following information:

1. (Mandatory if provided in the request) Request ID indicating the UUID of the transaction
2. Signature 
3. (Recommended) Origin to indicate the name of the offline signer

### Example

An example illustrates how the signing protocol works on Solana blockchain:

ToDo update example

<details>

<summary>Example Solana signature request</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'01000103c8d842a2f17fd7aab608ce2ea535a6e958dffa20caf669b347b911c4171965530f957620b228bae2b94c82ddd4c093983a67365555b737ec7ddc1117e61c72e0000000000000000000000000000000000000000000000000000000000000000010295cc2f1f39f3604718496ea00676d6a72ec66ad09d926e3ece34f565f18d201020200010c0200000000e1f50500000000', ; sign-data
    3: 303( ; #6.303(crypto-hdkey)
         {3: h'02EAE4B876A8696134B868F88CC2F51F715F2DBEDB7446B8E6EDF3D4541C4EB67B', ; key-data
    		  6: 304({1: [44, true, 501, true, 0, true, 0, true]}) ; origin m/44’/501’/0’/0’
         }),
    5: "Trust Wallet", ; wallet name
    6: 1 ; type = transaction
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A5                                      # map(5)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 96                                # bytes(150)
          01000103C8D842A2F17FD7AAB608CE2EA535A6E958DFFA20CAF669B347B911C4171965530F957620B228BAE2B94C82DDD4C093983A67365555B737EC7DDC1117E61C72E0000000000000000000000000000000000000000000000000000000000000000010295CC2F1F39F3604718496EA00676D6A72EC66AD09D926E3ECE34F565F18D201020200010C0200000000E1F50500000000 
       03                                   # unsigned(3)
       D9 012F                              # tag(303)
          A2                                # map(2)
             03                             # unsigned(3)
             58 21                          # bytes(33)
                02EAE4B876A8696134B868F88CC2F51F715F2DBEDB7446B8E6EDF3D4541C4EB67B 
             06                             # unsigned(6)
             D9 0130                        # tag(304)
                A1                          # map(1)
                   01                       # unsigned(1)
                   88                       # array(8)
                      18 2C                 # unsigned(44)
                      F5                    # primitive(21)
                      19 01F5               # unsigned(501)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
       05                                   # unsigned(5)
       6C                                   # text(12)
          54727573742057616C6C6574          # "Trust Wallet"
       06                                   # unsigned(6)
       01                                   # unsigned(1)
    ```
    
- UR encoding in `ur:sol-sign-request/<message>` format

</details>

<details>

<summary>Example Solana signature response</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'd4f0a7bcd95bba1fbb1051885054730e3f47064288575aacc102fbbf6a9a14daa066991e360d3e3406c20c00a40973eff37c7d641e5b351ec4a99bfe86f335f7' ; signature
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A2                                      # map(2)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 40                                # bytes(64)
          D4F0A7BCD95BBA1FBB1051885054730E3F47064288575AACC102FBBF6A9A14DAA066991E360D3E3406C20C00A40973EFF37C7D641E5B351EC4A99BFE86F335F7 
    ```
    
- UR encoding in `ur:sol-signature/<message>` format

</details>

### Transaction verification guidance

The watch-only wallet should indicate the transaction information when the signature request is generated. Once the request is received by the offline signer, the user should be able to verify the correspondence of the transaction content he is about to sign.

Solana transactions are based on instruction. The basic instruction to send SOL (see [[solanacookbook/basic-transactions]](https://solanacookbook.com/references/basic-transactions.html#how-to-send-sol)) should be decoded and displayed to the user with the necessary information:

- Sender address
- Receiver address
- Amount

For Solana, the transaction fees can only be estimated by the online watch-only wallet, making them unverifiable to the user.

In case of more complex transactions, the offline signer should be able to recognise the transaction and displays additional required information:

1. SPL token transfers (including NFT transfer): specific information are included in the data, such as contract address, receiver address, amount of token or NFT.
2. Smart contracts: as indicated, Solana transactions are based on instructions. Complex operations on the blockchain can contain multiple instructions that need to be decoded for user verification, as presented by Keystone in [[blinding-signing-on-solana]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/blinding-signing-on-solana.md).

## II - 4. Tezos transactions

[[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposes the specific UR types `xtz-sign-request` and `xtz-signature` for signing Tezos transactions. In this document, we present their integration into the common UR types `crypto-sign-request` and `crypto-signature`.

### Integration with the generic UR types registry

- **Tezos sign request**

A Tezos transaction is uniquely identified through the URI format `bc-coin://ed25519/1729`, `bc-coin://secp256k1/1729` or `bc-coin://P256/1729` depending on the elliptic curves selected to sign the transaction.This information is shared in `crypto-coin-identity` UR type.

Several fields are required for requesting the signature of a Tezos transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Type of data to sign, if not specified, operation type is used by default
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `xtz-sign-request` and `crypto-sign-request`.

| crypto-sign-request fields
Associated number corresponds to the order in UR type | Corresponding xtz-sign-request fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id specifying the elliptic curve for XTZ | 5. key-type   |
| 3. derivation-path | 4. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. master-fingerprint (optional) | 6. master-fingerprint (optional) |
| 6. origin (optional) | No corresponding field, provides an additional description |
| 7.1. xtz-meta.data-type (optional) | 3. data-type |

`crypto-sign-request` UR type provides the exact same information as `xtz-sign-request` and has even the advantage to provide an additional optional field with a text describing the origin (e.g. the wallet name).

- **Tezos signature**

A Tezos sign request results in a signed transaction after user verification. 

The required fields for a Tezos signature are the signature itself and potentially the request identifier to identify uniquely the response. 

The following table indicates the corresponding fields between the UR types `eth-signature` and `crypto-signature`.

| crypto-signature fields
Associated number corresponds to the order in UR type | Corresponding eth-signature fields
Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | No corresponding field, adds a description |

`crypto-signature` UR type provides the exact same information as `xtz-signature`, except regarding the optional `payload` field available in `xtz-signature`. This field is however a redundant information since the signature can be identified using the `request-id` field.

Additionally, `crypto-signature` has the advantage to provide an additional optional field to describe the origin with the device name for example.

### Use case

The watch-only wallet generates a QR code containing the `crypto-sign-request` UR type with the following information:

1. (Recommended) Request ID, an universally unique identifier (UUID) of the transaction
2. Tezos coin identity with the specific elliptic curve
3. Derivation path of the key requested to sign the request
4. Transaction to sign
5. (Recommended) Master fingerprint of the master public key
6. (Recommended) Origin to indicate the wallet name
7. (Optional) Data type (if not specified, operation type is defined by default)

The offline signer responds to the request with the signature containing the following fields:

1. (Recommended) Request ID indicating the UUID of the transaction
2. Signature 
3. (Recommended) Origin to indicate the name of the offline signer

### Example

An example illustrates how the signing protocol works on Tezos blockchain:

ToDo update example

<details>

<summary>Example Tezos signature request</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'a4025820190e4b0ed26eec520135c6deb5fd59434989ea811f1ccfc30205bea7816fa4ea030104446f01ffc8056f416972476170202d20426f756e6365', ; sign-data
    3: 1, ; operation data-type
    4: 303( ; #6.303(crypto-hdkey)
         {3: h'02320EC3FDC5604F151A90F36C88EC9588FC320981E3C26D73F290C016FD11CD9D', ; key-data
    		  6: 304({1: [44, true, 1729, true, 0, true, 0, true, 0, true]}) ; origin m/44’/1729’/0’/0’/0'
         }),
    5: 1 ; ed25519 key-type 
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A5                                      # map(5)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 3D                                # bytes(61)
          A4025820190E4B0ED26EEC520135C6DEB5FD59434989EA811F1CCFC30205BEA7816FA4EA030104446F01FFC8056F416972476170202D20426F756E6365 
       03                                   # unsigned(3)
       01                                   # unsigned(1)
       04                                   # unsigned(4)
       D9 012F                              # tag(303)
          A2                                # map(2)
             03                             # unsigned(3)
             58 21                          # bytes(33)
                02320EC3FDC5604F151A90F36C88EC9588FC320981E3C26D73F290C016FD11CD9D 
             06                             # unsigned(6)
             D9 0130                        # tag(304)
                A1                          # map(1)
                   01                       # unsigned(1)
                   8A                       # array(10)
                      18 2C                 # unsigned(44)
                      F5                    # primitive(21)
                      19 06C1               # unsigned(1729)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
       05                                   # unsigned(5)
       01                                   # unsigned(1)
    ```
    
- UR encoding in `ur:xtz-sign-request/<message>` format

</details>

<details>

<summary>Example Tezos signature response</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'9fb423ee0b1ad3d3ad359c22d1e79048789c232813663fd5d8a1223458082ea844f5e87bf77db3b997aa4c847e23047c042003e3b204cea9ae0e1bf6fdcaaf09' ; signature
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A2                                      # map(2)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 40                                # bytes(64)
          9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09 
    ```
    
- UR encoding in `ur:xtz-signature/<message>` format

</details>


### Transaction verification guidance

The watch-only wallet should indicate the transaction information when the signature request is generated. Once the request is received by the offline signer, the user should be able to verify the correspondence of the transaction content he is about to sign.

The Tezos transactions contain all the essential information for user verification:

- Sender address
- Receiver address
- Counter
- Amount
- Fee

Smart contracts decoding on Tezos are not addressed in this document.

## II - 5. MultiversX and Stellar transactions

For MultiversX and Stellar transaction, there are no existing signing protocol based on UR types. This document introduces the signing of MultiversX and Stellar transactions using the common UR types `crypto-sign-request` and `crypto-signature`.

### Integration with the generic UR types registry

- **MultiversX and Stellar sign request**

A MultiversX transaction is uniquely identified through the URI format `bc-coin://ed25519/508`, while a Stellar transaction is identified by `bc-coin://ed25519/148`. This information is shared in `crypto-coin-identity` UR type.

Several fields are required for requesting the signature of a MultiversX or Stellar transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Origin of the sign request

The listed information for requesting a MultiversX or Stellar signature can be included in the `crypto-sign-request` UR type. 

- **MultiversX and Stellar signature**

A MultiversX or Stellar sign request results in a signed transaction after user verification. 

The required fields for a MultiversX or Stellar signature are the signature itself and potentially the request identifier to identify uniquely the response. Additionally, a description can be provided to specify the device name for example.

This listed information for sending a MultiversX and Stellar signature can be included in `crypto-signature` UR type.

### Use case

The watch-only wallet creates a signature request containing the following fields:

1. (Recommended) Request ID, an universally unique identifier (UUID) of the transaction
2. MultiversX or Stellar coin identity
3. Data to sign
4. Derivation path of the key requested to sign the request
5. Transaction to sign
6. (Recommended) Master fingerprint of the master public key
7. (Recommended) Origin to indicate the wallet name

The offline signer responds to the request with the signature containing the following fields:

1. (Mandatory if provided in the request) Request ID indicating the UUID of the transaction
2. Signature 
3. (Recommended) Origin to indicate the name of the offline signer

### Example

An example illustrates how the signing protocol works on MultiversX blockchain:

ToDo update example

<details>

<summary>Example MultiversX signature request</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'f849808609184e72a00082271094000000000000000000000000000000000000000080a47f7465737432000000000000000000000000000000000000000000000000000000600057808080', ; sign-data
    3: 303( ; #6.303(crypto-hdkey)
          {3: h'024137C16B966ABAC1DF4CB893AF421EA0BE61A52266AA7B3AA993900F7139640F', ; key-data
           6: 304({1: [44, true, 508, true, 0, true, 0, true, 1, true]}) ; origin m/44’/501’/0’/0’/1'
         }),
    4: "Trust Wallet" ; wallet name
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A4                                      # map(4)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 4B                                # bytes(75)
          F849808609184E72A00082271094000000000000000000000000000000000000000080A47F7465737432000000000000000000000000000000000000000000000000000000600057808080 
       03                                   # unsigned(3)
       D9 012F                              # tag(303)
          A2                                # map(2)
             03                             # unsigned(3)
             58 21                          # bytes(33)
                024137C16B966ABAC1DF4CB893AF421EA0BE61A52266AA7B3AA993900F7139640F 
             06                             # unsigned(6)
             D9 0130                        # tag(304)
                A1                          # map(1)
                   01                       # unsigned(1)
                   8A                       # array(10)
                      18 2C                 # unsigned(44)
                      F5                    # primitive(21)
                      19 01FC               # unsigned(508)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
                      00                    # unsigned(0)
                      F5                    # primitive(21)
                      01                    # unsigned(1)
                      F5                    # primitive(21)
       04                                   # unsigned(4)
       6C                                   # text(12)
          54727573742057616C6C6574          # "Trust Wallet"
    ```
    
- UR encoding in `ur:egld-sign-request/<message>` format

</details>

<details>

<summary>Example MultiversX signature response</summary>

- CBOR diagnosis format:
    
    ```jsx
    {
    1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
    2: h'd4f0a7bcd95bba1fbb1051885054730e3f47064288575aacc102fbbf6a9a14daa066991e360d3e3406c20c00a40973eff37c7d641e5b351ec4a99bfe86f335f713', ; signature
    3: "NGRAVE Zero" ; device name
    }
    ```
    
- CBOR encoding (see playground [here](https://cbor.me/))
    
    ```jsx
    A3                                      # map(3)
       01                                   # unsigned(1)
       D8 25                                # tag(37)
          50                                # bytes(16)
             9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
       02                                   # unsigned(2)
       58 41                                # bytes(65)
          D4F0A7BCD95BBA1FBB1051885054730E3F47064288575AACC102FBBF6A9A14DAA066991E360D3E3406C20C00A40973EFF37C7D641E5B351EC4A99BFE86F335F713 
       03                                   # unsigned(3)
       6B                                   # text(11)
          4E4752415645205A65726F            # "NGRAVE Zero"
    ```
    
- UR encoding in `ur:egld-signature/<message>` format

</details>

Another example illustrates how the signing protocol works on Stellar blockchain:

ToDo update example

### Transaction verification guidance

The watch-only wallet should indicate the transaction information when the signature request is generated. Once the request is received by the offline signer, the user should be able to verify the correspondence of the transaction content he is about to sign.

- **MultiversX transaction decoding**

The MultiversX transactions contain all the essential information for user verification:

- Sender address
- Receiver address
- Amount
- Fee

More complex transactions include an additional data field that needs to be decoded in order to display the following information to the user:

- ESDT token transfer: the additional information contains the token identifier and the value to transfer (see [[docs.multiversx/esdt-transfers]](https://docs.multiversx.com/tokens/esdt-tokens#transfers)).
- ESDT NFT transfer: the additional information contains the collection identifier, the NFT nonce, the quantity and the destination address (see [[docs.multiversx/nft-transfers]](https://docs.multiversx.com/tokens/nft-tokens#transfers)).
- Smart contracts: the data field contains several information that needs to be decoded according to the recognized smart contract.
- **Stellar transaction decoding**

The Stellar transactions contain all the essential information for user verification:

- Sender address
- Receiver address
- Counter
- Amount
- Fee

At the time of writing this document, smart contracts on Stellar are only in a research phase and are therefore not addressed.

---

# IV - Considerations

## Backwards compatibility

We have listed below the watch-only wallets allowing the signing of transactions through UR types.

| Coins supported | Sign request UR type | Signature UR type | Watch-only wallet |
| --- | --- | --- | --- |
| Bitcoin (BTC) | `crypto-psbt` | `crypto-psbt` | [Bluewallet](https://support.keyst.one/3rd-party-wallets/bitcoin-wallets/bluewallet) <br> [Sparrow](https://support.keyst.one/3rd-party-wallets/bitcoin-wallets/sparrow) <br> [Specter](https://support.keyst.one/3rd-party-wallets/bitcoin-wallets/specter) <br> [Nunchuk](https://support.keyst.one/3rd-party-wallets/bitcoin-wallets/nunchuk) <br> [Simple Bitcoin Wallet](https://support.keyst.one/3rd-party-wallets/bitcoin-wallets/simple-bitcoin-wallet) |
| Ethereum (ETH) <br> EVM chains | `eth-sign-request` | `eth-signature` | [Metamask](https://support.keyst.one/3rd-party-wallets/eth-and-web3-wallets-keystone/bind-metamask-with-keystone) <br> [Rabby wallet](https://support.keyst.one/3rd-party-wallets/eth-and-web3-wallets-keystone/rabby-wallet) |
| Solana (SOL) | `sol-sign-request` | `sol-signature` | [Solflare](https://support.keyst.one/3rd-party-wallets/solana-wallets/solflare-mobile) |
| Aptos (APT) | `aptos-sign-request` | `aptos-signature` | [Fewcha wallet](https://support.keyst.one/3rd-party-wallets/aptos-wallets/fewcha-wallet-extension) <br> [Petra](https://support.keyst.one/3rd-party-wallets/aptos-wallets/petra-aptos-wallet-extension) |
| Cosmos (ATOM) | `cosmos-sign-request` | `cosmos-signature` | [Keplr](http://support.keyst.one/3rd-party-wallets/cosmos-wallets/keplr-extension) |
| Near (NEAR) | `near-sign-request` | `near-signature` | [Sender wallet](https://support.keyst.one/3rd-party-wallets/near-wallets/sender-wallet-extension) |
| Tezos (XTZ) | `xtz-sign-request` | `xtz-signature` | No known wallet |

The signing communication protocol is fully compatible with existing proposals and implementations based on UR types for several blockchains (e.g. Bitcoin with [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md), Ethereum with [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527), Solana with [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer) and Tezos with [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md)). For the blockchains without reference implementation (e.g. Elrond and Stellar), this document proposes new UR types as communication layer via QR code between a watch-only wallet and an offline signer to sign transaction. 

The existing QR protocol between the NGRAVE watch-only wallet, LIQUID, and the NGRAVE offline signer, ZERO, presents major changes with the proposed communication. These changes are however needed for better standardization between the QR-based hardware wallet and to enhance overall security with a common definition on the transaction verification. 

## Security consideration

The protocol should ensure the “What You See Is What You Sign” property, i.e. the message content can not be changed during the signature process. The user needs to confirm the transaction details he is about to sign on the offline signer based on the transaction verification guidance presented for each blockchain. Any inconsistency with the transaction initiated on the watch-only wallet will alert the user on the potential malicious transaction. 

The decoded transaction on the offline signer should be presented in a human-readable format, but in case of undecodable transaction (e.g. with smart contracts unknown to the device), the offline signer should warn the user and display the raw data.

---

# V - References

| Reference | Link |
| --- | --- |
| [BCR-2020-005] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md |
| [EIP-4527] | https://eips.ethereum.org/EIPS/eip-4527 |
| [TZIP-25] | https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md |
| [solana-qr-data-protocol] | https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer |
| [BCR-2020-006] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-006-urtypes.md |
| [BIP174] | https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki |
| [CBOR Tag] | https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml |
| [keytool-cli] | https://github.com/BlockchainCommons/keytool-cli/tree/master |
| [ur-registry-eth] | https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth |
| [ur-registry-sol] | https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-sol |
| [ur-registry-xtz] | https://socket.dev/npm/package/@airgap/ur-registry-xtz/overview/0.0.1-beta.0 |
| [BCR-2020-007] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-007-hdkey.md |
| [NBCR-2023-002] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md |
| [NBCR-2023-001] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md |
| [goethereumbook/transfer-tokens] | https://goethereumbook.org/en/transfer-tokens/ |
| [ABI]  | https://docs.soliditylang.org/en/develop/abi-spec.html |
| [solanacookbook/basic-transactions] | https://solanacookbook.com/references/basic-transactions.html#how-to-send-sol |
| [blinding-signing-on-solana] | https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/blinding-signing-on-solana.md |
| [docs.multiversx/esdt-transfers] | https://docs.multiversx.com/tokens/esdt-tokens#transfers |
| [docs.multiversx/nft-transfers] |  https://docs.multiversx.com/tokens/nft-tokens#transfers |