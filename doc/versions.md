<!-- title: Versioning -->

Design Document Versioning // DRAFT
--------------------------

The documents contained on specify intended program behaivor, because the documents themselves are changing things themselves it is useful to label them.
The label scheme is simple. Each document proceeds in order from Draft --> Revision --> Implemented. 

What follows below is the criterea for what you can expect from a document labeled in these ways.

### Draft
Nothing in a draft is final or an official decision or spell checked. 
A draft represents a rough take on a specific design and should not be relied upon.
The status line that documents will carry follows below.

    While this is not an RFC, the intention behind this document is to describe the system completely so that it is reproducible and well understood.
    Please note this is a **draft of a design** that is subject to change. 
    Do not rely the content of this document for critical decisions. 
    Information contained within may be inaccurate.
    It has not been reviewed.
    
    [Author] -- [Month Year]

### Revision
A revision is a work in progress that has been reviewed at least once by someone other than the original author.
A revision will be versioned in numerically increasing order and document changes if the document is popular enough to warrant that disscussion.
A revision will carry a **Status of this Document** section that is co-authored by the reviewer and the author.

### Implemented
A document that is implemented is running in live code. 
This document can be understood as the intended functionality of the system.
Any aspects of the system that do not conform to an implemented design doc are failures in the implementation.
Like Revisions implementation documents can be versioned in numerically increasing order.
An implemented document will carry: 

- a **Status of this Document** section that is written by the author of the code.
- a link to a tagged release of the code that implements the document.

