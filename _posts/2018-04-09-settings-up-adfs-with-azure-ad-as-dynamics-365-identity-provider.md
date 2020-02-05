---
author: Jan Hajek
title: Setting up ADFS with Azure AD as Dynamics 365 Identity Provider
slug: settings-up-adfs-with-azure-ad-as-dynamics-365-identity-provider
id: 30
date: '2018-04-09 08:00:55'
categories:
  - Azure
  - Dynamics 365 / CDS / PowerApps
tags:
  - Active Directory
  - Active Directory Federation Services
  - Azure AD
  - Dynamics 2016
  - Dynamics 365
  - Single Sign On
  - SSO
---

[In previous article](/2018/04/03/using-azure-active-directory-for-sso-with-dynamics-365-on-premise/), we have looked at the possibility to connect Dynamics 365 on-premise directly with Azure AD, which is on one hand really cool, on the other, it doesn't provide all the features like mobile apps integration. In this article, we are going to explore a production ready solution by leveraging Active Directory Federation Service and Azure AD as a Claims Provider Trust. Starting off, I am going to assume you already have ADFS installed and set up with Dynamics 365, if not I suggest starting [here](https://technet.microsoft.com/en-us/library/gg188595.aspx).

# Identities in Dynamics 365

Basically, you have two kinds of identities in Dynamics 365 - identities sourced from Active Directory and identities sourced from a federation provider. By default, and in most simple deployments with ADFS, you use the first one - user has an account in Active Directory and is created in Dynamics 365 with username like _AD\Jan Hajek_ which is their logon name into the AD. What happens on background in this case, is that Dynamics has a mapping table in _MSCRM_CONFIG_ database, which is called _SystemUserAuthentication_ and it basically maps the user's SID to the identity in Dynamics 365\. That is why it is super important, to include the user's SID in the claims (as the tutorial mentioned above states), else, even if you include the UPN or logon name, the user won't get mapped. Second option is sourcing the identity from a federation provider - ADFS. If you are a larger company which for example allows external users from other organizations to access their Dynamics instance, you have two options - either go with the option above and create a special account for that user in your directory - which may have certain negative security impact (like if the person gets fired from their company and their IT forgets to mention it, they still get to keep the account with all permissions etc.) or you can add their ADFS instance as a trusted claims provider for the Dynamics 365 relying party. The user will then get a prompt to choose which organization they belong to and be authenticated by proper ADFS instance. In this case, the user doesn't have to exist in your Active Directory, their account is only in the specific Dynamics 365 instance's database and no Active Directory is involved on your side for user authentication. Like mentioned above, the same _SystemUserAuthentication_ table is used to map the UPN (User Principal Name) to the correct Dynamics account.

# Why integrate with Azure AD?

I have already mentioned this in the previous article, but to recap - Azure AD adds enhanced security, logging and much more to the authentication and authorization process. It can help you easily enforce conditional access, multi-factor authentication and others while using it as your primary identity provider. Thanks to this, you don't have to use Azure AD Connect to synchronize users from your Active Directory to Azure Active Directory and therefor you don't need to keep users in your Active Directory at all and use pure cloud-only Azure AD instance. You also get a great set of [B2B functionalities](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b).

# Why is ADFS still needed?

This has been already mentioned before - however to recap this as well: Dynamics can directly integrate with Azure AD so no ADFS is needed, however, you will loose the ability to use both desktop and mobile applications because their application IDs are reserved for Dynamics 365 Online usage.

# What needs to get changed?

We are going to start off by adding Azure Active Directory as a claims provider trust into ADFS. It is basically reverse process of configure Azure AD to accept ADFS as the claims provider. Here's what you have to do:

1.  Create a new application in Azure AD using [Azure Portal](https://portal.azure.com) or [Azure AD Portal](https://aad.portal.azure.com).
2.  Configure the Application Identifier to your ADFS's address, for example: _https://sts.crmlabs.tntg.cz_
3.  Configure _Reply URLs_ for the application to include your ADFS's address - for example: _https://sts.crmlabs.tntg.cz/adfs/ls_
4.  _Tip:_ You may want to set the _Home page URL_ in application's _Properties_ to your CRM address - for example _https://1box-03.crmlabs.tntg.cz_ in order for the user to be redirected to your Dynamics instance when clicking the application in their App Launcher or My Apps.
5.  In ADFS console, create a new _Claims Provider Trust_.
6.  Choose _Import data about the claims provider..._ and put in your directory's Federation Metadata's address - for example: _https://login.microsoftonline.com/thenetw.org/FederationMetadata/2007-06/FederationMetadata.xml_ (you can get this endpoint for your tenant using _Endpoints _button on the _App Registration_ page in AAD). [![](/uploads/2018/04/Dynamics_ClaimsProviderTrust-300x246.png)](/uploads/2018/04/Dynamics_ClaimsProviderTrust.png)
7.  Specify a Display name, for example _Azure AD_ and add the trust.
8.  Now you can use Azure AD as a claims provider in your ADFS.
9.  One more thing that you need to do is to configure the UPN claim - since [Azure AD is not going to send it to you](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization#restricted-claims), because without it, Dynamics wouldn't identify the user correctly (alternatively you could modify _IdentityClaim_ in Dynamics database like mentioned in previous article).
10.  So after having the Trust created, go ahead and click _Edit Claim Rules..._
11.  Add a new rule from template _Transform an Incoming Claim_ and set _Incoming claim type_ to _Name_ and _Outgoing claim type_ to _UPN_. [![](/uploads/2018/04/Dynamics_ADFS_Rule-279x300.png)](/uploads/2018/04/Dynamics_ADFS_Rule.png)

## Configuring relying party trust

If you have set up the [Relying Party correctly](https://technet.microsoft.com/en-us/library/gg188595.aspx), you already have the UPN passthrough rule created for the relying party. If not, look at [Microsoft's tutorial](https://technet.microsoft.com/en-us/library/gg188595.aspx). Now we have to enable the Relying Party Trust to accept claims from Azure AD, we will have to use PowerShell (on the ADFS server) for this. First, we are going to check your relying party configuration:

<pre class="lang:ps decode:true">Get-ADFSRelyingPartyTrust -Name "<your CRM IFD Relying Party Name>"</pre>

When you check _ClaimsProviderName_ property, you should have _Active Directory_ there. Now if you want to add Azure AD alongside AD, you would execute:

<pre class="lang:default decode:true ">Set-AdfsRelyingPartyTrust -TargetName "<your CRM IFD Relying Party Name>" -ClaimsProviderName @{add="Azure AD"}</pre>

If you want to use exclusively Azure AD only, you would execute following, however, beware before executing this (read below):

<pre class="lang:default decode:true ">Set-AdfsRelyingPartyTrust -TargetName "<your CRM IFD Relying Party Name>" -ClaimsProviderName @("Azure AD")</pre>

  Before you use exclusively Azure AD, you should modify at least one system administrator's User Name in Dynamics to match their User Principal Name in Azure AD, so for example: _Jan Hajek@thenetw.org_, else no one will be able to sign in. Generally, you should change your user's User Names in Dynamics first before migrating to Azure AD login. If you have same UPNs in Azure AD and Active Directory, their logins should work both logging through Azure AD and Active Directory claims providers. [![](/uploads/2018/04/Dynamics_FederatedUser-300x168.png)](/uploads/2018/04/Dynamics_FederatedUser.png)