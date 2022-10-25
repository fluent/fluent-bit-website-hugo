---
title: 'How It Works'
description: 'Works for Logs and Metrics'
type: how-it-works
hero:
 heading: |
  Works for Logs and Metrics
 desc: |
  Fluent Bit was designed for speed, scale, and flexibility in a very lightweight, efficient package.
 image: "/images/overview-new.svg"
 mobileImage: "/images/sm-howitworks.svg"
 text: |
  Fluent Bit enables you to collect event data from any source, enrich it with filters, and send it to any destination.
howtoworks:
- text: >
   Fluent Bit can read from\
   
   local files and network devices, and can **scrape metrics in the Prometheus**\
   
   format from your server.
- text: > 
   All events are **automatically
   
   tagged** to determine filtering, 
   
   routing, parsing, modification 
   
   and output rules.
- text: > 
   Filters can modify data by calling

   **an API (E.g. Kubernetes),**\

   remove extraneous fields,\

   or add values.
- text: > 
   Built-in reliability means if you hit

   a network or server outage\

   **you will be able to resume**\

   from where you left off\

   without data loss.
- text: > 
   Fluent Bit can send data to a 
   
   **multitude of locations,**\

   including popular destinations\

   like Splunk, Elasticsearch,\
   
   OpenSearch, Kafka, and more.

btnAdvText: "Read documentation"
btnAdvUrl: "https://docs.fluentbit.io/manual"
linkNewTab: "true"
history:
  heading: "A Brief History of Fluent Bit"
  image: "/images/history.jpg"
  text: >
   In 2014, the Fluentd team at Treasure Data began to see the need for a more lightweight log processor to be used in resource-constrained environments like embedded Linux and gateways.  The objective was to **provide all the speed, scale, and flexibility** of Fluentd in a smaller, more efficient footprint. The result was Fluent Bit.


   While Fluent Bit did gain rapid adoption in embedded environments, its lightweight, efficient design also made it attractive to those working across the cloud.  Features to support more inputs, filters, and outputs were added, and Fluent Bit quickly **became the industry standard unified logging layer** across all cloud and containerized environments.
   
   
   **Fluent Bit has been deployed over three billion times and is trusted by some of the world’s largest and most complex organizations.**
---

## Overview

It has been designed as a lightweight solution with high performance in mind. From a design perspective, it's fully asynchronous (event-driven) and take the most of the operating systems API for performance and reliability.
