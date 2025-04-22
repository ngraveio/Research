# Signing Protocol

## NBCR-2023-003

© 2023 NGRAVE
Authors: Mathieu Da Silva, Irfan Bilaloglu <br/>
Date: April 26, 2023

Revised: April 14, 2025

---

# I - Introduction

The goal of this document is to propose a QR protocol standardizing the communication between an offline wallet signing the transactions and a watch-only wallet preparing and broadcasting the transactions to the blockchain.

This protocol specifies the QR formats for the signing request and the signature. The signing request is created by the watch-only wallet including the transaction to sign and the signer account to be sent via QR code to the offline signer. The transaction is signed by the offline signer that generates a QR code to send back the signed transaction. 

For some blockchains, standards already exist, as proposed in [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md) for Bitcoin, in [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) for Ethereum blockchain, in [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) for Tezos blockchain and in Keystone repository [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer) for Solana blockchain. 

This document defines a generic QR protocol to sign on any blockchain and extends at the same time the QR code-based protocol to blockchains where a standard has not been defined yet.

## Specification

**Offline signer**: An offline signer is a device or application which holds the user private keys and does not have network access.

**Watch-only wallet**: A watch-only wallet is a wallet that has network access and can interact with the blockchain.

## Motivation to use Uniform Resources (UR) types

The BlockchainCommons (BC) have published a series of data transmission protocol called Uniform Resources (UR). It provides a basic method to encode data into animated QR Codes. The UR have been used for different proposals of QR-code based data transmission (e.g. [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527),  [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) etc…). The UR types form the basis of the data transmission protocol for the following reasons:

- **Standardization**: a standard already commonly used by other hardware wallets (Keystone, AirGap, CoolWallet, D’Cent).
- **Data compression**: Byte savings using CBOR compared to text encoding.
- **Human-friendly text encoding**: Format is defined as `ur:<type>/<message>` (details can be found [here](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md))
- **Efficiency of QR code encoding**: data is encoded as CBOR (Concise Binary Object Representation) supporting sizes up to 2^32-1 bytes (~ 4 GB) compared to the largest QR code ("version 40") consisting of 2953 bytes (details can be found in [[BCR-2020-005]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md))
- **Time-sequence QR code**: ability to break up messages too large to fit into a QR code into a sequence of QR codes using sophisticated "fountain codes."
- **Error detection**: CRC32 checksum on the entire message in each part to tie them together and ensure the transmitted message has been reconstructed.

## Technical specifications

The following parameters are recommended for generating dynamic QR codes to ensure optimal readability and interoperability with a wide range of scanners:

- The fragment size should not exceed **200 characters** per QR code.
- The frame interval between successive QR codes should be defined at **150 ms**.
- QR codes should use **Medium (M)** error correction level.

## Limitation

New blockchains can be integrated seamlessly due to the blockchain-agnostic nature of the communication protocol defined in this document. However, blockchain-specific requirements—such as delegating account selection to the offline signer in the case of Cosmos—may necessitate adaptations to the QR protocol. Any such modifications must preserve backward compatibility with existing blockchain-agnostic UR types.

This document focuses only on the signing process between the offline signer and the watch-only wallet. The sync communication protocol via QR codes is detailed in [[NBCR-2023-002]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md) specification.

---

# II - Signing protocol

As preliminary step before signing any transaction, the offline signer needs to provide via QR code the necessary accounts’ information to the watch-only wallet. 

The user can initiate the signing protocol on a watch-only wallet synchronized with an offline signer by following the described process:

1. The watch-only wallet creates a QR code including the unsigned data to be signed based on the user inputs (sender and receiver addresses, amount, transaction fee, etc…)
2. The offline signer reads the QR code including the signature request, then decodes the transaction to display the transaction information in order to ensure the transaction integrity.
3. After user confirmation, the offline signer signs the transaction and provides the signature to the watch-only wallet via a QR code.
4. The watch-only wallet receiving the signature is able to broadcast the transaction to the blockchain.

The existing communication protocol for signing are based on UR types specific to each blockchain. Except for Bitcoin with `psbt` UR type, the signing request prepared by the watch-only wallet is embedded in a UR type with the extension `sign-request`. Once the transaction is signed, the offline signer sends the signature back to the watch-only wallet by encoding the data in a UR type with the extension `signature`.

This document proposes to standardize into one UR type `sign-request` for the signing request and another UR type `sign-response` for the signed message.

The following table listed the existing UR types depending on the blockchain and introduced the new UR types intended to be blockchain agnostic.

