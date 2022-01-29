---
author: Jan Hajek
title: Parallel upsert in Dataverse ends up with HTTP 412
date: "2022-01-29T17:00:00+0200"
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - WebAPI
  - OData
---

In some of our projects where we synchronize data from other systems, we use the [upsert](https://docs.microsoft.com/en-us/powerapps/developer/data-platform/webapi/update-delete-entities-using-web-api#upsert-a-table) operation, since we keep the same primary ID of the record. After reviewing the logs, we found a re-occuring error which we weren't able to reproduce - `A record with matching key values already exists.`.

The full exception (especially to have it properly index by search engines for future reference) is following:

```json
{
  "error": {
    "code": "0x80040237",
    "message": "A record with matching key values already exists."
  }
}
```

Upsert operation is generally a HTTP PATCH request sent to `/api/data/v9.1/entity(id)` endpoint with body containing the record values. If the specified `entity` with `id` already exists, the record will be updated, otherwise it will be created.

In our scenario, we were sending an upsert request to the same combination of `entity` and `id` at the same time (by parallelization). It didn't initially struck us that this could be the cause but after trying to reproduce the issue, it was the only option left.

So we tried to re-create the situation by code, simply by executing two HTTP requests at the same time and then waiting for the results of each and there it was - one succeeded and one failed.

I suppose this is by design, however the documentation doesn't mention this behavior at all (eg. with upsert, it should not happen). What I think that happens behind the scenes is:

```
Request 1: Check whether entity with id exists > No
Request 2: Check whether entity with id exists > No
Request 1: Insert record into the database > Success
Request 2: Insert record into the database > Fail - key already exists
```

Basically a race condition.

# Possible mitigations

You have to handle this behavior on your application level. If you are writing C# code, you can either write a lock (if async using SemaphoreSlim) and prevent this to happen (if using a single instance of the application). If using multiple instances (Azure Functions, multiple nodes, Power Automate, Logic Apps, ...), you will have to handle the error itself.

With handling the error, there are two possible ways in my opinion. When you are upserting in parallel (the same record value) but from multiple instances, you should probably just catch the error, verify that it is a response to your upsert operation and skip it and continue (since the data have already been written using another call and they are the same). If you are upserting the data but from multiple sources (HR system and org chart source at the same time for example), you need to implement a retry logic - when a HTTP 412 occurs with this error message, the upsert failed and you need to run the operation again to get the data in Dataverse.