# Caching for Web Service

Here's an example design doc: Adding caching for a web service.

## Context

From 12/01/2021 to 12/31/2021, we noticed a 80% spike in 5XX and 4XX errors sent by our data layer. This is most likely due to an increase in users who wanted to access our site for the Holiday season to buy goods. While there has been some performance deterioration among the other parts of the system, there have been the most errors from the data layer, so we need to consider how to scale our data layer to meet the rush of new customers. To do this, we propose adding a cache in front of our Postgres instances to reduce load on the database.

## Scope

This document is only concerned with improving the performance of the database layer by adding a cache in front of it. There is another design doc that is considering changing the indexing pattern for frequently accessed data to improve performance inside the database.

## Goals

Here are the goals we wish to meet with our implementation:

- Lower CPU load of instances from 80% during peak times to 30%.
- Reduce 90th percentile tail latency from 300ms to 100ms.
- Reduce ops load by 50% during the holiday season, measuring by the amount of tickets that are filed per user on the system.

## Non-Goals

Here are some non-goals:

- Increasing performance of the database by changing its internals
- Lowering cost of VPS instances by improving performance
- Changing the API interface in any way.

## Design

Given the constraints provided, we decided to use a Key-Value store as our caching layer, spun up on a different network which the web layer will query on all incoming requests to see if the cache contains the data required.

### Data Storage

The data will be stored in a Hash Table in Redis, which will allow for fast access by key, while saving on memory (on our benchmarks, we noted a 80% decrease in memory usage compared to a B-Tree structure).

### Alternatives

We were considering a B-Tree index, as that would allow for sorted searching, but we declined this approach as we realized that our data access pattern does not use range queries. As well, we declined to follow an approach using a Bloom Filter, since we wanted 100% accuracy in cache usage.

### Cross-cutting concerns

This project, if successful, strives to lower both ops load and developer on-call time by lowering the amount of 4XX and 5XX statuses that the data layer returns. This will allow us to have less support burden and allow us to scale our teams for an increased user load.

[Back](./index.md)
