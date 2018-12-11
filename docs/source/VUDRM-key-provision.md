# ENCRYPTION KEY PROVISION

The VUDRM key provision service exposes the generation of DRM encryption. It provides a secure REST API allowing you to retrieve encryption keys for all DRM types in order to encrypt your content.

## Key provider API

The key provider API will provide all the information required in order to encrypt various types of content. In order to be as flexible as possible it will return encryption keys in Base16 and Base64 as some systems require different values. 

The following sections describe the requests and responses in order to retrieve the keys. The most common scenario is to request CENC and Fairplay encryption keys. 

### Request

A `GET` should be made to the following URL:

```URL
https://key-provider.drm.technology/<DRM_TYPE>/<CLIENT_NAME>/<CONTENT_ID> 
```

The breakdown of this URL is:
- `DRM_TYPE`: The type of encryption keys being requested. Possible values are `cenc`, `fairplay`, `widevine`, `playready`, and `aes`. See [Using the encryption keys](#using-the-encryption-keys) for more information.
- `CLIENT_NAME`: The account name. Plese contact support@vualto.com if you do not have the account name.
- `CONTENT_ID`: A unique content identifier. This value will always generate the same encryption keys.

In order to provide the keys securely an API key is required in the header of the request. Please contact support@vualto.com if you do not have an API key.

Below is an example curl request for `cenc` encryption keys using the client `vualto` and with the `CONTENT_ID` of `test`

```bash
curl -X "GET" "https://key-provider.drm.technology/cenc/vualto/test" -H "API_KEY: 8930a415-6eb0-4393-a108-9780d9g687b5" 
```

### Response

In order to provide the keys securely the DRM encryption keys are encrypted in the response.

The format of the response will be:

```JSON
{
    "key":"r8tXpf8wSSHKVnW1fz+MgZY3BTnTO+/IGFglt9oUwtKWj8eyguSLd0bzOhcn4DMF4JWOAbhbKopJE/cZWPqIfl3RCIwl46UP57/m7870+z3NWxh/JSvrfWBq4SGmkykfDLKjyLBqb5F6dDlUB+flcfZMcQh6FGk5GNGEniXliB/kyCrVDiKdwAw3hft8rT4/itFQDMRKkhJf1Fh67AZ1LyzOI3CCY/oO4w/XF/XMAJ1z92tAKULI+cPMFaXT3I77N3iaQoYN78mJkKgAr71Tlf91+ASB8jqOCHD6nNgm6nGxZlsTVK5wxdhT4zbMJq4xb7daSy7xMWdH/1eXuIh0ZhWicKJqbWHKIeifZWFras/0QzqN27rupHF3UYnYQ40V3ESE2PUIe8Cs8471QRHlruYlwtgBBmGme2ERZWFjrRFEeUt8SobQkCIZrzDEKaQ2bJMyOy1yMt8oklpFaW9pBg3yHZEK1WZn5u8ynV+ZWLI6soCNpMgkPYe2UtnFOhFRZfT8WG09FGJiouzyL3fCZAguxP8u9jkzQTv15UhwO/StdpTKCn1yxr3KoyxGAk3L|166c43d4023880a99691fd9679e6ad785305f205"
} 
```

The value of `key` has two components separated by a pipe (ASCII code 124):
- Encrypted blob containing the DRM encryption keys. 
- Hash used in a checksum

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
```

### Using the encryption keys

The next section explains the various encryption keys provided by the Key Provider API. Each set has a different use case. For use with USP products [CENC](#cenc) and [Fairplay](#fairplay) encryption keys are recommended. See the [Use with mp4split](#use-with-mp4split) section for more details on how to use `mp4split` with the Key Provider.

#### CENC

CENC is the Common Encryption Scheme and it standardises encryption keys between different DRM systems. This allows a single set of encryption keys to be use to encrypt a single file using different DRM systems. VUDRM supports Widevine and PlayReady when generating CENC encryption keys.

Make the following request to the Key Provider API in order to retrieve `cenc` Keys.

```bash
curl -X GET https://key-provider.drm.technology/cenc/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

The keys return in this response can be used for PlayReady and Widevine scenarios. They allow a single piece of content to be used across multiple devices and browsers.

An example decrypted response would be:

```JSON
{
    "key_id_big":"fF5/kR95LyG+X+m8kZPDOw==",
    "key_id_hex":"7C5E7F911F792F21BE5FE9BC9193C33B",
    "content_key":"eI5JjujIo1Ek7lqO+3gg0A==",
    "content_key_hex":"788E498EE8C8A35124EE5A8EFB7820D0",
    "playready_key_iv":"cd340bf920a34bfb",
    "widevine_drm_specific_data":"CAESEJrENOhpDl58uLPtkqsm0/kaBnZ1YWx0byIgN0M1RTdGOTExRjc5MkYyMUJFNUZFOUJDOTE5M0MzM0IqAkhEMgA=",
    "playready_laurl":"http://playready.drm.technology/rightsmanager.asmx",
    "widevine_laurl":"https://license.widevine.com/cenc/getcontentkey/vualto"
} 
```

The values are:

- `key_id_big`: Unique ID for the encryption. Base64 in Big Endian. 
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64 in Big Endian.
- `content_key_hex`: 128bit encryption key in base16 in Little Endian. This one should be used with `mp4split`.
- `playready_key_iv`: Additional random value to strengthen the PlayReady encryption.
- `widevine_drm_specific_data`: The Widevine PSSH box.
- `playready_laurl`: The PlayReady license server URL.
- `widevine_laurl`: The Widevine license server URL.

#### Fairplay

[Fairplay](https://developer.apple.com/streaming/fps/) is Apple's DRM system and is commonly used in conjunction with CENC encryption to provide support to the widest amount of devices possible.

Make the following request to retrieve `fairplay` keys from the Key Provider API:

```bash
curl -X GET https://key-provider.drm.technology/fairplay/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```json
{
    "key_hex": "457A8E3300DE6D549A95037F1C7ADEB1",
    "iv_hex": "EB86EAFBD487391383E8FFF957561B0C",
    "laurl": "skd://fairplay-license.drm.technology/license/somecontentid"
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
curl -X GET https://key-provider.drm.technology/aes/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_hex":"dd682004123622a99c2a2afcdad6217c",
    "key_url":"http://keyprovider.drm.technology:9293/aes/getkey/vualto/test1"
}
```
- `key_hex`: Base16 AES Content key.
- `key_url`: URL to the AES Key Server 


#### Playready

[PlayReady](https://www.microsoft.com/playready/) is Microsoft's DRM system. Only use these keys directly to target PlayReady enabled devices. The use of CENC keys is preferred.

Make the following request to retrieve `playready` keys from the Key Provider API:

```bash
curl -X GET https://key-provider.drm.technology/playready/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
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
    "laurl":"http://vualto.playready.drm.technology/rightsmanager.asmx",
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
curl -X GET https://key-provider.drm.technology/widevine/<CLIENT>/<CONTENT_ID> -H 'API_KEY: <API_KEY>'
```

An example decrypted response would be:

```JSON
{
    "key_id":"NxB8uJFHVSuaO5BjiMzuEQ==",
    "key_id_hex":"37107CB89147552B9A3B906388CCEE11",
    "content_key":"Unyx1s3fMFUedA688fCNxw==",
    "content_key_hex":"527CB1D6CDDF30551E740EBCF1F08DC7",
    "laurl":"https://license.widevine.com/cenc/getcontentkey/vualto",
    "drm_specific_data":"CAESEDcQfLiRR1UrmjuQY4jM7hEaBnZ1YWx0byIFdGVzdDEqAkhEMgA="
} 
```

- `key_id`: Unique ID for the encryption. Base64 in Little Endian.
- `key_id_hex`: Unique ID for the encryption. Base16 in Little Endian. This one should be used with `mp4split`.
- `content_key`: 128bit encryption key in base64.
- `content_key_hex`: 128bit encryption key in base16. This one should be used with `mp4split`.
- `laurl`: The Widevine license server URL.
- `drm_specific_data`: The Widevine PSSH box.

#### Use with mp4split

The encryption keys provided by the Key Provider API are compatible with [Unified Streaming Platform's](https://www.unified-streaming.com/) products.

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
