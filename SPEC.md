# Extensible Path Frame Protocol (EPF) 0.1

## 1. Status

This document defines the draft 0.1 wire format of the Extensible Path Frame Protocol (EPF).

EPF is a framed-message protocol whose base abstraction is:

```text
ordered application-defined path
+
opaque payload
```

EPF defines framing and path structure. It does not define application semantics.

The version number in this document identifies the specification revision. EPF 0.1 does not place a version field in each frame.

## 2. Requirement language

The key words MUST, MUST NOT, REQUIRED, SHOULD, SHOULD NOT, and MAY in this document are to be interpreted as described by RFC 2119 and RFC 8174 when, and only when, they appear in all capitals.

## 3. Frame format

An EPF frame consists of a fixed 6-byte header followed by the path and payload:

```text
uint16_be path_length
uint32_be payload_length
byte[path_length] path
byte[payload_length] payload
```

The fields occur on the wire in exactly this order:

```text
+----------------------+  2 bytes
| path_length          |
+----------------------+  4 bytes
| payload_length       |
+----------------------+  path_length bytes
| path                 |
+----------------------+  payload_length bytes
| payload              |
+----------------------+
```

No other field is part of an EPF 0.1 frame.

When multiple EPF frames are carried in one ordered byte stream, each frame begins immediately after the preceding frame ends.

## 4. Integer encoding

`path_length` MUST be encoded as an unsigned 16-bit integer in big-endian byte order.

`payload_length` MUST be encoded as an unsigned 32-bit integer in big-endian byte order.

The largest values representable by these fields are:

```text
path_length:       65,535 bytes
payload_length:    4,294,967,295 bytes
```

These are wire-format limits. Implementations MAY impose smaller operational limits.

## 5. Path

The path contains zero or more ordered keys.

EPF assigns no meaning to a key or to its position in the path.

### 5.1 Key encoding

Each key MUST contain at least one byte.

A key MAY contain any byte value except `0x00`.

EPF does not define a character encoding for keys.

An implementation MUST NOT perform Unicode normalization, case conversion, transcoding, or other implicit transformation of key bytes as part of EPF framing.

### 5.2 Key separator

Adjacent keys MUST be separated by exactly one `0x00` byte.

There MUST NOT be a separator after the final key.

For example, the logical path:

```text
["users", "42", "avatar"]
```

has the following path bytes:

```text
75 73 65 72 73 00 34 32 00 61 76 61 74 61 72
```

### 5.3 Empty keys

Empty keys are not valid in EPF 0.1.

For a non-empty path, the encoded path:

- MUST NOT begin with `0x00`;
- MUST NOT end with `0x00`;
- MUST NOT contain two consecutive `0x00` bytes.

A path violating any of these requirements is malformed.

### 5.4 Zero-key path

A path containing zero keys MUST be encoded with:

```text
path_length = 0
```

No path bytes follow the header in this case.

This is the only valid representation of a zero-key path.

EPF assigns no special application semantics to the zero-key path.

## 6. Payload

The payload consists of exactly `payload_length` bytes.

The payload MAY contain any byte value, including `0x00`.

A zero-length payload is valid.

EPF assigns no encoding or interpretation to the payload.

## 7. Frame parsing

A parser MUST logically perform the following operations:

1. Obtain the complete 6-byte header.
2. Decode `path_length` and `payload_length`.
3. Check the declared lengths against local operational limits.
4. Obtain exactly `path_length` path bytes.
5. Validate the path encoding.
6. Obtain exactly `payload_length` payload bytes.
7. Deliver the completed frame.

A parser MUST NOT deliver a frame as complete before all declared bytes have been received and the path has been validated.

An implementation SHOULD reject an unacceptable declared length before allocating or buffering resources proportional to that length.

An implementation is not required to store the entire frame in one contiguous buffer.

## 8. Incomplete and truncated frames

Receiving only part of a frame is not itself a protocol error while additional bytes may still arrive.

A parser MAY retain partial input while waiting for more bytes, subject to its operational resource and progress limits.

If the underlying byte source ends after a frame has begun but before all declared bytes have arrived, that frame is truncated.

A truncated frame MUST NOT be delivered as a complete frame.

## 9. Malformed frames

A frame is malformed when its bytes violate the EPF 0.1 wire rules.

A malformed frame MUST NOT be delivered as a valid EPF frame.

A frame that exceeds a local implementation limit is not necessarily malformed on the wire, but the implementation MUST reject it locally.

## 10. Recovery after invalid input

EPF 0.1 defines no resynchronization marker and no byte-scanning recovery procedure.

On an ordered byte-stream transport, an implementation that encounters malformed input SHOULD stop decoding further EPF frames from that byte stream. If the implementation controls the surrounding transport, it MAY close that transport.

An implementation MUST NOT claim EPF resynchronization by searching arbitrary subsequent bytes for a guessed frame boundary.

EPF defines no protocol error frame, error number, or mandatory error response.

## 11. Resource limits

Implementations MUST enforce finite operational limits appropriate to their environment.

Implementations SHOULD consider limits for:

- accepted path bytes;
- accepted payload bytes;
- number of keys;
- individual key size;
- bytes buffered per parser or connection;
- time allowed for an incomplete frame;
- minimum progress required while a frame remains incomplete.

Operational limits MAY be smaller than the limits representable by the wire format.

Operational limits do not change the EPF encoding.

## 12. Transport

EPF defines frame bytes, not a transport protocol.

A transport carrying multiple EPF frames as a byte stream MUST preserve byte order and byte values.

Connection establishment, confidentiality, integrity protection, peer authentication, retransmission, congestion control, and transport-specific framing are outside the scope of EPF.

## 13. Protocol scope

EPF 0.1 defines only:

- frame boundaries;
- ordered application-defined path keys;
- opaque payload bytes.

EPF 0.1 does not define RPC, requests, responses, methods, services, topics, subscriptions, streams, request identifiers, stream identifiers, content types, serialization, compression, authentication, authorization, multipart messages, or application message types.

Applications MAY define such concepts above EPF.

## 14. Conformance

An encoder conforms to EPF 0.1 if every frame it emits follows this specification.

A decoder conforms to EPF 0.1 if it correctly accepts valid frames within its documented operational limits and rejects malformed frames as required by this specification.

Implementations MUST document operational limits that can cause an otherwise valid EPF frame to be rejected.
