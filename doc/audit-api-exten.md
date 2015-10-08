<!-- title: Audit Extension -->

An Auditable Public API Extension // DRAFT
---------------------------------

Overview
========
The goal of this document is to describe the auditing process and the relevant aspects of a system that verifies that a web server is faithfully serving public statements included in a block chain. 
The reference implementation along with demonstrative test cases of the API will be available [here](https://github.com/soapboxsys/ombudslib).


The goal of providing public auditing is to give clients that consume a third party's block chain data some autonomny in judging the trustworthyness of that third party. 
The tangible goal is to remove some of the attacks a malicious API server can mount against a client.
While not providing anonymity, this extension lets clients prove to themselves and others that the API server is not manipulating records.

Status of This Document
=======================
This is the initial discussion of a design that is subject to change. 
While this is not an RFC, the intention behind this document is to describe the extension completely so that it is reproducible and well understood.

Nick -- September 2015

Contents
========
1. Definitions
2. Requirements
    1. Server Descriptions
    2. Client Descriptions
3. Signatures
4. Inclusion Proofs
5. Managing Certificates
6. Disclaimer

### Definitions

Throughout this document the term `record` is used regularly. 
A record defined here is a single public statement that has been included in the block chain.
As of September 2015, the Ombuds protocol defines two types of records: bulletins and endorsements.
The schemas of these records and their encodings can be found in [section 2.7.4 record formats of DD01](https://github.com/soapboxsys/ombudslib).


### Introduction
A simple read-only HTTP JSON API server produces responses that contain data to clients that request it.
In this traditional setup, the client is dependent on the server honestly sending data.
If the server operator has malicous intent, he can manipulate existing data, fake the existence of data, or withhold data.
While the auditing system discussed below cannot hold an API server accountable for witholding data, its explicit purpose is to prevent an API server from faking or systematically changing the content of records stored in the public record.


### Requirements

To prevent the manipulation of data  from occuring on servers running Ombud's API, it is nessecary to require three things of API servers and four things of API clients.

#### API Servers MUST:
1. Provide signed responses to all requests.
2. Provide proofs of inclusion for all atomic records in its database.
3. Respond on a best effort basis to all requests from API clients.

#### API Clients MUST:
1. Select unpredictable challenges.
2. Regularly audit the API server.
3. Publish failed audits to peers.
4. Stop using the API server if it misbehaves or fails an audit.

If an API server and a group of Ombuds clients obey these requirements, then the clients can collectively rely on a single API server to faithfully provide them with accurate information. 
So long as the Ombuds clients can communicate amongst each other they will be able to adapt to a changing landscape of third parties of unknown reliablity.

In brief, the audit process works as follows: 

- An API client connects to the server and starts requesting records. 
- The client randomly selects a set of individual requests and records from the session.
- The client then checks the proofs of inclusion for each record it selected. 
    - If any of the proof fails in any way, the client publishes evidence of the failure to its peers and it stops trusting the server.
- If a client recieves a failed audit from a peer, it stops trusting the server.

Thus, with many active auditors the likelihood that a server can falsify or manipulate any record becomes extremely low.

#### Server Requirements Descriptions

```
1. Provide signed responses to all requests.
```

It is nessecary to sign every request from the server for the same reason https only works for a site if all traffic to the site is forced into https.
An attacker can selectively manipulate unsigned responses to specific API clients while those clients would not be able to discredit the API server.

The format of the signatures used is discussed in Section 6: [Signatures](#Signatures).

```
2. Provide proofs of inclusion for all atomic records in its database.
```

The server must provide the merkle path or an equivalent for each record it returns. 
This lets a client verify that any record requested is unmodified and placed within the block chain.


```
3. Respond on a best effort basis to all requests from API clients.
```

A malicious server could selectively respond to requests and simply return HTTP errors for things it does not want a client to view.
This means that an honest server must stay available and attempt to respond to every client request it gets.

Special care must be taken to balance DOS mitigation and valid challenges as an attacker could requests proofs only to discard them
in an attempt to waste server resources. 

#### Client Requirements Descriptions
```
1. Select unpredictable challenges.
```

If a malicious API server can predict the records the client is going to audit then it can lie through omission easily.

```
2. Regularly audit the API server.
```

If a client does not check the proofs the server is delivering then there is no point in sending them.

```
3. Publish failed audits to peers.
```

This scheme relies on a sort of herd immunity to prevent any one client from being lied to. 
If a client does not communicate or cannot communicate a failed proof to its peers then the entire herd is at risk.
Thus, it is imperative that peers remain in constant communication and that they quickly inform each other of malacious servers.


```
4. Stop using the API server if it misbehaves or fails an audit.
```

To function properly, clients should have next to no forgiveness for unresponsive API servers.
They should have __no trust__ in a server that fails an audit.

### Signatures

This scheme requires API servers sign all of their json responses according to the JWS standard described in [RFC 7515](https://tools.ietf.org/html/rfc7515).
This requires wrapping the server's JSON responses specified in [section 7.3 of DD01]() with the required JWS fields.

Each JSON response returned by the server MUST include a JWS message signature.

    {
        "payload": {
            // A standard bulletin
            author: "3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy",
            message: "I cant be undecided all of the time.",
            timestamp: 13040032,
            lat: 37043930,
            lon: 42093296
        },

        "protected": { 
            // JOSE Headers
            "x5t#S256": "34ed032...", 
            "alg": "HS512"
            // Need a timestamp that is signed.
        },
        "signature": "<signature N contents>"
    }

The deployment of JWS here will only include the use of these protected headers.
Special care must be taken to prevent the introduction of timing side-channel attacks.


### Inclusion Proofs

Each individual record can be provably demonstrated to be unchanged and included within the block chain.
Every record is contained within a Bitcoin transaction that has a unique hash based on double sha256, which is called the transaction id or 'txid' for short.

An inclusion proof is a merkle audit path that demonstrates that the txid is included within the merkle root of a block in the block chain.

#### Constructing and Validating a Proof

The exact format of the leaves and branch digests needed to construct the path is specified in [BIP 37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki#partial-merkle-branch-format).

The nessecary fields are then included in an individual transactions response are shown below in the example. 
The required JWS attributes are omitted from the example for clarity. 

    {
        // Fields required for audits
        "rawtx":    "01000000001344630cdbff6b...",
        "blk":      "0000000000000a3614515c3d...",
        "mpath": { 
            "branches": ["d334..." "eda4..." "3f92..."], 
            "flags": 10010100 
        }
    }

### Auditing a Record

The process a client takes for formally auditing a single record relayed by an API server is then specified below.
If at any point in the process one of the client checks fails, the client can use that as proof that the audit failed.

- The client requests the proof of inclusion for a specific record the API has previously served it.
- The server responds with a signed proof of inclusion and the raw tx.
- The client checks that the server has attached a valid signature to the audit
- The client reconstructs the record from the raw transaction.
    - It checks that the txid matches the hash of the raw tx.
    - It checks that all of the fields of the old record match the provided json.
    - It checks that the authors signature is valid.
- The client checks the proof of inclusion against its block headers.

### Distributing Failed Audits

If an audit has failed as per the requirements stated above an audit must be distributed to peers.
At this time it is not clear how this will be accomplished. 
Specifically it is not clear how peers to a specific API should discover each other and communicate.
Before the system described in this document is implemented that will be described.


### Managing Keys

The reference implementation will be using the standard defined in [RFC 6125](https://tools.ietf.org/html/rfc6125) where trusted signing keys are bundled in the client application and the user has the ability to import new ones and discard malicious keys.

The application will not trust a devices standard trusted certifcates and instead rely on the user explicitly importing new trusted keys.

### Disclaimers

It is important to dispell incorrect ideas about what this design makes possible.
This is not a complete list.

```
An API server cannot refuse to sign audits.
```

If an API server refuses to sign requests for a record that it has previously lied about, then the client must consider the failure malicious.
If the API server only refuses to sign audits to specific clients, then that client does not have a portable proof that the API lied to it.
To counter this attack, the client must present the suspected record to other clients to see if they can audit the server.
If they cannot then both clients should distrust the sever and actively inform other clients of the failure.


```
An API server cannot omit public records.
```

Detecting the exclusion of records is much harder then the manipulation of a regular.
With a record and a proof of inclusion, a client can quickly determine if the record is accurate and included within the public record.
If a malacious API omits a record, it will only appear as a signed response where the requested record does not exist or was omitted from an aggreagated response.
An SSV client running validation checks will not detect that it has been lied to.

```
An API server only needs a few clients to audit it.
```

The rate of challenges an API server must recieve is proportional to the size of the record. 
If the API server receives too few challenges then it can probabilistically get away with falsifying and omitting records.
Thus, if the size of the record grows so too must the number of auditors challenging the server.

```
API clients only need to check the proofs of records they are interested in.
```

It is possible and very likely that API servers will lie through omission, if they do not recieve enough random requests.
This is because a client cannot prove a record was left out of a response unless it has evidence of that record existing.
