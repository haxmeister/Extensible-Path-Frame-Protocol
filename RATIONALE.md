# EPF Rationale

This document explains why EPF 0.1 is shaped the way it is.

It is informative. [SPEC.md](SPEC.md) is the normative protocol definition.

## Core idea

EPF is deliberately built around only two application-visible pieces:

```text
ordered application-defined path
+
opaque payload
```

The path provides structure. The payload carries data. EPF does not decide what either means.

The design goal is not to create a smaller RPC protocol, broker protocol, object protocol, or serialization format. The goal is to provide a small framing primitive that applications can use to build those things when needed.

## Why a path?

Many application protocols need more than a single message type, but they do not necessarily need a large fixed header full of application-specific fields.

An ordered path gives an application a sequence of routing or classification coordinates without assigning semantics to them.

Examples might be interpreted by an application as:

```text
["rpc", "users", "get", "1742"]
["chat", "room-7", "message"]
["sensor", "building-a", "floor-3", "temperature"]
["file", "937", "chunk", "42"]
```

None of those meanings exist in EPF itself.

## Why an explicit path length?

The parser should know the complete path boundary before it begins interpreting key separators.

A 16-bit `path_length` gives the path a hard wire-level bound and prevents the path parser from searching indefinitely for a special terminator.

It also lets the parser locate the payload without reserving an end-of-path escape sequence.

## Why a 16-bit path length?

A maximum encoded path size of 65,535 bytes is already much larger than normal routing structures need.

Using a larger field would increase every frame header to support paths that implementations would usually reject for resource reasons.

Implementations are expected to enforce smaller operational limits where appropriate.

## Why a 32-bit payload length?

A 32-bit unsigned length permits payloads up to 4,294,967,295 bytes.

That is already very large for one framed message.

Applications that need to transfer larger logical objects can divide them across multiple application-defined EPF frames without making every EPF header larger.

## Why big-endian integers?

Big-endian byte order is conventional for network protocols, simple to specify, and widely supported.

EPF does not need a variable-length integer scheme for two fixed fields.

Fixed-width lengths also make the header size and parser state immediately known.

## Why NUL-separated keys?

EPF needs a compact way to preserve individual path keys inside one bounded path section.

Using `0x00` as the separator costs one byte between keys and allows every other byte value inside a key.

The path remains one framing object rather than turning every key into a separately length-framed transport object.

## Why arbitrary byte keys?

EPF does not need to impose a text model.

A key can be ASCII, UTF-8, a binary identifier, or another application-defined byte representation.

The only reserved byte inside the path is `0x00`.

This also avoids questions about Unicode normalization, case rules, or canonical text forms at the protocol layer.

## Why no empty keys?

Empty keys currently have no demonstrated protocol-level use case.

Forbidding them gives the path one unambiguous representation and makes malformed separator patterns easy to detect.

If an application needs a semantic empty value, it can define that meaning itself.

## Why is the zero-key path valid?

A zero-key path falls naturally out of the data model.

It can be useful for a default handler, connection-level data, or a single-purpose EPF connection.

No separate control-frame type is needed.

Its encoding is simply `path_length = 0`.

## Why no trailing NUL?

The path length already identifies the end of the path section.

A trailing separator would add a byte without adding information and would introduce a second possible interpretation involving an empty final key.

## Why no version field in every frame?

A version byte would be paid for by every frame even when an entire connection uses one protocol version.

If version negotiation becomes necessary, it can be handled by connection setup, transport negotiation, or another mechanism outside each EPF frame.

The document version "EPF 0.1" is therefore not a per-frame wire field.

## Why no flags?

A generic flags field tends to become a collection point for semantics that do not belong in the base protocol.

A proposed feature should first be expressible through application path keys, payload contents, or connection setup.

A flag should not be added merely because unused bits are available.

## Why no content type or serialization marker?

The payload is intentionally opaque.

Applications already know, negotiate, or define how their payloads are interpreted.

Adding a content-type system to EPF would couple framing to application data representation.

## Why no request ID, stream ID, or message type?

Those are useful concepts for some applications and unnecessary for others.

Applications can represent them in path keys or payload contents when needed.

EPF should not make every application pay for concepts it does not use.

## Why no multipart frame?

One EPF frame has one path and one payload.

Applications needing chunks, parts, or compound data can use multiple EPF frames or define structure inside the payload.

Keeping multipart semantics out of the base protocol keeps frame parsing simple.

## Design test for future changes

Before adding anything to the base frame, ask:

1. Is this required to identify EPF frame boundaries or the EPF path?
2. Does every EPF application need it?
3. Can it be represented cleanly by path keys, payload contents, or connection setup?
4. Does adding it make independent implementations materially safer or more interoperable?

If the answer does not strongly justify a base-protocol field, it should remain outside EPF.
