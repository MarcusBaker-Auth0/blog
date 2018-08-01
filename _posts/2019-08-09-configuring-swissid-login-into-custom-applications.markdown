---
layout: post
title: Configuring SwissID Login into Custom Applications
description: Learn how to support SwissID login into your custom applications with ease.
date: 2018-07-24 18:30
category: Technical Guide, Identity
author:
  name: Peter Fernandez
  url: https://twitter.com/HandsumQbn
  mail: peter.fernandez@auth0.com
  avatar: https://cdn.auth0.com/blog/guest-authors/peter-fernandez.png
design:
  bg_color: #4236c9
  image: https://cdn.auth0.com/blog/ml-kit-sdk/android-ml-kit-machine-learning-sdk-logo.png
tags:
- swissid
- oauth
- openid
- auth0
- social-connection
related:
- 2015-12-16-how-to-use-social-login-to-drive-your-apps-growth
- 2017-11-06-authenticated-identity-trusted-key-auth0
---

[SwissID](https://swissid.ch) is the standardized digital identity initiative for use across Switzerland. This initiative is a joint venture of state-affiliated businesses, financial institutions, and health insurance companies (including [Swiss Post](https://www.post.ch/en) and [SBB](https://www.sbb.ch/en/)) that provide a third party [Identity Provider](https://auth0.com/docs/identityproviders) service to application vendors. With SwissID, users retain exclusive ownership over their personal data making online transactions easier to process and, more importantly, more secure.

Usually, application vendors implementing support for SwissID would require a working knowledge of the [OAuth 2.0 protocol](https://auth0.com/docs/protocols/oauth2) and of the [OpenID Connect specification](https://auth0.com/docs/protocols/oidc). However, as [Auth0](https://auth0.com) now supports SwissID as a [Custom Social Connection](https://auth0.com/docs/extensions/custom-social-extensions), developers can easily integrate this identity provider into their applications. Also, alongside with SwissID, can support myriad of other Identity Provider services (such as Facebook, Google, Twitter, and LinkedIn) as well as add capabilities such as [Single Sign On](https://auth0.com/docs/sso/current) and [Multifactor Authentication](https://auth0.com/docs/multifactor-authentication).

## Signing up to SwissID

As an application vendor (a.k.a. a [Relying Party](https://auth0.com/identity-glossary#r)) you will first need to sign-up to SwissID. This process starts by [contacting SwissID in order to become a Business Partner](https://www.swissid.ch/en/business-partners#become-a-part-of-a-success-story). After the sign up with SwissID, you will have to provide information about your redirection URI’s. In return, SwissID will provide you a Client ID and a Client Secret. Keep handy this information as you will need both values while setting up SwissID within you Auth0 tenant.

## Signing up to Auth0

If you don't have an Auth0 account yet, <a href="https://auth0.com/signup" data-amp-replace="CLIENT_ID" data-amp-addparams="anonId=CLIENT_ID(cid-scope-cookie-fallback-name)">sign up for a free one right now</a>.

As part of [the free plan](https://auth0.com/pricing), you will get things like:
- 7,000 active users.
- Up to 2 [social identity providers](https://auth0.com/docs/identityproviders).
- Access to [Auth0's Passwordless feature](https://auth0.com/passwordless).

![Auth0 free plan](https://cdn.auth0.com/blog/swissid/auth0-free-plan.png)

While signing up to Auth0, you will have to define a _Tenant Domain_ (e.g., `your-company.auth0.com`) and you will have to choose a _Region_ (at the time of writing, three regions are available: US, Europe, and Australia). After that, you can complete your account creation and Auth0 will redirect you to your dashboard.

![Auth0 dashboard](https://cdn.auth0.com/blog/secure-your-gaming-company-with-auth0's-user-fraud-score-and-minfraud/auth0-dashboard.png)

## Setup a SwissID Custom Social Connection in Auth0

You will also need to setup install the Custom Social Connections extension with your Auth0 tenant (https://auth0.com/docs/extensions/custom-social-extensions). The Custom Social Connections extension provides an easy way to manage social connections within Auth0 that are not supplied as ‘first class citizens’ out-of-the-box.
Once the Custom Social Connections extension is installed it can be used to setup a SwissID Custom Social Connection within Auth0. From the Custom Social Connections dialog click + New Connection to add a new connection:

This will bring up the New Connection dialog which can then be used to configure your SwissID connection as described in the section below.

## Configuring the SwissID Custom Social Connection in Auth0

Once SwissID sign-up is complete and you have your SwissID registration details, you can configure the SwissID Custom Social Connection within Auth0. Creating a new Custom Social Connection (as described in the section above) will provide a New Connection dialog in which the configuration information for SwissID can entered as shown below: 

The Name of the connection of the connection obviously reflects the fact that it is for SwissID, but can be something else if you so desire. Client ID and Client Secret will typically be provided as part of SwissID registration details; Client ID is typically supplied as-is, and Client Secret is typically supplied as Password. 
The Fetch User Profile Script is expanded below. The function declared in the script is fully customizable JavaScript, and is used to obtain the various supported claims from SwissID in order to build the profile for a User. SwissID provide a reference application which can be used to explore what claims are available, and also some of the other functions that are provided; their reference application it is located at:  https://login.int.swissid.ch/swissid-ref-app

## Testing the SwissID Custom Social Connection in Auth0

Once configured the TRY button can be used to verify that the configuration is successful; pressing it will typically present the SwissID login page, and once your credentials have been successfully presented a page similar to the following will be displayed by Auth0:

You can then use the Apps tab on the Connection page to enable the applications you want to use with SwissID (see https://auth0.com/docs/extensions/custom-social-extensions for further information).

## Configuring Auth0 Lock to use SwissID

By default, the out-of-box hosted Login page (which by default uses the Lock widget; https://auth0.com/docs/hosted-pages/login) will automatically display any and all connections enabled for an application. For out-of-box, first class connections – such as Facebook or Google – the Lock widget will automatically display an appropriate icon for the connection. However for custom connections – such as Swiss ID – only a default icon is displayed. 
However, Lock supports configuration options that can be used to tailor the look and presentation for any connection on either the login or the registration dialog. This allows you to host a logo for SwissID on your own CDN, and then tell Lock to use this logo – as in the following example (see https://auth0.com/docs/hosted-pages/login#how-to-customize-your-login-page for further details on customizing Lock and the hosted Login page):   

{% include asides/about-auth0.markdown %}
