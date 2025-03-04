# Unique Asset ID (UAI) format
## NBCR-2024-001

**© 2024 Ngrave**

Authors: İrfan Bilaloğlu, Mathieu Da Silva, Maher Sallam<br/>
Date: 18 September 2024<br/>

Revised: October 04, 2024

---

### Introduction

By uniquely identifying an asset, the watch-only wallets and the offline signers are able to request the asset price and its logo, as well as to identify the protocol to derive addresses and to prepare transactions. Currently, many wallets are based on their homemade enumeration. 

We propose a new URI format, named *Unique Asset ID (UAI)*, to standardize the asset identification. 

### UAI format 

The UAI format is based on the following URL:

 **`uai://curve.type[.subtype1.subtype2]:tokenid[.subtype1.subtype2]/derivation_path?master_fingerprint=uint32`**

This format uses the reverse DNS notation because it groups closely related coins when sorting them lexically.

Using a URL format presents several benefits. 
1. URL parsers exist in the standard libraries of many languages. 
2. The format is human readable.

### UAI composition

An asset is identified by two ***required*** fields in the UAI format:

1. **`curve`** field carries the elliptic curve information of the coin, based on the **Name reference** in lowercase defined in the table from [[IANA COSE Elliptic Curves]](https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves). 

| Name                     | Value                           | Key Type | Description                        | Change Controller | Reference | Recommended |
|--------------------------|---------------------------------|----------|------------------------------------|-------------------|-----------|-------------|
| P-256                    | 1                               | EC2      | NIST P-256 also known as secp256r1 |                   | [RFC9053] | Yes         |
| P-384                    | 2                               | EC2      | NIST P-384 also known as secp384r1 |                   | [RFC9053] | Yes         |
| P-521                    | 3                               | EC2      | NIST P-521 also known as secp521r1 |                   | [RFC9053] | Yes         |
| X25519                   | 4                               | OKP      | X25519 for use w/ ECDH only        |                   | [RFC9053] | Yes         |
| X448                     | 5                               | OKP      | X448 for use w/ ECDH only          |                   | [RFC9053] | Yes         |
| Ed25519                  | 6                               | OKP      | Ed25519 for use w/ EdDSA only      |                   | [RFC9053] | Yes         |
| Ed448                    | 7                               | OKP      | Ed448 for use w/ EdDSA only        |                   | [RFC9053] | Yes         |
| secp256k1                | 8                               | EC2      | SECG secp256k1 curve               | IESG              | [RFC8812] | No          |

2. **`type`** field carries the coin type information as defined in [[SLIP44]](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) with the high bit turned off.    
In case additional information is required to identify the asset, the sub-type **`type[.subtype1.subtype2]`** fields must  be defined. For example, every EVM chain requires the additional chain ID as an identifier.

Extra **optional** information can be provided in the UAI format in order to link an asset to a wallet:

3. **`tokenid`** field carries the token information. For example, ERC20 tokens are identified through the token address and MultiversX tokens using their token identifier.   
For ERC721 and ERC1155 tokens, the token identifier is the combination of a contract address and a token ID. In this case, the sub-type **`tokenid[.subtype1.subtype2]`** fields will be used to specify the token ID associated to the contract address. 

5. **`derivation_path`** field carries the derivation path to indicate the account related to the asset.     
The derivation path is specified by `purpose/coin_type/account/change/address_index` format, without the `m/` prefix used in regular notation. Besides, the hardened paths in the derivation paths will be indicated using the letter `h`.    
The derivation path can be defined with a range of index in case of specifying specific account associated to the asset, e.g. `44h/60h/0h/0/0/[0-2]` to sync the first 3 ETH accounts.

6. **`master_fingerprint`** field provides the fingerprint for the master public key as per BIP32 and defined as `master_fingerprint=uint32` in the URI format where `uint32` is the master fingerprint value in the 32-bit unsigned integer format.

### Examples

