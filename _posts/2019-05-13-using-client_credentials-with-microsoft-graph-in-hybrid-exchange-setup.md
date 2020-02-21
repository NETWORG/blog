---
author: Jan Hajek
title: Using client_credentials with Microsoft Graph in Hybrid Exchange setup
slug: using-client_credentials-with-microsoft-graph-in-hybrid-exchange-setup
id: 638
date: '2019-05-13 09:00:11'
categories:
  - Azure
  - English
  - Office 365
tags:
  - Azure Active Directory
  - Exchange Server
  - Hybrid
  - Microsoft Graph
  - Office 365
---

If you or your customers are running hybrid Microsoft Exchange deployment and you are using [Microsoft Graph](https://graph.microsoft.com), you might have noticed that using the client_credentials grant flow doesn't really work and ends with errors. Last week, we have had a customer who we have been integrating few systems for, and hit the exactly same issue.

So far, Microsoft has been only [demonstrating](https://blogs.msdn.microsoft.com/deva/2019/03/16/deep-dive-how-to-configure-exchange-on-premise-server-hybrid-integration-with-office-365-test-rest-api-calls/) [hybrid Graph](https://docs.microsoft.com/en-us/graph/hybrid-rest-support) with regular authorization_code grant flow. If you wanted to run your application as a deamon, you had to rely on not-so-secure [resource_owner grant](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth-ropc) which is way less secure than [client_credentials flow](https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) which we will focus on.

Initially, when you try to call Microsoft Graph via client_credentials with user which has their mailbox on-prem, you will end up with _UnknownError_. What [you can find on the internet](https://social.msdn.microsoft.com/Forums/en-US/f6300a96-b762-404b-b585-34a420c633e7/client-credentials-token-is-not-working-for-onprem-exchange-server?forum=WindowsAzureAD) is to add _V1S2SAppOnly_ to your _web.config_'s (_...\FrontEnd\HttpProxy\rest\web.config_) line _<add key="OAuthHttpModule.Profiles"_. That's a good way to start, so make sure you have done it. And that's about all that the internet says about the issue, however if you try calling Graph again, you will end up with an error again.

So what to do next? Well I will describe a way how I figured it out, so if you want to find the fix, head to the end of this article.

**Disclaimer:** Please note that the steps described below are **unsupported by Microsoft** and while unlikely, it may corrupt your hybrid setup.

Initially, I thought that Exchange just didn't support the flow. But after doing some reading, I ended up with a conclusion that it had to be supported. I decided to try to enable IIS logging to capture the _WWW-Authenticate_ header, which Exchange Server sends along with responses. Thanks to that, I managed to get an error message:

<pre class="wp-block-preformatted">Bearer+client_id="00000002-0000-0ff1-ce00-000000000000",+trusted_issuers="00000001-0000-0000-c000-000000000000@<REDACTED>",+token_types="app_asserted_user_v1+service_asserted_app_v1",+error="invalid_token"  
2000008;reason="**The+token+should+have+valid+permissions+or+linked+account+associated+with+partner+application+'00000003-0000-0000-c000-000000000000**'.";error_category="invalid_grant" </pre>

This error message suggested that there is something wrong with the configuration. The key word here is _partner application_. If you do some search, you can find out that Partner Applications are used to allow access to the Exchange's REST API.

If you call _[Get-PartnerApplication](https://docs.microsoft.com/en-us/powershell/module/exchange/organization/get-partnerapplication)_ commandlet via PowerShell, you will end up with a list of two apps by default in hybrid setup - Exchange Online and Microsoft Graph. If you list the properties of Microsoft Graph, you end up with following:

```
RunspaceId                          : Redacted
Enabled                             : True
ApplicationIdentifier               : 00000003-0000-0000-c000-000000000000
CertificateStrings                  : {}
AuthMetadataUrl                     :
Realm                               :
UseAuthServer                       : True
AcceptSecurityIdentifierInformation : True
LinkedAccount                       :
DeploymentId                        :
IssuerIdentifier                    :
AccountType                         : OrganizationalAccount
AppOnlyPermissions                  :
ActAsPermissions                    : {Mail.Read, Mail.Write…}
AdminDisplayName                    :
ExchangeVersion                     : 0.20 (15.0.0.0)
Name                                : Microsoft Graph
DistinguishedName                   : CN=Microsoft Graph,CN=Partner Applications,CN=Auth Configuration,CN=EXLAB,CN=Microsoft Exchange,CN=Services,CN=Configuration,DC=exlab,DC=labs,DC=tntg,DC=cz
Identity                            : Microsoft Graph
Guid                                : Redacted
ObjectCategory                      : exlab.labs.tntg.cz/Configuration/Schema/ms-Exch-Auth-Partner-Application
ObjectClass                         : {top, msExchAuthPartnerApplication}
WhenChanged                         : 11\. 5\. 2019 18:45:51
WhenCreated                         : 8\. 5\. 2019 22:32:46
WhenChangedUTC                      : 11\. 5\. 2019 17:45:51
WhenCreatedUTC                      : 8\. 5\. 2019 21:32:46
OrganizationId                      :
Id                                  : Microsoft Graph
OriginatingServer                   : ex01.exlab.labs.tntg.cz
IsValid                             : True
ObjectState                         : Unchanged</pre>
```

In the configuration above,the two most important line of all for us is _AppOnlyPermissions_ which has no configuration. Now this is where it gets confusing, _[Set-PartnerApplication](https://docs.microsoft.com/en-us/powershell/module/exchange/organization/set-partnerapplication?view=exchange-ps)_ can set the property, however according to the docs, it is reserved to Microsoft's internal use. However, that doesn't mean that it won't work!

```powershell
$apps = Get-PartnerApplication
# Microsoft Graph is 2nd item in the array, if you are unsure, list the items by calling $apps first
$apps[1] | Set-PartnerApplication -AppOnlyPermissions $apps[1].ActAsPermissions
```

Simply running the PowerShell above is going to take the _ActAsPermissions_ and set it to _AppOnlyPermissions_. All you have to do then is to restart IIS and you should be good to go with using client_credentials flow!

Honestly, I don't understand why this is not set by default - maybe there are some security reasons behind this, but all in all, you can use client_credentials with hybrid deployments now!