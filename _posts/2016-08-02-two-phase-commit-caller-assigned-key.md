---
layout: post
title: "Two-phase commit and caller assigned key"
date: 2016-08-02 13:07:19
comments: true
keywords: "service architecture design"
categories:
- architecture

tags:
- architecture design
---
Come across two very interesting service design tops which I think is very useful. They are related to each other so I list them in one article. 

### Scenario
When a service/client need to make a call to another remote service. It can be creating a new object, update existing object, delete object, etc.

### Goal
- Track the progress 
- NO resource leak
- Easy system wise consolidation: which means we can easily clean up dead record and make sure different micro-service agree on one clean picture.

### Requirement
- Remote call needs to be **idempotent**

## Two-Phase Commit
Steps:
1. save a record about the object that the caller wants the remove service to handle
2. update the state of record to reflect that a remote call will be initiated
3. make the remote call
4. once response received from remote call, update the record to show whether the operation complete or not


|Operation|On-Call|On-Responce-Succeed|On-Responce-Failed|
|---|---|---|---|
|Create|create_requested|created|create_failed|
|Update|update_requested|updated|update_failed|
|Delete|delete_requested|deleted|delete_failed|

### What's the good points
Since we save the state before making the remote call, service will never lose track of the item. 
- If the service dies in any random time
- If the remote responses get lost in the middle

One can easily recover from the any given state. Since the remote call is **idempotent**, no bad thing will happen on retry.

## Caller Assigned Key
Caller Assigned Key can future help the two-phase commit on preventing resource leak. 

It requires that all remote resource be model like following:

|Column|Type|Description|
|---|---|---|
|object_key|str|caller assigned key|
|object_id|int|Internal Key within service|

When local service calls a remote service to create a new object, it first creates an object_key and passes it to the remote service. This will be the key to be referred to on all the future communications.

### What's the good points
Most service today just create an object and try to let the remote service hands back a key for future reference. The problem is that if the response gets lost, caller will never know that the resource has been actually allocated. So caller will give it another try and the first resource will be a leak in the system. There is no easy way to consolidate the system at this point since remote service doesn't know whether client still needs this objects.

With caller assigned key, even if the response is lost in the middle, the client will retry with same key and remote service will either create the item or realise it exist and just response ok. So there won't be any leaks around.

## Combine two-phase commit and caller assigned key
This will be the basic table to have both of the features:

|Column|Type|Description|
|---|---|---|
|key|str|caller assigned key|
|state|str|current state of the resource|
type|str|resource type, for Polymorphism|
