---
author: Jan Skala
title: >-
  Azure Logic Apps SQL Connector strikes again
slug: >-
  azure-logic-apps-sql-connector-strikes-again
date: '2020-05-12 09:00:00'
categories:
  - Azure
  - Azure Logic Apps
  - Power Platform
tags:
  - Azure
  - Azure Logic Apps
  - Power Platform
excerpt_separator: <!--more-->
---
Working with **Azure Logic Apps SQL Connector** can be tricky sometimes, especially if you work with datetime and OData queries. 
<!--more-->
# The problem
I was fetching rows from a specific table, based on their timestamp. So I wrote a simple OData query: `change ge 2020-05-12T15:17:43.1729282Z`. And to my surprise it didn't work.
# The mystery
The most annoying part of this problem was that I didn't know what went wrong. I received following error: "**BadRequest**. Http request failed: the content was not a valid JSON." 

Wait a minute... Did I send the invalid JSON or I received invalid JSON? Who knows. So anyway I started experimenting and noticed, that when I stop sending the OData query, it returns correct JSON. So it must be it right?

# The solution
And truly, you cannot compare datetime(s) this way. Or at least not when using Azure Logic Apps SQL Connector. My query had to be adjusted to a little bit ugly format.



Old Query: `change ge variables('Timestamp')`

New Query: `((year(change) ge year(@{variables('Timestamp')}) and month(change) ge month(@{variables('Timestamp')})  and day(change) ge day(@{variables('Timestamp')}) and hour(change) ge hour(@{variables('Timestamp')}) and minute(change) ge minute(@{variables('Timestamp')})))`

As you can see I had to compare every part separately and that solved my issue.