| Blockchain | Sign request  | Signature  | Owner | Definition | Implementation |
| --- | --- | --- | --- | --- | --- |
| Bitcoin | psbt <br> Tag: 40310 | psbt <br> Tag: 40310 | Blockchain Commons (BC) | [[BCR-2020-006]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-006-urtypes.md) | [[keytool-cli]](https://github.com/BlockchainCommons/keytool-cli/tree/master) |
| Bitcoin | (Deprecated) crypto-psbt <br> Tag: 310 | crypto-psbt <br> Tag: 310 | Blockchain Commons (BC) | [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md) | [[keytool-cli]](https://github.com/BlockchainCommons/keytool-cli/tree/master) |
| Ethereum/EVM | eth-sign-request <br> Tag: 401 | eth-signature <br> Tag: 402 | Keystone | [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) | [[ur-registry-eth]](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth) |
| Solana| sol-sign-request <br> Tag: 1101 | sol-signature <br> Tag: 1102 | Keystone | [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer)  | [[ur-registry-sol]](https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-sol) |
| Tezos | xtz-sign-request <br> Tag: 501 | xtz-signature <br> Tag: 502 | Airgap  | [[tzip-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) | [[ur-registry-xtz]](https://socket.dev/npm/package/@airgap/ur-registry-xtz/overview/0.0.1-beta.0) |
| Cosmos | cosmos-sign-request <br> Tag: 4101 | cosmos-signature <br> Tag: 4102 | Keystone  | ? | [[ur-registry-cosmos]](https://github.com/KeystoneHQ/keystone-sdk-base/tree/master/packages/ur-registry-cosmos) |
| **Generic** | **sign-request** <br> **Tag: 41411** | **sign-response** <br> **Tag: 41412** | **Ngrave** | **This document** | **[[ur-registry]](https://github.com/ngraveio/ur-registry/tree/main)** |

The specification for each UR type contains CBOR structure, expressed thereafter in Concise Data Definition Language [[CDDL]](https://datatracker.ietf.org/doc/html/rfc8610).

## II - 1. Blockchain-specific UR types registry

The UR types designed to be used for specific blockchain are based on existing specification referenced in each Section. 

### Definition of common embedded CBOR tags

The UR types for signing embed common tags, described in the following list:

- Universal Unique IDentifier, noted as UUID, are CBOR binary strings tagged with #6.37, per the IANA [[CBOR Tag]](https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml).
- `keypath` UR type specifies the key derivation path of the signer account. This structure is embedded with the tag #6.40304 as defined in [[BCR-2020-007]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-007-hdkey.md).

### Bitcoin UR type

For the Bitcoin blockchain, UR types are limited to the Partially Signed Bitcoin Transaction (PSBT) format as defined in [[BIP-174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki). 

- **CDDL for BTC PSBT** `psbt`

UR Type Tag: `#6.40310` (defined in [[BCR-2020-006]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-006-urtypes.md)). 

```
crypto-psbt = bytes
```

The `psbt` UR type is used identically in both signing requests and signature responses.

The main limitation is the inability to support signing other types of data, such as message signing defined in [[BIP-322]](https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki). This limitation will be addressed by the proposed blockchain-agnostic sign-request UR type.

### Ethereum UR types

For Ethereum and other EVM blockchains, the UR types are based on the [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposal. The EIP still uses a deprecated version of the UR type `crypto-keypath` (#6.304) differing only from the new version `keypath` by its CBOR tag and its UR name.

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

For Solana blockchain, the UR types are based on the proposed implementation by Keystone [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer). The proposal for Solana also uses the deprecated version `crypto-keypath` in the sign request.

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

For Tezos blockchain, the UR types are based on the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal. The proposal for Tezos also uses the deprecated version `crypto-keypath` in the sign request.

- **CDDL for XTZ signature request** `xtz-sign-request`

This UR type belongs to the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal and is tagged with #6.501. 

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

This UR type belongs to the [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposal and is tagged with #6.502.

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

### Cosmos UR types

The Cosmos-specific UR types are based on the [[ur-registry-cosmos]](https://github.com/KeystoneHQ/keystone-sdk-base/tree/master/packages/ur-registry-cosmos) implementation. This implementation still relies on the deprecated `crypto-keypath` UR type (CBOR tag `#6.304`), which differs from the newer `keypath` type only by its CBOR tag and UR name.

- **CDDL for Cosmos signature request** `cosmos-sign-request`

UR Type Tag: `#6.4101`

```
; Metadata for the signing request for Cosmos chains.
; 
sign-data-type = {
    type: int .default amino
}

amino = 1
direct = 2
textual = 3
message = 4

cosmos-sign-request = (
    ?request-id: uuid,
    sign-data: bytes,
    ?data-type: sign-data-type,
    derivation-paths: [#6.304(crypto-keypath)], 
    ?addresses: [string],
    ?origin: text  
)
request-id = 1
sign-data = 2
data-type = 3
derivation-paths = 4
addresses = 5
origin = 6
```

- **CDDL for Cosmos signature response** `cosmos-signature`

UR Type Tag: `#6.4102`

```
cosmos-signature  = (
    request-id: uuid,
    signature: bytes,
    public-key: bytes
)

request-id = 1
signature = 2
public-key = 3
```

## II - 2. Blockchain-agnostic UR types registry

This section defines a set of common UR types intended to support signing workflows across multiple blockchains.

- **CDDL for generic signature request** `sign-request`

UR Type Tag: `#6.41411`

This UR type encapsulates a generic signing request and includes:
- A `coin-identity`  UR type (`#6.41401`), as defined in [[NBCR-2023-001]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md), which identifies the elliptic curve and the blockchain for which the signing operation is intended.
- An optional derivation path and address, typically a single element. However, for certain blockchains like Cosmos, they may contain an array, allowing the signer to select an account interactively.

```
sign-request = {
    ?request-id: uuid,                            ; Unique identifier for the signing request
    coin-id: #6.41401(coin-identity),             ; Defines elliptic curve and blockchain context
    ?derivation-path: derivation-path-selection,  ; Key path(s) for signature derivation
    sign-data: bytes,                             ; Serialized transaction data
    ?origin: text,                                ; Origin of the request (e.g., wallet identifier)
    ?tx-type: int .default 1,                     ; Blockchain-specific transaction type
    ?address: address-selection                   ; Address(es) associated to the signer's account
}

; If both derivation paths and addresses are provided as arrays, they must contain the same number of elements.
; When multiple elements are present, the offline signer must select one account for signing.

derivation-path-selection = keypath-type / [2* keypath-type]
address-selection = address-type / [2* address-type]

keypath-type = #6.40304(keypath)
address-type = string / bytes

request-id = 1
coin-id = 2
derivation-path = 3
sign-data = 4
origin = 5
tx-type = 6
address = 7
```

The `tx-type` field reuses enumerations from existing UR types such as `sign-data-type` (Ethereum), `type` (Solana), `key-type` (Tezos), and `data-type` (Cosmos).
When integrating a new blockchain, a new enumeration should be defined if multiple transaction types are supported.

- **CDDL for generic signature response** `sign-response`

UR Type Tag: `#6.41412`

```
sign-response = {
    ?request-id: uuid,   ; Echoes the request-id if present in the corresponding sign-request
    signature: bytes,    ; Resulting signature
    ?origin: text,       ; Device or signer name
    ?public-key: bytes   ; Public key used to produce the signature (required only in multi-account selection)
}

request-id = 1
signature = 2
origin = 3
public-key = 4
```

The public key field is only required when the corresponding `sign-request` contains an array of derivation paths and/or addresses. It serves to indicate which account (from the provided array) was selected by the signer for this signature.

[View the sign-request protocol graph](../images/sign-request.pdf) for a visual overview of `sign-request` and its associated standards.

---

# III - Use cases on several blockchains

This Section gives more details on the use case for each blockchain, the signing process with examples and provides guidance to enable user verification of the transaction on the offline signer.

## III - 1. Bitcoin transactions

Regular Bitcoin transactions follow the [[BIP-174]](https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki) standard for Partially Signed Bitcoin Transactions (PSBT). Accordingly, the primary UR type used for Bitcoin signing is `psbt`. This section describes how the `psbt` format is integrated into the generic `sign-request` and `sign-response` UR types. Additionally, using the generic UR types supports signing other transaction types on Bitcoin, such as message signing.

### Integration with the generic UR types registry

- **Bitcoin sign request**

A Bitcoin transaction is identified via the `coin-identity` UR type, where:

- The elliptic curve is secp256k1
- The SLIP-44 coin type is 0

The required data for signing includes:
- The sign data byte 
- An optional tx-type indicating the nature of the Bitcoin operation

Supported Bitcoin tx-type values (to be extended to new transaction types when needed):

- `1`: PSBT (default if omitted)
- `2`: ECDSA (raw hex message signing using account private key)
- `3`: BIP-322 structured message signing

Field mapping between `sign-request` and `psbt`:

| sign-request fields <br> Associated number corresponds to the order in UR type | Corresponding crypto-psbt fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | — (Provides request correlation)  |
| 2. coin-id | — (Identifies Bitcoin context) |
| 3. derivation-path (optional) | — (Recommended for account origin) |
| 4. sign-data | `bytes` (raw unsigned transaction) |
| 5. origin (optional) | — (Describes the source wallet) |
| 6. tx-type (optional) | — (Describes the type of transaction to be signed) |

Using sign-request instead of raw `psbt` extends signing to other transaction types than PSBT and it allows for richer metadata:
- A request-id for signature correlation
- Derivation path(s) and master fingerprint to identify the signing account(s)
- A textual origin field to identify the wallet source

- **Bitcoin signature**

A Bitcoin sign request results in a signed transaction after user verification. A Bitcoin signature is contained in the same format as the request, a PSBT transaction.

The only required field for a Bitcoin signature is the bytes composing the signed PSBT transaction, corresponding to the `signature` fields of `sign-response` UR type.

The following table indicates the corresponding fields between the UR types `psbt` and `sign-response`.

| sign-response fields <br> Associated number corresponds to the order in UR type | Corresponding crypto-psbt fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id | — (Correlates with request) |
| 2. signature | `bytes` (signed transaction) |
| 3. origin | — (Identifies signer device) |

Advantages of using `sign-response` over raw `psbt`:
- Optional `request-id` for matching with sign-request
- Optional `origin` to describe the signer (e.g., device or firmware version)

### Example 1 - PSBT

An example illustrates how the signature request and response for Bitcoin PSBT are encoded.

<details>

<summary>Example BTC signature request</summary>

- BTC PSBT
    
```
0x70736274ff01009a020000000258e87a21b56daf0c23be8e7070456c336f7cbaa5c8757924f545887bb2abdd750000000000ffffffff838d0427d0ec650a68aa46bb0b098aea4422c071b2ca78352a077959d07cea1d0100000000ffffffff0270aaf00800000000160014d85c2b71d0060b09c9886aeb815e50991dda124d00e1f5050000000016001400aea9a2e5f0f876a588df5546e8742d1d87008f000000000000000000
```

- CBOR diagnosis format:
    
```
{
1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 8, ; secp256k1 curve
     2: 0 ; Bitcoin BIP44
    }),
3: 40304({1: [44, true, 0, true, 0, true, 0, false, 0, false], ; #6.40304(keypath) m/44'/0'/0'/0/0
          2: 934670036}), ; master fingerprint
4: h'70736274ff01009a020000000258e87a21b56daf0c23be8e7070456c336f7cbaa5c8757924f545887bb2abdd750000000000ffffffff838d0427d0ec650a68aa46bb0b098aea4422c071b2ca78352a077959d07cea1d0100000000ffffffff0270aaf00800000000160014d85c2b71d0060b09c9886aeb815e50991dda124d00e1f5050000000016001400aea9a2e5f0f876a588df5546e8742d1d87008f000000000000000000', ; sign-data
5: "NGRAVE LIQUID" ; wallet name
}
```

- CBOR encoding 
    
```
A5                                      # map(5)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         3B5414375E3A450B8FE1251CBC2B3FB5 
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         08                             # unsigned(8)
         02                             # unsigned(2)
         00                             # unsigned(0)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F4                          # primitive(20)
            00                          # unsigned(0)
            F4                          # primitive(20)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 A7                                # bytes(167)
      70736274FF01009A020000000258E87A21B56DAF0C23BE8E7070456C336F7CBAA5C8757924F545887BB2ABDD750000000000FFFFFFFF838D0427D0EC650A68AA46BB0B098AEA4422C071B2CA78352A077959D07CEA1D0100000000FFFFFFFF0270AAF00800000000160014D85C2B71D0060B09C9886AEB815E50991DDA124D00E1F5050000000016001400AEA9A2E5F0F876A588DF5546E8742D1D87008F000000000000000000 
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
```

- UR encoding 

```
ur:sign-request/onadtpdagdfrghbbemhyftfebdmyvydacerfdnfhreaotaoyrhoeadayaoaeaxtantjooeadlecsdwykaeykaeykaewkaewkaocyemrewytyaahdosjojkidjyzmadaenyaoaeaeaeaohdvsknclrejnpebncnrnmnjojofejzeojlkerdonspkpkkdkykfelokgprpyutkpaeaeaeaeaezmzmzmzmlslgaaditiwpihbkispkfgrkbdaslewdfycprtjsprsgksecdratkkhktikewdcaadaeaeaeaezmzmzmzmaojopkwtayaeaeaeaecmaebbtphhdnjstiambdassoloimwmlyhygdnlcatnbggtaevyykahaeaeaeaecmaebbaeplptoevwwtyakoonlourgofgvsjydpcaltaemyaeaeaeaeaeaeaeaeaeahjnglflgmfphffecxgsgagygogafyhebgfzdl
```

</details>

<details>

<summary>Example BTC signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; request-id
2: h'0200000001be7ae2cd723a2aea1db1ef208e7a1cc2e24cabe04b0662a37ae6d7e75e086519000000006b483045022100d2b7da967b6ca073595a0c00e4ab03e07c3e65a91fd92e0377b43a350d0afdf402200e87776838ca6f8753c947cd0dec77dd9a7a2ae3c50fd36aed2d887a522af65f012102c08b1d6b9534b9a8aa19b351fa9bdefc7d11ad1ed5ae02fb99736a51bc8b5475fdffffff02400d0300000000001976a914cd3dc3e2147620c8484ac6ba1ebdd53e9a23be7188ac905ef205000000001976a914036606840cc20ce9fddcbc384bc064f0e505afab88ac00000000
', ; Signed PSBT
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding 
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         3B5414375E3A450B8FE1251CBC2B3FB5 
   02                                   # unsigned(2)
   58 E2                                # bytes(226)
      0200000001BE7AE2CD723A2AEA1DB1EF208E7A1CC2E24CABE04B0662A37AE6D7E75E086519000000006B483045022100D2B7DA967B6CA073595A0C00E4AB03E07C3E65A91FD92E0377B43A350D0AFDF402200E87776838CA6F8753C947CD0DEC77DD9A7A2AE3C50FD36AED2D887A522AF65F012102C08B1D6B9534B9A8AA19B351FA9BDEFC7D11AD1ED5AE02FB99736A51BC8B5475FDFFFFFF02400D0300000000001976A914CD3DC3E2147620C8484AC6BA1EBDD53E9A23BE7188AC905EF205000000001976A914036606840CC20CE9FDDCBC384BC064F0E505AFAB88AC00000000 
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding 

```
ur:sign-response/otadtpdagdfrghbbemhyftfebdmyvydacerfdnfhreaohdvoaoaeaeaeadrnknvosnjpftdrwdcapawscxmnkncesavogspyvtgramidotknvatsvdhyayihcfaeaeaeaejefddyfeaoclaetdrltnmtkgjznbjkhkhtbnaevepyaxvtkefmihptcttadmaxktqzftecbtbkzcwkaocxbaltktisetsgjlltgusoflsnbtwpktutnykndrvlskbsteimwedplokngmdrynheadclaortlucajemdeerhpdpkcfqdgyzsndueztkibypmcktlplaozonljkimgyrflughkpzczmzmzmaofzbtaxaeaeaeaeaecfkoptbbsnfssrvobbkocxspfdgeswrdckrytlfmnycnrnjslopsmhhywzahaeaeaeaecfkoptbbaxiyamlrbnsabnwlzcuorfetgrrtiewtvwahpepylopsaeaeaeaeaxjeglflgmfphffecxhtfegmgwmntydtty
```

</details>

### Example 2 - ECDSA signing

An example illustrates how the signature request and response for Bitcoin ECDSA message signing are encoded.

<details>

<summary>Example BTC message signature request</summary>

- Message to sign
    
```
0x2fb2d3f6b0a0b914fb744be4a3a59e3f8f6398aa
```

- CBOR diagnosis format:
    
```
{
1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 8, ; secp256k1 curve
     2: 0 ; Bitcoin BIP44
    }),
3: 40304({1: [84, true, 0, true, 0, true, 0, false, 0, false], ; #6.40304(keypath) m/84'/0'/0'/0/0
          2: 934670036}), ; master fingerprint
4: h'2fb2d3f6b0a0b914fb744be4a3a59e3f8f6398aa', ; sign-data
5: "NGRAVE LIQUID", ; wallet name
6: 2 ; ECDSA message signing
}
```

- CBOR encoding 
    
```
A6                                      # map(6)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         3B5414375E3A450B8FE1251CBC2B3FB5 # ";T\u00147^:E\v\x8F\xE1%\u001C\xBC+?\xB5"
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         08                             # unsigned(8)
         02                             # unsigned(2)
         00                             # unsigned(0)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 54                       # unsigned(84)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F4                          # primitive(20)
            00                          # unsigned(0)
            F4                          # primitive(20)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   54                                   # bytes(20)
      2FB2D3F6B0A0B914FB744BE4A3A59E3F8F6398AA # "/\xB2\xD3\xF6\xB0\xA0\xB9\u0014\xFBtK䣥\x9E?\x8Fc\x98\xAA"
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
   06                                   # unsigned(6)
   02                                   # unsigned(2)
```

- UR encoding 

```
ur:sign-request/oladtpdagdfrghbbemhyftfebdmyvydacerfdnfhreaotaoyrhoeadayaoaeaxtantjooeadlecsghykaeykaeykaewkaewkaocyemrewytyaaghdlprteynpfnbrhbbzojygrveotonnnfhmyiamkpkahjnglflgmfphffecxgsgagygogafyamaojsaovove
```

</details>

<details>

<summary>Example BTC message signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'3B5414375E3A450B8FE1251CBC2B3FB5'), ; request-id
2: h'3046022100e4e87c417196c6e5cd63f93e94929ccda6d04fc0a7446922baf3070e854ec4f4022100a1ecd098008329de9bc93fb2ded6aaceecc921f7183d6b3cfc673b3ef8af219e', ; Signed message
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding 
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         3B5414375E3A450B8FE1251CBC2B3FB5 # ";T\u00147^:E\v\x8F\xE1%\u001C\xBC+?\xB5"
   02                                   # unsigned(2)
   58 48                                # bytes(72)
      3046022100E4E87C417196C6E5CD63F93E94929CCDA6D04FC0A7446922BAF3070E854EC4F4022100A1ECD098008329DE9BC93FB2DED6AACEECC921F7183D6B3CFC673B3EF8AF219E # "0F\u0002!\u0000\xE4\xE8|Aq\x96\xC6\xE5\xCDc\xF9>\x94\x92\x9Cͦ\xD0O\xC0\xA7Di\"\xBA\xF3\a\u000E\x85N\xC4\xF4\u0002!\u0000\xA1\xECИ\u0000\x83)ޛ\xC9?\xB2\xDE֪\xCE\xEC\xC9!\xF7\u0018=k<\xFCg;>\xF8\xAF!\x9E"
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding 

```
ur:sign-response/otadtpdagdfrghbbemhyftfebdmyvydacerfdnfhreaohdfddyfgaoclaevevskefpjsmtswvwsniaytfmmwmonssnoltigwrtosfyincprdwfatbalpglsswkaoclaeoywptimkaelsdtuendsofhpruetbpktowpsoclylcsfsjefnztiofrfmyapeclnnaxjeglflgmfphffecxhtfegmgwcahtstvy
```

</details>

## II - 2. Ethereum transactions

[[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527) proposes the specific UR types `eth-sign-request` and `eth-signature` for signing Ethereum transactions and in the same way transactions on every EVM blockchains. In this document, we present their integration into the common UR types `sign-request` and `sign-response`.

### Integration with the generic UR types registry

- **Ethereum sign request**

An Ethereum transaction is uniquely identified thanks to the information shared in `coin-identity` UR type, i.e. secp256k1 as elliptic curve, 60 as coin type and chain ID as subtype to identify the EVM chain (e.g. 1 for Ethereum).

Several fields are required for requesting the signature of an Ethereum transaction:

- Unsigned transaction
- Type of data to sign
- Chain ID
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `eth-sign-request` and `sign-request`.

| sign-request fields <br> Associated number corresponds to the order in UR type | Corresponding eth-sign-request fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id specifying the chain ID for EVM blockchains | 4. chain-id  |
| 3. derivation-path | 5. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. origin (optional) | 7. origin |
| 6. tx-type | 3. data-type |
| 7. address (optional) | 6. address (optional, completing the provided derivation path for account identification) |

`sign-request` UR type provides the exact same information as `eth-sign-request`, with the advantage of `sign-request` to be blockchain-agnostic.

- **Ethereum signature**

An Ethereum sign request results in a signed transaction after user verification. 

The required fields for an Ethereum signature are the signature itself and potentially the request identifier to identify uniquely the response. Additionally, a description can be provided to specify the device name for example.

The following table indicates the corresponding fields between the UR types `eth-signature` and `sign-response`.

| sign-response fields <br> Associated number corresponds to the order in UR type | Corresponding eth-signature fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | 3. origin (optional) |

`sign-response` and `eth-signature` provides identical information, with the advantage of `sign-response` to be blockchain-agnostic.

### Example

An example illustrates how the signing protocol works on EVM blockchains:

<details>

<summary>Example Polygon signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 8, ; secp256k1 curve
     2: 60, ; Ethereum BIP44
     3: [137] ; Polygon chain ID
    }),
3: 40304({1: [44, true, 60, true, 0, true, 0, false, 0, false], ; #6.40304(keypath) m/44'/60'/0'/0/0
          2: 934670036}), ; master fingerprint
4: h'f849808609184e72a00082271094000000000000000000000000000000000000000080a47f7465737432000000000000000000000000000000000000000000000000000000600057808080', ; sign-data
5: "NGRAVE LIQUID", ; wallet name
6: 1, ; RLP transaction
7: h'1efecb61a2f80aa34d3b9218b564a64d05946290' ; address
}
```
 
- CBOR encoding
    
```
A7                                      # map(7)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A3                                # map(3)
         01                             # unsigned(1)
         08                             # unsigned(8)
         02                             # unsigned(2)
         18 3C                          # unsigned(60)
         03                             # unsigned(3)
         81                             # array(1)
            18 89                       # unsigned(137)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            18 3C                       # unsigned(60)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F4                          # primitive(20)
            00                          # unsigned(0)
            F4                          # primitive(20)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 4B                                # bytes(75)
      F849808609184E72A00082271094000000000000000000000000000000000000000080A47F7465737432000000000000000000000000000000000000000000000000000000600057808080 
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
   06                                   # unsigned(6)
   01                                   # unsigned(1)
   07                                   # unsigned(7)
   54                                   # bytes(20)
      1EFECB61A2F80AA34D3B9218B564A64D05946290 
```

- UR encoding 

```
ur:sign-request/osadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhotadayaocsfnaxlycsldaxtantjooeadlecsdwykcsfnykaeykaewkaewkaocyemrewytyaahdgryagalalnascsgljpnbaelfdibemwaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaelaoxlbjyihjkjyeyaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaehnaehglalalaahjnglflgmfphffecxgsgagygogafyamadatghckzesbhsoeyabkotgtfrmocsreieolgtahmwidmhterpwnyn
```

</details>

<details>

<summary>Example Polygon signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'd4f0a7bcd95bba1fbb1051885054730e3f47064288575aacc102fbbf6a9a14daa066991e360d3e3406c20c00a40973eff37c7d641e5b351ec4a99bfe86f335f713', ; signature
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding
    
```
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
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding 

```
ur:sign-response/otadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfptywtosrftahprdctrkbegylogdghjkbafhflamfwlohghtpsseaozorsimnybbtnnbiynlckenbtfmeeamsabnaeoxasjkwswfkekiieckhpecckssptndzelnwfecylbwaxjeglflgmfphffecxhtfegmgwoxurfzby
```

</details>

## II - 3. Solana transactions

Keystone has proposed and implemented the specific UR types `sol-sign-request` and `sol-signature` for signing Solana transactions in [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer). In this document, we present their integration into the common UR types `sign-request` and `sign-response`.

### Integration with the generic UR types registry

- **Solana sign request**

A Solana transaction is uniquely identified thanks to the information shared in `coin-identity` UR type, i.e. ed25519 as elliptic curve and 501 as coin type.

Several fields are required for requesting the signature of a Solana transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Type of data to sign, if not specified, transaction type is used by default
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `sol-sign-request` and `sign-request`.

| sign-request fields <br> Associated number corresponds to the order in UR type | Corresponding sol-sign-request fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id | No corresponding field, serves as identifier for SOL transactions |
| 3. derivation-path | 3. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. origin (optional) | 5. origin (optional) |
| 6. tx-type (optional, transaction type by default) | 6. type (optional, transaction type by default) |
| 7. address (optional) | 4. address (optional, completing the provided derivation path for account identification) |

`sign-request` UR type provides the exact same information as `sol-sign-request` with the advantage of `sign-request` to be blockchain-agnostic.

- **Solana signature**

A Solana sign request results in a signed transaction after user verification. 

The required fields for a Solana signature are the signature itself and potentially the request identifier to identify uniquely the response. 

The following table indicates the corresponding fields between the UR types `sol-signature` and `sign-response`.

| sign-response fields <br> Associated number corresponds to the order in UR type | Corresponding sol-signature fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | No corresponding field, provides an additional description |

`sign-response` UR type provides the exact same information as `sol-signature` and has even the advantage to provide an additional optional field to describe the origin with the device name for example.

### Example

An example illustrates how the signing protocol works on Solana blockchain:

<details>

<summary>Example Solana signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 6, ; ed25519 curve
     2: 501 ; Solana BIP44
    }),
3: 40304({1: [44, true, 501, true, 0, true, 0, true], ; #6.40304(keypath) m/44’/501’/0’/0’
          2: 934670036}), ; master fingerprint
4: h'01000103c8d842a2f17fd7aab608ce2ea535a6e958dffa20caf669b347b911c4171965530f957620b228bae2b94c82ddd4c093983a67365555b737ec7ddc1117e61c72e0000000000000000000000000000000000000000000000000000000000000000010295cc2f1f39f3604718496ea00676d6a72ec66ad09d926e3ece34f565f18d201020200010c0200000000e1f50500000000', ; sign-data
5: "NGRAVE LIQUID", ; wallet name
6: 1, ; sign-type-transaction
7: "9FPebKDGZAdcpT7SpfB1UowuqobV8Zww9TwPDSyzXJMr" ; address
}
```

- CBOR encoding 
    
```
A7                                      # map(7)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D # "\x9B\u001D\xEBM;}K\xAD\x9B\xDD+\r{=\xCBm"
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         06                             # unsigned(6)
         02                             # unsigned(2)
         19 01F5                        # unsigned(501)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         88                             # array(8)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            19 01F5                     # unsigned(501)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 96                                # bytes(150)
      01000103C8D842A2F17FD7AAB608CE2EA535A6E958DFFA20CAF669B347B911C4171965530F957620B228BAE2B94C82DDD4C093983A67365555B737EC7DDC1117E61C72E0000000000000000000000000000000000000000000000000000000000000000010295CC2F1F39F3604718496EA00676D6A72EC66AD09D926E3ECE34F565F18D201020200010C0200000000E1F50500000000 # "\u0001\u0000\u0001\u0003\xC8\xD8B\xA2\xF1\u007Fת\xB6\b\xCE.\xA55\xA6\xE9X\xDF\xFA \xCA\xF6i\xB3G\xB9\u0011\xC4\u0017\u0019eS\u000F\x95v \xB2(\xBA\xE2\xB9L\x82\xDD\xD4\xC0\x93\x98:g6UU\xB77\xEC}\xDC\u0011\u0017\xE6\u001Cr\xE0\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0010)\\\xC2\xF1\xF3\x9F6\u0004q\x84\x96\xEA\u0000gmjr\xECf\xAD\t\xD9&\xE3\xEC\xE3OV_\u0018\xD2\u0001\u0002\u0002\u0000\u0001\f\u0002\u0000\u0000\u0000\u0000\xE1\xF5\u0005\u0000\u0000\u0000\u0000"
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
   06                                   # unsigned(6)
   01                                   # unsigned(1)
   07                                   # unsigned(7)
   78 2C                                # text(44)
      39465065624B44475A4164637054375370664231556F7775716F6256385A7777395477504453797A584A4D72 # "9FPebKDGZAdcpT7SpfB1UowuqobV8Zww9TwPDSyzXJMr"
```

- UR encoding

```
ur:sign-request/osadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhoeadamaocfadykaxtantjooeadlocsdwykcfadykykaeykaeykaocyemrewytyaahdmtadaeadaxsptpfwoewnlbtspkrpaytodmonecolwlhdurzscxsgyninqdflrhbysschcfihgubsmdkocxprderdvorhgslfuttyrtmumkftioengogorlemwpkiuobychvacejpvtaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaebedthhsawnwfneenaajslrmtwdaeiojnimjpwpiypmastadsvlwpvlgwhfhecstdadaoaoaeadbnaoaeaeaeaevyykahaeaeaeaeahjnglflgmfphffecxgsgagygogafyamadatksdwesfggdihidgrfyflhtfpieiajoghemgujoiyfwehgojlktkpjsjlidhfethtktktesghktgdfygukkknhdgegtjpcmztvlnt
```

</details>

<details>

<summary>Example Solana signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'd4f0a7bcd95bba1fbb1051885054730e3f47064288575aacc102fbbf6a9a14daa066991e360d3e3406c20c00a40973eff37c7d641e5b351ec4a99bfe86f335f7', ; signature
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding 
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D # "\x9B\u001D\xEBM;}K\xAD\x9B\xDD+\r{=\xCBm"
   02                                   # unsigned(2)
   58 40                                # bytes(64)
      D4F0A7BCD95BBA1FBB1051885054730E3F47064288575AACC102FBBF6A9A14DAA066991E360D3E3406C20C00A40973EFF37C7D641E5B351EC4A99BFE86F335F7 # "\xD4\xF0\xA7\xBC\xD9[\xBA\u001F\xBB\u0010Q\x88PTs\u000E?G\u0006B\x88WZ\xAC\xC1\u0002\xFB\xBFj\x9A\u0014ڠf\x99\u001E6\r>4\u0006\xC2\f\u0000\xA4\ts\xEF\xF3|}d\u001E[5\u001Eĩ\x9B\xFE\x86\xF35\xF7"
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding 

```
ur:sign-response/otadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfztywtosrftahprdctrkbegylogdghjkbafhflamfwlohghtpsseaozorsimnybbtnnbiynlckenbtfmeeamsabnaeoxasjkwswfkekiieckhpecckssptndzelnwfecylaxjeglflgmfphffecxhtfegmgwknfsytca
```

</details>

## II - 4. Tezos transactions

[[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md) proposes the specific UR types `xtz-sign-request` and `xtz-signature` for signing Tezos transactions. In this document, we present their integration into the common UR types `sign-request` and `sign-response`.

### Integration with the generic UR types registry

- **Tezos sign request**

A Tezos transaction is uniquely identified thanks to `coin-identity` UR type, indicating which elliptic curve should be used (either secp256k1, ed25519 or P256) for the coin type 1729.

Several fields are required for requesting the signature of a Tezos transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Type of data to sign, if not specified, operation type is used by default
- Origin of the sign request

The following table indicates the corresponding fields between the UR types `xtz-sign-request` and `sign-request`.

| sign-request fields <br> Associated number corresponds to the order in UR type | Corresponding xtz-sign-request fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id specifying the elliptic curve for XTZ | 5. key-type   |
| 3. derivation-path | 4. derivation-path |
| 4. sign-data | 2. sign-data |
| 5. origin (optional) | No corresponding field, provides an additional description |
| 6. tx-type (optional) | 3. data-type |
| 7. address (optional) | No corresponding field, provides an additional verification on the sender with the derivation path |

`sign-request` UR type provides the exact same information as `xtz-sign-request` and has even the advantage to provide an additional optional field with a text describing the origin (e.g. the wallet name).

- **Tezos signature**

A Tezos sign request results in a signed transaction after user verification. 

The required fields for a Tezos signature are the signature itself and potentially the request identifier to identify uniquely the response. 

The following table indicates the corresponding fields between the UR types `xtz-signature` and `sign-response`.

| sign-response fields <br> Associated number corresponds to the order in UR type | Corresponding xtz-signature fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | No corresponding field, adds a description |

`sign-response` UR type provides the exact same information as `xtz-signature`, except regarding the optional `payload` field available in `xtz-signature`. This field is however a redundant information since the signature can be identified using the `request-id` field.

Additionally, `sign-response` has the advantage to provide an additional optional field to describe the origin with the device name for example.

### Example

An example illustrates how the signing protocol works on Tezos blockchain:

<details>

<summary>Example Tezos signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 6, ; ed25519 curve
     2: 1729 ; Tezos BIP44
    }),
3: 40304({1: [44, true, 1729, true, 0, true, 0, true, 0, true], ; #6.40304(keypath) origin m/44’/1729’/0’/0’/0'
        2: 934670036}), ; master-fingerprint
4: h'f849808609184e72a00082271094000000000000000000000000000000000000000080a47f7465737432000000000000000000000000000000000000000000000000000000600057808080', ; sign-data
5: "NGRAVE LIQUID", ; wallet name
6: 1, ; operation data-type
7: "tz1gLTu4Yxj8tPAcriQVUdxv6BY9QyvzU1az" ; address
}
```

- CBOR encoding 
    
```
A7                                      # map(7)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         06                             # unsigned(6)
         02                             # unsigned(2)
         19 06C1                        # unsigned(1729)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            19 06C1                     # unsigned(1729)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
         05                             # unsigned(5)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 4B                                # bytes(75)
      F849808609184E72A00082271094000000000000000000000000000000000000000080A47F7465737432000000000000000000000000000000000000000000000000000000600057808080 
   02                                   # unsigned(2)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
   06                                   # unsigned(6)
   01                                   # unsigned(1)
   07                                   # unsigned(7)
   78 24                                # text(36)
      747A31674C54753459786A38745041637269515655647876364259395179767A5531617A # "tz1gLTu4Yxj8tPAcriQVUdxv6BY9QyvzU1az"
```

- UR encoding
```
ur:sign-request/osadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhoeadamaocfamseaxtantjooeadlecsdwykcfamseykaeykaeykaeykaocyemrewytyaahdgryagalalnascsgljpnbaelfdibemwaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaelaoxlbjyihjkjyeyaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaehnaehglalalaahjnglflgmfphffecxgsgagygogafyamadatksdkjyknehiogsghkpeehkksimetjygdfpiajpingyhfgoiekskoenfwhkesgykkkokngoehhsknmyfnwmfw
```

</details>

<details>

<summary>Example Tezos signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09', ; signature
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   58 40                                # bytes(64)
      9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09 
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding

```
ur:sign-response/otadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfzneqzcnwybdcytetepmecnscpttvdmhfdksnscndebwiyfhtltpoycpeehdaydmpdfyykvskgylkiqdrhmspkgslrkbcnaakeaacxaxvlpraatoptplbacwynzcsgpeasaxjeglflgmfphffecxhtfegmgwlkdifdrn
```

</details>

## II - 5. Cosmos transactions

The [[ur-registry-cosmos]](https://github.com/KeystoneHQ/keystone-sdk-base/tree/master/packages/ur-registry-cosmos) defines the dedicated UR types `cosmos-sign-request` and `cosmos-signature` for handling Cosmos transaction signing. This section outlines their integration into the standardized blockchain-agnostic UR types `sign-request` and `sign-response`.

### Integration with the generic UR types registry

- **Cosmos sign request**

A Cosmos transaction is identified using the coin-identity UR type, typically referencing secp256k1 with coin type 118 or other Cosmos-specific values (e.g., 529 for Secret Network).

Required fields for a Cosmos signing request include:

- Unsigned transaction payload (`sign-data`)
- A derivation path and/or address (either a single entry or an array for account selection scenarios)

Recommended fields for better verification and traceability (especially from a watch-only wallet):

- A request ID for correlating responses
- The transaction type to specify the transaction format (defaults to Amino)
- The origin to identify the wallet initiating the request

The table below maps fields between `sign-request` and `cosmos-sign-request`:

| sign-request fields <br> Associated number corresponds to the order in UR type | Corresponding cosmos-sign-request fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. coin-id | No direct equivalent (identifies Cosmos) |
| 3. derivation-path | 4. derivation-paths |
| 4. sign-data | 2. sign-data |
| 5. origin (optional) | 6. origin |
| 6. tx-type (optional) | 3. data-type |
| 7. address (optional) | 5. addresses |

The `sign-request` UR type fully encapsulates the same data as `cosmos-sign-request`, while maintaining a blockchain-agnostic structure.

- **Cosmos signature**

A successful Cosmos sign request results in a response containing:

- The generated signature
- The public key of the account used for signing
- Optionally, the request-id to correlate with the original request

Field mapping between `sign-response` and `cosmos-signature`:

| sign-response fields <br> Associated number corresponds to the order in UR type | Corresponding cosmos-signature fields <br> Associated number corresponds to the order in UR type |
| --- | --- |
| 1. request-id (optional) | 1. request-id (optional) |
| 2. signature | 2. signature |
| 3. origin (optional) | Not present |
| 4. public-key | 3. public-key |

The `sign-response` UR type provides equivalent signing output as `cosmos-signature`, with the added benefit of the optional `origin` field to include signer metadata (e.g., device name).

### Example

An example illustrates how the signing protocol works on Cosmos blockchain:

<details>

<summary>Example Cosmos signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 8, ; secp256k1 curve
     2: 118 ; Cosmos BIP44
    }),
3: [40304({1: [44, true, 118, true, 0, true, 0, false, 0, false], ; #6.40304(keypath) n°1 m/44’/118'/0’/0/0
        2: 78230804}), ; master-fingerprint
    40304({1: [44, true, 529, true, 0, true, 0, false, 0, false], ; #6.40304(keypath) n°2 m/44’/529'/0’/0/0
        2: 78230805})], ; master-fingerprint
4: h'8e53e7b10656816de70824e3016fc1a277e77825e12825dc4f239f418ab2e04e', ; sign-data
5: "NGRAVE LIQUID", ; wallet name
6: 1, ; Amino transaction
7: ["cosmos13nmjt4hru5ag0c6q3msk0srs55qd3dtme8wgep", "secret1e63tmjmhsac8ja86gk8kh06uqxh2h9hwkcs5cs"] ; addresses
}
```

- CBOR encoding 
    
```
A7                                      # map(7)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D # "\x9B\u001D\xEBM;}K\xAD\x9B\xDD+\r{=\xCBm"
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         08                             # unsigned(8)
         02                             # unsigned(2)
         18 76                          # unsigned(118)
   03                                   # unsigned(3)
   82                                   # array(2)
      D9 9D70                           # tag(40304)
         A2                             # map(2)
            01                          # unsigned(1)
            8A                          # array(10)
               18 2C                    # unsigned(44)
               F5                       # primitive(21)
               18 76                    # unsigned(118)
               F5                       # primitive(21)
               00                       # unsigned(0)
               F5                       # primitive(21)
               00                       # unsigned(0)
               F4                       # primitive(20)
               00                       # unsigned(0)
               F4                       # primitive(20)
            02                          # unsigned(2)
            1A 04A9B514                 # unsigned(78230804)
      D9 9D70                           # tag(40304)
         A2                             # map(2)
            01                          # unsigned(1)
            8A                          # array(10)
               18 2C                    # unsigned(44)
               F5                       # primitive(21)
               19 0211                  # unsigned(529)
               F5                       # primitive(21)
               00                       # unsigned(0)
               F5                       # primitive(21)
               00                       # unsigned(0)
               F4                       # primitive(20)
               00                       # unsigned(0)
               F4                       # primitive(20)
            02                          # unsigned(2)
            1A 04A9B515                 # unsigned(78230805)
   04                                   # unsigned(4)
   58 20                                # bytes(32)
      8E53E7B10656816DE70824E3016FC1A277E77825E12825DC4F239F418AB2E04E # "\x8ES\xE7\xB1\u0006V\x81m\xE7\b$\xE3\u0001o\xC1\xA2w\xE7x%\xE1(%\xDCO#\x9FA\x8A\xB2\xE0N"
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
   06                                   # unsigned(6)
   01                                   # unsigned(1)
   07                                   # unsigned(7)
   82                                   # array(2)
      78 2D                             # text(45)
         636F736D6F7331336E6D6A743468727535616730633671336D736B30737273353571643364746D653877676570 # "cosmos13nmjt4hru5ag0c6q3msk0srs55qd3dtme8wgep"
      78 2D                             # text(45)
         73656372657431653633746D6A6D68736163386A613836676B386B6830367571786832683968776B6373356373 # "secret1e63tmjmhsac8ja86gk8kh06uqxh2h9hwkcs5cs"
```

- UR encoding
```
ur:sign-request/osadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhoeadayaocskoaxlftantjooeadlecsdwykcskoykaeykaewkaewkaocyaaptrebbtantjooeadlecsdwykcfaobyykaeykaewkaewkaocyaaptrebzaahdcxmnguvdpaamhflyjnvdaydkvladjlseoektvdksdavydedauogwcnnefpleprvtglahjnglflgmfphffecxgsgagygogafyamadatlfksdpiajljkjnjljkeheojtjnimjyeeisjpkpechsiodyiaenjseojnjkjedyjkjpjkececjsieeoiejyjnihetktioihjoksdpjkihiajpihjyehiheneojyjnimjnisjkhsiaetimhseteniojeetjeisdyenkpjsksiseyisesisktjeiajkeciajkzeytjzjo
```

</details>

<details>

<summary>Example Cosmos signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'47e7b510784406dfa14d9fd13c3834128b49c56ddfc28edb02c5047219779adeed12017e2f9f116e83762e86f805c7311ea88fb403ff21900e069142b1fb310e', ; signature
3: "NGRAVE ZERO", ; device name
4: h'8e53e7b10656816de70824e3016fc1a277e77825e12825dc4f239f418ab2e04e' ; public key
}
```

- CBOR encoding
    
```
A4                                      # map(4)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D # "\x9B\u001D\xEBM;}K\xAD\x9B\xDD+\r{=\xCBm"
   02                                   # unsigned(2)
   58 40                                # bytes(64)
      47E7B510784406DFA14D9FD13C3834128B49C56DDFC28EDB02C5047219779ADEED12017E2F9F116E83762E86F805C7311EA88FB403FF21900E069142B1FB310E # "G\xE7\xB5\u0010xD\u0006ߡM\x9F\xD1<84\u0012\x8BI\xC5m\xDF\u008E\xDB\u0002\xC5\u0004r\u0019w\x9A\xDE\xED\u0012\u0001~/\x9F\u0011n\x83v.\x86\xF8\u0005\xC71\u001E\xA8\x8F\xB4\u0003\xFF!\x90\u000E\u0006\x91B\xB1\xFB1\u000E"
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
   04                                   # unsigned(4)
   58 20                                # bytes(32)
      8E53E7B10656816DE70824E3016FC1A277E77825E12825DC4F239F418AB2E04E # "\x8ES\xE7\xB1\u0006V\x81m\xE7\b$\xE3\u0001o\xC1\xA2w\xE7x%\xE1(%\xDCO#\x9FA\x8A\xB2\xE0N"
```

- UR encoding

```
ur:sign-response/oxadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfzflvdrebeksfyamuroygtnettfneteebglugaskjnursamnuyaoskaajpcfktnyuewebgadkbdlnebyjtlskodmlnyaahstehckpdmyqzaxzmclmhbaammefwpazoehbaaxjeglflgmfphffecxhtfegmgwaahdcxmnguvdpaamhflyjnvdaydkvladjlseoektvdksdavydedauogwcnnefpleprvtglctldimhe
```

</details>

## II - 5. Other blockchains transactions

There are no existing signing protocol based on UR types for other blockchains (at the time of writing). This document introduces the signing of MultiversX and Stellar transactions as examples for the blockchain-agnostic `sign-request` and `sign-response` UR types introduced.

### Integration with the generic UR types registry

- **MultiversX and Stellar sign request**

A MultiversX transaction is uniquely identified thanks to `coin-identity` UR type, i.e. ed25519 as elliptic curve and 508 as coin type. Identically, a Stellar transaction is identified with ed25519 as an elliptic curve and 148 as a coin type.

Several fields are required for requesting the signature of a MultiversX or Stellar transaction:

- Unsigned transaction
- Derivation path of the private key to sign the data

Additionally, the following fields are recommended to be supplied by the watch-only wallet for verification purposes:

- Unique request identifier
- Origin of the sign request

The listed information for requesting a MultiversX or Stellar signature can be included in the `sign-request` UR type. 

- **MultiversX and Stellar signature**

A MultiversX or Stellar sign request results in a signed transaction after user verification. 

The required fields for a MultiversX or Stellar signature are the signature itself and potentially the request identifier to identify uniquely the response. Additionally, a description can be provided to specify the device name for example.

This listed information for sending a MultiversX and Stellar signature can be included in `sign-response` UR type.

### Examples

An example illustrates how the signing protocol works on MultiversX blockchain:

<details>

<summary>Example MultiversX signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 6, ; ed25519 curve
     2: 508 ; MultiversX BIP44
    }),
3: 40304({1: [44, true, 508, true, 0, true, 0, true, 1, true], ; #6.40304(keypath) m/44’/501’/0’/0’/1'
          2: 934670036}), ; master-fingerprint
4: h'f849808609184e72a00082271094000000000000000000000000000000000000000080a47f7465737432000000000000000000000000000000000000000000000000000000600057808080', ; sign-data
5: "NGRAVE LIQUID" ; wallet name
}
```

- CBOR encoding
    
```
A5                                      # map(5)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         06                             # unsigned(6)
         02                             # unsigned(2)
         19 01FC                        # unsigned(508)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            19 01FC                     # unsigned(508)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            01                          # unsigned(1)
            F5                          # primitive(21)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 4B                                # bytes(75)
      F849808609184E72A00082271094000000000000000000000000000000000000000080A47F7465737432000000000000000000000000000000000000000000000000000000600057808080
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
```

- UR encoding 
```
ur:sign-request/onadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhoeadamaocfadztaxtantjooeadlecsdwykcfadztykaeykaeykadykaocyemrewytyaahdgryagalalnascsgljpnbaelfdibemwaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaelaoxlbjyihjkjyeyaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaeaehnaehglalalaahjnglflgmfphffecxgsgagygogafypynsuyny
```

</details>

<details>

<summary>Example MultiversX signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09', ; signature
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   58 40                                # bytes(64)
      9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09 
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding

```
ur:sign-response/otadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfzneqzcnwybdcytetepmecnscpttvdmhfdksnscndebwiyfhtltpoycpeehdaydmpdfyykvskgylkiqdrhmspkgslrkbcnaakeaacxaxvlpraatoptplbacwynzcsgpeasaxjeglflgmfphffecxhtfegmgwlkdifdrn
```

</details>

Another example illustrates how the signing protocol works on Stellar blockchain:

<details>

<summary>Example Stellar signature request</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: 41401( ; #6.41401(coin-identity)
    {1: 6, ; ed25519 curve
     2: 148 ; Stellar BIP44
    }),
3: 40304({1: [44, true, 148, true, 0, true, 0, true, 2, true], ; #6.40304(keypath) m/44’/148/0’/0’/2'
        2: 934670036}), ; master-fingerprint
4: h'00000002000000002df26f5fc2916d823126414b0cde52203a4f54222e1f3c82f2c82bf7c4e2d76d000000640011b3dc0000000100000001000000000000006400000000646e9655000000010000000c48656c6c6f20576f726c642100000001000000000000000100000000321911377e1664d677a85ab30acd1262522f989f0f31da613219e8396278cdb90000000000000002540be4000000000000000000', ; sign-data
5: "NGRAVE LIQUID" ; wallet name
}
```

- CBOR encoding 
    
```
A5                                      # map(5)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D # "\x9B\u001D\xEBM;}K\xAD\x9B\xDD+\r{=\xCBm"
   02                                   # unsigned(2)
   D9 A1B9                              # tag(41401)
      A2                                # map(2)
         01                             # unsigned(1)
         06                             # unsigned(6)
         02                             # unsigned(2)
         18 94                          # unsigned(148)
   03                                   # unsigned(3)
   D9 9D70                              # tag(40304)
      A2                                # map(2)
         01                             # unsigned(1)
         8A                             # array(10)
            18 2C                       # unsigned(44)
            F5                          # primitive(21)
            18 94                       # unsigned(148)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            00                          # unsigned(0)
            F5                          # primitive(21)
            02                          # unsigned(2)
            F5                          # primitive(21)
         02                             # unsigned(2)
         1A 37B5EED4                    # unsigned(934670036)
   04                                   # unsigned(4)
   58 A0                                # bytes(160)
      00000002000000002DF26F5FC2916D823126414B0CDE52203A4F54222E1F3C82F2C82BF7C4E2D76D000000640011B3DC0000000100000001000000000000006400000000646E9655000000010000000C48656C6C6F20576F726C642100000001000000000000000100000000321911377E1664D677A85AB30ACD1262522F989F0F31DA613219E8396278CDB90000000000000002540BE4000000000000000000
   05                                   # unsigned(5)
   6D                                   # text(13)
      4E4752415645204C4951554944        # "NGRAVE LIQUID"
```

- UR encoding

```
ur:sign-request/onadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaotaoyrhoeadamaocsmwaxtantjooeadlecsdwykcsmwykaeykaeykaoykaocyemrewytyaahdnbaeaeaeaoaeaeaeaedpwzjlhesamejnlfehdsfpgrbnuegmcxftgwghcpdmctfnlfwzspdnylssvotsjnaeaeaeieaebyqduoaeaeaeadaeaeaeadaeaeaeaeaeaeaeieaeaeaeaeiejtmtgoaeaeaeadaeaeaebnfdihjzjzjlcxhgjljpjzieclaeaeaeadaeaeaeaeaeaeaeadaeaeaeaeeycfbyemkbcmietbktpdhtqdbksnbgidgmdlmknebsehtnhseycfvsesidkssnrhaeaeaeaeaeaeaeaoghbdveaeaeaeaeaeaeaeaeaeahjnglflgmfphffecxgsgagygogafyeedshgwd
```

</details>

<details>

<summary>Example Stellar signature response</summary>

- CBOR diagnosis format:
    
```
{
1: 37(h'9b1deb4d3b7d4bad9bdd2b0d7b3dcb6d'), ; request-id
2: h'9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09', ; signature
3: "NGRAVE ZERO" ; device name
}
```

- CBOR encoding
    
```
A3                                      # map(3)
   01                                   # unsigned(1)
   D8 25                                # tag(37)
      50                                # bytes(16)
         9B1DEB4D3B7D4BAD9BDD2B0D7B3DCB6D 
   02                                   # unsigned(2)
   58 40                                # bytes(64)
      9FB423EE0B1AD3D3AD359C22D1E79048789C232813663FD5D8A1223458082EA844F5E87BF77DB3B997AA4C847E23047C042003E3B204CEA9AE0E1BF6FDCAAF09 
   03                                   # unsigned(3)
   6B                                   # text(11)
      4E4752415645205A45524F            # "NGRAVE ZERO"
```

- UR encoding

```
ur:sign-response/otadtpdagdndcawmgtfrkigrpmndutdnbtkgfssbjnaohdfzneqzcnwybdcytetepmecnscpttvdmhfdksnscndebwiyfhtltpoycpeehdaydmpdfyykvskgylkiqdrhmspkgslrkbcnaakeaacxaxvlpraatoptplbacwynzcsgpeasaxjeglflgmfphffecxhtfegmgwlkdifdrn
```

</details>

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

The signing communication protocol is fully compatible with existing proposals and implementations based on UR types for several blockchains (e.g. Bitcoin with [[BCR-2021-001]](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2021-001-request.md), Ethereum with [[EIP-4527]](https://eips.ethereum.org/EIPS/eip-4527), Solana with [[solana-qr-data-protocol]](https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer) and Tezos with [[TZIP-25]](https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md)). For the blockchains without a reference implementation (examples with MultiversX and Stellar), this document proposes new UR types as a communication layer via QR codes between a watch-only wallet and an offline signer to sign transactions. 

The existing QR protocol between the NGRAVE watch-only wallet, LIQUID, and the NGRAVE offline signer, ZERO, presents major changes with the proposed communication. These changes are however needed for better standardization between the QR-based hardware wallet and to enhance overall security with a common definition on the transaction verification. 

## Security consideration

The protocol must uphold the What You See Is What You Sign (WYSIWYS) principle: the content displayed for user confirmation on the offline signer must exactly match the data being signed. This ensures that no tampering or data alteration occurs during the signing process.

The user must explicitly review and approve the transaction details on the offline signer. These details must be derived and displayed according to blockchain-specific verification rules. Any discrepancy between the transaction shown on the offline signer and the one prepared by the watch-only wallet should trigger a warning, allowing the user to reject potentially malicious or altered requests.

To support this:

   - The offline signer must present decoded transaction data in a clear, human-readable format.
   - If the transaction cannot be decoded (e.g., due to an unknown or unsupported type), the device must clearly alert the user with a warning, indicating that the transaction could not be safely verified.

---

# V - References

| Reference | Link |
| --- | --- |
| [BCR-2020-005] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md |
| [EIP-4527] | https://eips.ethereum.org/EIPS/eip-4527 |
| [TZIP-25] | https://gitlab.com/tezos/tzip/-/blob/master/proposals/tzip-25/tzip-25.md |
| [solana-qr-data-protocol] | https://github.com/KeystoneHQ/Keystone-developer-hub/blob/main/research/solana-qr-data-protocol.md#sending-the-unsigned-data-from-wallet-only-wallet-to-offline-signer |
| [BCR-2020-006] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-006-urtypes.md |
| [BIP-174] | https://github.com/bitcoin/bips/blob/master/bip-0174.mediawiki |
| [BIP-322] | https://github.com/bitcoin/bips/blob/master/bip-0322.mediawiki |
| [CBOR Tag] | https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml |
| [keytool-cli] | https://github.com/BlockchainCommons/keytool-cli/tree/master |
| [ur-registry-eth] | https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-eth |
| [ur-registry-sol] | https://github.com/KeystoneHQ/keystone-airgaped-base/tree/master/packages/ur-registry-sol |
| [ur-registry-xtz] | https://socket.dev/npm/package/@airgap/ur-registry-xtz/overview/0.0.1-beta.0 |
| [ur-registry-cosmos] | https://github.com/KeystoneHQ/keystone-sdk-base/tree/master/packages/ur-registry-cosmos |
| [ur-registry] | https://github.com/ngraveio/ur-registry/tree/main |
| [BCR-2020-007] | https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-007-hdkey.md |
| [NBCR-2023-002] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md |
| [NBCR-2023-001] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md |
