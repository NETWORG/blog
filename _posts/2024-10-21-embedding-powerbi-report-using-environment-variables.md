---
title: "Embedding a PowerBI Report into a Form Using Environment Variables"
date: 2024-10-21T11:00:00+02:00
author: Stepan Houdek
categories:
  - Microsoft
  - PowerBI embedded
  - Environment variables
  - Model-driven apps
tags:
  - Microsoft
  - PowerBI embedded
  - Environment variables
  - Model-driven apps
  - Forms
---
# Embedding a PowerBI Report into a Form Using Environment Variables

Embedding a PowerBI report into a form often requires hard-coding the report ID directly into the form. This approach can be inflexible and difficult to maintain. In this post, we'll explore how to dynamically embed a PowerBI report by storing the report ID in environment variables.

## Embedding the Report

To embed the report, we follow the [Microsoft documentation](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/embed-powerbi-report-in-system-form). You can choose between embedding a filtered or non-filtered report. Typically, the report ID is hard-coded into the form, as shown below:

```xml
<section id="{d411658c-7450-e1e3-bc80-07021a04bcc2}" locklevel="0" showlabel="true" IsUserDefined="0" name="tab_4_section_1" labelwidth="115" columns="1" layout="varwidth" showbar="false">
    <labels>
        <label languagecode="1033" description="Unfiltered Power BI embedding demo"/>
    </labels>
    <rows>
        <row>
            <cell id="{7d18b61c-c588-136c-aee7-03e5e74a09a1}" showlabel="true" rowspan="20" colspan="1" auto="false" solutionaction="Added">
                <labels>
                    <label languagecode="1033" description="Accounts (Parent Account)"/>
                </labels>
                <control id="unfilteredreport" classid="{8C54228C-1B25-4909-A12A-F2B968BB0D62}">
                    <parameters>
                        <PowerBIGroupId>00000000-0000-0000-0000-000000000000</PowerBIGroupId>
                        <PowerBIReportId>544c4162-6773-4944-900c-abfd075f6081</PowerBIReportId>
                        <TileUrl>https://app.powerbi.com/reportEmbed?reportId=544c4162-6773-4944-900c-abfd075f6081</TileUrl>
                    </parameters>
                </control>
            </cell>
        </row>
        <row/>
    </rows>
</section>
```
## Dynamic Embedding with Environment Variables
To avoid hard-coding, we can use environment variables and a script to dynamically set the report ID.
### Step 1: Create Environment Variables
First, create a text environment variable to store the PowerBI settings:
```json
{
  "group": {
    "id": "280ff574-14c9-4b4d-af29-799f75acd380"
  },
  "report": {
    "id": "71ae687f-72b0-41df-83ee-44ee1e7a460b",
    "embedUrl": "https://app.powerbi.com/reportEmbed"
  }
}
```
Refer to the Microsoft documentation for locating these values.

### Step 2: Script to Set Report Values
Next, use a script to set the environment variable values to the PowerBI report control:
```typescript
public static async OpenReport(executionContext: Xrm.Events.EventContext | Xrm.FormContext) {
  // @ts-ignore
  const formContext= executionContext.getFormContext();
  const PowerBiSettingsString = await this.GetEnvironmentVariableValue("your_environment_variable_name");
  const PowerBiSettings = JSON.parse(PowerBiSettingsString);
  const reportId = PowerBiSettings.report.id;
  const reportControl = formContext.getControl("unfilteredreport");

  reportControl.controlDescriptor.Parameters["PowerBIGroupId"] = PowerBiSettings.group.id;
  reportControl.controlDescriptor.Parameters["PowerBIReportId"] = reportId;
  reportControl.controlDescriptor.Parameters["TileUrl"] = `${PowerBiSettings.report.embedUrl}?reportId=${reportId}`;
  reportControl.setVisible(true);
} 

public static async GetEnvironmentVariableValue(variableSchemaName) {
    const result = await Xrm.WebApi.retrieveMultipleRecords(
        'environmentvariabledefinition',
        `?$select=displayname,schemaname,defaultvalue` +
        `&$expand=environmentvariabledefinition_environmentvariablevalue($select=value,createdon;$orderby=createdon desc)` +
        `&$filter=(schemaname eq '${variableSchemaName}')`
    );
    const value = result.entities[0]?.environmentvariabledefinition_environmentvariablevalue[0]?.value;
    return value ?? result.entities[0]?.defaultvalue;
}
```

And that's it! By using environment variables, you can dynamically embed PowerBI reports into your forms without hard-coding the report IDs.

![DetailsList](/uploads/2024/10/embedded-powerbi.png)

## Credits
To the user "NODAL" from [Dynamics Community](https://community.dynamics.com/forums/thread/details/?threadid=55ffe60a-aec6-4d1a-8fca-7063a4a3661c) for coming up with the script.