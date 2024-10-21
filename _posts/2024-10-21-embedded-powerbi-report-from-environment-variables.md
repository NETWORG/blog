---
title: "Embedding a PowerBI report into a form while storing report ID in environment variables"
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

Ever wondered how to embed a PowerBI report into a form and not having to hard-code the report ID into the form itself? 

## Embedding the report
To embedd the report we follow MS documentation: https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/embed-powerbi-report-in-system-form

We can choose between embedding a filtered or a non-filtered report. The downside is that the report ID needs to be hard-coded into the form itself. See the PowerBIGroupId, PowerBIReportId and TileUrl:

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

## De-hard-coding the solution

Luckily, there's a programatic solution - a javascript function using XRM web api.
1. Firstly, we create a text environment variable which stores our values:
```json
{"group":{"id":"409776a5-ce6d-45d0-a05f-166e8f52e094"},"report":{"id":"a8311a29-2fae-419c-82c1-51705fdf26b5","embedUrl":"https://app.powerbi.com/reportEmbed"}}
```
2. Then, using a script we set the value to the PowerBI report control:
```typescript
public static async OpenReport(executionContext: Xrm.Events.EventContext | Xrm.FormContext) {
  //@ts-ignore
  const formContext = await TALXIS.Utility.Apps.Start.UCIClientExtensions.TryGetFormContext(executionContext);
  const PowerBiSettingsString = await this.GetEnvironmentVariableValue("ntg_foodsafetyfinepowerbireport");
  const PowerBiSettings = JSON.parse(PowerBiSettingsString);
  const reportId = PowerBiSettings.report.id;

  formContext.getControl("finereport").controlDescriptor.Parameters["PowerBIGroupId"] = PowerBiSettings.group.id;
  formContext.getControl("finereport").controlDescriptor.Parameters["PowerBIReportId"] = reportId;
  formContext.getControl("finereport").controlDescriptor.Parameters["TileUrl"] = `${PowerBiSettings.report.embedUrl}?reportId=${reportId}`;
  
  formContext.getControl("finereport").setVisible(true);
} 
```

## Solution

Ensure that the application name is configured correctly within the Azure Function startup. Use the `SetApplicationName()` method to explicitly set the application name to a fixed value, ensuring consistency across deployments and configurations. Application name should be unique unless you want to share protected data between multiple applications.

```csharp
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            var configuration = builder.GetContext().Configuration;

            builder.Services.AddHttpClient();
            builder.Services.AddDataProtection()
                .SetApplicationName("YourApplicationName");
            
        }
    }

```

## Related Links

Here are some related links to Microsoft documentation that can provide further insights:

- [SetApplicationName method - IDataProtectionBuilderExtensions](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setapplicationname?view=aspnetcore-8.0)
- [Overview of Data Protection Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview?view=aspnetcore-8.0#protectkeyswithazurekeyvault)
- [Key Management in ASP.NET Core Data Protection Implementation](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/implementation/key-management?view=aspnetcore-8.0)
