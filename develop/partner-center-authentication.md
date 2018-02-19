---
title: Partner Center authentication
description: Partner Center uses Azure AD for authentication, and to use the Partner Center APIs you must configure your authentication settings correctly.
ms.assetid: 2307F2A8-7BD4-4442-BEF7-F065F16DA0B2
ms.author: v-thpr
ms.date: 12/15/2017
ms.topic: article
ms.prod: partner-center
ms.technology: partner-center-sdk
---

# Partner Center authentication


**Applies To**

-   Partner Center
-   Partner Center operated by 21Vianet
-   Partner Center for Microsoft Cloud Germany
-   Partner Center for Microsoft Cloud for US Government

Partner Center uses Azure AD for authentication, and to use the Partner Center APIs you must configure your authentication settings correctly. This requires these three steps:

-   **App or App+user?**

    These two authentication styles are used depending on your end goal. If you are developing a web site, you'll probably use App+user credentials. App only credentials, meanwhile, work for both web sites and native apps.

    Under the **Prerequisites** heading, each [Scenario](scenarios.md) specifies whether App+user credentials, App credentials, or both are supported.

-   **REST or managed?**

    The managed API helps manage your tokens so that you do not have to refresh them. Additional steps are required for the REST API to keep your tokens current.

-   **TiP or regular?**

    You will use two sets of auth tokens over the course of development: one based on your integration sandbox Partner Center account for development and testing, and a second based on your primary Partner Center account for day-to-day management of real customer data.

## <span id="initial"></span><span id="INITIAL"></span>Initial setup


**Configure authentication for Partner Center APIs**

1.  To begin, you need to make sure that you have both a primary Partner Center account, and an integration sandbox Partner Center account. For more information, see [Set up Partner Center accounts for API access](set-up-api-access-in-partner-center.md). Make note of the Azure AAD App registration ID and Secret (client secret is required for App only identification) for both your primary account and your integration sandbox account.

2.  Sign in to Azure AD from the Azure management portal. In **permissions to other applications**, set permissions for **Windows Azure Active Directory** to **Delegated Permissions**, and select both **Access the directory as the signed-in user** and **Sign in and read user profile**.

3.  In the Azure management portal, **Add application**. Search for `fa3d9a0c-3fb0-42cc-9193-47c7ecd2edbd`, which is the Microsoft Partner Center application. Set the **Delegated Permissions** to **Access Partner Center API**.

## <span id="managedApp"></span><span id="managedapp"></span><span id="MANAGEDAPP"></span>Authentication with App credentials and the managed API


The following code shows how to get and use App authentication using the Partner Center managed API.

**Add Azure AD information to your project**

1.  Add the appropriate Azure AD information to your code. Choose the correct ID, Secret, and Domain depending on whether you're using the integration sandbox or the primary account.

    The Authority and ResourceUrl shown below may have to be updated with appropriate values if you're using Partner Center operated by 21Vianet, Partner Center for Microsoft Cloud Germany, or Partner Center for Microsoft Cloud for US Government.

    ```CSharp
    public static string PartnerServiceApiRoot = "https://api.partnercenter.microsoft.com";
    public static string Authority = "https://login.windows.net";
    public static string ResourceUrl = "https://graph.windows.net";
    public static string ApplicationId = "<web application id>";
    public static string ApplicationSecret = "<application secret>";
    public static string ApplicationDomain = "<partner tenant id>";
    ```

2.  Add code to manage your tokens.

    ```CSharp
    public static IAggregatePartner GetPartnerCenterTokenUsingAppCredentials()
    {
        // Get a user Azure AD Token.
        PartnerService.Instance.ApiRootUrl = PartnerApplicationConfiguration.PartnerServiceApiRoot;
        var partnerCredentials = 
            PartnerCredentials.Instance.GenerateByApplicationCredentials(
                PartnerApplicationConfiguration.ApplicationId, 
                PartnerApplicationConfiguration.ApplicationSecret, 
                PartnerApplicationConfiguration.ApplicationDomain);

        // Create operations instance with partnerCredentials.
        return PartnerService.Instance.CreatePartnerOperations(partnerCredentials);
    }
    ```

    Use the following code instead for Partner Center for Microsoft Cloud Germany.

    ```CSharp
    // Use the following code for Partner Center for Microsoft Cloud Germany.
    public static string Authority = "https://login.microsoftonline.de";
    public static string ResourceUrl = "https://graph.cloudapi.de";

    public static IAggregatePartner GetPartnerCenterTokenUsingAppCredentials()
    {
        // Get a user Azure AD Token.
        PartnerService.Instance.ApiRootUrl = PartnerApplicationConfiguration.PartnerServiceApiRoot;

        var partnerCredentials = 
            PartnerCredentials.Instance.GenerateByApplicationCredentials(
                PartnerApplicationConfiguration.ApplicationId, 
                PartnerApplicationConfiguration.ApplicationSecret, 
                PartnerApplicationConfiguration.ApplicationDomain, 
                PartnerApplicationConfiguration.Authority, 
                PartnerApplicationConfiguration.ResourceUrl);

        // Create operations instance with partnerCredentials.
        return PartnerService.Instance.CreatePartnerOperations(partnerCredentials);
    }
    ```

