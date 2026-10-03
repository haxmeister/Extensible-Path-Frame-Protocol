# EPF 0.1 Test Vectors

This directory contains language-neutral examples for testing EPF encoders and decoders.

The normative wire rules remain in [../SPEC.md](../SPEC.md).

## Encoding

All byte strings in `vectors.json` are lowercase hexadecimal with no separators or `0x` prefix.

For valid vectors:

- `path_keys_hex` contains the decoded key bytes in order;
- `payload_hex` contains the payload bytes;
- `frame_hex` contains the complete encoded EPF frame.

For invalid vectors:

- `frame_hex` contains the received bytes;
- `expected_error` identifies the condition being exercised.

The error names are test-vector labels. EPF does not define on-wire error codes.

## Header reminder

Every complete frame begins with:

```text
2 bytes  path_length, unsigned big-endian
4 bytes  payload_length, unsigned big-endian
```

## Use

A conforming implementation should be able to:

1. encode each valid vector from `path_keys_hex` and `payload_hex` and produce exactly `frame_hex`;
2. decode each valid `frame_hex` back to the listed keys and payload;
3. refuse to deliver invalid vectors as complete valid frames.

Truncated vectors represent end-of-input conditions. While more input may still arrive, those same byte sequences are simply incomplete rather than erroneous.
