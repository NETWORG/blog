---
author: Jan Hajek
title: Azure AD authentication for SSH
slug: azure-ad-authentication-for-ssh
id: 114
date: '2018-04-23 08:00:35'
categories:
  - Azure
  - English
  - Open Source
tags:
  - Azure Active Directory
  - Azure AD
  - Linux
  - Single Sign On
  - SSH
  - SSO
---

To be honest, managing authentication in Linux for multiple users/admins can be a huge pain. Different companies use various tools - generally, they use a centralized tool to distribute developer's SSH keys. This can still be a pain, however if the company has Azure AD (or Office 365), why not to use those accounts for authentication? _One of the SSH key distribution tools is [Teleport SSH server](https://github.com/gravitational/teleport) for example._

# Leveraging Device Code flow

If you ever used [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) (or logged in to YouTube/Spotify/Facebook on Xbox), you have already experienced a device code flow. Basically, if the client cannot offer interactive login session, the server is going to generate a short-lived (15 minute) single use code, which you can use on another device to authenticate towards the requested resource. _Device code is explained in-depth [here](https://joonasw.net/view/device-code-flow) for example or refer to the [IETF's draft](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-07)._ There are [some solutions in the wild](https://github.com/uberguru/azure-ad-ssh-pam/blob/master/aad-login.js), which offer Azure AD authentication however, those don't support MFA enabled users since they expect you to type your password directly into the SSH console and then try to authenticate with it in the plaintext (using _password_ grant). Due to the password input and lack of support for MFA, I didn't really like those solutions. Then suddenly, I discovered [Cyclone Project's implementation fo the PAM authentication](https://github.com/cyclone-project/cyclone-python-pam/). Built in Python and leveraging [_pam-python_](http://pam-python.sourceforge.net/) project, I decided to re-use some parts of their code. Their federation server doesn't directly support the device code flow, so what they do is that they boot up a tiny web server which you then access from your device and use it to login which actually mimics the device code flow in Azure AD (Azure AD leverages polling rather than running a local webserver). Thanks to Azure AD, I could remove most parts of their code and leave the heavy-lifting to [ADAL for Python](https://github.com/AzureAD/azure-activedirectory-library-for-python).

## Support in other apps

One of the very nice features is that this flow is even supported in applications like [WinSCP](https://winscp.net/)! Thanks to that, your users can also access the files using SFTP protocol. [![](/uploads/2018/04/ssh-aad-300x244.png)](/uploads/2018/04/ssh-aad.png)

# Next Steps

Since we would like to deploy this solution to our production servers the next steps are to add support for creating user accounts (eventually add support for temporary accounts as well) for incoming users from Azure AD and some advanced RBAC support.

# Try it out!

You can find a [working **Proof of Concept** on our GitHub](https://github.com/TheNetworg/ssh-aad). Please note that the code is not production ready, however I would welcome any Pull Requests and improvements.