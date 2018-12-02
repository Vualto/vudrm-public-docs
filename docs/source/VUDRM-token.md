# VUDRM TOKEN 

The VUDRM token has two purposes: authentication and the delivery of the DRM policy to the license server. It also represents a signed authorisation on the client's behalf for Vualto to issue a DRM license to the holder of the token, issuing a VUDRM token to a player will grant that player access to the DRM-protected content.

Due to the first purpose VUDRM tokens have a limited lifetime and are designed to be single use. Please contact support@vualto.com if you do not know what the VUDRM token's TTL is for your account.

The second purpose allows the user's playback rights for individual pieces of content to be set dynamically. A single user may be granted different rights on a single piece of content depending on business requirements.

VUDRM tokens should be generated using the [VUDRM token API](/projects/vudrm/en/latest/VUDRM-token.html#vudrm-token-api). The request to the [VUDRM token API](/projects/vudrm/en/latest/VUDRM-token.html#vudrm-token-api) should be made from a server side application and the VUDRM token should then be delivered to the client side for use by a player in a license request.

**VUDRM tokens should not be generated on the client side.**

## VUDRM token structure

```test-client|2018-26-11T11:01:04Z|cO9H1B2VKw0WyyNTOfoHEw==|05aff2dd06f4c52291b7a032c815ce8f599cf409```

The VUDRM token is comprised of four components each separated by the pipe character (ASCII code 124):
- The client name.
- The time the token was generated in an ISO8601 format (yyyy-MM-ddThh:mm:ssZ).
- The encrypted DRM policy.
- A signed hash.

## DRM policy

The DRM policy is included in the VUDRM token as the encrypted third component. When using multiple DRM providers the parameters that are applicable to `ALL` will work across all DRM technologies. Parameters specific to one DRM provider can be used but will be ignored if not relevant. For example, a VUDRM token can be created that pertains to both Widevine and PlayReady. Any PlayReady specific parameters will only be applied when a PlayReady licence is required and will be ignored for Widevine.

| Key                | Type     | Format                    | Applicable DRM | Description                                                                                                                     |
|--------------------|----------|---------------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------|
| `contentid`        | string   | Any                       | ALL            | The content identifier. This is used for tracking content in the stats and is not required.                                     |
| `polbegin`         | datetime | DD-MM-YYYY HH:mm:ss       | ALL            | Policy Begin. The start date for the license. Default value depends on the DRM provider.                                                      |
| `polend`           | datetime | DD-MM-YYYY HH:mm:ss       | ALL            | Policy End. The end date for the license. Default value depends on the DRM provider.                                                        |
| `liccache`         | string   | `yes` OR `no`             | ALL            | Licence Cache. Should the license be cached. Implementation depends on the DRM provider.                                                       |
| `firstplayback`    | integer  | Seconds                   | PlayReady      | Once playback initiated the user will have this window to complete playback. Once the window completes the license will expire. | 
| `securitylevel`    | integer  | `150` OR `2000` OR `3000` | PlayReady      | https://docs.microsoft.com/en-us/playready/overview/security-level                                                              |
| `type`             | string   | `r` OR `l` OR `p`         | Fairplay       | See [Fairplay DRM policy](fairplay-drm-policy) section for detail.                                                              |
| `duration_rental`  | string   | Seconds                   | Fairplay       | Overrides polend for rental type policies.                                                               |
| `duration_lease`   | string   | Seconds                   | Fairplay       | Overrides polend for lease type policies                                                              |
| `duration_persist` | string   | Seconds                   | Fairplay       | See [Fairplay DRM policy](fairplay-drm-policy) section for detail.                                                              |

This table is not an exhaustive list, for example it does not include advanced PlayReady settings. If you require the use of more advanced settings please contact support@vualto.com

The `polbegin` and `polend` settings use the timezone set at the account level. Please contact support@vualto.com to confirm the timezone for your account. 

There are also limitations depending on environments that are not explained in the table. Please refer to the following sections for more detail:

### PlayReady DRM policy

PlayReady has an extensive list of policy options described [here](https://docs.microsoft.com/en-us/playready/overview/license-and-policies). The majority of these options are supported via the VUDRM token's policy. Please contact support@vualto.com for more details.

### Fairplay DRM policy

Fairplay has three types of license; `rental`, `lease`, and `persist`. 

Fairplay licenses can only be persisted past the user session by an offline enabled iOS application. When using Safari a session will be preserved until the browser tab is closed.

Fairplay does not allow playback over non-HDCP connections. 

#### Fairplay rental DRM policy

You can specify a `rental` license by setting the `type` key to `r` in the DRM policy.

You can set the expiry of the license using the `duration_rental` key or the `polend` key. If both `duration_rental` and `polend` are set in a policy then `duration_rental` will be used.

When using the `type` of `rental` playback will continue after license expiry until the encryption keys change.

#### Fairplay lease DRM policy

You can specify a `lease` license by setting the `type` key to `l` in the DRM policy.

You can set the expiry of the license using the `duration_lease` key or the `polend` key. If both `duration_lease` and `polend` are set in a policy then `duration_lease` will be used.

Using the `type` of `lease` will cause playback to stop at the license expiry. Normally a player will make a new license request at this point, please ensure the VUDRM token is updated before further license requests are made.

#### Fairplay persist DRM policy

The use case of the `persist` license is for offline playback. 

You can specify a `persist` license by setting the `type` key to `p` in the DRM policy. Setting `licache` to `yes` will also cause a persist license to be generated. If `type` and `liccache` are set in the DRM policy then the license type will be set to `persist`.

You can set the expiry of the license using the `duration_persist` key or the `polend` key. If both `duration_persist` and `polend` are set in a policy then `duration_persist` will be used.

Using the `type` of `persist` will cause playback to stop at the license expiry and the license will be removed from the device.

### Widevine DRM policy

Widevine licenses can only be persisted past the user session by an Android application. When using Chrome a session will be preserved until the browser tab is closed.

### DRM policy examples

The following sections display some example polices. These polices apply to all DRM providers.

#### Rental

```JSON
{
    "contentid":"filename",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "type":"r"
}
```

#### Subscription

```JSON
{
    "contentid":"filename",
    "polbegin":"DD-MM-YYYY HH:mm:ss",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "type":"l"
}
```

#### Offline playback license

```JSON
{
    "contentid":"filename",
    "polend":"DD-MM-YYYY HH:mm:ss",
    "liccache":"yes"
}
```

## VUDRM token API

VUDRM tokens can be generated by making a request to the VUDRM token api: https://token-api.drm.technology/generate

The request to the VUDRM token API needs to be a POST and requires an `API_KEY` header with your account API key as the value. Please contact support@vualto.com if you do not have this.

The body of the POST request comprises of the account name and the DRM policy.

The quotes in the DRM policy must be **escaped**. 

For example:

```JSON
{
    "client": "YOUR_NAME",
    "policy": "{\"contentid\":\"filename\",\"polend\":\"26-11-2018 16:48:59\"}"
}   
``` 

### VUDRM token API examples

#### CURL

```bash
curl -X POST \
  https://token-api.drm.technology/generate \
  -H 'API_KEY: <your-api-key>' \
  -d '{"client": "<client>","policy": "{\"contentid\":\"<content-id>\",\"polend\":\"<pol-end>\",\"liccache\":\"no\"}"}'
```

#### GO

```go
package main

import (
	"fmt"
	"strings"
	"net/http"
	"io/ioutil"
)

func main() {

	url := "https://token-api.drm.technology/generate"

	payload := strings.NewReader("{\"client\": \"<client>\",\"policy\": \"{\\\"contentid\\\":\\\"<content-id>\\\",\\\"polend\\\":\\\"<pol-end>\\\",\\\"liccache\\\":\\\"no\\\"}\"}")

	req, _ := http.NewRequest("POST", url, payload)

	req.Header.Add("API_KEY", "<api-key>")

	res, _ := http.DefaultClient.Do(req)

	defer res.Body.Close()
	body, _ := ioutil.ReadAll(res.Body)

	fmt.Println(res)
	fmt.Println(string(body))

}
```

#### Ruby (NET::Http)

```ruby
require 'uri'
require 'net/http'

url = URI("https://token-api.drm.technology/generate")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Post.new(url)
request["API_KEY"] = '<api-key>'
request.body = "{\"client\": \"<client>\",\"policy\": \"{\\\"contentid\\\":\\\"<content-id>\\\",\\\"polend\\\":\\\"<pol-end>\\\",\\\"liccache\\\":\\\"no\\\"}\"}"

response = http.request(request)
puts response.read_body
```

#### C# (RestSharp)

```c#
var client = new RestClient("https://token-api.drm.technology/generate");
var request = new RestRequest(Method.POST);
request.AddHeader("API_KEY", "<api-key>");
request.AddParameter("undefined", "{\"client\": \"<client>\",\"policy\": \"{\\\"contentid\\\":\\\"<content-id>\\\",\\\"polend\\\":\\\"<pol-end>\\\",\\\"liccache\\\":\\\"no\\\"}\"}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

#### Python Requests

```python
import requests

url = "https://token-api.drm.technology/generate"

payload = "{\"client\": \"<client>\",\"policy\": \"{\\\"contentid\\\":\\\"<content-id>\\\",\\\"polend\\\":\\\"<pol-end>\\\",\\\"liccache\\\":\\\"no\\\"}\"}"
headers = {
    'API_KEY': "<api-key>"
    }

response = requests.request("POST", url, data=payload, headers=headers)

print(response.text)
```

#### PHP HttpRequest

```PHP
<?php

$request = new HttpRequest();
$request->setUrl('https://token-api.drm.technology/generate');
$request->setMethod(HTTP_METH_POST);

$request->setHeaders(array(
  'API_KEY' => '<api-key>'
));

$request->setBody('{"client": "<client>","policy": "{\\"contentid\\":\\"<content-id>\\",\\"polend\\":\\"<pol-end>\\",\\"liccache\\":\\"no\\"}"}');

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```
