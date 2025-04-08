---
title: "Decopiloting your dataverse environment"
date: 2025-04-207T11:00:00+02:00
author: Zdenek Srejber
categories:
  - Microsoft
  - Dataverse
  - Environment variables
  - Model-driven apps
  - Copilot
  - AI
tags:
  - Microsoft
  - Dataverse
  - Environment variables
  - Model-driven apps
  - Copilot
  - AI
---
# Decopiloting your dataverse environment
I might be the first one to come up with the term "Decopiloting" and it does exactly what you would expect. The following blogpost will show you how to remove copilot and AI clutter from your dataverse environment (and possibly more). 

## Option 1: Power Platform Admin center
Ideal when you managed just one or maybe a handful of environments without a strong ALM.
1. Open https://admin.powerplatform.microsoft.com/manage/environments
1. Open the target environment and hit Settings and then Features
   - All features here are well described and you should be able to clearly understand what you are disabling.
   - Its very likely that any newly released AI and Copilot features will be added here as well for you to turn them off.
1. I would recommend turning off the following:
   1. **AI form fill assistance: Automatic suggestion** - suggests content when editing form, good idea with the poorest execution.
   1. All setting under Copilot section
   1. AI builder
   1. AI prompts

## Option 2: Maker Portal
Suitable when you want to distribute decopiloting across multiple environments or code first approach is your jam.
1. Open https://make.preview.powerapps.com/
1. Select your environment and create a new solution
1. Add the following existing components from the Setting category
   1. AI insight cards on forms (logical name `EnableFormInsights`)
   1. Allow AI to generate predictions on edit forms. (logical name `FormPredictEnabled`)
   1. Copilot control (logical name `appcopilotenabled`)
   1. Enable Copilot answer control (logical name `EnablEnableCopilotAnswerControleFormInsights`)
1. Set properties of the components to `false` or `1` based on the input type and publish the changes.
1. From here you can export the solution and distribute the changes to other environments.
1. Every environment has an entity called 'organization'. In its data you will find exactly one record named after your environment. Its used for settings of the environment too. You will need to update the following properties of that record
   1. Display Preview Feature for this organization (logical name `paipreviewscenarioenabled`)
   1. Enable bot for makers. (logical name `powerappsmakerbotenabled`)
   1. Enable AI Promps (logical name `aipromptsenabled`)
1. If you want to distribute changes to the organization record, you can either use the code samples below.


### Code approach to update organization record
Simple JavaScript to run in you
```ts
const data = await Xrm.WebApi.retrieveMultipleRecords("organization",`?$select=organizationid,name`)
await Xrm.WebApi.updateRecord("organization", data.entities[0]['organizationid'], { 
    ["paipreviewscenarioenabled"]: false,
    ["powerappsmakerbotenabled"]: false,
    ["aipromptsenabled"]: false
})
```
Sample C# code to include in your package deployer extension. 
```c#
private void DisableDataverseAiTools(CrmServiceClient CrmSvc)
        {
            CreateProgressItem("Disabling AI helper tools");
            QueryExpression query = new("organization")
            {
                TopCount = 1,
                ColumnSet = new ColumnSet("organizationid"),
                Criteria = new FilterExpression
                {
                    FilterOperator = LogicalOperator.Or,
                    Conditions = {
                                new ConditionExpression{
                                    AttributeName = "paipreviewscenarioenabled",
                                    Operator = ConditionOperator.Equal,
                                    Values = { true }
                                },
                                new ConditionExpression{
                                    AttributeName = "powerappsmakerbotenabled",
                                    Operator = ConditionOperator.Equal,
                                    Values = { true }
                                },
                                new ConditionExpression{
                                    AttributeName = "aipromptsenabled",
                                    Operator = ConditionOperator.Equal,
                                    Values = { true }
                                }
                            },
                }
            };

            var organizationRecord = CrmSvc.RetrieveMultiple(query);
            if (organizationRecord.Entities.Count > 0)
            {
                CrmSvc.Update(new Entity("organization", new Guid(organizationRecord.Entities[0]["organizationid"].ToString()))
                {
                    ["paipreviewscenarioenabled"] = false,
                    ["powerappsmakerbotenabled"] = false,
                    ["aipromptsenabled"] = false
                });
            }
```

## Sources
https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/ai-overview#disable-copilot-in-power-apps

https://learn.microsoft.com/en-us/power-apps/developer/data-platform/webapi/reference/organization?view=dataverse-latest