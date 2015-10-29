<!-- title: Author -->

DD02 Author Explanation // DRAFT
=======================

Overview
--------
The author field is used in several places in Ombuds. 
Namely, it is used in bulletins and endorsements to specify the 'identity' of the records creator. 
Using the term author here is imprecise, but it helps regular folks understand what is going on.
For technical purposes the author field is filled by the bitcoin address that corresponds to the first public key that signed the Bitcoin transaction which contains the record.

Any record created in Ombuds must have a signature over its contents and that signature has to a conform to a strict format.
Otherwise, the whole bulletin will be discarded because the signature was invalid.


Valid Signature Format
----------------------

There is only one valid format that is used to assign the author of a record.
A record's first transaction input (short: *TxIn*) must contain a Script Sig that conforms to the standard pay to public key hash format.
This means that the OP_CHECKSIG included in the referenced Transaction Output, MUST be flagged as SIG_HASH_ALL or SIG_HASH_ANYONECANPAY

The author is then assigned to the public key hash in Base58Check notation.


Canonical Example
-----------------

Notice that the first signing address in this transaction is also the author of the bulletin contained within.

####Bulletin 
With irrelevant fields elided with ...

    {
        "txid" : "5a57ebcba9a52f770887ea1a4d039a67cf252b6ec25630ff4f42d09f0398faca",
        "author" : "n25H1Lpt1YbKhu6SU8L8rZ4HdZjAAaPbZN",
        "msg" : "What breaks old systems is ...",
        ...
        ...
        ...
    }
    
####Containing Transaction 
With irrelevant fields elided with ...

    {
        "txid" : "5a57ebcba9a52f770887ea1a4d039a67cf252b6ec25630ff4f42d09f0398faca",
        "vin" : [
        {
            "isConfirmed" : true,
            "value" : 0.13465879,
            "vout" : 2,
            "scriptSig" : {
                "hex" : "483045022100bcc5e560e3a3aa3d0afe67876364e1ac3e060a4f86bac71f3afeebafc9eee75202203a2f79d81fd297c9fe6bc237ffc88f2d702dfe0a7580def45b2550fbe7da38dd012103fef28f248fdb54b95cb8c27e4d2eceedc47d2f8f19fdc45d00f9ae8a7ec42fe3"
            },
            "addr" : "n25H1Lpt1YbKhu6SU8L8rZ4HdZjAAaPbZN",
            "txid" : "ba1821ca0a253e6dbf0476f4c341a3b2c591c00e54065c01bc759cdcf630027d",
        },
        ...
        ...
        ...
    }
[Source](https://testnet.blockexplorer.com/api/tx/5a57ebcba9a52f770887ea1a4d039a67cf252b6ec25630ff4f42d09f0398faca)

