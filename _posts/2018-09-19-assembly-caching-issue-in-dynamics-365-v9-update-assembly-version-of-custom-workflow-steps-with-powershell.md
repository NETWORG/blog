---
author: Tomas Prokop
title: >-
  Assembly Caching Issue in Dynamics 365 V9 - Update Assembly Version of Custom
  Workflow Steps With PowerShell
slug: >-
  assembly-caching-issue-in-dynamics-365-v9-update-assembly-version-of-custom-workflow-steps-with-powershell
id: 246
date: '2018-09-19 22:21:23'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - Assembly
  - Bug
  - CRM
  - Dynamics 365
  - Plugin
  - Plugin Registration Tool
  - PowerShell
  - Release Automation
---

As [Alex blogged](http://www.itaintboring.com/dynamics-crm/sometimes-you-may-have-to-re-build-the-whole-thing/) we are currently facing a caching bug of plugin assemblies in Dynamics 365 cloud instances (V9). You can do an in-place upgrade with Plugin Registration Tool without an exception now (see [Unable to in-place update plugin in Dynamics 365 (with build or revision number changed)](https://blog.thenetw.org/2018/07/10/unable-to-in-place-update-plugin-in-dynamics-365-with-build-or-revision-number-changed/)) but for some assemblies it just won't propagate to your workflow definitions. In the past we could fix it by clicking a save button in Properties tab of the assembly in Plugin Registration Tool but this does not work anymore in V9. I have put together another workaround which is usable in release automation scenarios. ![]/uploads7/2018/07/Update-AssemblyUsageToLatestVersion.gif)    You can use this when moving solutions between environments (unfortunately I had no luck with importing it back to the origin environment). The most common situation is exporting an unmanaged solution as managed from a sandbox to a production environment.

1.  Follow the instructions here and install a PowerShell module: [https://github.com/TheNetworg/dynamics365-release-automation-tools/blob/master/README.md](https://github.com/TheNetworg/dynamics365-release-automation-tools/blob/master/README.md)
2.  Then include the desired version of assembly in the solution. ![](/uploads/2018/09/chrome_2018-09-19_22-50-35.png)
3.  Export your solution. Both Managed and Unmanaged are supported.
4.  From the folder where you downloaded the solution open a PowerShell console. ![](/uploads/2018/09/2018-09-19_23-06-41.png)
5.  Run **Update-AssemblyUsageToLatestVersion** cmdlet with a path parameter. ![]/uploads7/2018/09/powershell_2018-09-19_23-08-21.png)
6.  Check the result. ![](/uploads/2018/09/powershell_2018-09-19_23-10-16.png)
7.  Now you can take the ZIP file which has been modified and import it to the target environment. If there was an older version of the plugin and solution, it will be updated.