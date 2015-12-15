<!-- title: Record Encoding Formats -->

DD04 Record Encoding Formats
============================

Overview
--------
Several encoding formats to store a serialized record within a Bitcoin transaction are accepted.
Only the encoding formats defined in this article are recognized as valid ways to append a [bulletin](/bulletin) or [endorsement](/endorsement) to the [public record](/public-record).
Each format has an introduction date and is not a valid encoding format before that date.
New encoding formats will likely be introduced.


Status of this Document
-----------------------

Encoding examples are not up to date and contain inaccurate information.
The introduction date of the encoding formats will be the relase date of the Ombuds specification.

Alex, Nick -- Dec 15, 2015


Contents
--------
1. Assumptions 
2. Criteria
3. Ombuds Header
3. Formats


Assumptions
-----------
This article assumes:

* a **valid [wirerecord type](https://github.com/soapboxsys/ombudslib/blob/master/protocol/wirerecord/types.proto)** is ready for encoding.
* the Bitcoin transaction inputs are controlled by the sender and hold enough coin to fund the new transaction.


Criteria
--------
To assess each encoding format, three criterion are used:

The likelyhood a record will be included in a block.
:   When records are stored at a sufficient depth in the block chain, these records are considered
    "public" meaning they are almost impossible to censor.
    As a derivative of the block chain, the public record is comprised of the records within mined blocks.
    It is __extremly important__ that records are selected by miner's for inclusion within their blocks.
    When considering each encoding format, the likelyhood of block inclusion is the top priority.

The cost of storing a record.
:   To broadcast any record, an author must spend bitcoin; while this cost is small, it is a non-negligible amount. 
    In Ombuds speech is not free in terms of cost, but it is free in terms of content.

The prunability of a record.
:   A growing bitcoin network provides Ombuds with a reliable and secure data store.
    If the network is a distrbuted, sustainbale entity, the long-term viability of dependent systems is possible.
    The impact an encoding format has on the network and block chain sustainability is thus crucial.


Ombuds Header
=============
The Ombuds Header indicates the presence, the type, and the length of an Ombuds Record. In every encoding format, the header must precede the serialized wirerecord.

    Ombuds Header
    6 byte : Record Flag 
    1 byte : Record Type
    varint : Record Length
           +
    Ombuds Record
    < 75KB : [serialized wirerecord]


Record Flag
-----------
The Record Flag is the serialized UTF-8 encoded string `"OMBUDS"`.

    Record Flag
    UTF-8 string : OMBUDS
    byte array   : [0x4f, 0x4d, 0x42, 0x55, 0x44, 0x53]


Record Type
-----------
The Record Type is either `1` for a bulletin or `2` for an endorsement. 
The record is invalid if this type does not correspond to the serialized wirerecord type.

    Record Type
    byte : 0x01 or 0x02
     

Record Length
-------------
The Record Length describes the byte length of the subsequent serialized wirerecord. 
It is encoded using Satoshi's variable-length unsigned integer standard, known as [CompactSize](https://bitcoin.org/en/glossary/compactsize).
See [btcsuite](https://github.com/btcsuite/btcd/blob/master/wire/common.go#L391-L418) or [bitcoinj](https://github.com/bitcoinj/bitcoinj/blob/master/core/src/main/java/org/bitcoinj/core/VarInt.java) for reference implementations.

The Record Length can be no larger than 75,000 kilobytes, the maximum size of an Ombuds Record.
If the Record Length does not correspond to the actual size of the subsequent wirerecord, the record is invalid.

    Record Length
    byte array : [0x??] through [0x124F8] , inclusive of end values.
    


Formats
=======

Value
-----
Allocating an "acceptable" amount of bitcoin for each transaction output increases the likelyhood of transaction propogation and block inclusion.
However, this specification will **not** provide a hard definition for calculating output values.
Instead, the responsibility of value calculation falls to Ombuds clients; please refer to [mobile architecture](/mobile-arch) to see how the reference implementation approaches this.


Scripts
-------
### Pay to Public Key Hash (P2PKH)

    Introduced : November XX, 2015
    Block #    : XXXXXX
    Block hash : 00000000000000cf6044ece29281718edad566543bb29f56889625481071f303 

* Encoding:
The combined data (Ombuds Header + serialized wirerecord) can be divided into 20-byte chunks and placed within sequential transaction output scripts.
Each chunk of data will reside where the `PubKeyHash` normally lives.
The first transaction output (index 0) must contain the first chunk of data. 
In other words, transaction output with index zero will hold the Ombuds Header followed by the first 12 bytes of the serialized wirerecord.
The last transaction output will contain zeros following the serialized wirerecord.

* Constraints
    - To be considered standard, a transaction must be less than 100,000 bytes.
    - Every transaction output must pay the current network's dust value.
        

`Likelihood of block inclusion: High`
:   The pay to public key hash script is the most common locking script. 
    Records encoded in this way will for the most part "blend in."

`Monetary cost: High`
:   This method of encoding has a low encoding efficieny and burns coins.

`Prunable: No`
:   Encoding records in `P2PKH` means that the transaction can never be removed from the unconfirmed transaction set under the current 'consensus rules'.
    This is not good for the health of Bitcoin and will probably change.

*Example:*
    
    JSON            : {"message":"Hello, world!","timestamp":12345678}
    Header          : 4f 4d 42 55 44 53 01 14
    Wirerecord      : 0a 0d 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 10 ce c2 f1 05
    ------------------------------------------------------

    *** Below this line is broken ***
    TX OUTPUT (record encoded)
    index  : 0
    value  : 567 (dust)
    script : 76a914425245544852454e0a0c48656c6c6f20776f726c88ac 

    TX OUTPUT (record encoded)
    index  : 1
    value  : 567 (dust)
    script : 76a914642110abc3e8b10500000000000000000000000088ac

    TX OUTPUT
    index  : 2
    value  : change
    script : 76a9149ad31da6187eda450d6bd215b80bcd7aa1f2b83688ac

    -----------------------------------------------------
    
    Inspecting tx out, index 0:
    76 a9 14 425245544852454e 0a0c48656c6c6f20776f726c 88 ac
    |  |  |  |--------------| |----------------------| |  |
    1  2  3   BRETHREN         wirerecord 1/2          4  5 

    Inspecting tx out, index 1:
    76 a9 14 642110abc3e8b105 000000000000000000000000 88 ac
    |  |  |  |--------------| |----------------------| |  | 
    1  2  3   wirerecord 2/2   null data               4  5

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

* Encoding: When the combined data (Ombuds Header + serialized wirerecord) is less than 80 bytes in size, it can be encoded entirely within a null script. To produce this script: add OP_RETURN, add size of data, add combined data.

* Constraints: The entire encoded payload must be less than 80 bytes.

`Likelihood of block inclusion: Medium`
:   Only a subset of Bitcoin miners will include transactions that use OP_RETURN in their output scripts.
    That might change in the future.

`Monetary cost: Low`
:   While the transaction will have a low priority in BitcoinCores case, the tx must only pay a mining fee and nothing else.

`Prunable: Yes`
:   OP_RETURN outputs where designed to be pruned from the regular ledger.

*Example:*

    JSON            : {"message":"Hello, world!","timestamp":12345678}
    Header          : 4f 4d 42 55 44 53 01 14
    Wirerecord      : 0a 0d 48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 21 10 ce c2 f1 05
    -----------------------------------------------------

    *** Below this line is broken ***
    TX OUTPUT (record encoded)
    index  : 0
    value  : 0
    script : 6a1c425245544852454e0a0c48656c6c6f20776f726c642110f4c2e8b105 

    TX OUTPUT
    index  : 1
    value  : change
    script : 76a9143e424382248e1499ab35e027ca08472a28a1e88b88a

    -----------------------------------------------------
    
    Inspecting tx out, index 0:
    6a 1c 425245544852454e 0a0c48656c6c6f20776f726c642110f4c2e8b105
    |  |  |--------------| |--------------------------------------|
    1  2   BRETHREN         wirerecord

    KEY
    1 : OP_RETURN
    2 : push 28 bytes

