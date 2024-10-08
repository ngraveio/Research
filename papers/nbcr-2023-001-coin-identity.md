# UR Type Definition for Coin Identity
## NBCR-2023-001

**© 2023 Ngrave**

Authors: İrfan Bilaloğlu, Mathieu Da Silva, Maher Sallam<br/>
Date: 18 September 2024<br/>

Revised: October 04, 2024

---

### Introduction

In this document, we are defining the new UR type `coin-identity` based on new types and COSE constants defined in [[RFC9053]](https://www.rfc-editor.org/rfc/rfc9053.html).

We propose to define a UR type standardizing the coin identification by aggregating the following information:

1. Curve of the coin (e.g. *`["secp256k1", "ed25519", "p256 (secp256r1)”, “X25519 (sr25519)”]` ).* This information is mandatory since some blockchains (e.g. Tezos) support multiple elliptic curves.
2. Coin type as defined in [[SLIP44]](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) with the high bit turned off.
3. Sub-type to define additional information to identify the coin. For example, every EVM chain is required to have the chain ID defined as a sub-type.

### EC Curve Definitions

The required `curve` field carries the elliptic curve information of the coin, with the **value** field defined in the table from [[IANA COSE Elliptic Curves]](https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves).


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

The required `type` field carries the coin type information as defined in [[SLIP44]](https://github.com/satoshilabs/slips/blob/master/slip-0044.md) with the high bit turned off.

### CDDL

The following specification of `coin-identity` is written in CDDL. When used embedded in another CBOR structure, this structure should be tagged #6.41401.

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

### Example/Test Vector 1: Bitcoin (BTC)

* In the CBOR diagnostic notation:

```
{
   1: 8, ; curve-secp256k1
   2: 0 ; Bitcoin SLIP44
}
```

* Encoded as binary using [[CBOR-PLAYGROUND]](https://cbor.me/):

```
A2    # map(2)
   01 # unsigned(1) curve
   08 # unsigned(8) secp256k1
   02 # unsigned(2) type
   00 # unsigned(0) Bitcoin SLIP44
```

* As a hex string:

```
A201080200

```

* As a UR:

```
ur:coin-identity/hdadbraoae
```

### Example/Test Vector 2: Solana (SOL)

* In the CBOR diagnostic notation:

```
{
   1: 6, ; curve-ed25519
   2: 501 ; Solana SLIP44
}
```

* Encoded as binary using [[CBOR-PLAYGROUND]](https://cbor.me/):

```
A2         # map(2)
   01      # unsigned(1) curve
   06      # unsigned(6) ed25519
   02      # unsigned(2) type
   19 01F5 # unsigned(501) Solana SLIP44
```

* As a hex string:

```
A20106021901F5 

```

* As a UR:

```
ur:coin-identity/hdadbdaoftadpk
```


### Example/Test Vector 3: Polygon (POL)

* In the CBOR diagnostic notation:

```
{
   1: 8, ; curve-secp256k1
   2: 60, ; Ethereum SLIP44 
   3: [137] ; Polygon chain ID
}
```

* Encoded as binary using [[CBOR-PLAYGROUND]](https://cbor.me/):

```
A3          # map(3)
   01       # unsigned(1) curve
   08       # unsigned(1) secp256k1
   02       # unsigned(2) type
   18 3C    # unsigned(60) Ethereum SLIP44
   03       # unsigned(3) sub type
   81       # array(1) 
      18 89 # unsigned(137) Polygon Chain Id
```

* As a hex string:

```
A3010802183C03811889

```

* As a UR:

```
ur:coin-identity/hgadbraoetpsatenetfd
```


### References

| Title | Link |
| --- | --- |
| [RFC9053] | https://www.rfc-editor.org/rfc/rfc9053.html |
| [IANA COSE Elliptic Curves] | https://www.iana.org/assignments/cose/cose.xhtml#elliptic-curves |
| [SLIP44]  | https://github.com/satoshilabs/slips/blob/master/slip-0044.md |
| [CBOR-PLAYGROUND]  | https://cbor.me/ |