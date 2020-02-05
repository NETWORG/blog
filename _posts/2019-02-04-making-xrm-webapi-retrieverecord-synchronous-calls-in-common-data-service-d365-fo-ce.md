---
author: jan.kostejn@thenetw.org
title: >-
  Making Xrm.WebApi.retrieveRecord Synchronous calls in Common Data Service
  (D365 fo CE)
slug: >-
  making-xrm-webapi-retrieverecord-synchronous-calls-in-common-data-service-d365-fo-ce
id: 445
date: '2019-02-04 08:11:21'
categories:
  - Dynamics 365 / CDS / PowerApps
  - English
tags:
  - CDS
  - Client API
  - Common Data Service
  - CRM
  - Dynamics 365
  - Dynamics CRM
  - JavaScript
  - PowerApps
  - powerplatform
---

With v9 a lot of changed. One of the major changes is client web API, some of calls were made deprecated and some were added. For example, Xrm.WebApi.

The new ‘Xrm.WebApi.retrieveRecord(entityLogicalName, id, options).then(successCallback, errorCallback)’ can’t be made synchronous. But imagine situation, in which you need to go through multiple entities (lookups). In this scenario, you need the result of retrieved record to access the next record. So how can this be done?

In this case, you need to use successCallback, which can be nested – so you can go through multiple entities:

    Xrm.WebApi.retrieveRecord(entityLogicalName, id, options)
        .then(function(x) {
           return Xrm.WebApi.retrieveRecord(x.lookupEntityLogicalName, x.lookupValue, options)
        }, errorCallback)
        .then(function(y) {
            return Xrm.WebApi.retrieveRecord(y.lookupEntityLogicalName, y.lookupValue, options)
         }, errorCallback)
         .then(function(z) {
            return Xrm.WebApi.retrieveRecord(z.lookupEntityLogicalName, z.lookupValue, options)
         }, errorCallback)

But if you are like me, you probably already found this out on your own. What’s the issue with this solution? You don’t want to nest calls to targeted record in advance, you want to specify path to this record and what should be done with the result. So, you can use the same code snippet.

I wrote simple recursion function that does exactly what’s described above. This may be exactly what you were looking for:

    function createRetrieveRecordPromise(entityLogicalName, entityId, pathToEntity, finalFunction, depth = 0) {
        Xrm.WebApi.retrieveRecord(entityLogicalName, entityId).then(
            function success(result) {
                if (depth < pathToEntity.length) {
                    entityLogicalName = result[`_${pathToEntity[depth]}_value@Microsoft.Dynamics.CRM.lookuplogicalname`];
                    entityId = result[`_${pathToEntity[depth]}_value`];
                    depth++
                    createRetrieveRecordPromise(entityLogicalName, entityId, pathToEntity, finalFunction, depth);
                }
                else {
                    finalFunction(result);
                }
            },
            function (error) {
                console.log(error.message);
            }
        );
    }

    // example call
    createRetrieveRecordPromise(
        "quotedetail",
        "CE6352EA-A9FE-E811-A968-000D3AB42B3B",
        ["quoteid", "customerid"],
        function (lastRecord) {
            alert(lastRecord.accountid);
        }
    );

I will write something more advanced next time, no worries.