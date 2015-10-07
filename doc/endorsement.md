<!-- title: endorsments -->

Endorsement Specification // DRAFT
-------------------------

Overview
========
An endorsement is a type of record that can be encododed in a Bitcoin transaction.
The goal of an endorsement is to provide a low cost, permanent way for organizations and individuals to publicly signal to the network that bulletin satisfies their criterea for endorsement. 
By providing an open mechanism for people to express approval of specific bulletins, more meaningful interactions can occur through Ombuds.

An example of this would be auditing a system that produces backups of politically sensitive tweets.
Multiple third parties could automatically validate that a tweet was accurately stored in the public record as a bulletin and endorse that bulletin.
This would provide a strong layer of audits that established that a tweet was created on Twitter at some point in the past.

Another use case is the aggregation of quality sources and material by news agencies interested in collecting digital ground truth.

What an endorsement really means is defined by the creator of the endorsement. 
Webster's dictionary gives us three definitions of the word 'endorsement'. 

    Endorsment (noun)
    : a public or official statement of support or approval

    : the act of publicly saying that you like or use a product or service 
    in exchange for money

    : the act or result of writing your name on the back of a check



All of the definitions or none of them can apply to an endoresment stored in the public record.

Contents
========
1. Specification
2. Public Record Schema
3. JSON Schema

Specification
=============

An endorsement is defined as a Bitcoin transaction which contains the following protocol buffer in a data carrying output.

    message endorsment {
        required bytes txid         = 1; // A 32 byte SHA hash
        required int64 timestamp    = 2; // Seconds since 00:00:00 Jan 1, 1970
    }

This buffer is encoded according to version 1.5.1 of Google's protocol buffer specification.
It is prefixed with the 4 byte indicator that informs parsers that the following bytes are an Ombuds record.
An example of the total payload encoded in hex follows this format:

                      head |        protocol buffer   |
    payload byte[] := 0xBEEF843F6724A347297987CD232304D
                             |        txid          | ts 

An endorsement can either be encoded in transactions in several ways.
Parsers will look in the valid encoding formats as described in [section 2.7.3 data encoding formats of DD01](/DD1).
Since endorsements are less than 40 bytes in size, the reference implementation will place endorsements in OP_RETURN outputs.

Public Record Schema
====================

An endorsement will be stored in the public record alongside bulletins by full nodes maintaining copies of the public record.
The SQL schema is the same as the protocol buffer format except for one notable addition.
An author field is included which is the Bitcoin address of the transactions first signing input as described in [section 1.4 author key formats of DD01](/DD01)

    CREATE TABLE endorsements (
        txid        TEXT, -- the enclosing transactions SHA hash
        bid         TEXT, -- the endorsed bulletins SHA hash
        timestamp   INT,  -- Unix time
        author      TEXT, -- formatted as a bitcoin address.

        FORIEGN KEY(bid) REFERENCES bulletins(txid),
        PRIMARY KEY(txid)
    );

[Source](https://github.com/soapboxsys/ombudslib/blob/master/protocol/schema.sql)

JSON Schema
===========

The JSON schema of an endorsement follows the same conventions laid out by the JSON format for bulletins.
An example endorsement returned by a JSON API looks like this.

    {
        txid:      "c9be2cbeb2da2dfe1f0158246938a3899f4e5c5108ea54e75c7c4f22580e42bc",
        bid:       "17756d6259242add26f728b8feb8f4b83a8c5af39d923b65f5ab6c0b27223d14",
        author:    "mkzkHkLn1Q5fRuNh2wbg3V7BjX4cKXQHYE",
        timestamp: 1444074578
    }
