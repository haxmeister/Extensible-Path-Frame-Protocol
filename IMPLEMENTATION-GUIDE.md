# EPF Implementation Guide

This guide describes a straightforward way to implement EPF 0.1.

It is informative. The normative rules are in [SPEC.md](SPEC.md).

## Keep the API close to the protocol

A generic EPF implementation needs very little application-facing behavior.

Conceptually:

```text
encode_frame(path, payload)
feed(bytes)
next_frame()
```

A callback-oriented implementation might deliver:

```text
on_frame(path, payload)
```

The EPF engine should not add RPC, publish/subscribe, automatic serialization, or application routing semantics.

## Encoding a frame

Given an ordered list of key byte strings and a payload:

1. Reject any empty key.
2. Reject any key containing `0x00`.
3. Join the keys with one `0x00` byte.
4. If there are no keys, use an empty path.
5. Reject the path if it exceeds 65,535 bytes or the implementation's lower path limit.
6. Reject the payload if it exceeds 4,294,967,295 bytes or the implementation's lower payload limit.
7. Encode the two lengths in big-endian order.
8. Emit the header, path bytes, and payload bytes.

Language-neutral pseudocode:

```text
function encode_frame(keys, payload):
    for key in keys:
        require length(key) > 0
        require key does not contain 0x00

    path = join(keys, 0x00)

    require length(path) <= local_path_limit
    require length(payload) <= local_payload_limit

    return uint16_be(length(path))
         + uint32_be(length(payload))
         + path
         + payload
```

## Incremental parser

A streaming parser can use four simple states:

```text
HEADER -> PATH -> PAYLOAD -> DELIVER
   ^                            |
   +----------------------------+
```

### HEADER

Wait until 6 bytes are available.

Decode:

```text
path_length
payload_length
```

Check both lengths against local limits before reserving large buffers.

If both lengths are zero, the complete frame is available immediately after the header.

### PATH

Wait for exactly `path_length` bytes.

If `path_length` is zero, the decoded path contains zero keys.

Otherwise validate that the path:

- does not begin with `0x00`;
- does not end with `0x00`;
- does not contain `0x00 0x00`.

The validated path can then be split on `0x00`.

Do not decode, normalize, or otherwise transform key bytes unless the application explicitly requests that behavior above EPF.

### PAYLOAD

Wait for exactly `payload_length` bytes.

The payload is opaque. The EPF parser performs no content validation.

### DELIVER

Deliver the path and payload as one completed frame, then return to HEADER.

## Buffering

A parser does not need to copy an entire frame into one buffer.

Possible implementations include:

- a single growable receive buffer;
- a small fixed header buffer plus bounded path and payload storage;
- slices into a larger receive buffer;
- a zero-copy view where object lifetime is carefully controlled.

The implementation strategy does not change the wire format.

The simplest correct implementation is preferable for a reference implementation.

## Operational limits

Implementations should expose or document finite limits for:

```text
maximum accepted path bytes
maximum accepted payload bytes
maximum number of keys
maximum key bytes
maximum buffered bytes
incomplete-frame timeout or progress policy
```

Defaults should be chosen for the implementation's intended environment rather than standardized by EPF.

## Rejection strategy

Reject unacceptable declared lengths immediately after decoding the header.

Reject malformed path bytes before delivering the frame to application code.

On a byte-stream transport, stopping EPF decoding after malformed input is the simplest behavior consistent with the specification. An implementation that controls the surrounding transport may close it.

Do not search arbitrary later bytes for a guessed replacement frame boundary.

## Multiple frames

A byte stream may contain frames back to back:

```text
frame frame frame frame ...
```

A parser should continue decoding while enough buffered bytes are available for another complete frame.

It should stop cleanly when the remaining bytes form only a partial frame.

## Zero-key and zero-payload frames

Both are ordinary EPF frames.

Root frame with no payload:

```text
path_length    = 0
payload_length = 0
```

Root frame containing five payload bytes:

```text
path_length    = 0
payload_length = 5
payload        = 5 opaque bytes
```

No special frame type is needed.

## Application dispatch

A protocol engine may provide optional dispatch helpers, but those helpers are above the EPF wire protocol.

The base decoder should expose the path as an ordered list of byte strings and the payload as bytes.

For example, a Perl-facing API could look like:

```perl
$epf->send(
    [ 'chat', 'room-7', 'message' ],
    $payload,
);
```

That is an API convenience. The strings `chat`, `room-7`, and `message` have no meaning to EPF itself.

## Testing

An implementation should test at least:

- zero-key frames;
- zero-length payloads;
- multiple keys;
- arbitrary non-NUL key bytes;
- payloads containing NUL;
- fragmented headers;
- fragmented paths;
- fragmented payloads;
- multiple concatenated frames;
- leading path separators;
- trailing path separators;
- consecutive path separators;
- truncated headers;
- truncated paths;
- truncated payloads;
- local limit rejection;
- integer overflow around buffer-size calculations.

Normative examples are provided in [test-vectors/](test-vectors/).
