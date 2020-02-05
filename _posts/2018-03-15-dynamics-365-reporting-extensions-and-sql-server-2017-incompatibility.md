---
author: Tomas Prokop
title: Dynamics 365 Reporting Extensions and SQL Server 2017 incompatibility
slug: dynamics-365-reporting-extensions-and-sql-server-2017-incompatibility
id: 92
date: '2018-03-15 13:33:30'
categories:
  - Dynamics 365 / CDS / PowerApps
tags:
  - OnPremise
  - Reporting Services
  - SQL Server
---

I came across a strange issue when trying to upgrade CRM deployment to Microsoft SQL Server 2017. Important note: It is not listed as a supported version [https://technet.microsoft.com/en-us/library/hh699754.aspx](https://technet.microsoft.com/en-us/library/hh699754.aspx) And here is probably why. Everything went smooth until I tried to reinstall Dynamics 365 Reporting Extensions via SetupSrsDataConnector.exe. Since SQL Server 2017 we have a standalone installer of Reporting Services. ![](/uploads/2018/04/report-server-install.png) It configures SSRS instance differently but I could find why Reporting Extensions wizard fails to locate it. I had even tried to use XML config to pass the instance parameters to the SetupSrsDataConnector.exe directly. After successful installation SrsDataConnector wizard was unable to locate my SSRS instance. ![](/uploads/2018/04/report-server-srs.png) Without correct setup of SrsDataConnector environment checks will fail when you try to provision a new organization or import an existing one. ![](/uploads/2018/04/report-server-env.png) You can set a flag in registry to ignore the errors if you are ok to proceed without reporting services. **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\MSCRM** DWORD **IgnoreChecks** = 1 It will let you create an org but I wasn't able to import or upgrade existing ones due to errors during the process. Only solution was using SQL Server 2016 end installing Reporting Services using the classic server installer.