---
author: Jan Hajek
title: Calling HTTP endpoints via On-premises data gateway
date: '2020-05-30 7:30:00'
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
tags:
  - PowerApps
  - Microsoft Flow
  - Power Automate
---

Recently, we have been working on an integration of a customer's on-premises system with Power Apps. The on-premises system is not accessible from the Internet and exposes the data on few REST-based endpoints. We needed to synchronize the data on daily basis and reflect the changes in Power Apps. In order to access the on-premises REST API, we decided to go with [On-premises data gateway](https://docs.microsoft.com/en-us/data-integration/gateway/service-gateway-onprem) which plays nicely with both Power Platform and Microsoft Azure.

The customer's API is protected by their custom OAuth2 authorization server (client_credentials flow is supported). Initially I thought - we did numerous integrations like this, both with REST APIs and various databases to which we connect via the gateway, so it's going to be quite simple. Yeah, well.. Turns out, we never did a combination of HTTP requests via the gateway.

So I started with trying out the [HTTP action](https://docs.microsoft.com/en-us/azure/connectors/connectors-native-http) but after doing some research, it turns out it isn't supported in scenarios with the gateway. So my next thought was - we are building lots of [custom connectors](https://docs.microsoft.com/en-us/connectors/custom-connectors/create-logic-apps-connector) which actually support the gateway. I went ahead and created a custom connector and [manually built](https://docs.microsoft.com/en-us/connectors/custom-connectors/define-blank) a Swagger definition around the REST endpoints. The issue came after I tried to setup authentication. Turns out, that when you [use the gateway](https://powerapps.microsoft.com/en-us/blog/on-premise-apis/), you are stuck with Windows and Basic authentication only.

Alrighty, we are engineers, let's come up with something creative - let's just define the `Authorization` header and give in a value with each request (that is the best way anyways, since client_credentials flow isn't supported by custom connectors as of now - only some first party connectors can use it). Well, turns out you can't do that:

![](/uploads/2020/05/customconnector-authorization-header.jpg)

This is where it gets confusing - the connector editor tells you that you can't use Authorization header and to use API key authentication type instead which is not supported with the gateway...

So while getting quite desperate with this entire situation, I started thinking about alternatives - like using [Azure Relay](https://docs.microsoft.com/en-us/azure/azure-relay/) to expose the endpoints - but I wanted to avoid this, because it would introduce another complexity with customer's security department (this one is kinda large corp) and we already had the gateway setup and approved.

Then, I started looking at [all the connectors](https://docs.microsoft.com/en-us/connectors/connector-reference/) supported by Flow. For a fact, I knew that the File System connector supports the gateway and has a setting called `gatewayParameters`. [Quick use of Google Search](https://www.google.com/search?q=%22gatewaySetting%22+site%3Ahttps%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fconnectors%2F&oq=%22gatewaySetting%22+site%3Ahttps%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fconnectors%2F&aqs=chrome..69i57.707j0j7&sourceid=chrome&ie=UTF-8) allowed me to get all the connectors which support the gateway (File System, BizTalk Server, MSSQL, MySQL, HTTP with Azure AD, SAP, ...).

Wait.. HTTP with Azure AD? That's like making a HTTP call right? Of course! So I went ahead and set up the HTTP with Azure AD connector's connection with the base URL of customer's on-premise REST API. When you create the HTTP with Azure AD connection and you don't choose the gateway, you are required to enter the Azure AD resource URI which you obviously don't have. When you choose the gateway, you get to specify the base URL, authentication type (choose anonymous) and the connector will work fine.

![](/uploads/2020/05/httpwithazuread-gateway.png)

I then tried to call their on-premises authorization server to obtain the token for the API via client_credentials flow... And there it was, I got the token! Cool!

Next up, was the need to make the actual call to customer's API. So I parsed the JSON response from the token endpoint, added the token into the `Authorization: Bearer <token>` header and made the call which went through the gateway and retrieved the data!

So a story with a good ending. I was honestly surprised that a connector called [HTTP with Azure AD](https://docs.microsoft.com/en-us/connectors/webcontents/) (I already used the connector in past to call Azure AD protected APIs in the cloud) would let me call on-premises REST API via the on-premises data gateway. Obviously, if you don't require the use of `Authorization` header in your on-premises API requests, you are best off with making the custom connector instead.