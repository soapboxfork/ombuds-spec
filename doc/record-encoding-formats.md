<!-- title: Record Encoding Formats -->

DD01 Record Encoding Formats
============================

Overview
--------
Several encoding formats to store a serialized record within a Bitcoin transaction are accepted.
Only the encoding formats defined in this article are valid ways to append a [bulletin](/bulletin) or [endorsement](/endorsement) to the [public record](/public-record).
Each format has an introduction date and is not a valid encoding format before that date.
New encoding formats will likely be introduced.


Status of this Article
----------------------

Encoding formats have not been finalized. 
Encoding examples must be fleshed out.
We must determine if we want wirecord length encoded.
The introduction date of the encoding formats will be the relase date of the Ombuds specification.

--Alex, November 3, 2015


Contents
--------
1. Assumptions 
2. Criteria
3. Ombuds Identifier
3. Formats


Assumptions
-----------
This article assumes:

* a **valid [wirerecord protobuf](https://github.com/soapboxsys/ombudslib/blob/master/protocol/wirerecord/types.proto)** is ready for encoding.
* the Bitcoin transaction inputs are valid and hold enough coin to fund the transaction outputs.


Criteria
--------
To assess each encoding format, three criterion are used:

The likelyhood a record is included within a block.
:   Bulletins and endorsements provide censorship resistent guarantees;
    these records are considered "secure" when they reside within the public record.
    As a subset of the block chain, the public record is comprised of the records within mined blocks.
    To that end, it is of *extreme importance records are chosen by Bitcoin miner's for block inclusion*.
    When considering each encoding format, the likelyhood of block inclusion is the top priority.'

The cost ($) of storing a record.
:   To broadcast any record, an author must spend Bitcoin.
    While a minimal cost, the Bitcoin required is a non-negligible amount. 
    Speech is not free (libre) when restricted to those fortunate with access to financing.
    Thus, the monetary cost of an encoding format must be considered.

The prunability of a record.
:   A healthy bitcoin network provides Ombuds with a reliable and guarantee-laden data store.
    If the network and block chain are sustainable entities, the long-term viability of digital currencies and dependent systems, at large, is possible.
    The impact each encoding format has on the network and block chain sustainability must be considered.


Ombuds Identifier
-----------------
In every encoding format, the serialized Ombuds identifier `"BRETHREN"` must precede the serialized wirerecord.
The UTF-8 encoded string indicates an Ombuds record follows.

    Ombuds Identifier
    UTF-8 string : BRETHREN
    byte array   : [0x42, 0x52, 0x45, 0x54, 0x48, 0x52, 0x45, 0x4e]
    

Formats
=======

Values
------
Sending an "acceptable" amount of bitcoin to each transaction output increases the likelyhood of transaction propogation and block inclusion.
However, this specification will **not** provide a hard definition for calculating output values.
Instead, the responsibility of value calculation falls to Ombuds clients; please refer to [mobile architecutre](/mobile-arch) for insight to how the Ombuds team implemented this computation.


Scripts
-------
### Pay to Public Key Hash (P2PKH)

    Introduced : November XX, 2015
    Block #    : XXXXXX
    Block hash : 00000000000000cf6044ece29281718edad566543bb29f56889625481071f303 

* Encoding:
The combined data (serialized Ombuds identifier + serialized wirerecord) can be divided into 20-byte chunks and placed within sequential transaction output scripts.
Each chunk of data will reside where the `PubKeyHash` normally lives.
The first transaction output (index 0) must contain the first chunk of data. 
In other words, transaction output with index zero will hold the eight byte `"BRETHREN"` identifier followed by the first 12 bytes of the serialized wirerecord.
The last transaction output will contain zeros following the wirerecord data.

* Constraints: to be considered standard, a transaction must be less than 100,000 bytes.

Likelihood of block inclusion: High
:   Considered the "standard of standard transactinos", the pay to public key hash script is the most common locking script. 

Monetary cost: High
:   Requires a dust value 

Prunibility: None
:   blah blah blah

*Example:*

    BRETHREN   : 425245544852454E
    wirerecord : 0a0c48656c6c6f20776f726c642110abc3e8b105
    -----------------------------------------------------

    TX OUTPUT (record encoded)
    index  : 0
    value  : xxx
    script : 76a914425245544852454e0a0c48656c6c6f20776f726c88ac 

    TX OUTPUT (record encoded)
    index  : 1
    value  : xxx
    script : 76a914642110abc3e8b10500000000000000000000000088ac

    TX OUTPUT
    index  : 2
    value  : change
    script : 76a9149ad31da6187eda450d6bd215b80bcd7aa1f2b83688ac

    -----------------------------------------------------
    
    Inspecting tx out, index 0:
    76 a9 14 425245544852454e 0a0c48656c6c6f20776f726c 88 ac
    || || || |--------------| |----------------------| || ||
    1  2  3  BRETHREN         wirerecord 1/2           4  5 

    Inspecting tx out, index 1:
    76 a9 14 642110abc3e8b105 000000000000000000000000 88 ac
    || || || |--------------| |----------------------| || ||
    1  2  3  wirerecord 2/2   null data                4  5

    KEY
    1 : OP_DUP
    2 : OP_HASH160
    3 : push 20 bytes
    4 : OP_EQUALVERIFY
    5 : OP_CHECKSIG


### Null Data

    Introduced : November XX, 2015
    Block #    : XXXXXX
    Block hash : 00000000000000cf6044ece29281718edad566543bb29f56889625481071f303 

* Encoding: When the combined data (serialized Ombuds identifier + serialized wirerecord) is less than 80 bytes in size, it can be encoded entirely within a null script. To produce this script: add OP_RETURN, add size of data, add combined data.

* Constraints: got to be less than 80 bytes 

Likelihood of block inclusion: Medium
:   Considered the "standard of standard transactinos", the pay to public key hash script is the most common locking script. 

Monetary cost: Medium
:   Requires a dust value 

Prunibility: Good
:   blah blah blah

*Example:*

    BRETHREN   : 425245544852454E
    wirerecord : 0a0c48656c6c6f20776f726c642110abc3e8b105
    -----------------------------------------------------

    TX OUTPUT (record encoded)
    index  : 0
    value  : xxx
    script : 6a1c425245544852454e0a0c48656c6c6f20776f726c642110f4c2e8b105 

    TX OUTPUT
    index  : 1
    value  : change
    script : 76a9143e424382248e1499ab35e027ca08472a28a1e88b88a

    -----------------------------------------------------
    
    Inspecting tx out, index 0:
    6a 1c 425245544852454e 0a0c48656c6c6f20776f726c642110f4c2e8b105
    || || |--------------| |--------------------------------------|
    1  2  BRETHREN         wirerecord

    KEY
    1 : OP_RETURN
    2 : push 28 bytes



------
Encoding:
A serialized wirerecord can be divided into 20-bytes chunks and placed within sequential transaction output scripts. The first output (output index 0) must contain the 

hard definition of article, however, will make no recomendation about output values.
Instead, please refer to [mobile architecture](/mobile-arch) to see how the Ombuds project calculates output values.

Defined in this document efines how a serialized Ombuds record ([bulletin](/bulletin) and [endorsement](/endorsement)) can be stored within a Bitcoin transaction.

I have a valid, serialized Ombuds protobuf -- how can I place it within a Bitcoin transactions such that it is recognized by an Ombuds Full Relay?

Balance between encoding flexibility / options / propertries and maintain computational efficiency.

Several methods to store a serialized record within a Bitcoin transaction are accepted.
To signify a record is stored within a transaction, the Ombuds identifer utf-8 "BRETHREN" is serialized and prepended to the serialized wirerecord. 
one can only expect a full relay to view these. 
note, additional methods will be created. 
This article documents each encoding format.
Ombuds clients can only expect [bulletins](/bulletins) and [endorsements](/endorsements) encoded using a format defined in this article to be recognized by full relays.

Encoding formats have not been finalized. The introduction date of the encoding formats will be the relase date of the ombuds-specification.

The Ombuds project seeks to increase the resilience of speech on the Internet. To be considered "secure" the record must be within the public record. For this reason, our top priority when assessing a encoding format is the *likelyhood of block inclusion*.

The monetary cost of record storage is a close second in priority. An expensive encoding format reduces the accessibility of the Ombuds system.

This data is either a [bulletin](/bulletin) or an [endorsement](/endorsement).

We consider the implications our encoding formats have on the health of the network and block chain, but admit 
This article will refer to the combined identifier and serialized record as the "encoded record".

BRETHREN + WireBulletin (msg: Hello world! ; time: 12345678)
