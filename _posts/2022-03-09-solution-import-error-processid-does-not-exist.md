---
author: Ondrej Juda
title: Solution import error - processid does not exist
date: "2022-03-09T08:00:00+0100"
categories:
  - English
  - Power Platform
  - Dynamics / CDS / PowerApps
  - Business Process Flow
  - Business Rule
  - Solution Import
tags:
  - PowerApps
  - Model-driven Apps
  - Business Rule
  - Business Process Flow
  - Solution Import
  - Error
---

In our [last post](/2022/03/07/business-rule-on-related-bpf-stage/), I explained how to make a business rule based on related business process flow.

While playing with this feature, I run into an error while trying to import it to a new environment:

**The dependent component Attribute (Id=processid) does not exist. Failure trying to associate it with ProcessTrigger (Id=c2f35ac1-be88-ec11-8d20-0022489e31fc) as a dependency. Missing dependency lookup type = AttributeNameLookup.**

After some research, I found these problems:

- Our custom entity in a solution doesn't contain the **processid** field.
- Compared to another entity with business process flow, it lacks two other fields: **stageid**, **traversedpath**.
- Based on [this article](https://docs.microsoft.com/en-us/dynamics365/customerengagement/on-premises/developer/model-business-process-flows?view=op-9-1#legacy-process-related-attributes-in-entities) in MS Docs, these fields are deprecated and should not be used.

After discussion with MS support, we thought that adding those fields to the solution would solve our problem, but it did not, because the fields were not imported. As we later found out, our problem was in the entity metadata. 

```xml
<IsBusinessProcessEnabled>1</IsBusinessProcessEnabled>
```

When you create a business process flow for an entity, it changes this property from 0 to 1. This ensures that when you import a solution with an entity that has a business process flow, it creates those fields.
Even though the three fields are marked as deprecated, they are still system required and used by the business rule, if you include the related business process flow in it.
