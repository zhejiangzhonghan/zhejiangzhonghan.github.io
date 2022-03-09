---
layout: post
title: "Distrubute locker with Container "
date: 2016-11-17 10:00:00
comments: true
keywords: "service architecture design"
categories:
- architecture
- design

tags:
- design 
---

One common issue with distrubuted locker is that if the process die somehow, who/how/when should we release the lock.

A noraml approach will be have a timeout and if the process doesn't hand back the lock within a period of time, the serivce just thanks that the process is dead.

But it is not always the case: for some long running operation, it could take a while. 

One way to fix it is having the process update the lock service every few seconds to renew the lock. But it introduces additional cost for the communication.

### With Container

With kubernete and container, now there is another approach:
- progress grab a lock from service and the service remember the spot-container_name as the owner of the lock
- service can talk to Kubernete and check whether the container is still alive. 
- if it no longer exists, just release the lock
