---
layout: post
title: "Stock Tracer Design"
date: 2016-07-15 15:12:36
comments: true
description: "Stock Tracer Design"
keywords: "service, design, container"
categories:
- development

tags:
- python
- mysql

---

## Overview
- [github](https://github.com/ning-yang/stock_tracer)
- [demo](http://stock.ningy.me)

Personal Project to track history data for stocks. Including:
- Frontend UI: dislapy quotes table and allow adding new stocks
- Backed Service: serving UI api query
- Woker: refresh quotes data 

## language framework
- python
- flask
- sqlalchemy
- jinja2
- rabbit MQ
- mysql 

## DevOps Stack
- github
- travis-ci
- docker hub 

## Production Stack
- Azure
- docker
- elastic search
- kibana

## Overall System
![test link](/assets/images/stock_tracer.png)
