# Extensible Path Frame Protocol (EPF)

[![Status: Draft 0.1](https://img.shields.io/badge/status-draft%200.1-blue)](SPEC.md)
[![License: CC0 1.0](https://img.shields.io/badge/license-CC0%201.0-lightgrey)](LICENSE)

EPF is a minimal framed-message protocol built around one primitive:

```text
ordered application-defined path
+
opaque payload
```

EPF provides structure without semantics.

The protocol defines how a path and payload are placed on the wire. It does not define what the path means or how the payload should be interpreted.

## Wire format

Every EPF frame begins with a fixed 6-byte header:

```text
+----------------------+  2 bytes
| path_length          |  unsigned, big-endian
+----------------------+  4 bytes
| payload_length       |  unsigned, big-endian
+----------------------+
| path                 |  path_length bytes
+----------------------+
| payload              |  payload_length bytes
+----------------------+
```

Equivalent notation:

```text
uint16_be path_length
uint32_be payload_length
byte[path_length] path
byte[payload_length] payload
```

The path contains zero or more ordered keys. Keys are non-empty byte strings, may contain any byte except `0x00`, and are separated by a single `0x00`.

For example, the logical path:

```text
users
42
avatar
```

is encoded as:

```text
users 00 42 00 avatar
```

There is no trailing separator.

A zero-key path is represented by `path_length = 0`.

The payload is arbitrary bytes. EPF does not define a serialization format, content type, compression scheme, or application message type.

## What EPF does not define

EPF intentionally does not define:

- RPC
- requests or responses
- topics or subscriptions
- streams
- methods or services
- objects
- serialization
- authentication or authorization
- content types
- compression
- multipart messages
- application message types

Applications may build any of those concepts above EPF if they need them.

## Size model

The wire format permits:

| Field | Encoding | Wire maximum |
| --- | --- | ---: |
| Path | unsigned 16-bit | 65,535 bytes |
| Payload | unsigned 32-bit | 4,294,967,295 bytes |

These are wire-format maxima, not recommended operating limits. Implementations are expected to enforce smaller limits appropriate to their environment.

## Documents

- [SPEC.md](SPEC.md) - normative EPF 0.1 wire specification
- [RATIONALE.md](RATIONALE.md) - reasons behind the wire-format choices
- [SECURITY.md](SECURITY.md) - security and resource-exhaustion considerations
- [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md) - practical parser and encoder guidance
- [test-vectors/](test-vectors/) - normative examples for implementation testing
- [CONTRIBUTING.md](CONTRIBUTING.md) - guidelines for protocol changes

## Status

EPF 0.1 is currently a draft specification.

The version number identifies this specification revision. It is not carried in every EPF frame.

The wire format should remain deliberately small. Proposed additions to the base frame should demonstrate that they cannot be cleanly expressed by the application path, the payload, or connection setup.

## License

The EPF specification and documentation are dedicated to the public domain under [CC0 1.0 Universal](LICENSE).
