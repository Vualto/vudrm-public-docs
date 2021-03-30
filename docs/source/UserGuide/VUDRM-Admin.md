# USER GUIDE

Our VUDRM Admin page gives you the tools you will need to test our DRM services with your content. If you need assistance or clarification in relation to anything in this guide, please contact <support@vualto.com>.

## Login

To access the VUDRM Admin site, please go to the [Admin Login page](https://admin.vudrm.tech/Account/Login) - login using the credentials on your onboarding documentation.

<img src="../_static/user-guide-images/1login.png" class="image" width="500" alt="VUALTO Admin - Login Page"/>

## Dashboard

Once you have successfully logged in, you will be redirected to the `Dashboard` page.

<img src="../_static/user-guide-images/2dashboardpage.png" class="image" width="750" alt="VUALTO Admin - Dashboard Page"/>

The dashboard will show you an overview of all DRM licenses served with your content.

By clicking on the `Range` dropdown field, you can set custom time frames to get an overview for that time period.

<img src="../_static/user-guide-images/3rangedropdown.png" class="image" width="750" alt="VUALTO Admin - Range Dropdown"/>

## Configuration

### Client Info

The `Client Info` section contains the details of how your account is configured, such as which DRM providers are enabled and which URLs have been set to retrieve each service's DRM licenses.

<img src="../_static/user-guide-images/4blurredinfopage.png" class="image" width="750" alt="VUALTO Admin - Client Info Page"/>

If you require amendments to your account, please contact <support@vualto.com>.

#### Client Info - Edit default token policies

At the bottom of the `Client Info` page, you can view each DRM service’s default token policy. This can be edited to give more control over how a service will act by default.

<img src="../_static/user-guide-images/5edittokenpolicies.png" class="image" width="750" alt="VUALTO Admin - Edit Token Policies"/>

Clicking `Edit Token Policy` will load the JSON editor. 

<img src="../_static/user-guide-images/5.1jsoneditor.png" class="image" width="750" alt="VUALTO Admin - Edit Token Policies - JSON Editor"/>

Please ensure that the JSON has been entered correctly, otherwise you will be informed of an error and will not be able to save the amended policy.

When the amends are complete, click `Close` and then `Save`. 

Each DRM service has limitations to how their policy can be altered. Please read the links below for more details:

* [DRM Policy](../DeveloperDocumentation/VUDRM-token.html#drm-policy)
* [PlayReady DRM Policy](../DeveloperDocumentation/VUDRM-token.html#playready-drm-policy)
* [Fairplay DRM Policy](../DeveloperDocumentation/VUDRM-token.html#fairplay-drm-policy)
* [Widevine DRM Policy](../DeveloperDocumentation/VUDRM-token.html#widevine-drm-policy)

### VUDRM Token

In this section, you can create, validate, decrypt, encode, and decode VUDRM Tokens.

<img src="../_static/user-guide-images/6vudrmtoken.png" class="image" width="1000" alt="VUALTO Admin - VUDRM Token Page"/>

There are a variety of templates to choose from which will automatically populate the `Policy` text box, please ensure that you replace all default values (such as `{"polend":"DD-MM-YYYY HH:MM:SS"}` to `{"polend":"09-05-2021 12:30:00"}`). When the default values have been changed, you can then generate a token with those parameters. 

The buttons on the page do the following:

* `Generate`        - This will generate a token to include the policy you have entered
* `Validate Policy` - This will ensure the policy is valid
* `Validate Token`  - Validate the token created
* `Decrypt Token`   - Decrypts the token
* `Encode Token`    - URL Encodes the token
* `Decode Token`    - URL Decodes the token

Once a token has been generated, a new button will appear - `Use Token in Demo`. 

<img src="../_static/user-guide-images/6-1vudrmtoken.png" class="image" width="1000" alt="VUALTO Admin - VUDRM Token Page - Use Token in Demo"/>

Clicking `Use Token in Demo` will load the `VUPLAY Demo` player with the generated token in the `Custom VUDRM Token` text box.

For an in depth guide of how our tokens work - please refer to the [VUDRM Token documentation](../DeveloperDocumentation/VUDRM-token.html#). 

### VUDRM Encryption Keys

Within this section, you can call the `CPIX Key Provider` to fetch VUDRM Encryption Keys in CPIX XML document format as well as the CPIX keys as JSON which is generated at the same time, both of which are available to copy. 

<img src="../_static/user-guide-images/7encryptkeys.png" class="image" width="1000" alt="VUALTO Admin - Encrypt Keys Page"/>

To create an Encryption key, do the following:

* Select which Key Provider API you wish to use by using the `Key Provider API`
* Select which DRM Providers by changing the relevant providers button to `YES`
* Enter a relevant name in the `Content ID` text field
* Click `Generate`
* Click `Copy` to copy the newly created key

We also have our `Legacy JSON Key provider` (available on the `Key Provider API` dropdown) to use if your product requires it. However, we recommend you use the `CPIX Key Provider` whenever possible.

For more information - please refer to our [Encryption Key Provision documentation](../DeveloperDocumentation/VUDRM-key-provision.html#).

### VUPLAY Demo

VUPLAY is our internal demo player. If you access this page by clicking the tab on the admin site, a token with an open policy is loaded into the player. If you access the player directly, the `Custom VUDRM Token` text box will be blank.

You can copy and paste a token you created in the `VUDRM Token` section of the `Info` page. This can be used to test content with your variation of the token. 

<img src="../_static/user-guide-images/8vuplay.png" class="image" width="1000" alt="VUALTO Admin - VUPLAY Page"/>

The different options are as follows: 

* `Default no DRM`  - this will play our demo content with no DRM
* `Default DRM`     - this will play our demo content with DRM. A `VUDRM Token` is required to view it
* `Custom Stream`   - this can be used to test custom content with a `VUDRM Token`

If you select `Custom Stream`, you will be presented with two text boxes to edit:

* `Custom stream url` - your onboarding document contains a link to an encrypted video for you to use with the player, copy and paste that link into the text box. The player will automatically transform this URL to HLS (`.m3u8`) when Fairplay is required
* `Custom VUDRM Token` - as mentioned above, this may be blank or contain a token depending on how the player was accessed. Please ensure the text box contains a valid token

Clicking `Load Player` will then load the content with the conditions of the token.

## User

On the `User` section of the VUDRM Admin site, you have the option of changing your password.

<img src="../_static/user-guide-images/9userpage.png" class="image" width="1000" alt="VUALTO Admin - User Page"/>

## Health

The `Health` page gives an overview of the general health for each DRM or DRM related service we provide. If one of these services are down, it will be down throughout all regions.

<img src="../_static/user-guide-images/10healthpage.png" class="image" width="1000" alt="VUALTO Admin - Health Page"/>

By default, the page is set to auto-refresh every 60 seconds, but you can toggle this to `No` if not required.

The world map contains region specific services. A regions health colour will change in the following circumstances: 

* `GREEN`   - All services are operational
* `AMBER`   - 1 or more services are not operational
* `RED`     - All services are not operational
