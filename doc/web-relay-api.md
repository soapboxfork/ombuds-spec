<!-- title: Web Relay Spec -->

Web Relay API Design Description // DRAFT
--------------------------------

Overview
========
The goal of this document is to describe the intention behind and the details of a JSON API that exposes the public record over HTTP.
This document will cover the specific requirements of an open-source API that serves public statements stored in Bitcoin's block chain.
How this software is intended to be used and how it must function is covered within.

Status of This Document
=======================
This is a draft design that is subject to change. Further documentation of the API methods themselves can be found in the source code [here](https://github.com/soapboxsys/ombudslib).

Nick -- October 2015


Contents
========
    1. Definitions
    1. Introduction
    2. Requirements Overview
    3. Available Records
    3. Physical Infrastructure 
       1. Minimum Requirements
       2. Deployment Process
    4. Authenticating the Server
    5. Protecting Users
        1. Server Logging
        2. TOR hidden services
    6. DOS Resistance Tradeoffs
    7. Release Process and Updates

## Definitions

Throughout this document the pronoun "we" is occasionally used. "We" in this context means Alex Kuck and Nick Skelsey.


## Introduction

The API discussed here is the interface that mobile application and web based users of Ombuds will rely on for information stored in the public record.
While it is clear that operator's of APIs and web servers can act maliciously, for the sake of usability and scalability we have decided to build a centralized piece of core infrastructure. 

The goal here is to produce a compact and performant JSON API that is easy for many organizations and individuals to deploy. 
That being said, the first organization to deploy it will be Soapbox Systems.
Due to size of Bitcoin's block chain we cannot expect new participants in this system to fully join the Bitcoin network, just to have access to Ombuds.
To make our system more accessible, clients that cannot or would rather not bear the cost of deploying and maintaining there own network infrastructure can instead trust a third party to serve them relevant application data.

It is important to note, that the API discussed below is READ only. The only interactions with the system that are conducted through the API is the retireval of records stored in the public record. 
An SSV client can always verify that the statement is unmoddified and actually included in the record. 
However, an SSV client can not be sure of whether or not statements have been excluded from the record.
This problem along with methods to address it is discussed further in [DD02](/An-Auditable-API).

## Requirements Overview
In general the goal of this server application is that it runs on modern enterprise grade servers.
To that end it must support thousands of simulataneous clients while remaining:

0. Secure
1. Available
2. Performant
3. Auditable

`0. Secure`

Clearly, the API code must not introduce any remote code execution vulnerabilities or unintentional leaks of user or operator information.
The API server must also be resitant to DOS attacks, both from malicious peers on the Bitcoin network and SSV clients.
Finally, updates to API servers must be cryptographically authenticated.

3. Available Records

The API will serve three fundamental things: bulletins, endorsements and inclusion proofs.
It will also serve aggregated lists of these things. 
While the exact schema and available routes of theses requests is descibed elsewhere this section describes 

The schema of bulletins is described in appendix A of this [paper](https://getombuds.org/research).
The JSON format of endorsements is described in section 2.2 of [DD2](/DD2).
The JSON format of inclusion proofs is described in [DD3](/DD3).


4. API Routes and Responses
    
The api itself will provide the following routes to the user. 
The required fields are surrounded in brackets (eg. [ ]).
The return type follows after the arrow (==>).
The available query parameters further restrict behaivor of the API.

    Individual

    /bltn/[txid]        ==>     bulletin
    /endo/[txid]        ==>     endorsement

    Paginated

    /author/[address]   ==>     records       ; The records an author has produced.
    /tag/[name]         ==>     records       ; The records that contain a specific tag.
    /loc/[lat],[lon],[r]==>     bulletins     ; The bulletins generated within r km of (lat, lon)
    /block/[hash]       ==>     records       ; All of the records within a block.
    
    Available Query parameters

        before=[blk hash]
        after=[blk hash]
        limit=[num records]
        type=[type records]

    Aggregates                                ; See 4.0 for more detailed explanations

    /pop-tags          ==>      tags          ; The most popular tags within T of now.
    /most-endo         ==>      bulletins     ; The most endorsed bulletins within T
    /dense-locs        ==>      locations     ; The most active locations within T
    /dense-areas       ==>      records       ; The densest areas of the graph within T
    /new               ==>      records       ; New records in the public record.

    Available Query Parameters
    
        T=after=[blk hash]
        before=[blk hash]
        limit=[num records]

    Information

    /status            ==>      See section 5.1.1 
    /config            ==>      See section 5.1.2

### 4. Aggregated Responses

`/pop-tags` returns the most commonly used tags in the last T blocks in a list. 
The schema looks like this:

    {
        tags: ["#bankcorp", "#homs", "#Jan25", "#utopia", "#gtown"]
         
    }


### 4.1 Pagination

For requests that could return many records, if the limit param is not specified or is higher than the server's MAX_RETURN field then the server will paginate the list of results
Pagination is based on two things, the number of records that could be returned by the query and the blocks which the records are in.
A paginated query will return all records within consecutive blocks where the number of records returned does not exceed the max number of records it is allowed to return. 

This means that if records in blocks are arranged like in the example below and a query asks for all of the records, then only records in the first 3 blocks are returned. (If the limit is set to 20.)

    query is /new?limit=20

    blocks:       a320f <--- b79de <--- c3421 <--- d3725 <--- e2934
    Num records:   15         73         4          7          9 

The response then looks like this:

    {
        records: [ { txid: "d932e..." ...},
                   { txid: "b3f1a..." ...},
                   { txid: "f3f65..." ...},
                   ...
                 ],

        stop: "c3421",
        start: "e2934"
    }

It may be suprising to see, but if the client responds with another query asking for the new records before c3421 the number of results will exceed the user set 'limit'. 
This is because the API servers will return all results in a block if they must return some even if it means violating the user set limit.
This is done to keep the api queries simple not to be maniacal.
    
    query is /new?before=c3421&limit=5

    {
        records: [ { txid: "4832d..." ...}
                   { txid: "b422e..." ...}    
                   { txid: "g3943..." ...}
                   ... // And 69 more
            ],
        stop:  "b79d4",
        start: "b79d4"
    }    


## 5 Physical Infrastructure

A properly functioning API server will have to run two binaries: ombfullnode and ombwebapi.
`ombfullnode` is a modified full node implementation of Bitcoin, while `ombwebapi` serves the JSON API over https. 
This means that a single server must meet the minimum requirements to run a Bitcoin full node _and_ fulfill requests from SSV nodes.

## 5.1 Minimum Requirements
Be aware that this is not the recommended setup and it makes more sense to use two different machines. 
This way one machine can communicate solely with the Bitcoin network and build the public record and the other machine can respond to SSV nodes.
In any configuration of servers though, the requirements stated above still apply.


## 5.2 Deployment Process

There are two ways to set up a deployment of an API server. 
The first only requires the application binaries and a connection to the Bitcoin network.
This requires a full download and validation of the bitcoin block chain and the construction of a public record from it.

The second method relies on a trusted checkpoint or a friends application data to supply the initial block chain and public record.
While we don't endorse this method of setting up an API.
We will be distributing a bootstrap file for Bitcoin's Testnet, so that folks can test their deployments of the server API before running the real thing.
That bundle of bootstrap data will be released with new versions of the software.
The process is decribed further in [section 7.2]().

## 6.1 Protecting Users

    `Honest servers do not keep logs.`

    `Available Over Tor`


