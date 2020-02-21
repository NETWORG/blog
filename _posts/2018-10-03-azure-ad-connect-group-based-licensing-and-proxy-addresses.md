---
author: Jan Hajek
title: 'Azure AD Connect, group-based licensing and proxy addresses'
slug: azure-ad-connect-group-based-licensing-and-proxy-addresses
id: 342
date: '2018-10-03 09:28:37'
categories:
  - Azure
  - English
  - Office 365
tags:
  - Azure AD
  - Azure AD Connect
  - Group-based licensing
  - proxyAddress
---

We have had the [group-based licensing](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-licensing-whatis-azure-portal) option available in preview for over a year. While this service is in preview, it makes provisioning hundreds of users from Active Directory really simple. You simply create users in your on-premise Active Directory, assign them a valid User Principal Name, add them to the correct group and then sync them with Azure AD Connect, right? Not that fast cowboy! Once the initial synchronization happens, the user is then assigned the respective licenses based on their groups. Once exchange is provisioned, you might notice the following state: [![](/uploads/2018/10/upn.png)](/uploads/2018/10/upn.png) The user's UPN is set correctly, however their primary e-mail address is going to contain _.onmicrosoft.com_ suffix and the correct address will be set as secondary address. After some search, you can find the following document - [How the proxyAddresses attribute is populated in Azure AD](https://support.microsoft.com/en-us/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad). The scenario which applies is _Scenario 1_ where the user has only UPN set in Active Directory. I tried to do some investigation on my own, but ended up opening a support ticket. After arguing and persuading the support agent to escalate this higher and few screen sharing sessions, we ended up getting a response that this is a bug - probably wasn't known until we reported it. So the solution? It is pretty much obvious - either set the desired address into the _mail_ attribute or _proxyAddresses_. While we knew we could do this from the very beginning, I wanted to get an official statement from Microsoft whether this is expected or we were doing something wrong. In case you would like to quickly fix it, you can use the PowerShell script below:

```powershell
Import-Module ActiveDirectory 

$users = Get-ADUser -Filter {UserPrincipalName -like '*'} -SearchBase "OU=Students,OU=Domain Users,DC=ad,DC=school,DC=cz" -Properties UserPrincipalName
foreach ($user in $users) {
    $upn = $user.UserPrincipalName

    Write-Host $user.UserPrincipalName

    $user | Set-ADUser -Replace @{mail=$upn}
}
```

This script simply takes the user's UPN and sets it into the mail attribute of users in specific OU.