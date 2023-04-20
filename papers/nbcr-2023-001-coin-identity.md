# UR Type Definition for Coin Identitiy
## NBCR-2023-001

**© 2023 Ngrave**

Authors: İrfan Bilaloğlu, Mathieu Da Silva<br/>
Date: 19 April 2023<br/>

---

### Introduction

In this document, we are defining the new UR type `crypto-coin-identity` based on new types and [IANA-defined](https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves) constants.

By uniquely identifying a coin, the watch-only wallet is able to request the coin price and its logo, as well as to identify the protocol to derive addresses and to prepare transactions. Currently, many QR-based offline signers are based the coin identification on their homemade enumeration. 

We propose to define a UR type standardizing the coin identification by aggregating the following information:

1. Curve of the coin (e.g. *`["secp256k1", "ed25519", "p256 (secp256r1)”, “X25519 (sr25519)”]` ).* This information is mandatory in the case of some blockchain (e.g. Tezos) supporting multiple elliptic curves.
2. BIP44 coin type as defined in [[SLIP44]](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).
3. Subtype to define additional information to identify the coin (e.g. the chain ID for an EVM chain).
4. Contract identifier as subtypes (e.g. ERC20 token address).



### EC Curve Definitions

The required `curve` field carries the elliptic curve information of the coin.

This table taken from [IANA COSE Elliptic Curves](https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves).
Should be updated as IANA updates the table.

The **value** field is used CBOR encoding and the **Name reference** field is used with lowercase and without special characters in URI encoding. 

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


### Type Data

The required `type` field carries the coin type information as stated in [BIP44 Document](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki) data.

Data is taken from [SLIP44](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)

### CDDL

The following specification of `crypto-coin-identity` is written in CDDL. When used embedded in another CBOR structure, this structure should be tagged #6.1401.

```
; Table should always be updated according to IANA registry 
; https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves
; https://www.rfc-editor.org/rfc/rfc9053.html#name-elliptic-curve-keys

P256=1	            ; NIST P-256 also known as secp256r1
P384=2	            ; NIST P-384 also known as secp384r1	
P521=3	            ; EC2	NIST P-521 also known as secp521r1		
X25519=4            ; X25519 for use w/ ECDH only		
X448=5              ; X448 for use w/ ECDH only		
Ed25519=6           ; Ed25519 for use w/ EdDSA only		
Ed448=7             ; Ed448 for use w/ EdDSA only		
secp256k1=8         ; SECG secp256k1 curve	IESG	

elliptic_curve = P256 / P384 / P521 / X25519 / X448 / Ed25519 / Ed448 / secp256k1

; Subtypes specific to some coins (e.g. ChainId for EVM chains)
sub_type_exp = uint32 / str

coin-identity = {
    curve: elliptic_curve,
    type: uint31, ; values from [SLIP44] with high bit turned off,
    ? subtype: [ sub_type_exp + ]  ; Compatible with the definition of several subtypes if necessary
}

curve = 1
type = 2
subtype = 3
```

### URI Format

The UR type is designed to be easily convertible to a URI format when human readability or deep linking are required.

The URI format is as follows: **`bc-coin://{tokenId@subtype1.subtype0}.{curve}/type?optional=parameters`**.


We are providing several examples in the following table:

| Coin | URI coin identity |
| --- | --- |
| Bitcoin (BTC) | bc-coin://secp256k1/0 |
| Ethereum (ETH) | bc-coin://secp256k1/60 |
| Polygon (MATIC) | bc-coin://137.secp256k1/60 |
| Solana (SOL) | bc-coin://ed25519/508 |
| Tezos (XTZ) based on ed25519 | bc-coin://ed25519/1729 |
| Tezos (XTZ) based on secp256k1 | bc-coin://secp256k1/1729 |
| Neo | bc-coin://p256/888 |
| Polkadot | bc-coin://x25519/354 |



#### Tokens

