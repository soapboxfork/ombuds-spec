<!-- title: Bulletins Def -->

Bulletin Specification // DRAFT
----------------------

Overview
========
Occasionally the term when talking about Ombuds the term public statement is used. 
Nine times out of ten, the speaker is referring to a bulletin stored in the public record.

A bulletin is the name for a statement containing text, a timestamp, and optionally a location.
A bulletin's author is attributed to a public key according to [section of 2.7 of DD03](/author#2.7).

For a bulletin to be considered part of the public record it must be included in Bitcoin's block chain.
See section 5.1 of the thesis for a further discussion on the security implications of the previous statement.

What follows below is a description of the formats bulletins can be in.

Contents
========
1. Wire Protocol Schema
2. Public Record Schema
3. JSON Schema

Wire Protocol Schema
====================

A bulletin is defined as a Bitcoin transaction that contains the following protocol buffer in a series of data carrying outputs.
A bulletin can either be encoded in a bitcoin in several ways. 
A parser will look for the message in various encoding formats as described in [section 2.7.3 Encoding Formats of DD01](/DD01).

    message WireBulletin {
        required string message     = 1;
        required int64 timestamp    = 2; // Seconds since 00:00:00 Jan 1, 1970
        optional location           = 3;
    }

    message location {
        required lat                = 1;
        required lon                = 2;
    }

Public Record Schema
====================

The SQl schema for a bulletin can be found at its [source](https://github.com/soapboxsys/ombudslib/blob/master/protocol/schema.sql).

JSON Schema
===========

The fields of a bulletin encoded in JSON is fully specified in [bulletin.go](https://github.com/soapboxsys/ombudslib/blob/master/ombjson/responses.go)
