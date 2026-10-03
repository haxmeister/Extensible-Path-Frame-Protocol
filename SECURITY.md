# EPF Security Considerations

EPF is intentionally small, but a small wire format still needs defensive implementations.

This document is informative except where [SPEC.md](SPEC.md) states normative requirements.

## Threat model

An EPF parser should assume that every received byte can be controlled by an untrusted peer.

The main protocol-level risks are resource exhaustion, integer mistakes, incomplete-frame retention, malformed path handling, and unsafe assumptions about opaque application data.

## Validate lengths before allocation

The 16-bit path length and 32-bit payload length are wire-format limits, not instructions to allocate that much memory.

An implementation should compare declared lengths with local limits before allocating buffers or reserving application resources.

A declared 4 GiB payload does not require an implementation to accept or buffer a 4 GiB frame.

## Use overflow-safe size arithmetic

Implementations commonly need to calculate values such as:

```text
6 + path_length + payload_length
```

That calculation must be performed in a type large enough to represent the result.

Code should not trust an arithmetic result until overflow has been ruled out.

This is particularly important in languages or APIs where allocation sizes use a narrower integer type.

## Bound incomplete frames

A peer can declare a legal frame size and then transmit it extremely slowly or stop making progress.

Implementations should enforce finite limits on:

- bytes retained for incomplete frames;
- number of simultaneous incomplete frames, when applicable;
- total buffering per peer or connection;
- time allowed without sufficient progress.

Exact values depend on the implementation and deployment and are not fixed by EPF.

## Bound path complexity

A path has a hard byte limit, but implementations should also consider lower limits for:

- total path bytes;
- number of keys;
- individual key size.

A parser should not create an unlimited number of application objects while scanning a hostile path.

Where possible, validate the bounded path bytes before performing expensive application-level dispatch or allocation.

## Treat keys as bytes

EPF keys are arbitrary non-NUL byte strings.

Implementations must not assume that keys are valid UTF-8, printable text, filesystem names, URLs, SQL identifiers, shell arguments, or safe log text.

Applications that interpret keys as any of those things must perform their own validation and escaping.

## Treat payloads as untrusted opaque bytes

EPF does not validate payload contents.

A successfully decoded EPF frame says only that the framing is valid.

If an application interprets the payload as JSON, an image, compressed data, executable input, another protocol, or any other structured format, that parser needs its own security controls.

## Avoid speculative resynchronization

EPF has no synchronization marker.

After malformed input on a byte stream, scanning arbitrary bytes for something that looks like another EPF header can misidentify attacker-controlled data as a new frame.

Implementations should follow the recovery behavior in [SPEC.md](SPEC.md).

## Transport security is separate

EPF does not provide:

- encryption;
- integrity protection;
- peer authentication;
- authorization;
- replay protection;
- confidentiality.

Applications that need those properties must obtain them from the transport or another surrounding security layer.

## Authentication does not belong in the frame format

Authentication requirements vary widely between deployments.

EPF therefore does not reserve frame fields for credentials, tokens, identities, or authorization metadata.

Applications may perform authentication during connection setup or define their own application-level messages.

## Logging

Binary keys and payloads can contain control bytes, secrets, or large amounts of data.

Implementations should avoid blindly writing raw frame contents to terminals or logs.

Diagnostic output should be bounded and safely escaped.

## Denial of service

Protocol conformance does not require accepting every representable frame size.

Operational limits are a required part of a robust EPF implementation.

An implementation should be able to reject excessive work before committing resources proportional to attacker-provided lengths whenever possible.
