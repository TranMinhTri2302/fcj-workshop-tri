---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# PAGINATION STRATEGY IN AMAZON DYNAMODB - HOW TO SAVE COSTS AND OPTIMIZE PERFORMANCE

---

Amazon DynamoDB provides a pagination mechanism through **LastEvaluatedKey** and **ExclusiveStartKey**, allowing you to split query results into smaller portions (pages) instead of loading all data at once. This is not an optional feature but an essential strategy that helps reduce Read Capacity Units (RCU) costs by thousands of times, improve response time from minutes to hundreds of milliseconds, and ensure your application can scale as data grows.

---

## The Real Problem I Encountered

This article is based on my hands-on experience working with DynamoDB across multiple projects. I noticed that many people new to DynamoDB often make a fairly common mistake — trying to load all data at once. This seems harmless when data is small, but when your DynamoDB table grows to hundreds of thousands or millions of records, that's when problems start to appear clearly.

Specifically, when you have a DynamoDB table containing millions of records and you perform a query or scan to retrieve all data, it's not just that it will be slow — it's also costly in terms of both money and system performance. DynamoDB charges based on **Read Capacity Units (RCU)** — each time you read a certain amount of data, you are charged accordingly. If you keep loading everything from scratch with every request, costs will grow exponentially, especially when the application has many concurrent users.

Beyond the cost issue, loading all data also causes **throttling** when exceeding provisioned capacity, severely increasing latency and in many cases leading to **timeouts** on the client side. Users will have to wait tens of seconds, even minutes, just to see a product list or order history.

---

## The Solution: Pagination with LastEvaluatedKey

The practical solution is to use **pagination** — instead of fetching everything, you only fetch a small portion first (for example, 10 or 20 records), then when the user wants to see more, you fetch the next batch.

How pagination works in DynamoDB is quite simple but extremely effective. When you perform a **Query** or **Scan**, in addition to the returned results (Items), DynamoDB also returns a value called **LastEvaluatedKey**. This key indicates exactly where you stopped in the previous query. For the next query, you simply pass this key into the **ExclusiveStartKey** parameter, and DynamoDB will know where to start and continue returning the next batch without scanning from the beginning.

---

## Key Points to Understand

- **LastEvaluatedKey** is the key that DynamoDB returns after each query/scan, indicating the last position that was read. When this value is `null`, it means all data has been retrieved.

- **ExclusiveStartKey** is the parameter you pass into the next request — it is the LastEvaluatedKey value from the previous response — so DynamoDB knows where to start.

- **The Limit parameter** restricts the number of items **evaluated** (checked), not the number of items **returned**. If you have a FilterExpression, the actual number of items returned may be less than Limit → this is the most common "gotcha" that many newcomers encounter.

- **1MB limit per response**: DynamoDB automatically paginates when results exceed 1MB, even if you don't set a Limit. This means that even if you want to retrieve everything, DynamoDB forces you to make multiple calls.

- **RCU costs decrease proportionally** with the amount of data read each time. Loading 20 items × 1KB = 20 RCU, instead of loading 1 million items = 1 million RCU. If a user only views 5 pages, the total cost is only about 100 RCU — **10,000 times** more efficient.

- **Response time improves significantly**: from several seconds or minutes (load all) down to a few hundred milliseconds (load per page), noticeably improving user experience.

- **Pagination works with both Query and Scan**, but Query should be preferred because Scan scans the entire table and consumes far more RCU.

- **LastEvaluatedKey includes the full primary key** (partition key + sort key if applicable), so its size depends on the table's key structure, but it is always much smaller than the entire dataset.

---

## Specific Cost Comparison

To clearly see the difference, consider the following example:

| Method | Items Read | RCU Consumed | Response Time |
|---|---|---|---|
| Load all 1 million items | 1,000,000 | ~1,000,000 RCU | Minutes or timeout |
| Pagination 20 items/page × 5 pages | 100 | ~100 RCU | ~200–500ms per page |

In practice, the majority of users only view the first 2–3 pages. This means pagination not only saves costs in theory but also accurately reflects real usage behavior — you only pay for what users actually need to see.

---

## Implementation Considerations

**First**, when the client receives LastEvaluatedKey, you should **encode it** (for example, base64) before returning it to the frontend, because this key contains information about the table structure that you don't want to expose directly.

**Second**, DynamoDB pagination is **forward-only cursor-based pagination** and does not support jumping to an arbitrary page (for example, jumping directly to page 50). If your application requires that type of pagination, you need to consider combining it with a caching layer or redesigning your data model.

**Third**, carefully handle cases where data changes between calls (items are added or deleted), as this can cause duplication or missing data at page boundaries.

**Fourth**, with FilterExpression, you may receive an empty page (0 items) but LastEvaluatedKey is still not null. In this case, you need to continue calling the query until LastEvaluatedKey is null or you have collected enough items.

---

## When Should You Use Pagination?

This feature is especially useful when building features such as product lists, transaction history, activity feeds, or any interface that displays large data lists. Instead of forcing users to wait and burdening the system with the entire dataset, pagination allows you to serve small portions quickly, efficiently, and smoothly.

If you are building any application with DynamoDB, think about pagination from the design stage. It is not a "nice-to-have" feature but an essential part of application architecture — helping you save operational costs, improve system performance, and create a better experience for your users.

---

**Blog Post Link:**  
https://www.facebook.com/share/p/1BxRgPHRBn/

**Main Reference:**  
[Paginating table query results — Amazon DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)