<!-- title: Web Relay Spec -->

Web Relay API Design Description // DRAFT
================================

Overview
--------
The goal of this document is to describe the intention behind and the details of a JSON API that exposes the public record over HTTPS.
This document will cover the specific requirements of an open-source API that serves public statements stored in Bitcoin's block chain.
How this software is intended to be used and how it must function is also covered within.

##Status of This Document
While this is not an RFC, the intention behind this document is to describe the system completely so that it is reproducible and well understood.
Please note this is a **draft of a design** that is subject to change. 
Do not rely the content of this document for any important decisions.
It has not been reviewed.

Nick -- Oct 2015


##Contents
    1. Definitions
    2. Introduction
    3. Requirements Overview
    4. Available Records
    5. API Routes and Responses
      5.1 Individual Routes
      5.2 Paginated Routes
      5.3 Aggregated Routes
      5.4 Informational Routes
    6. Physical Deployment
      6.1 Minimum Requirements
      6.2 Deployment Process
      6.3 Protecting Users
    
## 1. Definitions

Throughout this document the pronoun `we` is occasionally used. "We" in this context means Alex Kuck and Nick Skelsey.
A `web relay` is the noun used to describe a single deployment of the API described below.
It does not nessecarily refer to a single API server, but it can be conceptually thought of as one machine reachable from the rest of the Internet.


## 2. Introduction

The API discussed here is the interface that mobile application and web based users of Ombuds will rely on for information stored in the public record.
While it is clear that operators of APIs and web servers can act maliciously, for the sake of usability and scalability this web API functions as a centralized piece of core infrastructure. 

The goal of this document is to describe a compact and performant JSON API that is easy for organizations and individuals to deploy. 
With that said, the first organization to deploy it will be Soapbox Systems.
Due to the size of Bitcoin's block chain it is unreasonable to expect new participants to fully join the Bitcoin network just to have access to Ombuds.
Thus, to make Ombuds more accessible clients that cannot or would not bear the cost of deploying and maintaining there own network infrastructure can instead trust a third party to serve them application data.

It is important to note, that the API discussed below is **read** only.
The only interactions with the system that are conducted through the API is the retireval of records stored in the public record. 

## 3. Requirements Overview
In general the goal of this server application is that it runs on modern enterprise grade servers.
To that end it must support thousands of simulataneous clients while remaining:

