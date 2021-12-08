[Back to main page](README.md)

> Instructions designed for use in the Postman's desktop app v8.

# Installing the Collection

- [Download Postman](#postman-application)
- [Create Private Public Key Pair](#create-private-public-key-pair)
- [Connected App Setup](#connected-app-setup)
- [App Authorization](#app-authorization)
- [Fork the Collection](#fork-the-collection)
- [Configure the Collection](#configure-the-collection)
- [Execute a Request](#execute-a-request)

## Postman Application

### Desktop App

Download and install the Postman app from [Postman](https://www.postman.com/downloads).

### Web App
Not currently supported. Please use the desktop application.

>**Note**: These instructions are designed for use in the Direct APIs which are the recommended approach for invoking the CDP APIs. If you are going to be using the Connect APIs some steps such as the Key Pair, Connected App Setup, and App Authorization will not be required.

## Create Private Public Key Pair

1. You’ll need a .key and a .crt file (private and public key)
2. Open Terminal and change directories to any folder and run the following commands
    ```
    openssl genrsa 2048 > host.key
    chmod 400 host.key
    openssl req -new -x509 -nodes -sha256 -days 365 -key host.key -out host.crt
    ```
    > **Note:** There will be a series of questions (Country, State, Org Name, etc) you need to complete.

    ![Terminal key generation screenshot](images/terminal-key-generation.png)
    > This creates a host.key and host.crt files in that folder. These will be used later in the setup.

## Connected App Setup

This collection utilizes Salesforce's server to server [JWT bearer flow](https://help.salesforce.com/articleView?id=remoteaccess_oauth_jwt_flow.htm&type=5) for acquiring an access token. This portion of the setup will walk you through setting up the connected app.

1. Login to Salesforce → Setup and Search "**App Manager**"
2. In the Setup’s Quick Find search "**App Manager**"
3. Select **“New Connected App”**
    1. Connected App Name: CDP API
    2. API Name: CDP_API (or whatever default value is prepopulated)
    3. Contact Email: Your email address
    4. Under API Heading, check the box for **“Enable OAuth Settings”**
    5. Callback URL: https://oauth.pstmn.io/v1/callback
    6. Select the checkbox for **“Use digital signatures”**
    7. Select **“Choose File”** and select the **host.crt** file created in [Create Private Public Key Pair](#create-private-public-key-pair) section
    8. Under **“Selected OAuth Scopes”** move the following from the “Available OAuth Scopes” to “Selected OAuth Scopes”
        1. Manage user data via APIs (api)
        2. Perform requests at any time (refresh_token, offline_access)
        3. Perform ANSI SQL queries on Salesforce CDP data (cdp_query_api)
        4. Manage Salesforce CDP profile data (cdp_profile_api)
        5. Manage Salesforce CDP Ingestion API data (cdp_ingest_api)
        6. Note: feel free to select others if needed.

        > Your screen should look similar to this

        ![Connected App Setup 1 Screenshot](images/connected-app-setup-1.png)
    9. Select **Save** (on the next screen select **Continue**)
    10. Make note of the **Consumer Key** and **Consumer Secret** values. This will be used as the **“clientId”** and **“clientSecret”** variables in the Postman collection.

        ![Connected App Setup 2 Screenshot](images/connected-app-setup-2.png)

5. At the top of your newly created connected app click “Manage”
    1. Select **“Edit Policies”**
    2. Change **“IP Relaxation”** to **“Relax IP restrictions”**
    3. Select **Save**

## App Authorization

At this point your connected app has been configured however there is a ***_one time setup requirement_*** to authorize your user with the connected app. 

The URL format will look like:

```
<YOUR_ORG_URL>/services/oauth2/authorize?response_type=code&client_id=<YOUR_CONSUMER_KEY>&scope=api refresh_token cdp_profile_api cdp_query_api cdp_ingest_api&redirect_uri=https://oauth.pstmn.io/v1/callback
```
>Notice the scope parameter in the above URL. ***It’s important that you select all the required custom CDP scopes in this request***. All further JWT bearer flow requests will honor ONLY these scopes

**YOUR_ORG_URL** is the fully qualified instance URL.

![App URL Screenshot](images/app-url.png)

**YOUR_CONSUMER_KEY** is the consumer key noted in step 4.x above.

**Example URL:**
>https://aaroncates-20214005-demo.lightning.force.com/services/oauth2/authorize?response_type=code&client_id=asdlfjasldfjsaldfjaslfds&scope=api%20refresh_token%20cdp_profile_api%20cdp_query_api%20cdp_ingest_api&redirect_uri=https://oauth.pstmn.io/v1/callback

1. Paste that URL in a browser window. 
2. This prompts a consent dialog asking permission for each of the scopes requested above.  Select **Allow** and you should be redirected back.

    ![Access Prompt Screenshot](images/access-prompt.png)

3. You may also get an alert from the callback. If you do, select **Open Postman**

    ![Postman Redirect Screenshot](images/postman-redirect.png)

4. **Optional:** If you want to verify everything is authorized correctly, in the Quick Find search for **“Connected Apps OAuth Usage”**. Here you will see your connected app and should see a user count of 1.

    ![Connected App Verification Screenshot](images/connect-app-verification.png)

## Fork the Collection

Fork the collection using the following button or follow the instructions listed below.

##### Option 1:

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/14448118-8145476a-1797-4b2a-b219-37c6d438d956?action=collection%2Ffork&collection-url=entityId%3D14448118-8145476a-1797-4b2a-b219-37c6d438d956%26entityType%3Dcollection%26workspaceId%3D34382471-0c97-40e5-a206-f947271665c4#?env%5BCDP%20Template%20Environment%5D=W3sia2V5IjoibG9naW5VcmwiLCJ2YWx1ZSI6IiIsImVuYWJsZWQiOnRydWV9LHsia2V5IjoiY2xpZW50SWQiLCJ2YWx1ZSI6IiIsImVuYWJsZWQiOnRydWV9LHsia2V5IjoidXNlck5hbWUiLCJ2YWx1ZSI6IiIsImVuYWJsZWQiOnRydWV9LHsia2V5IjoicHJpdmF0ZUtleSIsInZhbHVlIjoiIiwiZW5hYmxlZCI6dHJ1ZX1d)

##### Option 2:

1. In Postman, click on the top search bar and type **Salesforce**
2. Click **Salesforce Developers** under Teams

    ![Searching for Salesforce screenshot](images/search-salesforce.png)

3. Click the **Salesforce CDP APIs** tile
4. Click **Fork**

    ![Fork button screenshot](images/fork-button.png)

5. Enter a label for your fork (e.g.: “My fork”)
6. Select a workspace (the default “My Workspace” workspace is fine)
7. Click **Fork Collection**

## Configure the Collection

The collection uses a series of collection variables to help streamline your calls. To successfully use the package it's important to be sure to update the collection variables.

1. Click **Salesforce CDP APIs**
2. Open the **Variables** tab 
3. Complete the following variables for your instance by placing the values in the **Current Value** column. 

|Variable|Example Value|Description|Used In Direct APIs|Used In Connect APIs
|-|-|-|-|-|
|loginUrl|login.salesforce.com|Using login.salesforce.com will be fine unless you are using a sandbox.|X|X|
|version|52.0|Salesforce API version number||X|
|clientId|3MVG9l2zHsylwlpR6H5xByqIHvFbLVATgzkY...|Consumer key from the connected app.|X|X|
|clientSecret|775C20434DB475FC326765353AF5210D4...|Consumer secret from the connected app.||X|
|privateKey|-----BEGIN RSA PRIVATE KEY----- MIIEpAIBAAKCAQEA6spOAo1NhTsOhj19M  <br />...<br />  rEOBZ458a3O4EOfHP1luZb4ZGrnTDRcA== <br /> -----END RSA PRIVATE KEY-----| Contents of host.key file.|X||
|userName|aaroncates@aaroncates-20210405.demo|User Name of the authorized user.|X|X|
|password|superSecretPassword1!|Password of the authorized user.||X|
|securityToken|fVhwzeDFMrAh4IC9hS|Salesforce security token for the authorized user. Details for securing a token available [here](https://developer.salesforce.com/docs/atlas.en-us.api.meta/api/sforce_api_concepts_security.htm).||X|

4. Click **Save**.

    ![Collection variables screenshot](images/collection-variables.png)

## Collection Authentication

**Direct APIs**

The collection is built to leverage the [OAuth 2.0 JWT Bearer Flow for Server-to-Server Integration](https://help.salesforce.com/articleView?id=sf.remoteaccess_oauth_jwt_flow.htm&type=5) for Salesforce Core authorization. The core token is then exchanged with the off core server hosting CDP for a final authorization token.

![Collection variables screenshot](images/authorization-flow.png)

> **NOTE: To simplify the use of the the collection this authorization process has been configured to run automatically prior to each request and check if a valid token exists.** 

This is accomplished by using the collection variables defined in the [Configure the Collection](#configure-the-collection) section combined with a pre-request script. 

The script creates six new variables that are used for token generation and should not be edited:
* dne_cdpTokenRefreshTime
* dne_cdpAssertion
* dne_cdpAuthToken
* dne_cdpInstanceUrl
* dne_cdpOffcoreToken
* dne_cdpOffcoreUrl

The Marketing Cloud authorization tokens are valid for 2 hours therefore when a token is requested we create a new variable called **dne_cdpTokenRefreshTime** that stores the time the token was generated. Each subsequent call will use this refresh time to determine if a new token should be requested.

The token returned in the authorization call is stored as the collection variable **dne_cdpOffcoreToken** and passed in the authorization header defined by a pre-request script at folder level.

**Connect APIs**

The connect APIs leverage the traditional Salesforce authentication request process. You must first run the `Auth Request` first and we leverage Postman's tests functionality to parse the response body and set the variables for `dne_cdpAuthToken` and `dne_cdpInstanceUrl` that are used in the remaining Connect API calls.

## Execute a Request

1. Expand the collection and select the `Profile API -> Metadata - DMO` request
1. Click `Send`

At this point, if your environment is correctly set up, you should see a `200 OK` status. This means that you have successfully authenticated with Salesforce CDP and that you can now use the other collection’s requests.

![DMO Metadata Success](images/metadata-dmo-200.png)

See [additional documentation](README.md#additional-documentation) for more information on how to keep the collection up to date and work with multiple Marketing Cloud instances.


[Back to main page](README.md)