3.  Get your token and use it to call the managed API. The following example retrieves a list of offer objects and then displays each offer identifier and name on the console.

    ```CSharp
    // Gets a list of offers using the SDK.
    public static void GetOffersUsingAppCredentials()
    {
        IAggregatePartner partner = GetPartnerCenterTokenUsingAppCredentials();

        // Get a list of offers.
        var offers = partner.Offers.ByCountry("US").Get();
        foreach (var offer in offers.Items)
        {
            Console.WriteLine("Offer ID:{0}, Name:{1}", offer.Id, offer.Name);
        }
    }
    ```

## <span id="RestApp"></span><span id="restapp"></span><span id="RESTAPP"></span>Authentication with App credentials and the REST API


The following code shows how to get and use App authentication using the Partner Center REST API.

**Add Azure AD information to your project**

1.  Add the appropriate Azure AD information to your code. Choose the correct ID, Secret, and Domain depending on whether you're using the integration sandbox or primary account.

    The Authority and ResourceUrl shown below may have to be updated with appropriate values if you're using Partner Center operated by 21Vianet, Partner Center for Microsoft Cloud Germany, or Partner Center for Microsoft Cloud for US Government.

    ```CSharp
    public static string PartnerServiceApiRoot = "https://api.partnercenter.microsoft.com";
    public static string Authority = "https://login.windows.net";
    public static string ResourceUrl = "https://graph.windows.net";
    public static string ApplicationId = "<web application id>";
    public static string ApplicationSecret = "<application secret>";
    public static string ApplicationDomain = "<partner tenant id>";
    ```

2.  Add code to manage your tokens.

    ```CSharp
    // Given the reseller domain, clientid and clientsecret of the app, this method helps to retrieve the AD token.
    //
    // Parameters:
    // resellerDomain - The domain of the reseller, including ".onmicrosoft.com".
    // clientId       - The AppId from the Azure portal registered for this app.
    // clientSecret   - The secret from the Azure portal registered for this app.
    //
    // Returns:
    // The authentication token object that contains access_token and expiration time.
    public static JObject GetADToken(string resellerDomain, string clientId, string clientSecret)
    {
        var request = WebRequest.Create(
            string.Format(
                "{0}/{1}/oauth2/token", 
                PartnerApplicationConfiguration.Authority, 
                resellerDomain));

        request.Method = "POST";
        request.ContentType = "application/x-www-form-urlencoded";
        string content = string.Format(
            "grant_type=client_credentials&amp;client_id={0}&amp;client_secret={1}&amp;resource={2}",
            clientId,
            HttpUtility.UrlEncode(clientSecret),
            HttpUtility.UrlEncode(PartnerApplicationConfiguration.ResourceUrl));

        using (var writer = new StreamWriter(request.GetRequestStream()))
        {
            writer.Write(content);
        }

        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                return adResponse;
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }

        return null;
    }

    // Gets the partner center app credential token.
    // Returns: Token response from server.
    public static JObject GetPartnerCenterAppCredentialsToken()
    {
        // Get the Azure AD token for user.
        var aadToken = 
            GetADToken(PartnerApplicationConfiguration.ApplicationDomain, 
                PartnerApplicationConfiguration.ApplicationId, 
                PartnerApplicationConfiguration.ApplicationSecret);

        var request = WebRequest.Create(string.Format(
            "{0}/generatetoken", 
            PartnerApplicationConfiguration.PartnerServiceApiRoot));

        request.Headers.Add(HttpRequestHeader.Authorization, 
            "Bearer " + aadToken["access_token"].ToString());
        request.Method = "POST";
        request.ContentType = "application/x-www-form-urlencoded";
        string content = string.Format("grant_type=jwt_token");

        using (var writer = new StreamWriter(request.GetRequestStream()))
        {
            writer.Write(content);
        }

        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                return adResponse;
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }

        return null;
    }
    ```

