# Batch signing Protocol

## NBCR-2025-002

© 2025 NGRAVE <br/>
Authors: Mathieu Da Silva <br/>
Date: April 16, 2025

---

# I - Introduction

In the evolving ecosystem of airgapped cryptocurrency wallets, enhancing flexibility and contextual understanding during the signing process is increasingly important. Existing communication protocols, such as those described in [[NBCR-2023-003]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-003-sign.md), enable standardized transaction signing via Uniform Resource (UR) types and QR communication. However, current standards assume that the transaction payload is fully formed and immutable at the point of signing, limiting the ability of the offline signer to participate dynamically in the transaction construction process.

This paper introduces the concept of **intents**—structured metadata embedded within the `sign-request`—which informs the offline signer about expected interactions or dynamic elements in the signing flow. This capability enables more flexible flows, such as signer-side input, context-aware signing, and adaptive transaction finalization.

---

# II - Intents

A signing request using intents introduces a controlled interaction flow between the watch-only wallet and the offline signer, allowing the signer to dynamically modify specific parts of the transaction prior to signature. This enables flexible signing scenarios while preserving end-user verification and transaction integrity.

The process follows these steps:

1. The watch-only wallet constructs a transaction with some additional steps required to be performed by the offline signer before signing the transaction, e.g. replacing a parameter inside the transaction.
2. The wallet encodes the sign-request and attaches the intent metadata as defined in this document. This informs the signer about required actions (e.g., field substitution) before signing.
3. The user scans the QR code containing the `sign-request` and associated `intent` using the offline signer.
4. The offline signer parses the intent and interprets the requested operations to be performed on the payload.
5. The offline signer applies the intent—e.g., replacing placeholder values with real data only available on the device (like a secure destination address).
6. The user verifies the final transaction content after intent application. This step ensures What You See Is What You Sign (WYSIWYS) by confirming the exact data to be signed.
7. Upon user confirmation, the offline signer signs the final transaction and encodes the signature into a sign-response UR type.
8. The user wallet scans the `sign-response` using the watch-only wallet to validate the signature and to broadcast the transaction to the blockchain.

## Intent UR type registry

The `intent` UR type is used to communicate metadata instructing the offline signer to apply modifications—such as substituting placeholder values—prior to signing.

**CDDL for `intent`**

UR Type Tag: `#6.41415`

```
; The encoding_type defines how the parameter value should be interpreted.
; This enumeration is extensible to support future encoding formats.
encoding_type = int
integer = 1
hex = 2
base32 = 3
base58 = 4
base64 = 5
ascii = 6
utf8 = 7

; Each parameter specifies a name that uniquely identifies a placeholder 
; in the transaction and the encoding used to interpret its replacement value.
parameter = [
    name: string,
    encoding: encoding_type
]

; substitute_fields defines a prefix used to mark placeholders in the transaction,
; and the list of parameters that should replace these placeholders.
substitute_fields = [
    placeholder_prefix: string,
    parameters: [parameter]
]

; The top-level intent structure is a tagged CBOR map.
; Additional intent types can be added in the future using new map keys.
intent = (
    substitute: substitute_fields
)

substitute = 1
```

## Use case and example

