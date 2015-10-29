<!-- title: Endorsment Def -->

DD04 Endorsement Specification // Revision-1
==============================

Overview
--------
An endorsement is a type of record that can be encododed in a Bitcoin transaction.
The goal of supporting endorsements is to provide a low cost, permanent way for organizations and individuals to publicly signal to others that a bulletin satisfies their criterea for endorsement. 
By providing an open mechanism for people to express approval of specific bulletins, more meaningful interactions can occur through Ombuds.

A plausible example is a system that produces backups of sensitive tweets.
Multiple third parties could automatically validate that a tweet was accurately stored in the public record as a bulletin and endorse that bulletin.
This would provide a strong set of independent audits that would establish the fact that a tweet was created on Twitter at some point in the past.

Another use case is the aggregation of quality sources and material by news agencies interested in collecting digital ground truth.
The institution could endorse bulletins produced by individuals they saw as reliable sources in the field.

What an endorsement really means is defined by the creator of the endorsement. 
Webster's dictionary gives us three definitions of the word 'endorsement'. 

    Endorsment (noun)
    : a public or official statement of support or approval

    : the act of publicly saying that you like or use a product or service 
    in exchange for money

    : the act or result of writing your name on the back of a check



All of the definitions or none of them could apply to an endoresment stored in the public record.

Status of This Document
----------------------

This is the working defintion of endorsements that will be implemented. 
Changes to the JSON are likely. 
Changes to the public record SQL are less likely, but may still occur.

Nick -- Oct 29, 2015


Contents
--------
1. Wire Protocol Schema
2. Public Record Schema
3. JSON Schema

A Web of Trust
--------------

Understanding how endorsements create a web of trust gives us insight into a graph.
A crucial takeaway from the discussion below is that this model only gives us a one-way system of trust.
Someone who's bulletins have been endorsed cannot transfer his legitmacy to other less reliable parties.

Consider the example where Reporter A is reliable, but his friend Person B is not.
    Reporter A                Voice of America                   

    bulletin 1     <----       endorses

This results only in the following structure.
    
    Reporter A

    bulletin 1  <----  VoA
                       HWR
                       AmI
    bulletin 2  <----  Glenn
                       HWR

    bulletin 3  <----  Glenn
        
This means that Reporter A cannot transfer his legitmacy to another person via some protocol level mechanism.

    Person B        X                   Institutions
                    X                         
     bulletin 1     X <--- Reporter A <--- VoA
                    X                      AmI 
                    X                      HWR 


The best he can do is mention Person B in a bulletin or endorse B's work to signal to his endorsers that Person B is someone they should endorse.
Here `bltn 1B` mentions Person B.

    Person B         Reporter A       Institutions

     bltn 1C           bltn 1B <---       Voa
                               <---       HWR
                               <---       AmI




This gives VoA an oppurtunity to review Person B's work and endorse bulletins he has produced.

    Person B        Institutions
     
     bltn 1C   <---   Voa
                X     HWR
                X     AmI



While institutions involved in the system can make decisions about what they are willing to support, an external reader just looks at the institutions at the edge of the network.
That reader uses those organisations that he or she trusts as the source of reliable information about some event.

Wire Protocol Schema
--------------------

An endorsement is defined as a Bitcoin transaction that contains the following protocol buffer in a series of data carrying outputs.

    message WireEndorsment {
        required bytes bid          = 1; // A 32 byte SHA hash of the referenced bulletin's txid
        required int64 timestamp    = 2; // Seconds since 00:00:00 Jan 1, 1970
    }

This buffer is encoded according to version 2.6.1 of Google's protocol buffer specification.
It is prefixed with the 4 byte indicator that informs parsers that the following bytes are an Ombuds record.
An example of the total payload encoded in hex follows this format:

                      head |        protocol buffer   |
    payload byte[] := 0xBEEF843F6724A347297987CD232304D
                             |        txid          | ts 

An endorsement can either be encoded in transactions in several ways.
Parsers will look for endorsements in all of the valid encoding formats as described in the [Encoded Message Formats of DD01](/public-record).
Since endorsements are less than 40 bytes in size, the reference implementation will place endorsements in OP_RETURN outputs.

Public Record Schema
--------------------

An endorsement will be stored in the public record alongside bulletins by full nodes maintaining copies of the public record.
The SQL schema is the same as the protocol buffer format except for one notable addition.
An author field is included which is the Bitcoin address of the transactions first signing input as described in [DD02](/author)

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
-----------

The JSON schema of an endorsement follows the same conventions laid out by the JSON format for bulletins.
An example endorsement returned by a JSON API looks like this. 
Be aware that the fields are specified in the [ombudslib](https://github.com/soapboxsys/ombudslib) package definitively.

    {
        txid:      "c9be2cbeb2da2dfe1f0158246938a3899f4e5c5108ea54e75c7c4f22580e42bc",
        bid:       "17756d6259242add26f728b8feb8f4b83a8c5af39d923b65f5ab6c0b27223d14",
        author:    "mkzkHkLn1Q5fRuNh2wbg3V7BjX4cKXQHYE",
        timestamp: 1444074578
    }

