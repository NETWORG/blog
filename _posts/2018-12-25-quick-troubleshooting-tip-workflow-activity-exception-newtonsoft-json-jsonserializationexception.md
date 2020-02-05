---
author: Tomas Prokop
title: >-
  Quick Troubleshooting Tip: Workflow Activity Exception
  'Newtonsoft.Json.JsonSerializationException'
slug: >-
  quick-troubleshooting-tip-workflow-activity-exception-newtonsoft-json-jsonserializationexception
id: 398
date: '2018-12-25 20:08:25'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - CodeActivity
  - Dynamics
  - JSON
  - Workflow Foundation
---

![](/uploads/2018/11/47007568_1178157702348930_5865726429164994560_n.png) **System.Runtime.Serialization.SerializationException: Type is not resolved for member 'Newtonsoft.Json.JsonSerializationException'** If you ever run into this exception without any trace available it is most likely not like it seems. Microsoft's internal wrapper for launching CodeActivity catches it and ignores your trace log entries. So if your code throws this exception the platform treats it like internal exception. It is probably because it serializes parameters for CodeActivity into JSON and this process can produce the same exception (in Microsoft's internal code).