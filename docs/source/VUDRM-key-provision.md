# ENCRYPTION KEY PROVISION

The VUDRM key provision service exposes the generation of DRM encryption. It provides a secure REST API allowing you to retrieve encryption keys for all DRM types in order to encrypt your content.

## Key provider API

The key provider API will provide all the information required in order to encrypt various types of content. In order to be as flexible as possible it will return encryption keys in Base16 and Base64 as some systems require different values. 

The following sections describe the requests and responses in order to retrieve the keys. The most common scenario is to request CENC and Fairplay encryption keys. 

### Request

A `GET` should be made to the following URL:

```URL
https://keyprovider.vudrm.tech/<DRM_TYPE>/<CLIENT_NAME>/<CONTENT_ID> 
```

The breakdown of this URL is:
- `DRM_TYPE`: The type of encryption keys being requested. Possible values are `cenc`, `fairplay`, `widevine`, `playready`, and `aes`. See [Using the encryption keys](#using-the-encryption-keys) for more information.
- `CLIENT_NAME`: The account name. Plese contact support@vualto.com if you do not have an account name.
- `CONTENT_ID`: A unique content identifier. This value will always generate the same encryption keys.

In order to provide the keys securely, an API key is required in the header of the request. Please contact support@vualto.com if you do not have an API key.

Below is an example curl request for `cenc` encryption keys using the client `vualto` and with the `CONTENT_ID` of `test`.

```bash
curl -X "GET" "https://keyprovider.vudrm.tech/cenc/vualto/test" -H "API_KEY: <API_KEY>" 
```

### Response

In order to provide the keys securely, the DRM encryption keys are encrypted in the response.

The format of the response will be:

```JSON
{
    "key":"r8tXpf8wSSHKVnW1fz+MgZY3BTnTO+/IGFglt9oUwtKWj8eyguSLd0bzOhcn4DMF4JWOAbhbKopJE/cZWPqIfl3RCIwl46UP57/m7870+z3NWxh/JSvrfWBq4SGmkykfDLKjyLBqb5F6dDlUB+flcfZMcQh6FGk5GNGEniXliB/kyCrVDiKdwAw3hft8rT4/itFQDMRKkhJf1Fh67AZ1LyzOI3CCY/oO4w/XF/XMAJ1z92tAKULI+cPMFaXT3I77N3iaQoYN78mJkKgAr71Tlf91+ASB8jqOCHD6nNgm6nGxZlsTVK5wxdhT4zbMJq4xb7daSy7xMWdH/1eXuIh0ZhWicKJqbWHKIeifZWFras/0QzqN27rupHF3UYnYQ40V3ESE2PUIe8Cs8471QRHlruYlwtgBBmGme2ERZWFjrRFEeUt8SobQkCIZrzDEKaQ2bJMyOy1yMt8oklpFaW9pBg3yHZEK1WZn5u8ynV+ZWLI6soCNpMgkPYe2UtnFOhFRZfT8WG09FGJiouzyL3fCZAguxP8u9jkzQTv15UhwO/StdpTKCn1yxr3KoyxGAk3L|166c43d4023880a99691fd9679e6ad785305f205"
} 
```

The value of `key` has two components separated by a pipe (ASCII code 124):
- Encrypted blob containing the DRM encryption keys. 
- Hash used in a checksum.

#### Decrypting the response

This section contains examples of how to decrypt the response from the key provider API.

You will need the `CLIENT_NAME`, `SHARED_SECRET` and the `KEY_PROVIDER_RESPONSE` value from the request to the Keyprovider API.

##### C#

```c#
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

namespace Vudrm.EncryptionExamples
{
    class Program
    {
        private const string Client = "CLIENT_NAME";
        private const string SharedSecret = "SHARED_SECRET";
        private const string KeyProviderResponse = "KEY_PROVIDER_RESPONSE";
        
        static void Main(string[] args)
        {
            Console.WriteLine("Decrypting keyprovider API response...");
            var splitKeyProviderResponse = KeyProviderResponse.Split('|');
            var computedHash = ComputeHash(SharedSecret, Client, splitKeyProviderResponse[0]);
            if (computedHash == splitKeyProviderResponse[1])
            {
                Console.WriteLine("key providers response hash passed validation");
                var decryptedKeyProviderResponse = Decrypt(splitKeyProviderResponse[0], SharedSecret);
                Console.WriteLine("Decrypted key provider response: " + decryptedKeyProviderResponse);
            }
            else
            {
                Console.WriteLine("Key provider response hash did not validate");
            }
            Console.ReadLine();
        }

        private string Decrypt(string policy, string sharedSecret)
        {
            try
            {
                var encrypted = Convert.FromBase64String(policy);
                var key = Encoding.ASCII.GetBytes(sharedSecret.Substring(0, 16));
                var rj = new RijndaelManaged
                {
                    Mode = CipherMode.ECB,
                    BlockSize = 128,
                    Key = key
                };
                var decryptor = rj.CreateDecryptor(key, null);
                var ms = new MemoryStream(encrypted);
                var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read);
                var plain = new byte[encrypted.Length];
                var decryptcount = cs.Read(plain, 0, plain.Length);
                ms.Close();
                cs.Close();
                return (Encoding.UTF8.GetString(plain, 0, decryptcount));
            }
            catch
            {
                return policy;
            }
        }

        private string ComputeHash(string sharedSecret, string client, string encryptedMessage)
        {
            var preComputed = sharedSecret + client + encryptedMessage;
            var algorithm = new SHA1Managed();
            var bytesOf = Encoding.UTF8.GetBytes(preComputed);
            var hashOf = algorithm.ComputeHash(bytesOf);
            var result = new StringBuilder();
            foreach (var b in hashOf)
            {
                result.Append(b.ToString("x2").ToLower());
            }
            return result.ToString();
        }
    }
}
```

##### Ruby

```ruby
require 'base64'
require 'crypt/rijndael'
require 'digest'
require 'openssl'

client = "CLIENT_NAME"
shared_secret = "SHARED_SECRET"

encrypted_keys = "KEY_PROVIDER_RESPONSE"

message, message_hash = encrypted_keys.split('|')

pre_computed = client + message
computed_hash = Digest::SHA1.new.hexdigest(shared_secret + pre_computed)

if computed_hash == message_hash
  decoded_message = Base64.strict_decode64(message)
  decipher = OpenSSL::Cipher::Cipher.new('AES-128-ECB').decrypt
  decipher.key = shared_secret[0..15]
  unencrypted_keys = decipher.update(decoded_message) + decipher.final
else
  raise "Key provider response hash did not validate"
end
```

##### GO

```go
package main

import (
	"crypto/aes"
	"crypto/sha1"
	"encoding/base64"
	"errors"
	"fmt"
	"strings"
)

var (
	Client = "CLIENT"
	SharedSecret = "SHARED_SECRET"
	KeyProviderResponse = "KEY_PROVIDER_RESPONSE"
	PKCS5 = &pkcs5{}
)

type pkcs5 struct{}

func main() {
	s := strings.Split(KeyProviderResponse, "|")
	encryptedMessage, hash := s[0], s[1]
	generatedHash := generateKeyProviderHash(SharedSecret, Client, encryptedMessage)
	if generatedHash == hash {
		decryptedKeys := decrypt(encryptedMessage, SharedSecret)
		fmt.Printf(decryptedKeys)
	} else {
		fmt.Printf("hash failed validation")
	}
}

func generateKeyProviderHash(sharedSecret string, client string, encryptedMessage string) string {
	v := fmt.Sprintf("%s%s%s", sharedSecret, client, encryptedMessage)
	hasher := sha1.New()
	hasher.Write([]byte(v))
	return fmt.Sprintf("%x", hasher.Sum(nil))
}

func decrypt(encryptedData string, sharedSecret string) string {
	source, _ := base64.StdEncoding.DecodeString(encryptedData)
	key := []byte(sharedSecret)[0:16]

	cipher, _ := aes.NewCipher([]byte(key))
	buffer := make([]byte, len(source))
	size := 16

	for bs, be := 0, size; bs < len(source); bs, be = bs+size, be+size {
		cipher.Decrypt(buffer[bs:be], source[bs:be])
	}

	plainBytes, _ := PKCS5.unPadding(buffer, 16)
	return string(plainBytes)
}

func (p *pkcs5) unPadding(src []byte, blockSize int) ([]byte, error) {
	srcLen := len(src)
	paddingLen := int(src[srcLen-1])
	if paddingLen >= srcLen || paddingLen > blockSize {
		return nil, errors.New("padding size error")
	}
	return src[:srcLen-paddingLen], nil
}
```

##### Python

```python
import base64
import binascii
import json
import hashlib

from Crypto.Cipher import AES

client = "CLIENT"
shared_secret = "SHARED_SECRET"
key_provider_response = "KEY_PROVIDER_RESPONSE"

print("Decrypting Key Provider Response...")
split_key_provider_response = key_provider_response.split("|")
pre_computed = (shared_secret + client + split_key_provider_response[0]).encode("utf-8")
computed_hash = hashlib.sha1(pre_computed).hexdigest()
if computed_hash == split_key_provider_response[1]:
    key = shared_secret[0:16]
    decoded_text = base64.b64decode(split_key_provider_response[0])
    aes = AES.new(key.encode("utf8"), AES.MODE_ECB)
    padded_text = aes.decrypt(decoded_text)
    unpadded_text = padded_text[:-padded_text[-1]]
    decrypted_keys = json.loads(unpadded_text)
    print("Decrypted keys: " + json.dumps(decrypted_keys))
else:
    print("Hash validation failed for key provider response!")
```

## Using the encryption keys

The next section explains the various encryption keys provided by the Key Provider API. Each set has a different use case. For use with USP products [CENC](#cenc) and [Fairplay](#fairplay) encryption keys are recommended. See the [Unified Streaming Integration](#unified-streaming-integration) section for more details on how to use `mp4split` with the Key Provider.

#### CENC

CENC is the Common Encryption Scheme and it standardises encryption keys between different DRM systems. This allows a single set of encryption keys to be use to encrypt a single file using different DRM systems. VUDRM supports Widevine and PlayReady when generating CENC encryption keys.

Make the following request to the Key Provider API in order to retrieve `cenc` Keys.

```bash
curl -X GET https://keyprovider.vudrm.tech/cenc/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

The keys returned in this response can be used for PlayReady and Widevine scenarios. They allow a single piece of content to be used across multiple devices and browsers.

An example decrypted response would be:

```JSON
{
    "key_id_big": "qMTumUz5drgbusWY+fi5LA==",
    "key_id_hex": "A8C4EE994CF976B81BBAC598F9F8B92C",
    "content_key": "ETweRxyDIuDqAadsWdx0HQ==",
    "content_key_hex": "113C1E471C8322E0EA01A76C59DC741D",
    "playready_key_iv": "dd80b66b2fe44be4",
    "playready_laurl": "https://playready-license.vudrm.tech/rightsmanager.asmx",
    "playready_checksum": "j+7cluk2J58=",
    "widevine_drm_specific_data": "Ig1zb21lY29udGVudGlkSOPclZsG",
    "widevine_laurl": "https://widevine-license.vudrm.tech/proxy"
}
```

The values are:

- `key_id_big`: Unique ID for the encryption. Base64 in Big Endian. 
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64 in Big Endian.
- `content_key_hex`: 128bit encryption key in base16 in Little Endian. This one should be used with `mp4split`.
- `playready_key_iv`: Additional random value to strengthen the PlayReady encryption.
- `playready_laurl`: The PlayReady license server URL.
- `playready_checksum`: A PlayReady specific security value required by some older PlayReady solutions and [Wowza](#wowza-integration).
- `widevine_drm_specific_data`: The Widevine PSSH box.
- `widevine_laurl`: The Widevine license server URL.

#### Fairplay

[Fairplay](https://developer.apple.com/streaming/fps/) is Apple's DRM system and is commonly used in conjunction with CENC encryption to provide support to the widest amount of devices possible.

Make the following request to retrieve `fairplay` keys from the Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/fairplay/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```json
{
    "key_hex": "457A8E3300DE6D549A95037F1C7ADEB1",
    "iv_hex": "EB86EAFBD487391383E8FFF957561B0C",
    "laurl": "skd://fairplay-license.vudrm.tech/license/somecontentid"
}
```

The values are:
- `key_hex`: Unique ID for the encryption.
- `iv_hex`: The encryption key.
- `laurl`: The Fairplay license server URL. At the point of the request to the license server the `skd` protocol should be replaced with `https`. 

#### HLS AES encryption

HLS AES-128 is the Advanced Encryption Standard using a 128 bit key, Cipher Block Chaining (CBC) and PKCS7 padding.

Make the following request to retrieve HLS AES encryption keys from the Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/aes/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_hex":"dd682004123622a99c2a2afcdad6217c",
    "key_url":"http://keyprovider.vudrm.tech:9293/aes/getkey/vualto/somecontentid"
}
```

The values are:
- `key_hex`: Base16 AES Content key
- `key_url`: URL to the AES Key Server 


#### PlayReady

[PlayReady](https://www.microsoft.com/playready/) is Microsoft's DRM system. Only use these keys directly to target PlayReady enabled devices. The use of CENC keys is preferred.

Make the following request to retrieve `playready` keys from the Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/playready/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON    
{
    "key_id":"kX9efHkfIS++X+m8kZPDOw==",
    "key_id_guid":"7c5e7f91-1f79-2f21-be5f-e9bc9193c33b",
    "key_id_uuid":"917f5e7c791f-2f21-be5fe9bc9193c33b",
    "key_id_hex":"917F5E7C791F212FBE5FE9BC9193C33B",
    "content_key":"eI5JjujIo1Ek7lqO+3gg0A==",
    "content_key_hex":"788E498EE8C8A35124EE5A8EFB7820D0",
    "laurl":"http://vualto.playready-license.vudrm.tech/rightsmanager.asmx",
    "service_id":"gwICI8yfIUGf4R/5qOWuqg==",
    "service_id_guid":"23020283-9fcc-4121-9fe11ff9a8e5aeaa",
    "key_iv":"07261ee6ee074f1d",
    "checksum":"wf0goCGB2O4="
}
```

The values are:
- `key_id`: Unique ID for the encryption. Base64 in Big Endian.
- `key_id_guid`: Unique ID for the encryption. GUID in Big Endian.
- `key_id_uuid`: Unique ID for the encryption. UUID in Little Endian.
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endean. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64.
- `content_key_hex`: 128bit encryption key in base16. This one should be used with `mp4split`.
- `laurl`: The PlayReady license server URL.
- `service_id`: Unique VUDRM service ID for Vualto in Base64.
- `service_id_guid`: Unique VUDRM service ID for Vualto as a GUID.
- `key_iv`: Additional random value to strengthen the PlayReady encryption.
- `checksum`: Extra security value required by some older PlayReady solutions.

#### Widevine 

[Widevine](https://www.widevine.com/) is Google's DRM system. Use these keys to target Widevine enabled devices directly. The use of CENC keys is preferred.

Make the following request to retrieve `widevine` keys from the Key Provider API:

```bash
curl -X GET https://keyprovider.vudrm.tech/widevine/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_id":"NxB8uJFHVSuaO5BjiMzuEQ==",
    "key_id_hex":"37107CB89147552B9A3B906388CCEE11",
    "content_key":"Unyx1s3fMFUedA688fCNxw==",
    "content_key_hex":"527CB1D6CDDF30551E740EBCF1F08DC7",
    "laurl":"https://widevine-license.vudrm.tech/proxy",
    "drm_specific_data":"CAESEDcQfLiRR1UrmjuQY4jM7hEaBnZ1YWx0byIFdGVzdDEqAkhEMgA="
} 
```

The values are:
- `key_id`: Unique ID for the encryption. Base64 in Little Endian.
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64.
- `content_key_hex`: 128bit encryption key in base16. This one should be used with `mp4split`.
- `laurl`: The Widevine license server URL.
- `drm_specific_data`: The Widevine PSSH box.

## Unified Streaming Integration

The encryption keys provided by the Key Provider API are compatible with [Unified Streaming Platform's](https://www.unified-streaming.com/) mp4split product.

The recommended approach is to call the Key Provider API and retrieve CENC and Fairplay Keys. Once the encryption keys have been retrieved they can be used with mp4split to generate an ism:

```bash
mp4split --license-key=$LICENSE_KEY -o $ISM \
 --iss.key=${key_id_hex}:${content_key_hex} \
 --iss.key_iv=${playready_key_iv} \
 --iss.license_server_url=${playready_laurl} \
 --widevine.key=${key_id_hex}:${content_key_hex} \
 --widevine.license_server_url=${widevine_laurl} \
 --widevine.drm_specific_data=${widevine_drm_specific_data} \
 --hls.client_manifest_version=4 \
 --hls.key=:${key_hex} \
 --hls.key_iv=${iv_hex} \
 --hls.license_server_url=${laurl} \
 --hls.playout=sample_aes_streamingkeydelivery
```

The HLS values come from the Fairplay encryption keys, all other values are from the CENC keys.

## Wowza Integration

The encryption keys provided by the Key Provider API are compatible with [Wowza](https://www.wowza.com/).

The following steps detail how to configure Wowza to support VUDRM:

### Configure Wowza Stream Settings

1. Stop Wowza appending the session ID to the license server URL: 
	- [how to control the streaming session id](https://www.wowza.com/docs/how-to-control-streaming-session-id-appended-to-encryption-urls-in-chunklist-responses-cupertinoappendqueryparamstoencurl)

2. Change the `EXT-X-VERSION` for HLS streaming to `5` by setting the `cupertinoExtXVersion` value: 
	- [how to change the ext x version](https://www.wowza.com/docs/how-to-change-the-ext-x-version-for-apple-http-live-streaming)

### Manually Set VUDRM Settings

This method of integrating VUDRM with Wowza uses key files:
- [how to secure mpeg dash](https://www.wowza.com/docs/how-to-secure-mpeg-dash-streaming-using-common-encryption-cenc#dash_cenc)
- [https://www.wowza.com/docs/how-to-secure-apple-hls-streaming-using-drm-encryption#keyfiles](https://www.wowza.com/docs/how-to-secure-apple-hls-streaming-using-drm-encryption#keyfiles)

NB: Widevine, PlayReady, and Fairplay key values can be set in the same file.

An example key file is:

```bash
mpegdashstreaming-cenc-key-id: MT8jItzhV+ay+3QEtoyIxQ==
mpegdashstreaming-cenc-content-key: EcZ9tM1adkoleC1hwFlpQw==
mpegdashstreaming-cenc-algorithm: AESCTR
mpegdashstreaming-cenc-keyserver-widevine: true
mpegdashstreaming-cenc-keyserver-widevine-system-id: edef8ba9-79d6-4ace-a3c8-27dcd51d21ed
mpegdashstreaming-cenc-keyserver-widevine-pssh-data: Igp3b3d6YS1kZW1vSOPclZsG
mpegdashstreaming-cenc-keyserver-playready: true
mpegdashstreaming-cenc-keyserver-playready-system-id: 9a04f079-9840-4286-ab92-e65be0885f95
mpegdashstreaming-cenc-keyserver-playready-license-url: https://playready-license.vudrm.tech/rightsmanager.asmx
mpegdashstreaming-cenc-keyserver-playready-checksum: Mv5YB5EhHKw=
cupertinostreaming-aes128-method: SAMPLE-AES
cupertinostreaming-aes128-url: skd://fairplay-license.vudrm.tech/license/wowza-demo
cupertinostreaming-aes128-key: 24F79075974B8E1BC8AF576925B8458F
cupertinostreaming-aes128-iv: A140A11A450DBC04E67F39B850C13D41
cupertinostreaming-aes128-iv-include-in-chunklist: false
cupertinostreaming-aes128-key-format: com.apple.streamingkeydelivery
cupertinostreaming-aes128-key-format-version: 1
```

The values are:
- `mpegdashstreaming-cenc-key-id`: The `key_id_big` value from the [CENC response](#cenc).
- `mpegdashstreaming-cenc-content-key`: The `content_key` value from the [CENC response](#cenc).
- `mpegdashstreaming-cenc-algorithm`: Must be set to `AESCTR`.
- `mpegdashstreaming-cenc-keyserver-widevine`: Must be set to `true`.
- `mpegdashstreaming-cenc-keyserver-widevine-system-id`: Must be set to `edef8ba9-79d6-4ace-a3c8-27dcd51d21ed`
- `mpegdashstreaming-cenc-keyserver-widevine-pssh-data`: The `widevine_drm_specific_data` value from the [CENC response](#cenc).
- `mpegdashstreaming-cenc-keyserver-playready`: Must be set to `true`.
- `mpegdashstreaming-cenc-keyserver-playready-system-id`: Must be set to `9a04f079-9840-4286-ab92-e65be0885f95`
- `mpegdashstreaming-cenc-keyserver-playready-license-url`: The `playready_laurl` value from the [CENC response](#cenc).
- `mpegdashstreaming-cenc-keyserver-playready-checksum`: The `checksum` value from the [CENC Response](#cenc)
- `cupertinostreaming-aes128-method`: Must be set to `SAMPLE-AES`.
- `cupertinostreaming-aes128-url`: The `laurl` value from the [Fairplay Response](#fairplay)
- `cupertinostreaming-aes128-key`: The `key_hex` value from the [Fairplay Response](#fairplay)
- `cupertinostreaming-aes128-iv`: The `iv_hex` value from the [Fairplay Response](#fairplay)
- `cupertinostreaming-aes128-iv-include-in-chunklist`: Must be set to `false`.
- `cupertinostreaming-aes128-key-format`: Must be set to `com.apple.streamingkeydelivery`.
- `cupertinostreaming-aes128-key-format-version`: Must be set to `1`.
