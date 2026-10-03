# Contributing to EPF

EPF is intentionally small.

Contributions are welcome, but changes to the base wire format should be held to a high bar.

## Protocol changes

A proposal that changes the wire format should explain:

1. the problem being solved;
2. why the problem belongs in EPF rather than the application;
3. why path keys, payload contents, or connection setup cannot solve it cleanly;
4. the exact byte-level change;
5. compatibility consequences;
6. security and resource-consumption consequences;
7. new or changed test vectors.

New base fields should not be added merely because they are convenient for a particular application model.

EPF should remain useful without assuming RPC, publish/subscribe, streams, object models, serialization formats, or authentication schemes.

## Specification changes

Normative behavior belongs in [SPEC.md](SPEC.md).

Design explanation belongs in [RATIONALE.md](RATIONALE.md).

Security analysis belongs in [SECURITY.md](SECURITY.md).

Implementation advice that is not part of wire conformance belongs in [IMPLEMENTATION-GUIDE.md](IMPLEMENTATION-GUIDE.md).

## Test vectors

A change affecting encoded bytes should include corresponding updates under [test-vectors/](test-vectors/).

Test vectors should use hexadecimal byte strings so they can be consumed by implementations in any language.

## Compatibility

Until EPF 0.1 is declared stable, incompatible draft changes may still occur.

Such changes should be explicit, documented, and accompanied by updated test vectors.

Once a wire version is declared stable, compatibility should be treated conservatively.

## Scope

The protocol specification is primary.

Reference implementations and language bindings should demonstrate the protocol without turning their APIs into normative requirements.
