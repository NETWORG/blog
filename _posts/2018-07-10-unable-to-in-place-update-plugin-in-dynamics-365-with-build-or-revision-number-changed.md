---
author: Tomas Prokop
title: >-
  Unable to in-place update plugin in Dynamics 365 (with build or revision
  number changed)
slug: >-
  unable-to-in-place-update-plugin-in-dynamics-365-with-build-or-revision-number-changed
id: 223
date: '2018-07-10 07:40:50'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - Assembly
  - Dynamics 365
  - Dynamics CRM
  - Plugin Registration Tool
  - TLS
---

UPDATE 19/9/18: It has been fixed in 9.0.2.5 version of Plugin Registration Tool https://www.nuget.org/packages/Microsoft.CrmSdk.XrmTooling.PluginRegistrationTool/9.0.2.5 If you use 9.0 version of Plugin Registration Tool to update your assemblies in Dynamics 365 you may encounter the following exception:

<pre class="wrap:true lang:default highlight:0 decode:true ">ERROR: Occurred while checking whether the assembly exists

The PluginType(00000000-0000-0000-0000-000000000000) component cannot be deleted because it is referenced by 1 other components. For a list of referenced components, use the RetrieveDependenciesForDeleteRequest.</pre>

## **Here are the official guidelines for plugin upgrades and versioning:**

### **The build or revision assembly version number is changed.**

This is considered an in-place upgrade. The older version of the assembly is removed when the solution containing the updated assembly is imported. Any pre-existing steps from the older solution are automatically changed to refer to the newer version of the assembly.

### **The major or minor assembly version number, except for the build or revision numbers, is changed.**

When an updated solution containing the revised assembly is imported, the assembly is considered a completely different assembly than the previous version of that assembly in the existing solution. Plug-in registration steps in the existing solution will continue to refer to the previous version of the assembly. If you want existing plug-in registration steps for the previous assembly to point to the revised assembly, you will need to use the Plug-in Registration tool to manually change the step configuration to refer to the revised assembly type. This should be done before exporting the updated assembly into a solution for later import.  

# **Resolution**

It looks like the issue is in newer builds. I couldn't make it work with the latest tools from NuGet:  [https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/download-tools-nuget](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/download-tools-nuget) In the meantime you can use the 8.2 plugin registration tool from the good old SDK which is working properly. [https://www.microsoft.com/en-us/download/details.aspx?id=50032](https://www.microsoft.com/en-us/download/details.aspx?id=50032)   ![](/uploads/2018/07/PluginRegistration_2018-07-01_19-09-12.png) ![](/uploads/2018/07/devenv_2018-07-01_19-10-44.png)  

# Encryption

Although you may not be able to connect due to TLS 1.2 restriction in the cloud environment. [https://support.microsoft.com/en-ph/help/4077479/unable-to-connect-to-dynamics-365-online-version-9-0-using-sdk-tools](https://support.microsoft.com/en-ph/help/4077479/unable-to-connect-to-dynamics-365-online-version-9-0-using-sdk-tools) There is a workaround. You can use a proxy to upgrade the version of TLS by a man in the middle. My favorite tool is for packet inspection is Fiddler 4 which supports TLS 1.2 and can be used for this: [https://www.telerik.com/download/fiddler](https://www.telerik.com/download/fiddler)