3.  Get your token and use it to call the REST API. The following example retrieves a list of offer objects and then displays information about each offer.

    ```CSharp
    // Gets a list of offers using the REST API.
    public static void GetOffersWithAppCredentials()
    {
        JObject tokenObject = GetPartnerCenterAppCredentialsToken();

        // Get a list of offers.
        var request = WebRequest.Create(
            string.Format(
                "{0}/v1/offers?country={1}", 
                PartnerApplicationConfiguration.PartnerServiceApiRoot, 
                "US"));

        request.Headers.Add(HttpRequestHeader.Authorization, 
            "Bearer " + tokenObject["access_token"].ToString());
        request.Method = "GET";
        
        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                Console.WriteLine(adResponse.ToString());
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }
    }
    ```

## <span id="managedUser"></span><span id="manageduser"></span><span id="MANAGEDUSER"></span>Authentication with App+User credentials and the managed API


The following code shows how to get and use App+User authentication using the Partner Center managed API.

**Add Azure AD information to your project**

1.  Add the appropriate Azure AD information to your code. Choose the correct ID, Domain, user name and password depending on whether you're using the integration sandbox or the primary account.

    The Authority and ResourceUrl shown below may have to be updated with appropriate values if you're using Partner Center operated by 21Vianet, Partner Center for Microsoft Cloud Germany, or Partner Center for Microsoft Cloud for US Government.

    ```CSharp
    private static class PartnerUserConfiguration
    {
        public static string PartnerServiceApiRoot = "https://api.partnercenter.microsoft.com";
        public static string Authority = "https://login.windows.net";
        public static string UserName = "<username>@<domainprefix>.onmicrosoft.com";
        public static string Password = "<password>";
        public static string ResourceUrl = "https://api.partnercenter.microsoft.com";
        public static string ClientId = "<native app id>";
        public static string ApplicationDomain = "<partner tenant>";
    }
    ```

2.  Add code to manage your tokens.

    ```CSharp
    // Sign the user into Azure AD and get token.
    // Returns: User token.
    public static AuthenticationResult UserLogin()
    {
        var addAuthority = new UriBuilder(Configuration.Authority)
        {
            Path = Configuration.CommonDomain
        };

        UserCredential userCredentials = 
            new UserCredential(PartnerUserConfiguration.UserName, PartnerUserConfiguration.Password);
        AuthenticationContext authContext = 
            new AuthenticationContext(addAuthority.Uri.AbsoluteUri);

        return 
            authContext.AcquireTokenAsync(Configuration.ResourceUrl, Configuration.ClientId, userCredentials).Result;
    }

    // Gets the partner center token using the managed API.
    // Returns: Partner Center SDK partner interface.
    public static IAggregatePartner GetPartnerCenterTokenWithUserCredentials()
    {
        // Get a user Azure AD Token.
        var aadAuthResult = UserLogin();
        PartnerService.Instance.ApiRootUrl = PartnerUserConfiguration.PartnerServiceApiRoot;
        var partnerCredentials = 
            PartnerCredentials.Instance.GenerateByUserCredentials(PartnerUserConfiguration.ClientId, 
                new AuthenticationToken(aadAuthResult.AccessToken, aadAuthResult.ExpiresOn), async delegate
        {
            // Token Refresh callback.
            aadAuthResult = UserLogin();
            return await Task.FromResult<AuthenticationToken>(new AuthenticationToken(aadAuthResult.AccessToken, aadAuthResult.ExpiresOn));
        });

        // Get operations instance with partnerCredentials.
        return PartnerService.Instance.CreatePartnerOperations(partnerCredentials);
    }
    ```

3.  Get your token and use it to call the managed API.

    ```CSharp
    public static void GetCustomersWithUserCredentials()
    {
        IAggregatePartner partner = GetPartnerCenterTokenWithUserCredentials();

        // Get a customer list.
        var customers = partner.Customers.Get();
        foreach (var customer in customers.Items)
        {
            Console.WriteLine("Customer ID:{0}, Name:{1}", 
                customer.Id, 
                customer.CompanyProfile.CompanyName);
        }
    }
    ```

## <span id="RestUser"></span><span id="restuser"></span><span id="RESTUSER"></span>Authentication with App+User credentials and the REST API