The following table provides several examples to identify a coin with only the required information:

| Asset | UAI |
| --- | --- |
| Bitcoin (BTC) | uai://secp256k1.0 |
| Ethereum (ETH) | uai://secp256k1.60.1 |
| Polygon (POL) | uai://secp256k1.60.137 |
| Solana (SOL) | uai://ed25519.501 |
| Tezos (XTZ) based on ed25519 | uai://ed25519.1729 |
| Tezos (XTZ) based on secp256k1 | uai://secp256k1.1729 |
| Neo | uai://p256.888 |
| Polkadot | uai://x25519.354 |

The following table provides additional examples to identify some specific assets on a blockchain using the optional fields:

| Asset | UAI |
| ---------------- | --- |
| Bitcoin (BTC) Taproot accounts | uai://secp256k1.0/86h/0h/0h |
| USDC on Polygon | uai://secp256k1.60.137:0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359 |
| USDC on Solana | uai://ed25519.501:EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v |
| USDC on MultiversX | uai://ed25519.508:USDC-c76f1f |
| [NFT](https://etherscan.io/nft/0x495f947276749ce646f68ac8c248420045cb7b5e/30215980622330187411918288900688501299580125367569939549692495859506871271425) on Ethereum | uai://secp256k1.60.1:0x495f947276749Ce646f68AC8c248420045cb7b5e.30215980622330187411918288900688501299580125367569939549692495859506871271425 |
| USDC on Ethereum account 2 | uai://secp256k1.60.1:0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48/44h/60h/0h/0/0/1?master_fingerprint=934670036 |
| Ethereum accounts 1 to 3 | uai://secp256k1.60.1/44h/60h/0h/0/0/[0,2]?master_fingerprint=934670036 |


### UAI format/UR types conversion

The UAI format and the UR types defined in [[NBCR-2023-001]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md) and [[NBCR-2023-002]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md) share the same information. While the UAI format is human readable and designed to be used as asset identifier in a database and in communication between different wallets or between a website and a mobile app to link unique crypto assets, the UR types are a compact format designed for efficient communication between an offline signer and a watch-only wallet.

The UR types and the UAI format can easily be converted to each other:

- `coin-identity` UR type defined in [[NBCR-2023-001]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md) contains the same mandatory UAI fields, i.e. the curve, the type and the subtypes. 

- `detailed-account` UR type defined in [[NBCR-2023-002]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md) contains the same optional UAI fields, i.e. the token ID and the derivation path.

We provide an example how UR types and UAI format can be converted to each other:

- USDC on Polygon account `44h/60h/0h/1/0`:

```
uai://secp256k1.60.137:0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359/44h/60h/0h/1/0?master_fingerprint=2819587291
     |_______________| |________________________________________________________|
              |                                     |
      coin-identity                       detailed-account

CBOR diagnostic notation          
      {                                   {
         1: 8, ; curve-secp256k1            1: [40303({  ; #6.303(hdkey)
         2: 60, ; Ethereum SLIP44                    6: 304({1: [44, true, 60, true, 0, true]}), ; origin m/44'/60'/0' 
         3: [137] ; Polygon chain ID                 7: 304({1: [1, false, 0, false]}) ; children m/44'/60'/0'/1/0
      }                                             })],
                                            2: [ h'0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359' ]  ; USDC ERC20 token on Polygon 
                                           }

    |_________________________________________________________________________________________________________|
                                                        |
                                                portfolio-coin
                                                { 
                                                    1: coin-identity
                                                    2: detailed-account
                                                    3: 2819587291 ; master-fingerprint
                                                }
```


### References

| Title | Link |
| --- | --- |
| [SLIP44]  | https://github.com/satoshilabs/slips/blob/master/slip-0044.md |
| [IANA COSE Elliptic Curves] | https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves|
| [NBCR-2023-001] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-001-coin-identity.md |
| [NBCR-2023-002] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-002-multi-layer-sync.md |
