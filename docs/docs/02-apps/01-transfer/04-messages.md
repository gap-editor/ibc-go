---
title: Messages
sidebar_label: Messages
sidebar_position: 4
slug: /apps/transfer/ics20-v1/messages
---

# Messages

## `MsgTransfer`

A fungible token cross chain transfer is achieved by using the `MsgTransfer`:

```go
type MsgTransfer struct {
  SourcePort        string
  // with IBC v2 SourceChannel will be a client ID
  SourceChannel     string 
  Token             sdk.Coin
  Sender            string
  Receiver          string
  // If you are sending with IBC v1 protocol, either timeout_height or timeout_timestamp must be set.
  // If you are sending with IBC v2 protocol, timeout_timestamp must be set, and timeout_height must be omitted. 
  TimeoutHeight     ibcexported.Height
  // Timeout timestamp in absolute nanoseconds since unix epoch.
  TimeoutTimestamp  uint64
  // optional Memo field
  Memo              string
  // optional Encoding field 
  Encoding          string 
}
```

This message is expected to fail if:

- `SourcePort` is invalid (see [24-host naming requirements](https://github.com/cosmos/ibc/blob/master/spec/core/ics-024-host-requirements/README.md#paths-identifiers-separators).
- `SourceChannel` is invalid (see [24-host naming requirements](https://github.com/cosmos/ibc/blob/master/spec/core/ics-024-host-requirements/README.md#paths-identifiers-separators)).
- `Token` is invalid:
    - `Amount` is not positive.
    - `Denom` is not a valid IBC denomination as per [ADR 001 - Coin Source Tracing](/architecture/adr-001-coin-source-tracing).
- `Sender` is empty.
- `Receiver` is empty or contains more than 2048 bytes.
- `Memo` contains more than 32768 bytes.
- `TimeoutHeight` and `TimeoutTimestamp` are both zero for IBC Classic.
    - Note that `TimeoutHeight` as a concept is removed in IBC v2, hence this must always be emitted and only `TimeoutTimestamp` used. 

This message will send a fungible token to the counterparty chain represented by the counterparty Channel End connected to the Channel End with the identifiers `SourcePort` and `SourceChannel`. Note that in IBC v2 a pair of clients are connected and the `SourceChannel` is referring to the source `ClientID`.

The denomination provided for transfer should correspond to the same denomination represented on this chain. The prefixes will be added as necessary upon by the receiving chain.

If the `Amount` is set to the maximum value for a 256-bit unsigned integer (i.e. 2^256 - 1), then the whole balance of the corresponding denomination will be transferred. The helper function `UnboundedSpendLimit` in the `types` package of the `transfer` module provides the sentinel value that can be used.

### Memo

The memo field was added to allow applications and users to attach metadata to transfer packets. The field is optional and may be left empty. When it is used to attach metadata for a particular middleware, the memo field should be represented as a json object where different middlewares use different json keys.

For example, the following memo field is used by the callbacks middleware to attach a source callback to a transfer packet:

```jsonc
{
  "src_callback": {
    "address": "callbackAddressString",
    // optional
    "gas_limit": "userDefinedGasLimitString",
  }
}
```

You can find more information about other applications that use the memo field in the [chain registry](https://github.com/cosmos/chain-registry/blob/master/_memo_keys/ICS20_memo_keys.json).

### Encoding

In IBC v2, the encoding method used by an application has more flexibility as it is specified within a `Payload`, rather than negotiated and fixed during an IBC classic channel handshake. Certain encoding types may be more suited to specific blockchains, e.g. ABI encoding is more gas efficient to decode in an EVM than JSON or Protobuf. 

Within ibc-go, JSON, protobuf and ABI encoding are supported and can be used, see the [transfer packet types](https://github.com/cosmos/ibc-go/blob/14bc17e26ad12cee6bdb99157a05296fcf58b762/modules/apps/transfer/types/packet.go#L36-L40).
 