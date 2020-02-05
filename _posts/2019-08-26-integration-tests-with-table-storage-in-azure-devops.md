---
author: jan.skala@thenetw.org
title: Integration tests with Table Storage in Azure DevOps
slug: integration-tests-with-table-storage-in-azure-devops
id: 852
date: '2019-08-26 08:58:12'
categories:
  - Azure
  - English
tags:
  - Azure DevOps
  - CI/CD
  - Table Storage
---

We need database so we need SQL Server. How many times have you heard this nonsense? There are many alternatives to traditional relation databases that are more suitable for some use cases. One of them is [Table Storage](https://azure.microsoft.com/en-us/services/storage/tables/). In this article, I will show you how to test your code that is using Table Storage in your Azure DevOps CI pipeline.

## Getting started

Before we move to cloud, let's explore our posibilites of testing Table Storage locally. Microsoft created a tool called [Azure Storage Emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator). You can use that for testing Tables, Queues or even Blobs. In order to connect to your Azure Storage Emulator, you have to use a special connection string.

    "UseDevelopmentStorage=true"

Good thing about this, is that you can commit this and push into your repo, because it is same for everyone. Just don't forget to change it for production. Now every time we will call Table Storage with this connection string, it will be caught by Azure Storage Emulator.

## Running Azure Storage Emulator in Azure DevOps

Unfortunately, Azure Storage Emulator runs only on Windows. There is an open source alternative called [Azurite](https://github.com/Azure/Azurite), but I have never user that in a pipeline.

First step to do is to run your CI pipeline on Windows agent.

<figure class="wp-block-image">![](/uploads/2019/08/image-3.png)</figure>

Next step is to run Azure Storage Emulator on this particular agent. You can do that with following script.

    echo Starting MsSql localdb
    sqllocaldb create MSSQLLocalDB
    sqllocaldb start MSSQLLocalDB
    sqllocaldb info MSSQLLocalDB

    echo Starting table storage emulator
    "C:\Program Files (x86)\Microsoft SDKs\Azure\Storage Emulator\AzureStorageEmulator.exe" start

    echo Table storage emulator started

<figure class="wp-block-image">![](/uploads/2019/08/image-4-1024x259.png)

<figcaption>Usage of the script</figcaption>

</figure>

Now you can use the real Table Storage inside your tests instead of a mocked one.

    var connectionString = "UseDevelopmentStorage=true";
    var storageAccount = CloudStorageAccount.Parse(connectionString);
    var tableClient = storageAccount.CreateCloudTableClient();
    // rest of the test...