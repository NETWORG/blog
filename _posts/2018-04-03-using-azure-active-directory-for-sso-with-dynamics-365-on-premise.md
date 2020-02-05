---
author: Jan Hajek
title: Using Azure Active Directory for SSO with Dynamics 365 On-Premise
slug: using-azure-active-directory-for-sso-with-dynamics-365-on-premise
id: 7
date: '2018-04-03 08:00:59'
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

While Dynamics 365's documentation is full of articles and tutorials about setting it up with Active Directory Federation Services, there is no mention of using Azure Active Directory for Single Sign On. Many replies in communities say that this is not possible, but today we are going to prove them wrong. As you might have guessed from the intro, using Azure Active Directory for authentication is possible even with Dynamics 365 on-premise. Which we are going to explore in this article.

> Please note, that this is more of a Proof of Concept article and due to the limitations we discovered along the way, this method is not recommended for production.

# Why use Azure AD directly?

There may be many reasons: First off, it provides more security, advanced attack protection methods, auditing, logging and [much, much more](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-whatis). Next, even tho that Dynamics 365 installation requires Active Directory, you don't need to use AD as an identity provider (except for the Deployment Administration) nor store users there (yes, for real! Dynamics doesn't have to source users from Active Directory like many of the forums incorrectly state). Removing user's dependency on Active Directory can allow you to deploy cloud-only IdP with on-premise Dynamics 365\. And one more important thing - thanks to Azure AD, you can have Internet Facing Deployment (IFD) of Dynamics 365 without having to expose (or even run) your ADFS to the internet while being protected by Azure AD.

## Existing solutions

