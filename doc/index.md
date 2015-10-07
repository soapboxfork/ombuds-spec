<!-- title: Ombuds Spec -->

# Ombuds Technical Specification
Ombuds ensures a person's words are neither forgotten nor modified.
This document defines and specifies the implementation of [Ombuds: A Public Space With a Single Shared History](https://getombuds.org/research/).

## Protocol Definitions
[Public Record](/public_record)
:   A single, consistent database of all public bulletins and public endorsements. Derived from a block chain, a public record is an immutable, append only data set.

[Author](/author)
:   The Base58Check encoded address of the public key that signed a record.

[Bulletin](/bulletin)
:   The common format for statements stored within the public record.

[Endorsement](/endorsement)
:   A record signing one's approval to some bulletin, also stored within the public record.

## System Design

[Mobile Architecture](/mobile-arch)
:   The design of the planned Android app written by [Soapbox Systems](http://soapbox.systems).

[Web Relay API Design](/web-relay-api)
:   The design of the HTTP api that Web and mobile clients consume.

## Future Work

[Publicly Auditable API](/audit-api-exten)
:   A draft of an extension for the web relay API that lets clients audit the servers they rely on.

---------------------

## Meta

[How this Spec is Versioned](/versions)
: Each document in this specification is governed by a review process