The following code shows how to get and use App+User authentication using the Partner Center REST API.

**Add Azure AD information to your project**

1.  Add the appropriate Azure AD information to your code. Choose the correct ID, Domain, user name and password depending on whether you're using the integration sandbox or the primary account.

    The Authority and ResourceUrl shown below may have to be updated with appropriate values if you're using Partner Center operated by 21Vianet, Partner Center for Microsoft Cloud Germany, or Partner Center for Microsoft Cloud for US Government.

    ```CSharp
    private static class PartnerUserConfiguration
    {
        public static string PartnerServiceApiRoot = "https://api.partnercenter.microsoft.com";
        public static string Authority = "https://login.windows.net";
        public static string UserName = "<username>@<domainprefix>.onmicrosoft.com";
        public static string Password = "<password>";
        public static string ResourceUrl = "https://api.partnercenter.microsoft.com";
        public static string ClientId = "<native app id>";
        public static string ApplicationDomain = "<partner tenant>";
    }
    ```

2.  Add code to manage your tokens.

    ```CSharp
    // Given the reseller domain, clientid and username/password of the app, this method helps to retrieve the AD token
    //
    // Parameters:
    // resellerDomain - The domain of the reseller including ".onmicrosoft.com".
    // clientId - AppId from the Azure portal.
    //
    // Returns:
    // An authentication token object that contains access_token, expiration time, and can be used to 
    // get the authorization token from a resource.
    public static JObject GetADToken(string resellerDomain, string clientId)
    {
        string loginUrl = 
            string.Format(
                "{0}/{1}/oauth2/token", 
                PartnerUserConfiguration.Authority, 
                resellerDomain);

        var request = WebRequest.Create(loginUrl);

        request.Method = "POST";
        request.ContentType = "application/x-www-form-urlencoded";
        
        string content = string.Format(
            "resource={0}&amp;client_id={1}&amp;grant_type=password&amp;username={2}&amp;password={3}&amp;scope=openid",
            HttpUtility.UrlEncode(Configuration.ResourceUrl),
            HttpUtility.UrlEncode(clientId),
            HttpUtility.UrlEncode(Configuration.UserName),
            HttpUtility.UrlEncode(Configuration.Password));

        using (var writer = new StreamWriter(request.GetRequestStream()))
        {
            writer.Write(content);
        }

        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                return adResponse;
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }

        return null;
    }

    // Gets the partner center App+User token.
    // Returns: Token response from server.
    public static JObject GetPartnerCenterToken()
    {
        // Get Azure AD token for user.
        var aadToken = 
            GetADToken(PartnerUserConfiguration.ApplicationDomain, PartnerUserConfiguration.ClientId);

        var request = 
            WebRequest.Create(
                string.Format(
                    "{0}/generatetoken", 
                    PartnerUserConfiguration.PartnerServiceApiRoot));

        request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + aadToken["access_token"].ToString());
        request.Method = "POST";
        request.ContentType = "application/x-www-form-urlencoded";
        string content = string.Format("grant_type=jwt_token");

        using (var writer = new StreamWriter(request.GetRequestStream()))
        {
            writer.Write(content);
        }

        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                return adResponse;
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }

        return null;
    }
    ```

3.  Get your token and use it to call the REST API.

    ```CSharp
    // Gets a list of customers using App+User credentials.
    public static void GetCustomersUsingUserCredentials()
    {
        JObject tokenObject = GetPartnerCenterToken();

        // Get a list of customers.
        var request = WebRequest.Create(
            string.Format(
                "{0}/v1/customers", 
                PartnerUserConfiguration.PartnerServiceApiRoot));

        request.Headers.Add(HttpRequestHeader.Authorization, "Bearer " + tokenObject["access_token"].ToString());
        request.Method = "GET";
        try
        {
            var response = request.GetResponse();
            using (var reader = new StreamReader(response.GetResponseStream()))
            {
                var responseContent = reader.ReadToEnd();
                var adResponse = 
                    Newtonsoft.Json.JsonConvert.DeserializeObject<JObject>(responseContent);
                Console.WriteLine(adResponse.ToString());
            }

        }
        catch (WebException webException)
        {
            if (webException.Response != null)
            {
                using (var reader = new StreamReader(webException.Response.GetResponseStream()))
                {
                    var responseContent = reader.ReadToEnd();
                }
            }
        }
    }
    ```

 

 




