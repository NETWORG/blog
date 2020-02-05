---
author: marek.dubsky@thenetw.org
title: Office 365 domain ForceTakeOver
slug: office-365-domain-forcetakeover
id: 920
date: '2019-09-17 08:06:22'
categories:
  - Azure
  - English
  - Office 365
tags:
  - Azure Active Directory
  - AzureAD
  - Domain TakeOver
  - Office365 Admin
  - PowerShell
---

If you are an Office 365 administrator, I believe that you have experienced this scenario. The situation when you want to add a domain to an Office 365 tenant, but you cannot because it is blocked by a different tenant .

More specifically, a customer wants you to set up a domain in their current tenant and when you try to add the domain it keeps telling you that the domain is being used already.

But when you ask the customer they do not know, they had used the domain previously in a different tenant. Then usually it goes down that the customer gets angry or impatient but you cannot solve it easily because you do not have an access to the historical tenant to remove the domain.

If you know the feeling, this blog will save you a lot of time and grey hair.  
The solution is actually very easy and you do not even need the access to the historical tenant.

**All you need is:**

*   Basic PowerShell knowledge
*   Access to the domain provider
*   Administrator access to the current tenant.

**Here are the steps, you need to do:**

1.  To connect to Azure AD via PowerShell , install the Sign in assistant:  
    [https://www.microsoft.com/en-us/download/details.aspx?id=41950](https://www.microsoft.com/en-us/download/details.aspx?id=41950)
2.  Run PowerShell as an adminstrator

**And now the commands:**

    Set-ExecutionPolicy unrestricted
    Install-Module MSOnline (install module)
    Connect-MsolService
    Sign in as an Administrator of the current tenant (where the domain should be verified to)
    New-MsolDomain –name domain.cz (adding domain)
    Get-MsolDomain (check the status - should be Unverified)
    Get-MsolDomainVerificationDns –DomainName domain.cz –Mode DnsTxtRecord (to get the TXT record, which should be set at domain provider)
    Add the generated TXT record to the domain provider (it should look like MS=xxxxxx)
    As soon as you are sure, that the record is visible at MX Toolbox or whatismydns.net, you can move forward to the next step
    Confirm-MsolDomain –DomainName domain.cz –ForceTakeover Force

Let us know in comments if it helped you or if you have any questions!

[https://docs.microsoft.com/en-us/powershell/azure/authenticate-azureps?view=azps-2.4.0](https://docs.microsoft.com/en-us/powershell/azure/authenticate-azureps?view=azps-2.4.0)  
[https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/domains-admin-takeover](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/domains-admin-takeover)