- Secure
- Available
- Performant
- Auditable (planned & described in [DD04](/audit-api)

## 4. Available Records

The API serves two fundamental things: bulletins and endorsements.
It also serve aggregated lists of these things, along with simple data-types like locations and tags.
While the exact schema and available routes of theses requests is completely described in the code, any difference between the description of data types here and there implementation should be reported as a bug.

The complete schema of JSON responses will reside in the golang implmentation [on Github](https://github.com/soapboxsys/ombudslib).

## 5. API Routes and Responses
    
All of the API routes the server exposes are documented below.
The required fields are surrounded in brackets (eg. [ ]).
The return type follows after the arrow (==>).
The available query parameters further restrict behaivor of the API.

    Individual

    /endo/[txid]        ==>     endorsement   ; A single endorsement
    /bltn/[txid]        ==>     bulletin      ; A single bulletin with all of its endorsements
    /block/[hash]       ==>     records       ; All of the records within a block.

    Paginated

    /author/[address]    ==>     records       ; The records an author has produced.
    /tag/[name]          ==>     records       ; The records that contain a specific tag.
    /loc/[lat],[lon],[r] ==>     bulletins     ; The bulletins generated within r km of (lat, lon)
    /range               ==>     records       ; All of the records between ?before and ?after.
    
    Available Query parameters

        before=[blk hash]
        after=[blk hash]
        limit=[num records]
        type=[type records]

    Aggregates

    /pop-tags          ==>      tags          ; The most popular tags within T of now.
    /most-endo         ==>      bulletins     ; The most endorsed bulletins within T
    /dense-locs        ==>      locations     ; The most active locations within T
    /dense-vertices    ==>      records       ; The densest areas of the graph within T
    /new               ==>      records       ; New records in the public record.

    Available Query Parameters
    
        T=after=[blk hash]
        before=[blk hash]
        limit=[num records]

    Information

    /status            ==>     object       ; See section 5.4
    /config            ==>     object       ; See section 5.4

### 5.1 Individual Routes

For queries that request single elements the schema is straight forward.

`/endo/txid]` returns a single endorsement that conforms to [section 3a of DD03](/endorsement).

`/bltn/[txid]` returns a bulletin conforming to the schema described in section [2.5 JSON Schema of DD02](/bulletin) with one additional field.
That field is the list of endorsements for that bulletin.

    {
        txid: "74cffe12b2d58ac...",
        author: "mn348fu23aG23...",
        message: "I am four on the floor and I'm happy",
        ...
        ...
        ...
        endorsements: [
            {
                txid: "8cee5dce2fa177...",
                bid:  "74ffe12b2d58ac...",
                ...
            },
            {
                txid: "3f293491823cab...",
                bid:  "4cffe12b2d58ac...",
                ...
            },
        ]
    }

`/block/[hash]` returns all of the records with a block.

    {
        bltns: bulletins,
        endos: endorsements
    }

### 5.2 Paginated Routes

For requests that can return many records, if the limit param is not specified or is higher than the server's MAX_RETURN field then the server will paginate the list of results.
Pagination is based on two things, the number of records that could be returned by the query and the blocks which the records are in.
A paginated query will return all records within consecutive blocks where the number of records returned does not exceed the max number of records the server is allowed to return. 

This means that if records in blocks are arranged like in the example below and a query asks for all of the records, then only records in the first 3 blocks are returned -- if the limit is set to 20.

    query is /new?limit=20

    blocks:       a320f <--- b79de <--- c3421 <--- d3725 <--- e2934
    Num records:   15         73         4          7          9 

The response then looks like this:

    {
        stop: "c3421",
        start: "e2934"
        records: [ { txid: "d932e..." ...},
                   { txid: "b3f1a..." ...},
                   { txid: "f3f65..." ...},
                   ...
                 ],

    }

It may be suprising to see, but if the client responds with another query asking for the new records before block `c3421` the number of results will exceed the user set 'limit'. 
This is because a web relay will return **all** of the results in a block if it must return some **even if** it means violating the MAX_RETURN limit.
This is done to keep the API queries simple, not to be maniacal.
    
    query is /new?before=c3421&limit=5

    {
        stop:  "b79d4",
        start: "b79d4"
        records: [ { txid: "4832d..." ...}
                   { txid: "b422e..." ...}    
                   { txid: "g3943..." ...}
                   ... // And 69 more
            ],
    }    

### 5.3 Aggregated Routes

Ony the unusual responses are documented here.

`/pop-tags` returns the most commonly used tags in the last T blocks in a list. 
The schema looks like this:

    {
        "tags": ["#bankcorp", "#homs", "#Jan25", "#utopia", "#gtown"]
    }

`/dense-locs` returns the most active locations with the last T blocks.

    {
        "locations": ["72.2323,-43.3222,5", "34.2323,43.5891,4", ...]
    }

### 5.4 Informational Routes

The last section of API methods provides information about the web relay itself.
These routes should be used by clients to determine if the API supports their version and use case.
The JSON fields are documented below.

`/config` returns the publicly available configuration details that the API is willing to share.
    
Schema:

    {
        apiVersion:   [semantic-version]    ; The API's version
        btcdVersion:  [semantic-version]    ; The trusted node's version
        hs-addr:      [hidden-service-addr] ; Optional TOR HS address
        maxReturn:    [int]                 ; The max num of records the server will return.
    }

Example: 

    {
        "apiVersion":  "0.2.1",
        "btcdVersion": "0.10.0-BETA",
        "hs-addr":     "493092esdfaefa.onion"
    }

`/status` returns the status of the web relay.

Schema:

    {
        bestblk:      [blk-hash]     ; Current chain tip
        uptime:       [secs]         ; Seconds since start
        timestamp:    [secs]         ; Unix time
        pubrecHash:   [rec-hash]     ; SHA3 hash of publice record.
    }


Example:

    {
        "bestblk":    "00000000054b84667...",
        "uptime":     53943,
        "timestamp":  123943321,
        "pubrecHash": "d35d83395a46b16e..."
    }



## 6. Physical Deployment

A properly functioning web relay will have to run two binaries: ombfullnode and ombwebapi.
`ombfullnode` is a modified full node implementation of Bitcoin, while `ombwebapi` serves the JSON API over https. 
This means that a single server must meet the minimum requirements to run a Bitcoin full node _and_ fulfill requests from SSV nodes.

Be aware that this is not the recommended setup and it makes more sense to use two different machines. 
This way one machine can communicate solely with the Bitcoin network and build the public record and the other machine can respond to SSV nodes.

### 6.1 Minimum Requirements

A server will generally not be able to support the application if it does not have at least:

- 1 GB of Ram.
- 4 2.5Ghz CPUs.
- 250 GB of disk storage.
- 30 GB per month of IO out.
- A 200 Mbits connection to the broader internet.

### 6.2 Deployment Process

There are two ways to set up a deployment of a web relay. 
The first only requires the application binaries and a connection to the Bitcoin network.
This requires a full download and validation of the bitcoin block chain and the construction of a public record from it.

The second method relies on a trusted checkpoint or a friends application data to supply the initial block chain and public record.
While we don't endorse this method of setting up an API.
We will be distributing a bootstrap file for Bitcoin's Testnet, so that folks can test their deployments of the server API before running the real thing.
That bundle of bootstrap data will be released with new versions of the software.
The process is decribed further in [section 7.2]().

### 6.3 Protecting Users

A web relay operating in good faith should do all of the following things:

- Do not keep server logs.
- Setup a tor hidden service
- Use proper public key authentication for software updates and HTTPS

