# VUDRM TOKEN 

The VUDRM token has two purposes: authentication and the delivery of the DRM policy to the license server. It also represents a signed authorisation on the client's behalf for Vualto to issue a DRM license to the holder of the token, issuing a VUDRM token to a player will grant that player access to the DRM-protected content.

Due to the first purpose VUDRM tokens have a limited lifetime and are designed to be single use. Please contact support@vualto.com if you do not know what the VUDRM token's TTL is for your account.

The second purpose allows the user's playback rights for individual pieces of content to be set dynamically. A single user may be granted different rights on a single piece of content depending on business requirements.

VUDRM tokens should be generated using the [VUDRM token API](#vudrm-token-api). The request to the [VUDRM token API](#vudrm-token-api) should be made from a server side application and the VUDRM token should then be delivered to the client side for use by a player in a license request.

<div class="admonition warning">
    <p class="first admonition-title">Warning</p>
    <p class="last">VUDRM tokens must not be generated on the client side.</p>
</div>

## VUDRM token structure

```vualto-demo|2021-07-08T10:55:49Z|v2|Yv84nG7Eys0znaR2EUZ6/Y3okFv/0RPj54BLgu3YvyE=|2c6c5404278ab1196d3b17cd6d535563b686f822899ffc3184f4ab7721b817f0```

The VUDRM token is comprised of five components each separated by the pipe character (ASCII code 124):
- The client name.
- The time the token was generated in an ISO8601 format (yyyy-MM-ddThh:mm:ssZ).
- `v2` to denote the version of the token.
- The encrypted DRM policy.
- A signed hash.

<div class="admonition danger">
    <p class="first admonition-title">Danger</p>
    <p class="last">VUDRM tokens are designed to be single use and have a one to one relationship with the license request. Each license request must use a new VUDRM token. VUDRM tokens will expire, the TTL is set at an account level and in production will be set to around 30 seconds. The time a token will expire is calculated by the time the token is generated plus the TTL; The time a token was generated is displayed in the token. This is a separate time to the DRM policy times set in the encrypted DRM policy.</p>
</div>

## DRM policy

The DRM policy is included in the VUDRM token as the encrypted third component. When using multiple DRM providers the parameters that are applicable to `ALL` will work across all DRM technologies. Parameters specific to one DRM provider can be used but will be ignored if not relevant. For example, a VUDRM token can be created that pertains to both Widevine and PlayReady. Any PlayReady specific parameters will only be applied when a PlayReady licence is required and will be ignored for Widevine.

### DRM policy structure

```json
{
    ... general values ...
    "fairplay" : {
        ... fairplay specifc values ...
    },
    "playready" : {
        ... playready specifc values ...
    },
    "widevine" : {
        ... widevine specifc values ...
    }
}
```

The VUDRM token is can be comprised of up to four sections:
- General values which will apply to **all** DRM providers.
- A `fairplay` section that wil only apply to Fairplay.
- A `widevine` section that wil only apply to Widevine.
- A `playRady` section that wil only apply to PlayReady.

Values specified in a DRM specifc section will override values in the general section of the policy.

The values that can be specified in the general section are as follows:

| Key                       | Type                    | Format            | Description
|---------------------------|-------------------------|-------------------|-------------
| `content_id`              | string                  | Any               | The content identifier. This is used in conjunction with `match_content_id` and for tracking content in the stats; it is not required.
| `rental_duration_seconds` | int                     | Seconds           | How long the license is valid for before it is used.
| `firstplayback_seconds`   | int                     | Seconds           | Once playback is initiated the user will have this window to complete playback. Once the window completes the license will expire. **You can not use `firstplayback` when `liccache` is set to `no`.**
| `offline`                 | bool                    | `true` or `false` | Should the license be cached for offline playback.
| `can_renew`               | bool                    | `true` or `false` | Should the license be renewable.
| `match_content_id`        | bool                    | `true` or `false` | Boolean for if the `content_id` in the token should be compared with the content id used in the license request; the content id in the license request will be the same as the content id used to generate the keys that encrypted a given piece of content.
| `block_vpn_and_tor`       | bool                    | `true` or `false` | Boolean for if ips coming from known vpn or tor networks should be blocked.
| `geo_whitelist`           | string array            | Array of 3 letter country codes (ISO 3166-1 alpha-3) | ALL | A list of 3 letter country codes, ISO 3166-1 alpha-3, for all countries that are allowed access. 
| `session`                 | JSON object             | See below         | A JSON object containing information about a DRM session that can be used to allow/deny a user's license requests.
| `client_data`             | list of key value pairs | JSON              | A list for extra information you wish to provide. Can be used to filter license stats.

The values that can be specified in the Fairplay section are as follows:
| Key                        | Type                    | Format            | Description
|----------------------------|-------------------------|-------------------|-------------
| `rental_duration_seconds`  | int                     | Seconds           | How long the license is valid for before it's used.
| `license_duration_seconds` | int                     | Seconds           | How long the license is valid for after it's initial use.
| `firstplayback_seconds`    | int                     | Seconds           | Once playback is initiated the user will have this window to complete playback. Once the window completes the license will expire. **You can not use `firstplayback` when `liccache` is set to `no`.**

The values that can be specified in the PlayReady section are as follows:
| Key                        | Type                    | Format                    | Description
|----------------------------|-------------------------|---------------------------|-------------
| `begin_datetime`           | datetime                | ISO 8601                  | The start date for the license.
| `end_datetime`             | datetime                | ISO 8601                  | The end date for the license.
| `firstplayback_seconds`    | int                     | Seconds                   | Once playback is initiated the user will have this window to complete playback. Once the window completes the license will expire. **You can not use `firstplayback` when `liccache` is set to `no`.**
| `security_level`           | int                     | `150` OR `2000` OR `3000` | https://docs.microsoft.com/en-us/playready/overview/

The values that can be specified in the Widevine section are as follows:
| Key                        | Type                    | Format            | Description
|----------------------------|-------------------------|-------------------|-------------
| `rental_duration_seconds`  | int                     | Seconds           | How long the license is valid for before it's used.
| `license_duration_seconds` | int                     | Seconds           | How long the license is valid for after it's initial use.
| `firstplayback_seconds`    | int                     | Seconds           | Once playback is initiated the user will have this window to complete playback. Once the window completes the license will expire. **You can not use `firstplayback` when `liccache` is set to `no`.**
| `security_level`           | int                     | `1` - `5`         | See below

### Online renewable licenses in Widevine

In order to have online licenses automatically renew when using Widevine you must first ensure the policy your VUDRM token is using contains `can_renew` set to `true`. With this value set Widevine will try to renew the license. This renewal will happen every 1800 seconds, although we can configure value so that license requests are more/less frequent based on your business needs. Although new licenses will be fetched at regular intervals, this interval is not based on when a license will expire and as such it is required that the license is valid for longer than the length of the interval between renewals. Below is an example policy for a rewnable license that is only valid for 45 minutes but will renew ever 30 minues.

```JSON
{
    "can_renew": true,
    "rental_duration_seconds": 2700
}
```

### Match content ID
If `content_id` has been set and `match_content_id` has been set to `true`, when a license request is made the content ID in the license request and the content ID in the VUDRM token will be compared. If they **are** the same a license will be served as normal; if they are **not** the same the license request will be denied. If `match_content_id` has been set to `false` **or** `content_id` has not been specified then this comparision will not occur. 

### GEO whitelisting
If `geo_whitelist` has been added to the DRM policy, when a license is requested the GEO location of the client's ip will be checked. If the country is in the whitelist the license request will be successful. If the country is not in the whitelist the request will be denied. E.g. if the DRM policy is `{ "geo_whitelist":[ "gbr", "deu" ] }` only users from the UK and Germany will be able to make successful license requests.

### DRM session in policy

It is possible to set in the policy used by VUDRM token information about the current DRM session. With this information we will make a request to make to an API of your choosing with an ID and any needed headers. You can set the request to be either a GET request or a POST request. In the case of a GET request we would make a GET request to the URL you specify with a query string parameter called `sessionId` set to ID the passed in the policy. E.g. a GET request to `https://some-url.com/valid?sessionId=abc123`. In the case of a POST request we would make a POST request to the URL you specify with the body of the request being the `sessionId` in JSON. E.g. a POST request to `https://some-url.com/valid` with the body being `{"sessionId":"abc123"}`. The result of the request we make will dictate whether or not a license is returned. If the result **is** a `200` we will serve a valid license back. If the result is **not** a `200` we will not return a valid license. 

There are four parts to the session information: 

| Key         | Type                     | Format                     | Example
|-------------|--------------------------|----------------------------|--------
| `id`        | string                   | Any                        | `abc123`
| `url`       | string                   | URL                        | `https://some-url.com/valid`
| `method`    | string                   | `GET` OR `POST`            | `GET`
| `headers`   | key-value pairs          | `{ "key", "value", ... }`  | `{ "someKey", "someValue" }`

The structure of the session information in a policy should be as follows:

```
{
    ... other policy values ...
    "session": {
        "id": "...",
        "url": "...",
        "method": "...",
        "headers": { "headerKey": "headerValue", ... }
    }
}
```

#### Example policy with session information
```
{
    "polbegin": "02-04-2019 12:00:00",
    "polend": "02-04-2019 17:00:00",
    "session": {
        "id": "someID",
        "url": "https://www.some-url.com/valid",
        "method": "GET",
        "headers": { "myHeader": "someValue", "myOtherHeader": "someOtherValue" }
    }
}
```

### Default DRM policy
It is possible to specify a default DRM policy that will be used when we receive a VUDRM token. For example if you wish for all licenses requested to default to caching the license, the default DRM policy would be `{ "liccache":"yes" }`, meaning a VUDRM token with the policy `{ "polend":"DD-MM-YYYY HH:mm:ss" }` would be the same as `{ "liccache":"yes", "polend":"DD-MM-YYYY HH:mm:ss" }`. The values in the default policy can be overridden by simply putting them in the policy you pass in the VUDRM token. For example if the default DRM policy was `{ "liccache":"yes" }` but you wanted a license to not be cached, the policy in the VUDRM token would be `{ "liccache":"no" }`. Any of the values listed above can be be set in the default DRM policy.

If you wish to utilise this feature please contact support@vualto.com.

### VUDRM token API

VUDRM tokens can be generated by making a request to the VUDRM token api: https://token.vudrm.tech/v2/generate

The request to the VUDRM token API needs to be a POST and requires an `x-api-key` header with your account API key as the value. Please contact support@vualto.com if you do not have this.

The body of the POST request comprises of the account name and the DRM policy.

For example:

```JSON
{
    "clientName": "YOUR_NAME",
    "policy": {
        "content_id": "filename",
        "rental_duration_seconds": 3600
    }
}   
``` 

An example request to the VUDRM token API in curl is:

```bash
curl -X POST \
  https://token.vudrm.tech/generate \
  -H 'x-api-key: <your-api-key>' \
  -d '{"clientName": "<client>","policy": {"content_id":"<content-id>","rental_duration_seconds":"<duration>","offline":false}}'
```

## Legacy

### VUDRM token structure

```test-client|2018-26-11T11:01:04Z|cO9H1B2VKw0WyyNTOfoHEw==|05aff2dd06f4c52291b7a032c815ce8f599cf409```

The VUDRM token is comprised of four components each separated by the pipe character (ASCII code 124):
- The client name.
- The time the token was generated in an ISO8601 format (yyyy-MM-ddThh:mm:ssZ).
- The encrypted DRM policy.
- A signed hash.

<div class="admonition danger">
    <p class="first admonition-title">Danger</p>
    <p class="last">VUDRM tokens are designed to be single use and have a one to one relationship with the license request. Each license request must use a new VUDRM token. VUDRM tokens will expire, the TTL is set at an account level and in production will be set to around 30 seconds. The time a token will expire is calculated by the time the token is generated plus the TTL; The time a token was generated is displayed in the token. This is a separate time to the DRM policy times set in the encrypted DRM policy.</p>
</div>

### DRM policy

The DRM policy is included in the VUDRM token as the encrypted third component. When using multiple DRM providers the parameters that are applicable to `ALL` will work across all DRM technologies. Parameters specific to one DRM provider can be used but will be ignored if not relevant. For example, a VUDRM token can be created that pertains to both Widevine and PlayReady. Any PlayReady specific parameters will only be applied when a PlayReady licence is required and will be ignored for Widevine.

| Key                | Type     | Format                    | Applicable DRM | Description                                                                                                                     |
|--------------------|----------|---------------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------|
| `content_id`       | string   | Any                       | ALL            | The content identifier. This is used for tracking content in the stats and is not required.                                     |
| `polbegin`         | datetime | ISO 8601 or DD-MM-YYYY HH:mm:ss      | ALL            | **The non ISO 8601 format is no longer recommended, please use the ISO 8601 format.** Policy Begin. The start date for the license. Default value depends on the DRM provider.  If using the ISO 8601 format the timezone required can be specified in the value given, e.g. `2021-03-19T14:30:00+01:00`, otherwise the default timezone for this value is Europe/London. This value comes from [this database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), of which we support a subset; if you wish to use a different timezone please contact support@vualto.com.                                                     |
| `polend`           | datetime | ISO 8601 or DD-MM-YYYY HH:mm:ss       | ALL            | **The non ISO 8601 format is no longer recommended, please use the ISO 8601 format.** Policy End. The end date for the license. Default value depends on the DRM provider. If using the ISO 8601 format the timezone required can be specified in the value given, e.g. `2021-03-19T14:30:00+01:00`, otherwise the default timezone for this value is Europe/London. This value comes from [this database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones), of which we support a subset; if you wish to use a different timezone please contact support@vualto.com.                                                       |
| `liccache`         | string   | `yes` OR `no`             | ALL            | Licence Cache. Should the license be cached. Implementation depends on the DRM provider.                                                       |
| `firstplayback`    | integer  | Seconds                   | PlayReady, Widevine      | Once playback is initiated the user will have this window to complete playback. Once the window completes the license will expire.  **You can not use `firstplayback` when `liccache` is set to `no`.** | 
| `securitylevel`    | integer  | `150` OR `2000` OR `3000` | PlayReady      | https://docs.microsoft.com/en-us/playready/overview/security-level                                                              |
| `type`             | string   | `r` OR `l` OR `p`         | Fairplay       | rental, lease, or persist. See [Fairplay DRM policy](#fairplay-drm-policy) section for details.                                                              |
| `duration_rental`  | string   | Seconds                   | Fairplay       | Overrides polend for Fairplay rental licenses.                                                               |
| `duration_lease`   | string   | Seconds                   | Fairplay       | Overrides polend for Fairplay lease licenses.                                                              |
| `duration_persist` | string   | Seconds                   | Fairplay       | Overrides polend for Fairplay persist licenses.                                                           |
| `match_content_id` | bool     | `true` or `false`         | ALL       | Boolean for if the content ID in the token should be compared with the content ID used in the license request; the content ID in the license request will be the same as the content ID used to generate the keys that encrypted a given piece of content. 
| `geo_whitelist`     | string array | Array of 3 letter country codes (ISO 3166-1 alpha-3) | ALL | A list of 3 letter country codes, ISO 3166-1 alpha-3, for all countries that are allowed access. 
| `block_vpn_and_tor` | bool    | `true` or `false`         | ALL   | Boolean for if ips coming from known vpn or tor networks should be blocked. |
| `session`          | JSON     | [See below](#drm-session-in-policy) | ALL | A JSON object containing information about a DRM session that can be used to allow/deny a user's license requests.

This table is not an exhaustive list, for example it does not include advanced PlayReady settings. If you require the use of more advanced settings please contact support@vualto.com

The `polbegin` and `polend` settings use the timezone set at the account level. Please contact support@vualto.com to confirm the timezone for your account. 

#### Default values

| Key               | Fairplay | PlayReady             | Widevine |
|-------------------|----------|-----------------------|----------|
| `content_id`      | Not set  | Not set               | Not set  | 
| `polbegin`        | Now*     | `01-01-0001 00:00:00` | Now*     | 
| `polend`          | Now*     | `12-31-9999 23:59:59` | Now*     | 
| `liccache`        | `no`     | `yes`                 | `no`     | 
| `firstplayback`   | N/A      | 10,675,199 days       | Not set  | 
| `securitylevel`   | N/A      | `2000`                | N/A      | 
| `type`            | `l`      | N/A                   | N/A      | 
| `duration_rental` | `0`      | N/A                   | N/A      |
| `duration_lease`  | `0`      | N/A                   | N/A      |
| `duration_persist`| `0`      | N/A                   | N/A      |
| `match_content_id`| `false`  | `false`               | `false`  | 
| `geo_whitelist`   | Not set  | Not set               | Not set  | 
| `session`         | Not set  | Not set               | Not set  | 

*When these values have not been set in the policy, as long as no other values would cause playback to stop, content will play indefinitely.

There are also limitations depending on environments that are not explained in the table. Please refer to the following sections for more detail:

#### Match content ID
If `content_id` has been set and `match_content_id` has been set to `true`, when a license request is made the content id in the license request and the content ID in the VUDRM token will be compared. If they **are** the same a license will be served as normal; if they are **not** the same the license request will be denied. If `match_content_id` has been set to `false` **or** `content_id` has not been specified then this comparision will not occur. 

#### GEO whitelisting
If `geo_whitelist` has been added to the DRM policy, when a license is requested the GEO location of the client's ip will be checked. If the country is in the whitelist the license request will be successful. If the country is not in the whitelist the request will be denied. E.g. if the DRM policy is `{ "geo_whitelist":[ "gbr", "deu" ] }` only users from the UK and Germany will be able to make successful license requests.

#### DRM session in policy

It is possible to set in the policy used by VUDRM token information about the current DRM session. With this information we will make a request to make to an API of your choosing with an ID and any needed headers. You can set the request to be either a GET request or a POST request. In the case of a GET request we would make a GET request to the URL you specify with a query string parameter called `sessionId` set to ID the passed in the policy. E.g. a GET request to `https://some-url.com/valid?sessionId=abc123`. In the case of a POST request we would make a POST request to the URL you specify with the body of the request being the `sessionId` in JSON. E.g. a POST request to `https://some-url.com/valid` with the body being `{"sessionId":"abc123"}`. The result of the request we make will dictate whether or not a license is returned. If the result **is** a `200` we will serve a valid license back. If the result is **not** a `200` we will not return a valid license. 

There are four parts to the session information: 

| Key         | Type                     | Format                     | Example
|-------------|--------------------------|----------------------------|--------
| `id`        | string                   | Any                        | `abc123`
| `url`       | string                   | URL                        | `https://some-url.com/valid`
| `method`    | string                   | `GET` OR `POST`            | `GET`
| `headers`   | key-value pairs          | `{ "key", "value", ... }`  | `{ "someKey", "someValue" }`

The structure of the session information in a policy should be as follows:

```
{
    ... other policy values ...
    "session": {
        "id": "...",
        "url": "...",
        "method": "...",
        "headers": { "headerKey": "headerValue", ... }
    }
}
```

##### Example policy with session information
```
{
    "polbegin": "02-04-2019 12:00:00",
    "polend": "02-04-2019 17:00:00",
    "session": {
        "id": "someID",
        "url": "https://www.some-url.com/valid",
        "method": "GET",
        "headers": { "myHeader": "someValue", "myOtherHeader": "someOtherValue" }
    }
}
```

#### Default DRM policy
It is possible to specify a default DRM policy that will be used when we receive a VUDRM token. For example if you wish for all licenses requested to default to caching the license, the default DRM policy would be `{ "liccache":"yes" }`, meaning a VUDRM token with the policy `{ "polend":"DD-MM-YYYY HH:mm:ss" }` would be the same as `{ "liccache":"yes", "polend":"DD-MM-YYYY HH:mm:ss" }`. The values in the default policy can be overridden by simply putting them in the policy you pass in the VUDRM token. For example if the default DRM policy was `{ "liccache":"yes" }` but you wanted a license to not be cached, the policy in the VUDRM token would be `{ "liccache":"no" }`. Any of the values listed above can be be set in the default DRM policy.

If you wish to utilise this feature please contact support@vualto.com.

#### PlayReady DRM policy

PlayReady has an extensive list of policy options described [here](https://docs.microsoft.com/en-us/playready/overview/license-and-policies). The majority of these options are supported via the VUDRM token's policy. Please contact support@vualto.com for more details.

#### Fairplay DRM policy

Fairplay has three types of license; `rental`, `lease`, and `persist`. 

Fairplay licenses can only be persisted past the user session by an offline enabled iOS application. When using Safari a session will be preserved until the browser tab is closed.

Fairplay does not allow playback over non-HDCP connections. 

##### Fairplay rental DRM policy

You can specify a `rental` license by setting the `type` key to `r` in the DRM policy.

You can set the expiry of the license using the `duration_rental` key or the `polend` key. If both `duration_rental` and `polend` are set in a policy then `duration_rental` will be used.

When using the `type` of `rental` playback will continue after license expiry until the encryption keys change.

##### Fairplay lease DRM policy

You can specify a `lease` license by setting the `type` key to `l` in the DRM policy.

You can set the expiry of the license using the `duration_lease` key or the `polend` key. If both `duration_lease` and `polend` are set in a policy then `duration_lease` will be used.

Using the `type` of `lease` will cause playback to stop at the license expiry. Normally a player will make a new license request at this point, please ensure the VUDRM token is updated before further license requests are made. Using an expired VUDRM token will cause the license request to fail.

##### Fairplay persist DRM policy

A `persist` license is used for offline playback. 

Setting `liccache` to `yes` or the `type` to `p` will generate a persisted license. Setting `liccache` to `yes` will override any other `type` settings.

Setting `type` to `p` will override setting `liccache` to `no`.

You can set the expiry of the license using the `duration_persist` key or the `polend` key. If both `duration_persist` and `polend` are set in a policy then `duration_persist` will be used.

Using the `type` of `persist` will cause playback to stop at the license expiry and the license will be removed from the device.

#### Widevine DRM policy

Widevine licenses can only be persisted past the user session by an Android application. When using Chrome a session will be preserved until the browser tab is closed.

#### DRM policy examples

The following sections display some example polices. These polices apply to all DRM providers.

##### Rental

This policy is designed for a rental business model. The license generated from this policy will expire at the time set by the `polend` value and will allow immediate playback.

```JSON
{
    "content_id":"filename",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "type":"r"
}
```

##### Subscription

This policy is designed for a subscription business model. The license generated from this policy will not allow playback until `polbegin` is reached and will expire when `polend` is reached.

```JSON
{
    "content_id":"filename",
    "polbegin":"DD-MM-YYYY HH:mm:ss",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "type":"l"
}
```

##### Offline playback license

This policy is designed for an offline playback scenario. The license generated from this policy will allow playback immediately and the license will expire when the `polend` value is reached. The license will be cached until the `polend` value is reached.

```JSON
{
    "content_id":"filename",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "liccache":"yes"
}
```

### VUDRM token API

VUDRM tokens can be generated by making a request to the VUDRM token api: https://token.vudrm.tech/generate

The request to the VUDRM token API needs to be a POST and requires an `API_KEY` header with your account API key as the value. Please contact support@vualto.com if you do not have this.

The body of the POST request comprises of the account name and the DRM policy.

For example:

```JSON
{
    "client": "YOUR_NAME",
    "policy": {
        "content_id": "filename",
        "polend": "26-11-2018 16:48:59"
    }
}   
``` 

An example request to the VUDRM token API in curl is:

```bash
curl -X POST \
  https://token.vudrm.tech/generate \
  -H 'API_KEY: <your-api-key>' \
  -d '{"client": "<client>","policy": {"content_id":"<content-id>","polend":"<pol-end>","liccache":"no"}}'
```
