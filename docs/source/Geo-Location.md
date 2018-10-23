# GEO LOCATION API

This API can be used to retrieve geographical location information about IPv4 or IPv6 addresses.

This service requires an API Key and client name. Please contact support@vualto.com if you do not have this information. 

## 1. Make a Request

A `GET` request should be made to the following URL:

`https://geo.drm.technology/ip_lookup/<client>/<ip_address>`

Replace `<client>` with your client name and `<ip_address>` with the IPv4 or IPv6 address.

The `X_AUTH_KEY` header should be set and the value should be the Geo Location API Key.

*This is not the same as your VUDRM token API key.*

## 2. The Response

The response will be in JSON and will look like this:

```json
{
    "ip": "188.39.163.11",
    "geo": {
        "timezone_name": "europe/london",
        "country": "gbr",
        "country_code": 826,
        "country_confidence_percentage": 99,
        "two_letter_country_code": "uk",
        "continent_code": 5,
        "region": "ply",
        "region_code": 25486,
        "region_confidence_percentage": null,
        "area_codes": null,
        "metro_code": 826046,
        "postal_code": "pl1 1ab",
        "postal_confidence_percentage": 30,
        "city": "plymouth",
        "city_code": 12140,
        "city_confidence_percentage": 60,
        "longitude": 0,
        "latitude": 50,
        "gmt_offset": "+0",
        "in_dst": false,
        "connection_speed": "xdsl"
    },
    "proxy": {
        "identification": null,
        "type": null
    }
}
```
The key `ip` details the requested IP address. The following sections explain the other sections in the returned JSON.

### `geo`

| Key | Type | Description |
|-----|------|-------------|
| `timezone_name` | string | |
| `country` | string | | 
| `country_code` | integer | The ISO-3166-1 numeric country code |
| `country_confidence_percentage` | integer|null | The confidence percentage that the country is correct |
| `two_letter_country_code` | string | |
| `continent_code` | string | |
| `region` | string | |
| `region_code` | integer | Includes AOL identification for the associated country, reserved ranges and RFC 1918 reserved ranges. Follows the ISO 3166-2 standardization. |
| `region_confidence_percentage` | integer | The confidence percentage that the region is correct |
| `area_codes` | string | |
| `metro_code` | integer|  Metro codes included Designated Market Areas (DMA’s) in the U.S., ITV regions in the U.K., Department Codes in France, German Nielsen TV Markets, South Korean Si/Gun/Gu, Chinese Diji Shi Cities,  Russian Federal Districts, Norwegian municipalities, urban areas in New Zealand, and the Greater Capital City Statistical Area and Significant Urban Area in Australia. |
| `postal_code` | string |
| `postal_confidence_percentage` | integer | The confidence percentage that the postal code is correct |
| `city` | string | |
| `city_code` | integer | |
| `city_confidence_percentage` | integer | |
| `longitude` | integer | |
| `latitude` | integer | |
| `gmt_offset` | string | Current GMT/UTC offset - format: `+hhmm` |
| `in_dst` | boolean | Are they currently observing DST? |
| `connection_speed` | string | Can have the following values: `dialup`, `DSL`, `cable`, `broadband` (non-specific high-speed connection), `t1`, `t3`, `oc3`, `oc12`, `mobile`, `wireless` and `satellite`. |

NB:
- Confidence values range from 0 to 100, with 0 representing least confidence in data sources and 100 representing total confidence in data sources.
- All values are nullable.

### `proxy`

Possible values for `proxy.identification`:

| Value | Description |
|-------|-------------|
| `tor_exit` | The gateway nodes where encrypted/anonymous Tor traffic hits the Internet. |
| `tor_relay` | Receives traffic on the Tor network and passes it along. Also referred to as "routers". |
| `cloud` | Enables ubiquitous network access to a shared pool of configurable computing resources. |
| `vpn` | Virtual private network that encrypts and routes all traffic through the VPN server, including programs and applications. |
| `dns` | A proxy used by overriding the client’s DNS value for an endpoint host to that of the proxy instead of the actual DNS value. |
| `web_browser` | This value will indicate connectivity that is taking place through mobile device webbrowser software that proxies the user through a centralized location. A couple of examples of  this type of connectivity are Opera mobile browsers and UCBrowser. |
| `cloud_security` | A host accessing the Internet via a web security and data protection cloud provider. Example providers with this type of service are Zscaler, Scansafe, and Onavo. |
| `null` | The IP has not been identified as a proxy. |

Possible values for `proxy.type`:

| Value | Description |
|-------|-------------|
| `anonymous` | Actual IP address of client is not available. Includes services that change location to beat DRM, TOR points, temporary proxies and other masking services. |
| `transparent` | Actual IP address of client is available via HTTP headers, though value is not necessarily reliable (i.e., it can be spoofed). |
| `hosting` | Address belongs to a hosting facility or cloud provider and is likely to be a proxy as end users are not typically located in a hosting facility. |
| `corporate` | Generally considered harmless, but location can be a concern. Can identify if multiple users are proxied through a central location or locations, and thus share a single networkapparent IP address |
| `public` | Multiple users proxied from a location allowing public Internet access. |
| `edu` | Proxied users from an educational institution. |
| `aol` | AOL proxy |
| `blackberry` | This value will identify an IP address owned by RIM (Research In Motion) who is responsible for Blackberry mobile devices. All Blackberry users go through a centralized proxy location and thus cannot be accurately geo-targeted. |
| `null` | Not determined to be a proxy. |

### Example Requests

#### CURL

```bash
curl -X GET \
https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11 \
-H 'X_AUTH_KEY: <geo-location-api-key>'
```

#### GO

```go
package main

import (
	"fmt"
	"net/http"
	"io/ioutil"
)

func main() {

	url := "https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11"

	req, _ := http.NewRequest("GET", url, nil)

	req.Header.Add("X_AUTH_KEY", "<geo-location-api-key>")

	res, _ := http.DefaultClient.Do(req)

	defer res.Body.Close()
	body, _ := ioutil.ReadAll(res.Body)

	fmt.Println(res)
	fmt.Println(string(body))

}
```

#### Ruby (Net:HTTP)

```ruby
require 'uri'
require 'net/http'

url = URI("https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11")

http = Net::HTTP.new(url.host, url.port)

request = Net::HTTP::Get.new(url)
request["X_AUTH_KEY"] = '<geo-location-api-key>'

response = http.request(request)
puts response.read_body
```

#### C# (RestSharp)

```C
var client = new RestClient("https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11");
var request = new RestRequest(Method.GET);
request.AddHeader("X_AUTH_KEY", "<geo-location-api-key>");
IRestResponse response = client.Execute(request);
```

#### Python Requests

```python
import requests

url = "https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11"

headers = {
    'X_AUTH_KEY': "<geo-location-api-key>",
    }

response = requests.request("GET", url, headers=headers)

print(response.text)
```

#### PHP HttpRequest

```PHP
<?php

$request = new HttpRequest();
$request->setUrl('https://geo.drm.technology/ip_lookup/vualto-demo/188.39.163.11');
$request->setMethod(HTTP_METH_GET);

$request->setHeaders(array(
  'X_AUTH_KEY' => '<geo-location-api-key>'
));

try {
  $response = $request->send();

  echo $response->getBody();
} catch (HttpException $ex) {
  echo $ex;
}
```
