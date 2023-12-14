---
title: Power Automate Delete Trigger - Missing record values
date: 2023-10-29T11:00:00+02:00
author: Adel Sabic
categories:
  - Microsoft
  - Power Platform
  - Power Automate
tags:
  - Flows
  - Work-around
---

If you are using Power Automate Flow with MS Dataverse Delete trigger and trying to find a way to retrieve attributes value from a deleted record, you are in the right place!

## ISSUE DESCRIPTION
MS Dataverse Delete trigger is triggered post action. That means that record values are already missing from the database when trigger is triggered. Therefore, if you want to reuse some value that was part of that record, you will not be able to.

## HOW TO DO IT
### REQUIREMENTS
- Audit needs to be enabled globally in environment
- Audit need to be enabled on entity you are targeting

### IMPLEMENTATION
- You need to get last Audit record related to deleted record.
     - OData example: `/audits?$filter=_objectid_value eq '{{ DELETED_RECORD_ID }}'&$orderby=createdon desc&$top=1`
- This call will return Audit record with `changedata` attribute
- `changedata` value is stringified JSON object which will contain `changedAttributes` array
- Parse this JSON object and you'll get last existed values in `oldValue` object key value of `changedAttributes` array


Hope this helps.