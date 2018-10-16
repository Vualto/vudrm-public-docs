# Token Integration 

## 1. Summary

The purpose of this document is to outline the work required to integrate the vudrm token solution into the client’s DRM workflow. This document replaces previous versions and instead of providing integration instructions for each individual DRM provider, now allows a common set of parameters to be used across multiple DRM platforms. This new common layer supports the following DRM technologies: 

• Adobe Primetime (formerly Access)

• Microsoft PlayReady 

• Google Widevine Modular 

There are some settings that do now apply to every DRM. These have been indicated. 

## 2. Token Overview

In order to authenticate a request for playback, a system of signed tokens is used.  
 
• The signed token includes all the policy information and the playback rules for the content. 

• These tokens are usually created by the client CMS and are passed to the player along with the URL of the content. This means that DRM policies can be created on-the-fly and can change according to the playback/user scenario. This methods avoids having to ‘burn in’ the policy information into the content and avoids having to re-encypt the content. 

• When playback of the content is requested, the player sends the token to the appropriate DRM Licence Server. 

• The signed token identifies and authenticates the player to Vualto, and specifies what licence parameters should be set in the new licence. 

• This token represents a signed authorisation on the client’s behalf for Vualto to issue a license to the holder of the token, so issuing a token to a player will grant that player access to the DRM-protected content. The lifetime of a token is specified in the client account settings but the default is 60 seconds. If the token is not used by the player within the lifetime then the token expires and will no longer be accepted by the license server.  
 
(For Adobe Primetime only, we also support ‘pre-defined’ DRM policies files.) 


### 2.1 Token Format for On-the-fly Policies

The Signed tokens contain:

• The client Account name

• Creation timestamp (Format ISO8601 yyyy-MM-ddThh:mm:ssZ)

• JSON parameters appropriate to DRM Policy requirements (Encrypted)

• Signed hash 

**Example Encrypted Token:**

```TestClientX|2014-0906T14:33:04Z|cO9H1B2VKw0WyyNTOfoHEw==|05aff2dd06f4c52291b7a032c815ce8f599cf409```


Each component is separated with a vertical bar character (ASCII code 124).   
The JSON is encrypted using AES (rijndael 128) so should anyone intercept the request the DRM policy token information is secure. 


### 2.2 Token Format for Pre-Defined Policy (Primetime Only) 

If the policy is to be ‘burnt’ into the content, then instead of specifying the Policy JSON, a policy name is specified. 

The signed token contains: 

• The client Account name

• Creation timestamp (Format ISO8601 yyyy-MM-ddThh:mm:ssZ)

• Policy Name (eg. 30 days, DTO)

• Signed hash 

Example Encrypted Token 

```TestClient|2012-03-09T11:27:57Z|30 Day|e03a31b675a5191235dc49c83ddbfe06cf66f44a```

Each component is separated with a vertical bar character (ASCII code 124).   
The JSON is encrypted using AES (rijndael 128) so should any one intercept the request the DRM policy token information is secure. 
 


### 2.3 JSON Parameters for DRM Policies

The following parameters are the most commonly used to specify DRM policies. It is not an exhaustive list but covers off most common scenarios.  

For those clients using multiple DRM providers, the Parameters that are applicable to All will work across DRM technologies. Parameters specific to one DRM can be used but will be ignored if not relevant. For example, a token can be created that pertains to both Widevine and PlayReady. Any PlayReady specific parameters from the list below will only be applied when a PlayReady licence is required and will be ignored for Widevine. 


