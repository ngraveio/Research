# UR Type Definition for Coin Identity
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
4. Contract identifier as subtypes (e.g. ERC20 token address, ERC721 NFTs).



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
hex_string = #6.263(bstr) ; byte string is a hexadecimal string no need for decoding
sub_type_exp = uint32 / str / hex_string

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

* Solana coin 
* URI: `bc-coin://ed25519/501`

* In the CBOR diagnostic notation:

```
{
   1: 6, ; curve-ed25519
   2: 501, ; type Solana BIP44
}
```


* Encoded as binary using [CBOR-PLAYGROUND]:

```
A2         # map(2)
   01      # unsigned(1) curve
   06      # unsigned(6) ed25519
   02      # unsigned(2) type
   19 01F5 # unsigned(501) Solana
```

* As a hex string:

```
A20106021901F5 

```

* As a UR:

```
ur:crypto-eckey/oeaoykaxhdcxlkahssqzwfvslofzoxwkrewngotktbmwjkwdcmnefsaaehrlolkskncnktlbaypkrphsmyid
```

* UR as QR Code:

![](bcr-2020-008/1.png)

### Example/Test Vector 2

- Polygon coin, ETH with a different chain ID.
- URI: `bc-coin://137.secp256k1/60`

* In the CBOR diagnostic notation:

```
{
   1: 8, ; curve-secp256k1
   2: 60, ; type ETH BIP44
   3: [137], ; sub type Polygon
}
```

* Encoded as binary using [CBOR-PLAYGROUND]:

```
A3          # map(3)
   01       # unsigned(1) curve
   08       # unsigned(1) secp256k1
   02       # unsigned(2) type
   18 3C    # unsigned(60) ETH
   03       # unsigned(3) sub type
   81       # array(1) 
      18 89 # unsigned(137) Polygon Chain Id
```

* As a hex string:

```
A3010802183C03811889

```

# Example/Test Vector 3
* USDT Token on Polygon (MATIC)
* URI: `bc-coin://0xc2132D05D31c914a87C6611C10748AEb04B58e8F@137.secp256k1/60`


* In the CBOR diagnostic notation:

```
{
   1: 8, ; curve-secp256k1
   2: 60, ; type ETH BIP44
   3: [137, "@", 263(h'c2132D05D31c914a87C6611C10748AEb04B58e8F')], ; subtype Polygon + contract address
}
```

* Encoded as binary using [CBOR-PLAYGROUND]:

```
A3              # map(3)
   01           # unsigned(1) curve
   08           # unsigned(1) secp256k1
   02           # unsigned(2) type
   18 3C        # unsigned(60) ETH
   03           # unsigned(3) sub type
   83           # array(3) 
      18 89     # unsigned(137) Polygon Chain Id
      61        # text(1)
         40     # "@"
      D9 0107   # tag(263)
         54     # bytes(20)
            C2132D05D31C914A87C6611C10748AEB04B58E8F # contract address
```

* As a hex string:

```
A3010802183C038318896140D9010754C2132D05D31C914A87C6611C10748AEB04B58E8F
```

### Example/Test Vector 4

* NFT on ETH
* URI: `bc-coin://30215980622330187411918288900688501299580125367569939549692495859506871271425.0x495f947276749Ce646f68AC8c248420045cb7b5e@secp256k1/60`

* In the CBOR diagnostic notation:

```
{
   1: 8, ; curve-secp256k1
   2: 60, ; type ETH BIP44
   3: ["@", 263(h'495f947276749Ce646f68AC8c248420045cb7b5e)', 30215980622330187411918288900688501299580125367569939549692495859506871271425], ; subtype contract address + token id
}
```

* Encoded as binary using [CBOR-PLAYGROUND]:

```
A3                                      # map(3)
   01                                   # unsigned(1)
   08                                   # unsigned(8)
   02                                   # unsigned(2)
   18 3C                                # unsigned(60)
   03                                   # unsigned(3)
   83                                   # array(3)
      61                                # text(1)
         40                             # "@"
      D9 0107                           # tag(263)
         54                             # bytes(20)
            495F947276749CE646F68AC8C248420045CB7B5E # contract address
      C2                                # tag(2)
         58 20                          # bytes(32)
            42CDA393BBE6D079501B98CC9CCF1906901B10BF000000000000020000000001 # token id
```

* As a hex string:

```
A3010802183C03836140D9010754495F947276749CE646F68AC8C248420045CB7B5EC2582042CDA393BBE6D079501B98CC9CCF1906901B10BF000000000000020000000001
```

* As a UR **TODO**:


### References

| Title | Link |
| --- | --- |
| [BIP44] | https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki |
| [SLIP44]  | https://github.com/satoshilabs/slips/blob/master/slip-0044.md |
| COSE| https://www.rfc-editor.org/rfc/rfc9053.html |
| IANA COSE Elliptic Curves | https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves |
| IANA CBOR Tags | https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml |
| Hex String Tag for CBOR | https://github.com/toravir/CBOR-Tag-Specs/blob/master/hexString.md |