Like I already mentioned, there are some solutions existing already, however none of them integrate Azure AD directly. I am going to shortly describe the alternative scenarios. [**Azure Access Control Service (ACS)**](http://teameasi.com/blog/dynamics-crm-using-azure-active-direction-instead-of-adfs) This was probably the best solution, however since [ACS is discontinued](https://azure.microsoft.com/en-us/blog/time-to-migrate-off-access-control-service/), you cannot use it anymore. **Using independent ADFS with Azure AD as Claims Provider** This solution is something we have being doing internally as well. Basically an independent ADFS deployment which has Azure AD configured as a Claims Provider (so ADFS acts like a proxy). There are few downsides of this - for example, you will have hard time extending login expiration period, since it is being inherited from Azure AD token. This is what we use in production.

# Getting started

There are two paths for getting this deployed. First is migrating from existing Claims Based Authentication setup with ADFS and second (trickier) is getting a vanilla deployment of Dynamics 365 setup with Azure AD. We are going to start with the common setup - registering the Dynamics 365 instance into Azure Active Directory:

1.  Navigate to [Azure Portal](https://portal.azure.com) and select _Azure Active Directory_ or alternatively use [Azure AD Portal](https://aad.portal.azure.com) directly.
2.  Select _Enterprise Applications_ and from _Add your own app_ create a _Non-gallery application_ and create it with your preffered name (I will be using _1box-01.crmlabs.tntg.cz_).
3.  Once you create the application navigate into the properties (there you can set a Logo for the users for example), and optionally turn off _User assignment required_ option. This is going to allow everyone from your AAD to authenticate with Dynamics 365 (you can keep it on if you want to assign users to it manually or use group assignment).
4.  Next navigate to _Single sign-on_ from the left menu. Select _SAML-based Sign-on_ from the drop down.
5.  There, you have to set the application's identifier and reply URL. Both should have the same value depending whether you are setting up the IFD or not:
    *   **IFD: **You should have _https://auth.your-ifd-address.tld _there (you will have to do few more steps in the IFD specific section).
    *   **Non-IFD:** You should have your instance address there, so for example: _https://1box-01.crmlabs.tntg.cz_
6.  Next, download the _Metadata XML_ from the _SAML Signing Certificate_ section. You will want to upload that to your web server so it can be accessible by the Dynamics instance. Note the URL down, because you are going to need it in the next step (for me it was _https://share.hajekj.net/1box-01.crmlabs.tntg.cz/FederationMetadata.xml_). And save the changes.

Now, we are going to proceed to the deployment specific configuration.

## Setting up Claims Based Authentication without IFD

This is the starting point if you are creating a new Dynamics 365 deployment. The pre-requisities are:

*   Dynamics 365 deployed with Windows Authentication login
*   Enabled HTTPS for Dynamics 365 deployment
*   [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) or [SQL Operations Studio](https://docs.microsoft.com/en-us/sql/sql-operations-studio/download) if you feeling experimental
*   Application created in Azure AD (which we did in previous step)

This part is a bit tricky, so bear with me on this one. What we need to achieve is to set AAD as a Claims Based Authentication provider while no local users exist in your AD. By default, Dynamics 365 stores user names in _DOMAIN\alias_ format (for example _AD\hajekj_, my UPN is _hajekj@ad.crmlabs.tntg.cz_), however I haven't found a way to force login through AAD with the NT logon name, so we have to do following:

1.  Now, we are getting to the tricky part: Open an _In-Private_ _browser window_, navigate to your Dynamics 365 instance and login using Windows Authentication. Then navigate to the User management under _Settings > Security_ _> Users_.
2.  On the server, open the Dynamics 365 _Deployment Manager_. From the left menu, choose _Configure Claims-Based Authentication_.
3.  Stepping through the wizard, enter the address of the _FederationMetadata.xml_ you uploaded before. [![](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_ConfigureClaimsBasedAuthentication-300x295.png)](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_ConfigureClaimsBasedAuthentication.png)
4.  Choose the encryption certificate (usually, the same certificate you are using for HTTPS).
5.  Once you apply the changes, switch to the In-Private browser window you opened before and choose to create a new user. You may get few Windows Authentication prompts, just skip them and continue filling out the details.
6.  Fill out the UPN of a user from AAD which you will use for Administrator - _Jan Hajek@thenetw.org_. Fill out the user's Full Name and save (optionally set _CAL _related information depending on your licensing). Then, you have to assign _System Administrator_ role to the user so you can sign-in and perform administrative tasks. [![](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_FederatedUser-300x168.png)](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_FederatedUser.png)
7.  _Note:_ If you now try to login, you are going to end up in a redirect loop. If you enable trace logging, you are going to fing out the error relates to an exception being thrown by username being undefined. This is caused by the fact, that Dynamics expects the username to be passed in _http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn_ claim, however, Azure AD is unable to send it because it is [part of restricted claims](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-saml-claims-customization#restricted-claims) for some reason whereas ADFS has no issue with sending the claim. So we have to override the _IdentityClaim_ configuration in the database. So far, I am not aware of this change having effect on any Dynamics functionality so you should be safe, however, if you are unfamiliar with SSMS, I suggest you backup the database, snapshot the server or something so you can revert the change.
8.  Next, open up your SSMS, connect to your Dynamics 365 SQL instance and open _MSCRM_CONFIG_ database. Find the table named _dbo.FederationProvider_, right click and choose _Edit Top 200 Rows_.
9.  Find the row which is named _IdentityClaim_ and change the value for all providers to _http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier_. This is going to make Dynamics to source the username from a different claim.
10.  Now once you save the changes and restart the site (from Internet Information Services for example), you will be open to open the Dynamics and login with the account which you created in step 8.
11.  Next, under this account, open the Users page again (_Settings > Security > Users_) and modify the original user's User Name to their UPN (so _hajekj@ad.crmlabs.tntg.cz_ in my case).
12.  You shouldn't have to use that account since your Azure AD account is already a System Administrator, however it may become necessary at some point (for example if you need Deployment Manager permission for deploying certain solutions).
13.  _Optional:_ In order to access CRM as the original account sourced from Azure AD, you have two options: Disable Claims-Based Authentication temporarily, which will return back to Windows Authentication (which will prevent AAD users from signing in), however when turning it back on, you just click through the wizard and the configuration including the _IdentityClaim_ is going to stay.
14.  Now you can create your users with their AAD User Principal Names and you are good to go. The user doesn't have to exist in the Active Directory, since Dynamics will treat them as Federated users which they are anyways.

## Modifying existing IFD deployment

The beginning and most of the steps are going to be similar as above, so I am going to refer to those by numbers so that I don't have to copy paste same text multiple times.

1.  First, go to your User administration (_Settings > Security > Users_) and verify whether users have their usernames set to their User Principal Names in AAD. If not, either modify existing user (who has System Administrator permissions) or create a new user (step 8 above).
2.  Next, go to your Dynamics 365 Deployment Manager and choose _Configure Claims-Based Authentication_. Use the metadata you uploaded in the _Getting Started_ section and generally follow steps 2, 3, and 4 from above. You can keep the encryption certificate the same you used before.
3.  Once finished, restart your Dynamics 365 through IIS. If you then head to your IFD address - _https://auth.your-ifd-address.tld_, you should authenticate with Azure AD and access your default instance. However, you are very likely to have more instances in separate subdomains - _https://prod.*_, _https://dev.*_ etc. If you try to access those, you will get an _AADSTS70001_ error from Azure AD stating that the identifier is invalid. Adding those as identifiers is the tricky part.
4.  In Azure Portal, navigate to _App Registrations_ tab (on the same level as _Enterprise Applications_), from the dropdown, choose _All apps_ and from the list select the app you created above (_1box-02.crmlabs.tntg.cz_ in my case) and select _Manifest_. In the manifest, find _identifierUris_ and _replyUrls_ and add all known addresses to the JSON lists. Leave the other identifiers and URLs as is. I don't suggest modifying anything else in the manifest, since you could break the application. [![](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_AAD_CustomManifest-294x300.png)](http://blog.thenetw.org/wp-content/uploads/2018/04/Dynamics_AAD_CustomManifest.png)
5.  Save the manifest and you should be good to go.
6.  Next step would be to modify existing user's User Names in Dynamics so they can access it and it's done.

# Summary

In this article, I have demonstrated how to setup both non-IFD and IFD deployments of Dynamics 365 with Azure Active Directory as Claims-Based Authentication Provider directly, which can reduce your infrastructure overhead. Additionally, I suggest looking at [Dynamics 365 Online](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/admin/try-dynamics-365-online) offering from Microsoft which is a hosted solution which communicates with Azure AD directly and therefor makes a lot of things much easier. Next time, we might explore the ADFS > Azure AD setup with Dynamics on-premise.

## Downsides

So far we discovered one downside with this solution, but they may be more. Like I have already mentioned above, this shouldn't be used for production. **Mobile Applications** If you plan to use Dynamics 365 mobile/Outlook applications, they are not likely to work. The issue is that they require to be registered with the identity provider, which is actually impossible with AAD, since Microsoft has claimed those [application IDs](https://technet.microsoft.com/en-us/library/hh699726.aspx#Configure%20Windows%20Server%202012%20R2%20for%20Dynamics%20365%20applications%20that%20use%20OAuth) for Dynamics Online obviously and you cannot integrate with your own application. You could probably work around it with URL rewrite - rewrite the _clientId_ to a clientId of yours back and forth, but I think it could be problematic.