|Context          |Parameter Name | Parameter Contents/Format  | Description                  | Appicable DRM
|-----------------|---------------|----------------------------|------------------------------|------
|                 | contentid     | name of the file           | An identifier of the content |ALL
| Policy Duration | polbegin      | dd-mm-yyyy hh:mm:ss        | The policy Start date        |ALL
|                 | polend        | dd-mm-yyyy hh:mm:ss        | The policy Expiration date   |ALL
| Licence Caching | licdelete     | dd-mm-yyyy hh:mm:ss        | Date licence is removed from local device if it still exists | ALL
|                 | liccache      | yes/no                     | Whether licence is cached (persistent) or not cached (nonpersistent) on local device | ALL
|                 | lic.cache.after  | Integer (seconds)       | Delete licence after number of 6 seconds | Primetime
| Playback Window | firstplayback | Integer (seconds)          | Time from the first playback to licence expiry. Used in rental situations | ALL
| Output Protection| out.analog   | [noprot = no protection, req = required, use = use if available, noplay = no playback] req/use/noplay |   | Primetime
|                 | out.digital   | [noprot = no protection, req = required, use = use if available, noplay = no playback] req/use/noplay |   | Primetime
| **              | right.AnalogVideoOPLl | Integer value eg. 200 | Minimum Analog Video Output Protection Level | PlayReady
| **              | right.UncompressedDigitalAudioOPL  | Integer value eg. 200 | Minimum Uncompressed Digital Audio Output Protection Level | PlayReady
| **              | right.UncompressedDigitalVideoOPL  | Integer value eg. 200 | Minimum Uncompressed Digital Video Output Protection Level | PlayReady
| **              | right.CompressedDigitalVideoOPL    | Integer value eg. 200 | Minimum Compressed Digital Video Output Protection Level   | PlayReady
|                 | track_type                         | String                | Track type name such as video/audio etc.                   | Widevine
|                 | security_level                     | Integer               | Client robustness requirements for playback: **1.** The Software based whitebox crypto is required **2.** Software crypto & obfuscated decoder is required  **3.** Key material and crypto ops must be performed within the hardware backed, trusted execution environment. **4.** Crypto and decoding performed within hardware backed trusted execution environment. **5.** The crypto, decoding and all handling of the media must be handled in a hardware backed  trusted execution environment | Widevine
|                 | required_output_protection.hdcp | HDCP_NONE, HDCP_V1, HDCP_V2 | Indicates whether HDCP is required. | Widevine
|                 | key           | Base64                      | Content key to use for this track. If used, then the track_type or key_id is required. | Widevine
|                 | key_id        | Base64 encoded string       | Unique identifier for the key | Widevine
| Other Common Parameters  | Securitylevel | 150/2000           | Refers to the robustness level of DRM SDK. 150  for non-commercial application (testing/development) 2000 for commercial application | PlayReady
|                 | airplay       | Use "1" to enable Airplay   |              | PlayReady
|                 | lic.chain     | yes / no (default)          | Whether a root licence can unlockleaf licences | Primetime
| Fairplay Token Requirement | type        | string "r"  | "r" for rental, "l" for lease, "b"string  for both | Fairplay
| Fairplay Token Requirement | duration_rental | 3600    | The period before a new FPS license request is needed | Fairplay

** PlayReady output protection is complex and there are too many parameter values to list in this table. 

For a full explanation of PlayReady output protection, please see: 

```url
https://www.microsoft.com/playready/licensing/compliance/
```

#### Simple JSON Examples

**Example Rental** : All DRM Providers:

```JSON

    {
        "contentid":"filename",
        "polbegin":"DD-MM-YYYYHH:MM:SS",
        "licdelete":"DD-MM-YYYYHH:MM:SS",
        "liccache":"yes",
        "firstplayback":172800
    }

```
The policy needs to be timestamped with the correct date and time of when the rental period ends. This ensures that any request for a license outside the policy window will fail and that any cached license is also removed. 

Each time a user requests playback it will check the local cache for a license. If it finds one it will play (if the license is valid) and if not it will request one from the server. As long as the client continues to timestamp the policy end date based on the initial purchase it will be valid for the correct period.


**Example Download to Own** : All DRM Providers:

```JSON

    {
        "contentid":"filename",
        "polbegin":"DD-MM-YYYY HH:MM:SS",
        "liccache":"yes"
    }
```

**Example Subscription** : All DRM Providers:

``` JSON
    {
        "contentid":"filename",
        "polend":"DD-MM-YYYY HH:MM:SS",
        "liccache":"no"
    }

```

### 2.5 Using the Token API

We provide a secure API to generate a token for your content. This is accessed from the following URL:

``` URL
https://token-api.drm.technology/generate
```

 
The request needs to be a post and requires the following: 

• A Header called “API-KEY” this key is your api-key which is provided behind your login at ```admin.drm.technology```

• A body comprising of your name and the policy (which is generated using the above guides).

The body contains a JSON object which will need to be **escaped**. For example: 

```JSON

    {
        "client": "YOUR_NAME",
        "policy": "{
                    \"contentid\":\"test\",
                    \"polend\":\"04-01-2016 16:48:59\",
                    \"liccache\":\"no\"
                   }"
    }   
``` 



#### 2.5.1 Token API Request Example

##### CURL

```bash
curl -X POST \
  https://token-api.drm.technology/generate \
  -H 'API_KEY: <your-api-key>' \
  -d '{"client": "<client>","policy": "{\"contentid\":\"<content-id>\",\"polend\":\"<pol-end>\",\"liccache\":\"no\"}"}'
```

##### GO

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

##### Ruby (NET::Http)

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

##### C# (RestSharp)

```c
var client = new RestClient("https://token-api.drm.technology/generate");
var request = new RestRequest(Method.POST);
request.AddHeader("API_KEY", "<api-key>");
request.AddParameter("undefined", "{\"client\": \"<client>\",\"policy\": \"{\\\"contentid\\\":\\\"<content-id>\\\",\\\"polend\\\":\\\"<pol-end>\\\",\\\"liccache\\\":\\\"no\\\"}\"}", ParameterType.RequestBody);
IRestResponse response = client.Execute(request);
```

##### Python Requests

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

##### PHP HttpRequest

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





















 
