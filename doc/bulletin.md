<!-- title: Bulletins Def -->

DD03 Bulletin Specification // DRAFT
===========================

Overview
--------

A bulletin is the name for a public statement - generically called a record - containing a text message, a timestamp, and optionally a location.
A bulletin's author is attributed to a public key according to [DD02](/author).
For a bulletin to be considered part of the public record it must be included in Bitcoin's block chain.
The contents below describe the valid formats a bulletin can take.

Status of this Document
-----------------------

This document has been reviewed by the author.
It's contents have not been implemented and it lacks concrete types.
This will be rememdied.

Nick -- Oct 29, 2015



Contents
--------
1. Wire Protocol Schema
2. Public Record Schema
3. JSON Schema

Wire Protocol Schema
--------------------

A bulletin is defined as a Bitcoin transaction that contains the following protocol buffer in a series of data carrying outputs.
A bulletin can either be encoded in a bitcoin in several ways. 
A parser will look for the message in various encoding formats as described in [Encoded Message Formats of DD01](/public-record).

     message WireBulletin {
             required string message     = 1;
             required uint64 timestamp   = 2; // Seconds since 00:00:00 Jan 1, 1970
             optional Location location  = 3;
     }

     // A single WGS84 Datum
     message Location {
             required double lat          = 1;
             required double lon          = 2;
             required double h            = 3;
     }


Public Record Schema
--------------------

The SQl schema for a bulletin can be found at its [source](https://github.com/soapboxsys/ombudslib/blob/master/protocol/schema.sql).

JSON Schema
-----------

The fields of a bulletin encoded in JSON is fully specified in [bulletin.go](https://github.com/soapboxsys/ombudslib/blob/master/ombjson/responses.go)
