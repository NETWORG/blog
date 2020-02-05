---
author: Jan Skala
title: Logic App in a multi tenant environment
slug: logic-app-in-a-multi-tenant-environment
id: 489
date: '2019-08-20 07:34:29'
categories:
  - Azure
  - English
  - Microsoft Flow
tags:
  - Azure Logic Apps
---

Since 2018 I am playing with Logic Apps. All that time I have been thinking about the best approach for authentication and authorization. There are certain scenarios where the standard approach (OAuth2, basic, static API key) are not ideal. Consider sending REST API call to someone else's Azure Active Directory. You can use HTTP with Azure AD, but you need to have someone's username and password, which is not the nice way of doing this.

## Calling Microsoft Graph

Imagine you are building an integration, that needs to get all users from a certain Azure Active Directory. Well that's easy right? We just need to call:

    GET
    https://graph.microsoft.com/v1.0/users

Ok, but how to authorize against it without someone creating us an account. We can use Active Directory OAuth Authentication. For that, we will create an Application Registration.

## Register your application in your Azure Portal

Add new app registration. Specify the name of your app, optionally you can also specify the Redirect URI.

<figure class="wp-block-image">![Add new App registration](/uploads/2019/08/image-2-1024x500.png)

<figcaption>Add new App registration</figcaption>

</figure>

Don't forget to make your app Multitenant. It is really up to you if you allow personal Microsoft accounts, it depends on your use case.

![](/uploads/2019/08/chrome_2D61LJb9B6.png)

Supported account types

## Approve the app

If you want your customer to approve, that your app can access his or her tenant, you can send him a link that will request an admin consent. Keep in mind, that only an admin can approve that. Every time you change permissions that your app requires, the admin consent must be approved again, for obvious reasons.

    https://login.microsoftonline.com/common/oauth2/authorize?client_id={your_app_client_id}&redirect_uri=https%3A%2F%2Ftalxis.com&response_type=code%20id_token&scope=openid%20profile&response_mode=form_post&nonce=static_nonce&prompt=admin_consent&state=static_state&sso_reload=true﻿

In this link, replace {your_app_client_id} with your application id. You can find that in Azure portal. You can even setup _**redirect_uri**_ which is basically telling where to redirect after successful login, but that is up to you.

<figure class="wp-block-image">![Application client id ](/uploads/2019/04/image.png)

<figcaption>Application client id</figcaption>

</figure>

## Use the app

First, we need to get client credentials for our app. You can either use Client secrets or Certificates. I will use client secrets for the sake of this demo. We can get them from App registrations.

<figure class="wp-block-image">![Generate client secrets](/uploads/2019/08/image-1024x535.png)

<figcaption>Generate client secrets</figcaption>

</figure>

Now in order to use the credentials in a standard connector, you need to switch to Active Directory OAuth. And then fill in the client credentials. If you are about to use Microsoft Graph, make sure to specify the Audience as well.

<figure class="wp-block-image">![Use client credentials](/uploads/2019/08/image-1-1024x596.png)

<figcaption>Use client credentials</figcaption>

</figure>

Now you can easily get your app approved and don't need anyone to give you username or password. Also this approach is more suitable for cases when you need some organization wide data, like reading all calendars or modifying all contact lists.

One last note. Sometimes it takes time before the new app permissions are propagated into all Microsoft Graph replica servers.