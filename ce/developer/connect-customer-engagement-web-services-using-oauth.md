---
title: "Connect to Dynamics 365 for Customer Engagement web services using OAuth (Developer Guide for Dynamics 365 for Customer Engagement apps)| MicrosoftDocs"
description: "Learn how to connect to Dynamics 365 for Customer Engagement web services using OAuth and how the ADAL API manages OAuth 2.0 authentication with the Dynamics 365 for Customer Engagement web service identity provider"
ms.custom: 
ms.date: 03/29/2019
ms.reviewer: pehecke
ms.service: crm-online
ms.suite: 
ms.tgt_pltfrm: 
ms.topic: get-started-article
applies_to: 
  - Dynamics 365 for Customer Engagement (online)
ms.assetid: 05696c45-2a01-4787-aad5-87e2afef2b7f
caps.latest.revision: 20
author: JimDaly
ms.author: jdaly
manager: amyla
search.audienceType: 
  - developer
search.app: 
  - D365CE
---
# Connect to Dynamics 365 for Customer Engagement web services using OAuth

[!INCLUDE[](../includes/cc_applies_to_update_9_0_0.md)]

OAuth is the authentication method supported by the [!INCLUDE[pn_dynamics_crm](../includes/pn-dynamics-crm.md)] apps Web API, and is one of two authentication methods for the Organization Service – the other being [!INCLUDE[pn_Active_Directory](../includes/pn-active-directory.md)] authentication. One benefit of using OAuth is that your application can support multi-factor authentication. You can use OAuth authentication when your application connects to either the Organization service or the Discovery service.  
  
 Method calls to the web services must be authorized with the identity provider for that service endpoint. Authorization is approved when a valid              OAuth 2.0 (user) access token, issued by [!INCLUDE[pn_microsoft_azure_active_directory](../includes/pn-microsoft-azure-active-directory.md)], is provided in the headers of the message requests.  
  
 The recommended authentication API for use with the [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] Web API is [Azure Active Directory Authentication Library (ADAL)](https://azure.microsoft.com/en-us/documentation/articles/active-directory-authentication-libraries/), which is available for a wide variety of platforms and programming languages. The ADAL API manages OAuth 2.0 authentication with the [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] web service identity provider. For more details on the actual OAuth protocol used, see [Use OAuth to Authenticate with the CRM Service](http://blogs.msdn.com/b/crm/archive/2013/12/12/use-oauth-to-authenticate-with-the-crm-service.aspx).  
 
> [!NOTE]
> All Dynamics 365 for Customer Engagement tools, assemblies, and utilities currently require the authentication patterns supported by ADAL 2.x. When writing custom code, you must use ADAL 2.x for authentication only if your code uses the XRM Tooling APIs found in the Microsoft.Xrm.Tooling.* namespaces. Use of ADAL 3.x or newer for authentication with Dynamics 365 for Customer Engagement is possible when the XRM Tooling APIs are not used.
> 
> For an example of using ADAL 3.x to authenticate, see the GitHub [sample](https://github.com/Microsoft/PowerApps-Samples/tree/master/cds/webapi/C%23/ADALV3WhoAmI). For more information about the benefits of using XRM Tooling APIs in your code see [Build Windows client applications using the XRM tools](/powerapps/developer/common-data-service/xrm-tooling/build-windows-client-applications-xrm-tools).


Before you can use OAuth authentication to connect with the [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] web services, your application must first be registered with [!INCLUDE[pn_microsoft_azure_active_directory](../includes/pn-microsoft-azure-active-directory.md)]. [!INCLUDE[pn_azure_active_directory](../includes/pn-azure-active-directory.md)] is used to verify that your application is permitted access to the business data stored in a [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] tenant.  
  
## Authenticate using ADAL  
 Basic OAuth web service authentication using ADAL is done using just a few lines of code.  
  
```csharp  
// TODO Substitute your correct CRM root service address,   
string resource = "https://mydomain.crm.dynamics.com";  
  
// TODO Substitute your app registration values that can be obtained after you  
// register the app in Active Directory on the Microsoft Azure portal.  
string clientId = "e5cf0024-a66a-4f16-85ce-99ba97a24bb2";  
string redirectUrl = "http://localhost/SdkSample";  
  
// Authenticate the registered application with Azure Active Directory.  
AuthenticationContext authContext =   
    new AuthenticationContext("https://login.windows.net/common", false);  
AuthenticationResult result = authContext.AcquireToken(resource, clientId, new Uri(redirectUrl));  
```  
  
 The authentication context is returned using a well-known authority provider. When you don’t know the [!INCLUDE[pn_azure_active_directory](../includes/pn-azure-active-directory.md)] tenant associated with the [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] apps instance you’re calling, you can use a constant string of `https://login.microsoftonline.com`, which is the authority URL for a multiple tenant scenario. An alternate method to dynamically discover the authority at run time is described later in this topic.  
  
 The next line of code gets the authentication result that contains the access token you’re looking for. You can send message requests to the web service with this token.  
  
 A few more items of interest in this code are the string values used. The `resource` variable contains the Transport Layer Security (TLS) or Secure Sockets Layer (SSL) root address, including the domain (organization), of your [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] server. The `clientId` and `redirectUrl` variables contain the app registration information that is the result of registering the app with [!INCLUDE[pn_Active_Directory](../includes/pn-active-directory.md)]. For more information on app registration, see [Walkthrough: Register a Dynamics 365 for Customer Engagement apps with Azure Active Directory](walkthrough-register-dynamics-365-app-azure-active-directory.md).  
  
## Use the access token in message requests  
 Depending on the [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] API you’re using, there are two different methods to send a message request to the web services. For the Web API, you would typically send an HTTP message request. For the Organization Service, you would send a message request using the web client proxy.  
  
### HTTP message request  
 Once you have the access token, you must set the Authorization header of the message request that you are sending to the web service to the access token value and specify the token type of `Bearer`. For more information on the Authorization header, see section 14.8 of the [HTTP/1.1 protocol](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). The following code demonstrates how this is done using the                          `System.Net.Http.HttpClient` class.  
  
```csharp  
using (HttpClient httpClient = new HttpClient())  
{  
    httpClient.Timeout = new TimeSpan(0, 2, 0);  // 2 minutes  
    httpClient.DefaultRequestHeaders.Authorization =   
        new AuthenticationHeaderValue("Bearer", result.AccessToken);  
```  
  
### Web client requests

Simply set the <xref:Microsoft.Xrm.Sdk.WebServiceClient.WebProxyClient`1.HeaderToken> property value to the access token when using <xref:Microsoft.Xrm.Sdk.WebServiceClient.OrganizationWebProxyClient> or <xref:Microsoft.Xrm.Sdk.WebServiceClient.DiscoveryWebProxyClient> of the Organization Service.  
  
## Refresh the access token

It’s a recommended best practice to refresh the access token before each call to a [!INCLUDE[pn_crm_shortest](../includes/pn-crm-shortest.md)] web service method. To refresh the access token, which is cached by ADAL, you simply call the `AcquireToken` method again using the same context.  
  
```csharp    
AuthenticationResult result = authContext.AcquireToken(resource, clientId, new Uri(redirectUrl));  
```  
  
Afterwards, you once again set the Authorization header with `result.AccessToken` when using the Web API, or the <xref:Microsoft.Xrm.Sdk.WebServiceClient.WebProxyClient`1.HeaderToken> when using the Organization Service.  
  
```csharp    
httpClient.DefaultRequestHeaders.Authorization =   
    new AuthenticationHeaderValue("Bearer", result.AccessToken);  
```  
  
## Discover the authority at run time

The authentication authority URL, and the resource URL, can be determined dynamically at run time using the following ADAL code. This is the recommended method to use as compared to the well-known authority URL shown previously in a code snippet.  
  
```csharp    
AuthenticationParameters ap = AuthenticationParameters.CreateFromResourceUrlAsync(  
                        new Uri("https://mydomain.crm.dynamics.com/api/data/")).Result;  
  
String authorityUrl = ap.Authority;  
String resourceUrl  = ap.Resource;  
```  
  
For the Web API, another way to obtain the authority URL is to send any message request to the web service specifying no access token. This is known as a         *bearer challenge*. The response can be parsed to obtain the authority URL.  
  
```csharp  
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", "");  
```  
  
### See also  
 [Walkthrough: Register a Dynamics 365 for Customer Engagement app with Azure Active Directory](walkthrough-register-dynamics-365-app-azure-active-directory.md)   
 [Multi-Factor Authentication documentation](https://azure.microsoft.com/en-us/documentation/services/multi-factor-authentication/)   
 [OAuth 2.0](http://oauth.net/2/)
