---
layout: post
title: Design Pattern for container
date: 2016-09-15 10:00:00
comments: true
keywords: "service architecture design container"
categories:
- architecture
- design
- container

tags:
- design pattern 
- container
---
Notes from this [Design patterns for container-based distributed systems](https://www.usenix.org/system/files/conference/hotcloud16/hotcloud16_burns.pdf)

A few interesting ideas and patterns for container based system learns from reading the article above.

## Single-container management patterns

### Upward

Have http-based (separate port, maybe Json format) for data transition web server to expose rich informations:

- monitoring metrics
- profiling information
- component logs

### Downward

Add lifecycle APIs. 

- Instead of just killing the container, inform the container the shutdown signal and give container a chance to save information.

## Single-node, multi-container application patterns

### Sidecar pattern

Have a separate container running along with main container to extend/enhance the main container. 

#### Example

- have a log container collecting logs
- have a container which populate local disk with git/CMS/data from server

#### Benifites

- main container can have priority over sidecar container and some times, sidecar container can be stopped to free resources
- sidecar container is portable and can be reused
- allow two container to be updated independently

### Ambassador pattern

Provide proxy communication to and from a main container.

#### Example

App that is that is speaking the memcache protocol with a twemproxy ambassador.

#### Benifites

- presents an application with a simplified view of the outside world
- it simplifies the app design as the app is just talking to a server on local host
- app and the ambassador proxy can be in different language

### Adapter pattern

 Standardizing output and interfaces across multiple containers. Present the outside world with a simplified, homogenized view of an application.

#### Example

- Adapters that ensure all containers in a system have the same monitoring interface



## Multi-node application patterns

### Leader election pattern

A set of leader-election containers, each one co-scheduled with an instance of the application that requires leader election, can perform election amongst themselves, and they can present a simplified HTTP API over localhost to each application container that requires leader election (e.g. becomeLeader, renewLeadership, etc.). 

These leader election containers can be built once, by experts in this complicated area, and then the subsequent simplified interface can be re-used by application developers regardless of their choice of implementation
language

### Work queue pattern

- Use container run() and mount() interfaces
- build a container that can take an input data file on the filesystem, and transform it to an output file; this container would become one stage of the work queue.

### Scatter/gather pattern

- an external client sends an initial request to a “root” or “parent” node. 
- This root fans the request out to a large number of servers to perform computations in parallel. 
- Each shard returns partial data, and the root gathers this data into a single response to the original request.