A practical use case for the intent UR type is found in the Babylon Delegation flow, specifically in the `MsgCreateBTCDelegation` transaction format defined in [[Babylon/docs/register-bitcoin-stake]](https://github.com/babylonlabs-io/babylon/blob/main/docs/register-bitcoin-stake.md).

This scenario involves a batch-sign-request (see [[NBCR-2025-001]](https://github.com/ngraveio/Research/blob/main/papers/nbcr-2025-001-batch-sign.md)) containing five distinct transactions:
1. Staking PSBT
2. Slashing PSBT
3. Unbonding slashing PSBT
4. Proof of Possession 
5. Babylon Delegation message

The fifth transaction (`MsgCreateBTCDelegation`) references values derived from the other transactions in the batch (e.g. signatures and PSBTs) and includes placeholders that must be dynamically replaced by the offline signer before the final signature.

In this case, the watch-only wallet constructs the delegation transaction with placeholder strings (e.g., $BATCH.SIG(4), $BATCH.TX(1)) and includes an intent instructing the signer to:
- Interpret `$BATCH.` as the placeholder prefix
- Replace placeholders with corresponding batch elements based on type (`TX` or `SIG`) and index

Each parameter is uniquely identifiable and unambiguous, directly referencing the structure and position of other batch entries.

```
{
    "btc_sig": "$BATCH.SIG(4)"
    "staking_tx": "$BATCH.TX(1)",
    "slashing_tx": "$BATCH.TX(2)",
    "delegator_slashing_sig": "$BATCH.SIG(2)",
    "unbonding_slashing_tx": "$BATCH.TX(3)",
    "delegator_unbonding_slashing_sg": "$BATCH.SIG(3)"
}
```

These placeholders are resolved by the offline signer using metadata from the `batch-sign-request`. The final transaction, after intent resolution, is displayed to the user for verification before signing.

This use case demonstrates how intents can orchestrate dynamic transaction composition while ensuring full user control and integrity through offline verification.

### Intent example for parameter substitutions on Babylon Message delegation

- CBOR diagnosis format

```
{
    1: ["$BATCH.", 
        [["TX(1)", 5],
         ["TX(2)", 5],
         ["TX(3)", 5],
         ["SIG(2)", 5],
         ["SIG(3)", 5],
         ["SIG(4)", 5]
        ]
    ]
}
```

- CBOR encoding

```
A1                          # map(1)
   01                       # unsigned(1)
   82                       # array(2)
      67                    # text(7)
         2442415443482E     # "$BATCH."
      86                    # array(6)
         82                 # array(2)
            65              # text(5)
               5458283129   # "TX(1)"
            05              # unsigned(5)
         82                 # array(2)
            65              # text(5)
               5458283229   # "TX(2)"
            05              # unsigned(5)
         82                 # array(2)
            65              # text(5)
               5458283329   # "TX(3)"
            05              # unsigned(5)
         82                 # array(2)
            66              # text(6)
               534947283229 # "SIG(2)"
            05              # unsigned(5)
         82                 # array(2)
            66              # text(6)
               534947283329 # "SIG(3)"
            05              # unsigned(5)
         82                 # array(2)
            66              # text(6)
               534947283429 # "SIG(4)"
            05              # unsigned(5)
```

- UR encoding

```
oyadlfiodkfwfpghfxfddmlnlfihghhddeehdtahlfihghhddeeydtahlfihghhddeeodtahlfiygugafldeeydtahlfiygugafldeeodtahlfiygugafldeeedtahoshevtmd
```

---

# III - Considerations

## Open framework

Although this document focuses on placeholder substitution as the primary use case for intents, the framework is intentionally designed to be extensible and future-proof. Intents are structured to support a broad range of signer-driven behaviors beyond field replacement.

Future intent types may include, for example:
- Explicit user confirmation: Enforce user approval for critical transaction fields (e.g., destination address, transfer amount), even if the data is already present in the payload.
- Signer-side fee adjustment: Allow the offline signer to edit the transaction fee based on predefined rules.

These examples are illustrative and not exhaustive. The intent mechanism is deliberately flexible, allowing the ecosystem to define additional use cases as needed—without altering the core signing protocol.

## Security consideration

The use of intents introduces dynamic behavior in the signing flow, allowing the offline signer to perform operations such as substituting placeholder values. While this increases flexibility, it also requires additional safeguards to uphold the integrity of the signing process.

To preserve the What You See Is What You Sign (WYSIWYS) principle, it is critical that:
- All intent-based modifications MUST be applied before user confirmation. The user must verify the fully resolved transaction on the offline signer, after all placeholders have been substituted. No changes—intent-driven or otherwise—must occur after user approval.
- The offline signer MUST clearly present the final transaction to the user in a human-readable format that reflects the actual data to be signed, including any substituted values.
- If a placeholder cannot be resolved (e.g., missing data, index mismatch), the signer MUST display a warning and refuse to proceed with signing.

Failure to enforce user verification post-intent application could result in signature over tampered or unintended transactions, compromising user funds or trust. Therefore, any implementation of the intent mechanism must treat intent resolution as a precondition to signature verification and user approval.

---

# IV - References

| Reference | Link |
| --- | --- |
| [NBCR-2023-003] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2023-003-sign.md |
| [Babylon/docs/register-bitcoin-stake] | https://github.com/babylonlabs-io/babylon/blob/main/docs/register-bitcoin-stake.md |
| [NBCR-2025-001] | https://github.com/ngraveio/Research/blob/main/papers/nbcr-2025-001-batch-sign.md |