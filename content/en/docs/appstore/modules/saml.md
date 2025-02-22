---
title: "SAML"
url: /appstore/modules/saml/
category: "Modules"
description: "Describes the configuration and usage of the SAML module, which is available in the Mendix Marketplace."
tags: ["marketplace", "marketplace component", "saml", "IdP", "identity provider", "platform support"]
#If moving or renaming this doc file, implement a temporary redirect and let the respective team know they should update the URL in the product. See Mapping to Products for more details.
---

## 1 Introduction

The [SAML](https://marketplace.mendix.com/link/component/1174/) module can be used to give end-users access to your Mendix application based on their identity in your Identity Provider (IdP). A Mendix application that uses the SAML SSO module will delegate user login to your Identity Provider using SAML 2.0.

By configuring the information about all identity providers in this module, you will allow the users to sign in using the correct identity provider (IdP). There is no limit on the number of different identity providers you can configure.

{{% alert color="info" %}}
Mendix also offers an [OIDC SSO](/appstore/modules/oidc/) module if you want to authenticate your end-users using the OAuth/OpenID Connect protocol.
{{% /alert %}}

### 1.1 Typical Use Cases

Examples of the use of the SAML module include the following:

* authenticating against your Microsoft Active Directory server in a secure manner utilizing the SAML capabilities of Active Directory Federation Services (ADFS) — the SAML protocol allows for the encryption of all information transferred between the two servers, so VPN connections, LDAP, or Kerberos authentication are no longer needed
* implementing SSO in your Mendix App through a Shibboleth identity Provider
* identifying the end-users of your Mendix app through SAML-enabled national identity schemes such as eHerkenning, a Dutch eID scheme for B2B or B2G scenarios, or DigiD, which gives Dutch citizens access to (semi) governmental services
    {{% alert color="info" %}}Some of these identity schemes use optional features of SAML which are not yet supported in the SAML SSO module — see [Limitations](#limitations) for more information{{% /alert %}}
* authenticating within a Mendix session — for example requiring end-users to re-authenticate shortly before they are allowed to do critical transaction in your app or having a second user authenticate within the context of the first user’s session in your Mendix app

### 1.2 Features

#### 1.2.1 SAML Protocol Adherence

The SAML SSO module supports the following [SAML 2.0](https://docs.oasis-open.org/security/saml/v2.0/saml-core-2.0-os.pdf) profiles for your Mendix app acting as a Service Provider (SP):

* Web browser SSO profile using one of the following bindings
    * HTTP redirect, 
    * HTTP POST bindings
    * Artifact binding for SAML responses
* Single Logout profile

The Mendix SAML SSO supports usage of SAML metadata in the following way:

* Daily synchronization of the IdP metadata, so your Mendix app will always have the latest IdP metadata.
    * For daily synchronization of IdP metadata, configure the `SE_SynchronizeIdPMetadata` scheduled event. For local development this can be done from Studio Pro. In the Mendix Cloud, you can do this on the [Environments Details](/developerportal/deploy/environments-details/#model-options) page for your app.
* Downloading of the metadata for your Mendix application that acts as an SP in the SAML protocol

For encryption of SAML messages the following options are supported:

* No Encryption
* 1024 or 2048 bit encryption
* SHA1 or  SHA256 algorithms

For easy configurability, the SAML module offers the following:

* a SAML administration screen that allows you to configure one or multiple SAML IdP’s. IdP discovery is supported by an endpoint that returns a page listing all configured IdPs so the end user can select the IdP where they have an account.
* various options as per the SAML 2.0 specification and as indicated on this page

#### 1.2.2 Other Features

The SAML module keeps a log/audit trail of login attempts. These can be downloaded.

### 1.3 Limitations{#limitations}

The Mendix SAML SSO module does not support the following:

* SAML1.0
* Enhanced Client/Proxy SSO profile
* HTTP artifact binding for SAML requests

Some SAML services, such as eHerkenning and DigID in the Netherlands, use optional features of SAML which are not yet supported by the Mendix SAML SSO module. These include:

* Signature included as a query string parameter in URL (for HTTP-REDIRECT)
* restriction of RelayState to 80 characters (i.e. SAML SSO may generate RelayState values that exceed 80 characters)
* ForceAuthn
* ProviderName
* Scoping
* RequestedAuthnContext in the SAML requests
* HTTP-SOAP Logout Request

If you need any of these features, contact your Mendix CSM to discuss inclusion of these features on the Mendix roadmap or customization of the SAML SSO module.

If you want to connect your app to multiple SAML IdPs, you cannot use different key pairs and certificates for each of the SSO federations. Instead, you must use a single key pair and certificate for all SAML IdPs. The certificate can be either a self-signed certificate or a certificate issued by a Certificate Authority (CA) (see [Use a Certificate Issued by a Certificate Authority](#use-ca) for more details)

The URL for downloading the SP metadata of your app is independent of the value of the EntityID that you configure for your app (see [Configuring Service Provider](#configure-sp)) and which is included in the SP metadata. Instead, the metadata URL is based on the alias for the connected IDP where the SP metadata will be used.

### 1.4 Dependencies{#dependencies}

{{% alert color="warning" %}}
If you are running your app outside of the Mendix Cloud, make sure you have [external file storage](/refguide/system-requirements/#file-storage) configured. The SAML module writes configuration data to a file document on the file storage to read it later. Without external file storage, this configuration will be lost when you restart your app. The SAML module will not work correctly without reading the configuration data from the file storage.
{{% /alert %}}

* [Mx Model Reflection](/appstore/modules/model-reflection/)

There are different versions of the module, depending on which version of Mendix you are using. These versions may change, see the versions available in the [SAML module](https://marketplace.mendix.com/link/component/1174/).

* Mendix version 7 – SAML module version 1.17.1
* Mendix version 8 – SAML module version 2.3.1
* Mendix version 9 (upgraded from version 8) – SAML module version 3.3.8
* Mendix version 9 (new app using Atlas version 3.0) – SAML module version 3.3.9

For Mendix 9, the versions for new apps (odd-numbered patch versions) differ from those for upgrading from Mendix version 8 (even numbered patch versions) as they have the newer version of Atlas UI (version 3). This is because using Atlas 3 with an app upgraded from Mendix version 8.x (which uses Atlas version 2) would result in issues because the templates for the SAML module pages would not exist.

## 2 Installation

1. Configure the **Startup** microflow to run as the startup microflow. This microflow will initialize the custom request handler `/SSO/` (please note the importance of using the final `/` for all instances of `/SSO/`), validate all IdP configurations, and prepare the configuration entities required during the configuration.
    {{% alert color="info" %}}If you have set up path based access restrictions in your cloud (for example [Path-Based Access Restrictions](/developerportal/deploy/environments-details/#path-based-restrictions) in the Mendix Cloud), ensure that access to `/SSO/` is allowed.{{% /alert %}}
2. Add the **OpenConfiguration** microflow to the navigation, and then allow the administrator to access this page.
3. Review and configure all the constants:
    * **DefaultLoginPage** – You can specify a different login page here for when the login process fails. When the end-user cannot be authenticated in the external IdP, a button will appear, and by clicking this button, they will be redirected to the specified login page. If this is left blank, an unauthenticated user will be redirected to `/login.html`.
    * **DefaultLogoutPage** – Removing the sign-out button is recommended, but if you choose to keep it, the end-user will be redirected to a page. You can choose where the end-user is redirected to (for example, back to `/SSO/` or your `login.html` page). Every user signed in via SAML is redirected to this location when they are logged out.
    * **SSOLandingPage** – Set this if you redirect the `index.html` to log into your app automatically. See [Using SSOLandingPage](#ssolandingpage) for further information about this.
    * **HybridAppLoginTimeOutInMinutes** – If you are using a [hybrid mobile](/refguide/hybrid-mobile/) app and have the `com.mendix.webui.HybridAppLoginTimeOut` [custom runtime setting](/refguide/custom-settings/#web-client-settings) configured to customize the expiration of mobile authentication tokens, you will have to set the value of the `HybridAppLoginTimeOutInMinutes` constant to match the value of the custom runtime setting. When you use the SAML module for SSO in your Mendix app, the authentication token is not created by the Mendix runtime, which uses the custom runtime setting. Instead, the authentication token is created by the Java code in the SAML module. This Java code does not have access to the custom runtime setting value, and thus requires the constant value to be set.
    
        Only check this if you are using SAML on a hybrid mobile app. Note that this functionality also requires mobile authentication tokens to be enabled in your [IdP Configuration](#additional-functionality) as well as changes to the hybrid app package as described in [How To Implement SSO on a Hybrid App with Mendix and SAML](/howto8/mobile/implement-sso-on-a-hybrid-app-with-mendix-and-saml/).

        * If you use the default login handler in your hybrid app, you must change the `com.mendix.webui.HybridAppLoginTimeOut` custom runtime setting to change the validity of the authentication token used by the hybrid mobile app.
        * If you use the SAML module in your hybrid app, you must change the `SAML20.HybridAppLoginTimeOutInMinutes` constant to change the validity of the authentication token used by the hybrid mobile app.
        * If you use both the default login handler and the SAML module in your hybrid app, you must change both.

        {{% alert color="warning" %}}Hybrid mobile apps are deprecated in Mendix version 9.{{% /alert %}}

4. Sign in to the application and configure the SAML module as described in the [Configuration](#config) section.

### 2.1 Using SSOLandingPage{#ssolandingpage}

You can use single sign on to automatically sign users in to your app by redirecting every user accessing `index.html` to the Mendix `/SSO/` endpoint. To do this, you need to add `<meta http-equiv="refresh" content="0;URL=/SSO/" />` to the `index.html` file.

{{% alert color="info" %}}
For Mendix version 9, there is no `index.html` file, so you need to create this file first. You can find instructions on how to do this in the [Customizing index.html (Web)](/howto/front-end/customize-styling-new/#custom-web) section of *Customize Styling*.
{{% /alert %}}

If you use this method, do not forget to set the **SSOLandingPage** constant to a value different than `index.html`. Otherwise, the app will come back to `index.html` which will be redirected again to single sign on, resulting in an endless loop. **SSOLandingPage** specifies a different landing page so the end-user does not end up on `index.html` again after a login attempt. We recommend that you change this constant to `/index3.html` and create an `index3.html` page in your `/theme` folder and copy contents of the original `index.html` (without the added redirect) into it. The authenticated end-user will then land on `index3.html` which will display the content of the app. If the user authentication fails, the user will be directed to the **DefaultLoginPage** instead. 

{{% alert color="info" %}}If you want to redirect users who have not yet signed in automatically to `/SSO/` when opening `index.html`, while still allowing users to open `login.html` directly and sign in using a local account, bypassing single sign on, then you should not add `<meta http-equiv="refresh" content="0;URL=/SSO/" />` to the `index.html` file as described above; instead, you should edit the `index.html` file by changing the URL within the `originURI` to `/SSO/`, for example: `document.cookie = "originURI=/SSO/" + (window.location.protocol === "https:" ? ";SameSite=None;Secure" : "");`. This cookie determines to which location the Mendix Client will redirect users when they need to sign in. If you have already signed in, you are not redirected again.{{% /alert %}}

## 3 Configuration{#config}

To configure SAML, start your app and navigate to the **OpenConfiguration** microflow which you added to the navigation.

You can now finish configuring your SAML module in your app by reviewing/updating the Service Provider (SP), and creating/updating the IdP configuration.

### 3.1 Configuring Service Provider{#configure-sp}

Before any IdP can be configured, you need to configure the SP, which is your current application. The SP configuration allows you to configure some basic information for the SP metadata file. This information will be be available in the IdP for the reference of the IdP administrator.

{{% alert color="info" %}}
The base URL used for the links in your SP metadata is determined by the **Application Root URL** [custom runtime setting](/refguide/custom-settings/#general) of your app. Change the value for this runtime setting to change the base URL of the links in your SP metadata. After changing the **Application Root URL** setting, you have to import the SP metadata into your IdP again.
{{% /alert %}}

You can choose what you want to enter for the **Entity Id**, **Organization**, and **Contact person**. The SAML module imposes no restrictions and doesn’t apply any validations. The SAML core specification recommends that you “use a URL containing its own domain name to identify itself“ as the value of the EntityID of your app.

* **Log available days** – If **Log SAML Requests** is checked in the IdP configuration, all login attempts are tracked in the **SAMLRequest** and **SSOLog** entities. This setting configures how long those records are kept before removing them. A scheduled event runs daily to remove all the files outside that date range. This value is mandatory. If it is set to 0, all records will be removed daily.
* **Use Encryption** – This setting controls the encryption and signing of messages being exchanged between your app (as an SP) and the IdP. This is in addition to the encryption provided by using a secure HTTPS connection. For security and privacy reasons it is recommended that you enable this. Limitations in your IdP may be a reason not to enable it, but this should typically not be the case.

    If you choose to use encryption, you will need to select the appropriate values:

    * **Encryption method** – `SHA1 - RSA` or `SHA256 - RSA`
    * **Encryption key length** – 1024 or 2048 bits

    Enabling encryption has the following effects on messages being exchanged:

    * all the messages between your app and the IdP will be encrypted, this requires the IdP’s metadata to contain a KeyDescriptor having ‘use’ value ‘encryption’ or empty — see [Identity Provider Metadata](#idp-metadata) for more information
    * all the messages between your app and the IdP will be signed. The SP metadata exported from your app will have `AuthnRequestSigned` with value `true`. This corresponds to IdPs whose metadata have `WantAuthnRequestSigned` with value `true`
    * in SAML SSO module versions 1.17.3 and above, 2.3.0 and above, and 3.3.0 and above, your IdP is expected to sign all SAML assertions sent to your app. The SP metadata will have `WantAssertionsSigned` with value `true`. Any assertions that are not properly signed will be rejected. SAML’s signature inheritance is supported as well; if the SAML response message is signed by the IdP, it is not necessary for the assertion to be signed as well

    See [Managing the Keys and Key Store](#keystore), below, for additional information and options related to encryption and signing keys.

{{% alert color="info" %}}
The SP metadata that you supply to the IdP is only available after you have configured the [IdP-specific settings](#idp-specific-settings) following the instructions below.
{{% /alert %}}

#### 3.1.1 Managing the Keys and Key Store{#keystore}

SAML implements encryption and signing using asymmetric keys. If encryption is enabled, all the certificates required for encryption are stored in the key store. When you choose **Use encryption** a key store is automatically created using the URL of the application, or the custom EntityID, and shown as the **Key store alias**.

The key-pair(s) for your app (as an SP) can be self-generated by the SAML module or you can upload keypair certificate(s) provided by your IdP. 

You can usually leave the key store settings as the default. The SAML module will generate distinct key pairs for signing and encryption. The self-generated key pairs are used for all IdPs, if your app is configured to use multiple IdPs.

However, there may be a requirement to use a specific key store.

{{% alert color="warning" %}}
Resetting the key store or uploading another key store will require you to export the SP metadata and import it to all applicable IdPs.

It is currently not possible to have different key pairs for communication with different IdPs; the same SP key pair will be used with all configured IdPs.
{{% /alert %}}

If you want to upload a key pair certificate, click **Upload** to upload a key store file. Use the **Entity Id** as the the alias of the key store.

Click **Reset** to return the key store settings to their defaults. 

Click **Download** to download the key store file and use it when configuring other SAML SPs.

{{% alert color="info" %}}
Remember to set the new key store password in the `KeystorePassword` constant of your app.
{{% /alert %}}

### 3.2 Configuring the IdP-Specific Settings{#idp-specific-settings}

Each IdP (entity descriptor) should have its own configuration set. Every IdP can be configured and enabled separately. All changes made in the configuration are immediately applied when you save the configuration. 

#### 3.2.1 Creating a New IdP Configuration

{{% alert color="warning" %}}
If you have multiple IdPs, please make sure each IdP has a unique **Entity descriptor**. If you add multiple IdPs with the same entity descriptor, you might experience unexpected behavior where a different SSO configuration is selected than the alias provided.
{{% /alert %}}

When creating a new IdP configuration, you are guided through a workflow to help you configure everything required for the IdP configuration. Each option in the workflow is explained below, and can be changed by editing an existing IdP Configuration.

Use **Previous** to go back to the previous dialog, or **Cancel** to abandon your changes. Click **Save** on the last step to save the configuration.

Upon completing these steps, you only need to send the SP metadata file to the IdP and have them configure the authentication on their end. The SAML module generates separate SP metadata for every connected SAML IDP (see [Configuring the IdP-Specific Settings](#idp-specific-settings)).  The SP metadata for your app can be obtained by clicking **Download SP Metadata** on the final configuration step to download the XML file or by opening `http://<Application Root URL>/SSO/metadata/<IDP-Alias>` for your app's URL.

{{% alert color="warning" %}}
The XML for the SP metadata is signed. If you make any changes to the metadata (even just opening it in an editor) this can mean that the signature no longer matches the content and the metadata will be rejected.
{{% /alert %}}

#### 3.2.2 General

The following settings apply to this IdP configuration:

* **Alias** – The alias for your IDP can be used in the URL of the application to indicate the IdP configuration that should be used during login. There are no validations on this field (except that it is required), but you should make sure that this alias is compatible with usage in an URL (meaning, no `/`, `&`, `?`, or special character that could get lost in the communication).
* **Log SAML Requests** – Determines whether all requests and login attempts should be logged and stored in an entity.

#### 3.2.3 Identity Provider Metadata{#idp-metadata}

The following settings are for the IdP metadata:

* **IdP Metadata Location** – The module is capable of re-importing all IdP metadata files on a daily basis. You can also choose to import the metadata from a file.
* **Metadata Overview** – This overview shows all the information that has been found in the IdP metadata information. It is usually not necessary to do anything here, but it can be useful in order to review the possible IdP and SP configuration options.

{{% alert color="info" %}}
If you want to automatically synchronize the IdP metadata, make sure the **SE_SynchronizeIdPMetadata** [scheduled event](/refguide/scheduled-events/) is enabled. This is in the **\_USE ME** > **Scheduled Events** folder of the SAML module.
{{% /alert %}}

If you need to change your identity provider metadata you can find more information in the [Configuration of SAML Binding](#saml-binding) section.

#### 3.2.4 User Provisioning 

The following settings control user provisioning:

* **Uses InCommon Federation Standard** – IdPs that use the InCommon standard often do not specify the assertion attributes. When following the InCommon standard, a fixed set of assertion attributes will be available to choose from later.
* **Identifying Assertion** (aka "Principal Key") – Specifies which of the assertion attributes identifies the user name.

    {{% alert color="info" %}}In Mendix version 7,  all user names passing through the SAML module are converted to lower-case before login and creation. We strongly advise you to convert all existing and new user names to lower-case as well.{{% /alert %}}

* **User Entity** – The Mendix entity in which you will store and look up the user account. Most often something like `Administration.Account`.
* **Attribute On** – Determines the attribute on which you want to do the lookup. This attribute will be compared against the passed **Identifying Assertion** (see above).
* **User Action** – The module will always search for the user, based on the **Identifying Assertion**. You can allow the module to create users with a predefined user role. If you allow the module to create users, it will automatically create a new user account if the user cannot be found.  If the module is not allowed to create users, it will present a message to the user stating that the login action was successful but no user has been configured. 
* **Default User Role** – This role will be assigned to newly created users.
* **Use Custom Logic in User Provisioning** – If you want to add your own logic to the user provisioning, enable this function and use [CustomUserProvisioning](#customuserprovisioning) to point to your custom logic. This microflow will be executed after the user has been created, or it will be updated through the default user provisioning provided by the module (using the settings above).
* **Just in Time Provisioning** – During the login process, all fields from the assertions can be copied into the user account entity. All the **Claim** fields from the assertion will be copied into the selected **Mx User Attribute**. 
    * **Claim**
    * **Mx User attribute**

##### 3.2.4.1 Additional Functionality{#additional-functionality}

{{% alert color="info" %}}
These settings are only available in the following versions of the module (depending on which Mendix version you are using)

* v3.1.8/v3.1.9 and above for Mendix version 9
* v2.2.0 and above for Mendix version 8
* v1.16.4 and above for Mendix version 7
{{% /alert %}}

* **Use custom logic for user provisioning** and **Use custom after sign-in logic**

    If you want to add your own logic to the user provisioning, enable one, or both, of these functions.The microflow you select will be executed after the user signs in.

    1. If the **Custom microflow** field is *None* then the default Mendix custom microflows `CustomUserProvisioning` and `CustomAfterSigninLogic`, respectively, is/are executed
    2. You can also implement your own custom microflow and then select that in the Custom microflow field to override the Mendix custom microflows. For this:
        * the custom microflow name must begin with the string “Custom”, (for example, `CustomMyUserProvisioning`)
        * to see the latest custom microflows in the dropdown, refresh the modules in the *Model Reflection* of your application whenever you add/remove any custom microflow — see [Mx Model Reflection](/appstore/modules/model-reflection/) for information on how to do this

* **Enable delegated authentication** *(deprecated)* - See [Multi-tier Delegated Authentication](#delegated-auth), below, for information on when you might set this.

* **Enable mobile authentication Token** - If you are using a [hybrid mobile](/refguide/hybrid-mobile/) app and you enable this, you can log in to your Mendix hybrid mobile app after the app is closed, using an auth token cookie. Only check this if you are using SAML on a hybrid mobile app. Note that this functionality also requires changes to the hybrid app package as described in [How To Implement SSO on a Hybrid App with Mendix and SAML](/howto8/mobile/implement-sso-on-a-hybrid-app-with-mendix-and-saml/).

#### 3.2.5 Authentication Context

The following settings set the authentication context:

* **Preferred Entity Descriptor** – The IdP metadata can contain references to many different IdPs or SPs. This option allows you to select which of the IdPs ought to be used when a user tries to login using this IdP configuration.
* **IdP Authentication Properties** – When sending out requests, this option has to be configured according to the specification of the IdP server: **Authentication context comparison**.
* **Disable Name ID Policy** – Check this box to disable the use of a name ID policy. This means you will use another attribute or claim to identify users.
* **Preferred Name ID** – If using a name ID policy, it is mandatory to specify the name ID method. Every IdP is supposed to support transient, but that does not have to guarantee a fixed user name. It is up to the IdP and your user provisioning implementation what the best solution is here. 
* **Authentication Context Classes** – This passes the allowed authentication methods. This has to be whatever the IdP requests, as there are no requirements within this module and all options are available. For the SAML protocol, the SP is required to pass in the authentication context. However, passing all options is not recommended, since that leads to significantly bigger (and slower) message exchange.
* **Allow Idp Initiated Authentication** – By default, the module does not allow for unsolicited request. That means that every login has to be initiated from the Mendix application, and all the messages have to be exchanged using the same RequestID and RelayState. Some IdPs do not allow for the RelayState to be passed, or the authentication could be initiated by the IdP instead of at the SP. For all situations where the RelayState is not being generated or passed from the original login action at Mendix, this option should be enabled. By default, it does not allow for unsolicited requests, because that would be considered less secure.

## 4 Advanced Configuration

### 4.1 Multi-tenant Behavior

The resource folder contains a file called *SAMLConfig.properties*. In this file, you can optionally override advanced settings from the SAML module. Usage of this file is optional. When the file does not exist or you do not specify a setting, the module will use its default behavior.

This file contains the documented properties, and example lines show the default values of these options.

With these settings, you can configure the behavior of this module and improve multi-tenant behavior of your application. For plain SAML authentication, it is best to leave this file unchanged. 

### 4.2 In-session Authentication{#in-session}

{{% alert color="info" %}}
In-session authentication at the SAML IdP is only available in the following versions of the module (depending on which Mendix version you are using)

* v3.3.0/v3.3.1 and above for Mendix version 9
* v2.3.0 and above for Mendix version 8
{{% /alert %}}

In-session authentication is a process that takes place within a session that was initiated by a (primary) end-user that signed in to your app or from within an anonymous session. This can be useful in the following situations:

* to require the primary end-user to re-authenticate shortly before they are allowed to do a critical transaction in your app
* to have a second end-user add their authentication (for example, for electronic signing) but leave the primary end-user associated with the overall session
* to let the user interact anonymously with your app at first, but ask them to identify and authenticate themselves during that session

Usage of the in-session authentication does change the user roles that apply to the current session. If your app is configured with multiple IDP configurations, the in-session authentication will use the same SAML IDP as the initial (non-anonymous) session. If there is no current session, it lets the user select their IDP.

This flow can be initiated by using the URL `https://{app-url}/sso/login?action=verify`

To enable in-session authentication, you need to use the `OpenConfiguration` microflow to configure two microflows in the SAML SSO module. In the **Provisioning** tab of the SAML configuration, you need to do the following:

* set **Custom Prepare In-Session Authentication microflow** to `CustomPrepareInSessionAuthentication`
* set **Custom Evaluate In-Session Authentication microflow** to `CustomEvaluateInSessionAuthentication`

The `CustomPrepareInSessionAuthentication` microflow sets up specific data in the current user session so that it can be recovered after the SAML in-session authentication flow returns to the app. The microflow can use the context information that is passed via the `on` query parameter.

The `CustomEvaluateInSessionAuthentication` microflow implements the logic that handles the authentication details of the in-session authentication. The SAML SSO module comes with an empty default `EvaluateInSessionAuthentication` flow, which can be enhanced to combine information from the original session with information received in the assertion from the in-session authentication..

### 4.3 Requesting user attributes at the SAML IdP

{{% alert color="info" %}}
Requesting user attributes at the SAML IdP is only available in the following versions of the module (depending on which Mendix version you are using)

* v3.3.0/v3.3.1 and above for Mendix version 9
* v2.3.0 and above for Mendix version 8
{{% /alert %}}

Your app using the SAML protocol can request specific attributes from the SAML IdP, such as Date of Birth or Gender. You need to clarify with your SAML IdP what attributes can be requested. In the request you can also indicate whether you consider the attribute as mandatory or optional for your app’s logic.

You can configure these attribute requests in the **Attribute Consuming Service** tab of the `OpenConfiguration` microflow. This allows you to add new attributes, edit existing attributes, or remove selected attributes.

You can set up two sets of attributes, provided at different times. Those listed under **I want to request attribute(s) at my IDP during initial login** will be returned when the end user initially signs in. Those listed under **I want to request attribute(s) at my IDP during in-session login** will be returned during [In-session Authentication](#in-session).

Although the typical use case for requesting attributes is to obtain information about the user, you can request an attribute with a specific value. In this case, you can configure the optional **Attribute value that must be returned**.

### 4.4 Configuration of SAML Binding{#saml-binding}

{{% alert color="info" %}}
Using artifact binding for SAML responses at the SAML IdP is only available in the following versions of the module (depending on which Mendix version you are using)

* v3.3.0/v3.3.1 and above for Mendix version 9
* v2.3.0 and above for Mendix version 8
{{% /alert %}}

You may need to choose a different SAML binding to match your IdP. You can configure the SAML binding in the **Identity Provider Metadata** tab of the `OpenConfiguration` microflow. 

#### 4.4.1 Response Protocol Binding

By default, the SAML SSO module uses `POST_BINDING` for the SAML response. In most cases (for example, when using AzureAD) you will want to stick to this default.

Some IdPs, however, require your app to use the more secure `ARTIFACT_BINDING`. 

To use artifact binding, select ARTIFACT_BINDING option for **Response protocol binding**.
This configuration helps enable the Post/Artifact binding, used as the following:

* `ProtocolBinding` attribute in Auth-request.
* `AssertionConsumerService` binding in SP-MetaData.

#### 4.4.2 Use AssertionConsumerService Concept

Also in typical cases (e.g. with AzureAD), you don’t want to use the AssertionConsumerService concept in requests. However, some IdPs require requests to include an AssertionConsumerServiceIndex in requests, which refer to the definition of the Assertion Consumer Service in the SP-metadata.

If **Use AssertionConsumerService Concept** is set to `Yes…` then Auth-Request contains only the '`AssertionConsumerServiceIndex`' attribute.

If **Use AssertionConsumerService Concept** is set to `No…` then Auth-Request contains the `AssertionConsumerServiceURL` and `ProtocolBinding` attributes.

Set **Assertion consumer service index** to the value you want to use for `AssertionConsumerServiceIndex` in both the Auth-Request and also in the SP-Metadata. 

The configured binding will be included in the SP metadata, as indicated in the [URLs](#urls} section. 

### 4.5 Use a Certificate Issued by a Certificate Authority{#use-ca}

By default the SAML SSO module will use self-signed certificates. It is, however, also possible to use certificates issued by a Certificate Authority (CA).

SAML SSO supports 2 file formats:

* a PKCS 12 file, which typically has extension .pfx or .p12.
* a jks file.

To use a CA-certificate, upload it as your key store file as described in [Managing the Key Store](/appstore/modules/saml/#keystore).

Remember to do the following:

* set the certificate password in the `KeystorePassword` constant of your app to be able to read the contents of the uploaded key store.
* use an alias for the certificate — this must be the name parameter that is provided when creating the certificate you are uploading. If the values do not match, the SAML module will fall back to using a self-signed certificate instead.
* the value of the configured SP EntityID must match the alias that is included in the uploaded key store.

### 4.6 Multi-tier Delegated Authentication{#delegated-auth}

{{% alert color="warning" %}}
This feature is deprecated. The complexity of the necessary configurations doesn’t correspond with Mendix's ambition to provide an easy ‘low code experience’ and Mendix cannot provide support for it.

Customers are advised to use OAuth bearer tokens instead.
{{% /alert %}}

When you use the SAML SSO module in your app, your app will typically be a front-end app that redirects the user to their IdP via the browser for login.

Using SAML protocols to secure the APIs of your back-end app is more challenging. We advise you to use OAuth access tokens by installing the [OIDC SSO](https://marketplace.mendix.com/link/component/120371) module instead of the SAML module. This is a common, and a best, practice.

The SAML module does currently allow you to use multi-tier delegation (which makes use of the SAML ECP profile) if you need it. Your front-end app can request a token during login that has the right characteristics so it can be shared with a back-end app.  This is an advanced scenario which requires in-depth knowledge of the SAML protocol and the configuration of all integrating systems to get it working.

In the SAML module, you can enable this by checking “Enable delegated authentication” on the provisioning tab. By checking this box you are able to access the authorized SAML token, the module will automatically keep the token alive. Only enable this functionality if you are actually using multi-tier delegated authentication.

## 5 Debugging the Configuration

When testing and debugging the configuration, an option is to view the messages in the log files. A detailed cause of the failure will be printed in case something goes wrong.

When enabling the log node SSO to show trace messages, you can find detailed information from every step in the process. This allows for an easy analysis of where potential configuration errors recite. Enabling trace messages for the SSO log node will also allow for detailed response messages to the user trying to sign in. By default, every failed login attempt always results in this message: "Unable to validate the SAML message!" After enabling trace logging, you can see the exact cause of the failure in the browser. In case of exceptions, you can even see the stack trace.  Obviously, you should not have this enabled in production, but it does allow for easier and faster testing of the configuration.

### 5.1 Error Messages

* **"The application hasn't been properly configured to support Single Sign On."** – This message indicates an incomplete IdP configuration. In more detailed error messages (via the log file), you are able to see which property in the IdP configuration has not been configured.
* **"Unable to complete the request"** – A message has been received that does not have a RelayState/RequestID that matches any of the previously generated IDs (or the message has been answered already). If you get this message, you should validate the message communication and confirm that you are not using unsolicited requests. Or, you can enable that check the box to allow for IdP initiated authentication. 
* **"The authentication was successful, but there is no account available in this application."** – There is no account that matches the identifying assertion, by downloading the SAMLResponse message, you can see the assertion attributes in the XML file to validate which user name has been sent.
* **"Your account hasn't been configured to access this application."** – There is a user account available in the application that matches the identifying assertion, but the user does not have user roles or the user is not active. 
* **"An unexpected error occurred while creating a session"** – An uncaught exception occurred, which could be a configuration error or situation that has not been supported by the module. More information should be available in the stack trace.
* **"The response from the identity provider isn't valid."** – The response from the IdP does not contain any assertion attributes.
* **"No valid SSO Configuration could be found for entity Id: [IdP Alias]"** – Either the specified IdP configuration has not been activated, or an error occurred when reloading the configuration. The error message when reloading the configuration should give more information about the exact problem. The configuration is loaded on startup, when (de-)activating the configuration or when saving an active configuration. 
* **"Unsupported action: [action], only ....."** – The URL is incorrect. Validate that the URL is correctly structured as *action: login, assertion, metadata, discovery*.
* **“MSIS7046: The SAML protocol parameter ‘RelayState’ was not found or not valid.”** – This error can be shown on the ADFS server, most likely when you are using Mac OSX and a Safari browser. Setting the `BindingURI_Redirect` constant to true might help resolve the issue. By default, Mendix favors the `Post` binding, as the maximum size exceeds that of a `Redirect` binding due to its use of cookies and post information instead of URL parameters. The size can be a factor when using encryption.
* **"Unable to validate Response, see SAMLRequest overview for detailed response. Error: An error occurred while committing user: p:'johndoe@company.com'/u:'JoHnDoE@CoMpAnY.CoM'"** – All user names passing through the SAML module are converted to lower-case, so make sure all the existing user names and new user names are also converted to lower-case. This is because certain systems are not case-sensitive (for example, Active Directory), and also because it is a good idea to create two unique users (for example, "JoHnDoE@CoMpAnY.CoM" and "johndoe@company.com").
* **“Could not create a session for the provided user principal.”** – This error can be shown if the IdP configuration does not contain any application attributes for the entity where the user (and user principal) is to be found (and stored).

### 5.2 Troubleshooting an Endless Redirect Loop in Mendix Version 9

When using the [SAML](/appstore/modules/saml/) module for SSO in Mendix 9 and above, you might get stuck in an endless redirect loop. This is because the default value for SameSite cookies is `"Strict"`, and the session cookies cannot be forwarded.

To avoid this issue, make sure your IdP (identity provider) and your app are in the same domain, and thus on the same site. For example, if your app is on `app.domain.com` and you open the deep link `app.domain.com/link/test`, then you are redirected to your IdP to sign in on `idp.domain.com/SSO`. After you sign in successfully, you are sent back to `app.domain.com/SSO/assertion`. Finally, you are forwarded to `app.domain.com/link/test`. Since your requests always stay on the same site, the cookie can be forwarded each time.

If it is not an option to have the IdP and the app in the same domain, set the value for the SameSite cookies to `"None"` or`"Lax"` to solve the problem. See also [Runtime Customization](/refguide/custom-settings/).

## 6 URLs{#urls}

The following diagram gives an overview of all endpoints that the SAML SSO module exposes and consumes:

{{< figure src="/attachments/appstore/modules/saml/saml-endpoints.png">}}

End users can access your app through the following endpoints when using the SAML SSO module:

* **/SSO/discovery** – If there are multiple active IdP configurations and discovery is enabled, this page can give a list of all the IdP configuration. It also allows the user to click the correct URL to sign in.
* **/SSO/login/[IdP Alias]** or **/SSO/login?_idp_id=[IdP_Alias]&action=verify&on={contextname}** – For logging using a specific IdP, you have to open either of these two URLs and pass the IdP alias as a parameter in the URL.
    The (optional) parameters for this end point are as follows:
    * **idp_id** – this indicates which idp will be used to sign the end user in if you connect your app using multiple SAML IdPs
    * **action=verify** – indicates that in-session authentication is being requested (see [In-session Authentication](#in-session) for more information)
    * **on={contextname}** – this gives context to the initiation of in-session authentication,  (see [In-session Authentication](#in-session) for more information)
* **/SSO/login/SSO/** or **/SSO/login/SSO?action=verify&on={contextname}** – If you have only one active IdP, opening these URLs will automatically try to log you in using the active IdP. In the case of multiple active IdPs and discovery enabled, the user will be redirected to the discovery page.  If discovery is not allowed, the user will receive an error message. The optional parameters work as described above.

Your SAML IdP can consume the following endpoints at your app. Typically the SP-metadata is used to communicate the URLs to your SAML IdP. As a Low-Code Developer you don’t have to consider these endpoints. This information is included here for completeness and as a reference when questions arise around integration with your SAML IdP.

* **/SSO/metadata/[IDP-Alias]** – This provides a point for the IdP to automatically download the metadata from this SP
* **/SSO/assertion** – This is the endpoint where the IdP submits the SAML assertion to the so-called ‘Assertion Consumer Service’
* **/SSO/attribute** – This is the endpoint where the SAML-IdP submits requested attributes about the authenticated user
* **/SSO/logout** – This URL will trigger a single logout

## 7 Custom Behavior

### 7.1 evaluateMultipleUserMatches

The module tries to look up the user that matches the provided user name. When multiple `System.User` records are found, this microflow is always executed.

It is possible to customize this microflow to determine the correct user. Whichever user instance is returned will be signed in to the application (and passed on to any other microflow).

### 7.2 CustomUserProvisioning {#customuserprovisioning}

When selecting in the SSO configuration to run the `customUserProvisioning` action (previously known as `CustomLoginLogic`), you can update the new or retrieved user with additional information from the assertion. All the assertions are passed into the microflow, and these can be transformed and stored in the user record. Also, additional roles can be granted to the users based on the assertion attributes.

### 7.3 CustomAfterSigninLogic

After a new session is created for the user, this microflow can be called to copy any data from the previous session to the new session. This microflow behaves similarly to the platform after the sign-in microflow. By using this microflow, it is possible to copy records from the anonymous user to the newly signed-in user.

## 8 Custom Settings

The resources folder contains the *SAMLConfig.properties* file, and through this file, advanced settings can be configured for the module. This file contains the settings along with documentation on the settings. Through this file, it is possible to alter the URLs used as well as how the application behaves in a multi-tenant environment. The file also specifies all the default values and behavior in more detail.

If you are using a custom URL, see [How Do I Get my SAML Metadata or CommunityCommons.GetApplicationUrl to Use the Custom URL?](/developerportal/deploy/custom-domains/#use-custom-url) in the *Custom Domains* documentation.

## 9 Read More

* Academy lecture [SSO Using SAML](https://academy.mendix.com/link/modules/115/lectures/938/2.3.1-SSO-using-SAML)

    {{% alert color="info" %}}You must log into the Mendix Platform to see the lecture above.{{% /alert %}}