Tokens under the respective coin can be distinguished by the `@` seperator. Anything under the left of `@` is considered the contract identifier. 
For EVM based coin and ERC20 tokens, the contract identifier is the token address. For Elrond its tokenID.
For ERC721 tokens, the contract identifier is contract address and the token ID. Token ID will be the sub type of contract in this case.

**Examples:**

| Token | Coin | URI coin identity |
| --- | --- | --- |
| USDT |ETH | bc-coin://0xdAC17F958D2ee523a2206206994597C13D831ec7@secp256k1/60|
| USDT |Polygon (MATIC) | bc-coin://0xc2132D05D31c914a87C6611C10748AEb04B58e8F@137.secp256k1/60 |
| [NFT](https://etherscan.io/nft/0x495f947276749ce646f68ac8c248420045cb7b5e/30215980622330187411918288900688501299580125367569939549692495859506871271425) | ETH | bc-coin://30215980622330187411918288900688501299580125367569939549692495859506871271425.0x495f947276749Ce646f68AC8c248420045cb7b5e@secp256k1/60 |
| USDC | MultiversX (EGLD) | bc-coin://USDC-c76f1f@ed25519/508|
| USDC | SOL | bc-coin://EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v@ed25519/501|




















### Example/Test Vector 1

* An EC private key:

```
$ seedtool --count 32
8c05c4b4f3e88840a4f4b5f155cfd69473ea169f3d0431b7a6787a23777f08aa
```

* In the CBOR diagnostic notation:

```
{
	; `curve` is implied to be 0 (secp256k1)
	2: true, ; is-private
	3: h'8c05c4b4f3e88840a4f4b5f155cfd69473ea169f3d0431b7a6787a23777f08aa' ; data
}
```

* Encoded as binary using [CBOR-PLAYGROUND]:

```
A2                                      # map(2)
   02                                   # unsigned(2) is-private
   F5                                   # primitive(21) true
   03                                   # unsigned(3) data
   58 20                                # bytes(32)
      8C05C4B4F3E88840A4F4B5F155CFD69473EA169F3D0431B7A6787A23777F08AA
```

* As a hex string:

```
A202F50358208C05C4B4F3E88840A4F4B5F155CFD69473EA169F3D0431B7A6787A23777F08AA
```

* As a UR:

```
ur:crypto-eckey/oeaoykaxhdcxlkahssqzwfvslofzoxwkrewngotktbmwjkwdcmnefsaaehrlolkskncnktlbaypkrphsmyid
```

* UR as QR Code:

![](bcr-2020-008/1.png)

### Example/Test Vector 2

* Convert the private key above into a public key:

```
$ bx ec-to-public
8c05c4b4f3e88840a4f4b5f155cfd69473ea169f3d0431b7a6787a23777f08aa
^D
03bec5163df25d8703150c3a1804eac7d615bb212b7cc9d7ff937aa8bd1c494b7f
```

* In the CBOR diagnostic notation:

```
{
	3: h'03bec5163df25d8703150c3a1804eac7d615bb212b7cc9d7ff937aa8bd1c494b7f' ; data
}
```

* Encoded as binary using [CBOR-PLAYGROUND]:

```
A1                                      # map(1)
   03                                   # unsigned(3) data
   58 21                                # bytes(33)
      03BEC5163DF25D8703150C3A1804EAC7D615BB212B7CC9D7FF937AA8BD1C494B7F
```

* As a hex string:

```
A103582103BEC5163DF25D8703150C3A1804EAC7D615BB212B7CC9D7FF937AA8BD1C494B7F
```

* As a UR:

```
ur:crypto-eckey/oyaxhdclaxrnskcmfswzhlltaxbzbnftcsaawdsttbbzrkcldnkesotszmmuknpdrycegagrlbemdevtlp
```

* UR as QR Code:

![](bcr-2020-008/2.png)






### References

| Title | Link |
| --- | --- |
| [BIP44] | https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki |
| [SLIP44]  | https://github.com/satoshilabs/slips/blob/master/slip-0044.md |
| COSE| https://www.rfc-editor.org/rfc/rfc9053.html |
| IANA COSE Elliptic Curves | https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves |