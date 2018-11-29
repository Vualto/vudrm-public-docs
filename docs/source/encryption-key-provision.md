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
- `DRM_TYPE`: The type of encryption keys being requested. Possible values are `cenc`, `fairplay`, `widevine`, `playready`, and `aes`.
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
    "key":"jmr3FPUDpUeAhruWEv565gXgjvrYOEyajtSVfOxLWyYydBSLPfbp0kCJH5UbloEqAlGNKP15ZQfNpIEeKK4AM3c8j22p10XD6mHqqsa8n2LWrVXDFEQ2k1psNdaxJzLMHLzSdA9BXgYUjAVdRD6+K6u0coW4T8l7PTTzJSFGKqg=Kfddff0b08aca3d6b47df62b5fdab748879224fee"
} 
```

#### Decrypting the response examples

This section contains examples of how to decrypt the response from the key provider API.

You will need the `CLIENT_NAME`, `SHARED_SECRET` and the `key` value from the request to the Keyprovider API.

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
        private const string Client = "<CLIENT_NAME>";
        private const string SharedSecret = "<SHARED_SECRET>";
        private const string KeyProviderResponse = "<KEY_PROVIDER_RESPONSE>";
        
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
                Console.WriteLine("Key provider response has did not validate");
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

client = "<CLIENT_NAME>"
shared_secret = "<SHARED_SECRET>"

encrypted_keys = "<KEYPROVIDER_ENCRYPTED_KEY_VALUE>"

message, message_hash = encrypted_keys.split('|')

pre_computed = client + message
computed_hash = Digest::SHA1.new.hexdigest(shared_secret + pre_computed)

if computed_hash == message_hash
  decoded_message = Base64.strict_decode64(message)
  decipher = OpenSSL::Cipher::Cipher.new('AES-128-ECB').decrypt
  decipher.key = shared_secret[0..15]
  unencrypted_keys = decipher.update(decoded_message) + decipher.final
else
  raise "hash check failed"
end
```

##### PHP

```PHP
<?php
    class Crypto
    { 
        private $_cipher;
        private $_mode;
        private $_keySize; 

        public function __construct($cipher = MCRYPT_RIJNDAEL_128, $mode = MCRYPT_MODE_ECB) 
        {
            $this->_cipher = $cipher;
            $this->_mode = $mode;
            $this->_keySize = mcrypt_get_key_size($this->_cipher, $this->_mode); 
        }

        public function decrypt($encryptedText, $sharedSecret)
        { 
            $cipherText = base64_decode($encryptedText, true);
            $paddedPlainText = mcrypt_decrypt($this->_cipher, substr($sharedSecret, 0, 16), 
                $cipherText, $this->_mode);
            $plainText = $this->removePkcs5Pad($paddedPlainText);
            return $plainText; 
        }

        private function removePkcs5Pad($padded) 
        {
            $len = strlen($padded);
            $padSize = ord($padded[$len - 1]);
            return substr($padded, 0, $len - $padSize); 
        }
    }
?> 
```

### CENC encryption key 

### 2.1 Microsoft Playready

**Example Resonse:**

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

```Content Encryption Key``` : 128bit encryption key in Base16 and Base64, we will also provide Big and Little Endian values due to compatability with common encryption. 

```key_id```  : Unique ID for the encryption (Base64, Base16 and GUID).

```laurl```  : License Server Acquisition URL.

```key_iv``` : Provides an additional random value to strengthen the encryption.

```checksum``` : Required in some older Playready Solutions.

```service_id``` : Uniqiue Vualto DRM Service ID.


### 2.2 Adobe Primetime (nee Access)

**Example Response:**

```JSON

{
"content_key":"7AD77B22434F85F9BA990D2582A953C5",
"drm_specific_data":"AgARfEFkZGl0aW9uYWxIZWFkZXIDAApFbmNyeXB0aW9uAwAHVmVyc2lvbgBAAAAAAAAAAAAGTWV0aG9kAgAIU3RhbmRhcmQABUZsYWdzAAAAAAAAAAAAAAZQYXJhbXMDAAdWZXJzaW9uAD/wAAAAAAAAABNFbmNyeXB0aW9uQWxnb3JpdGhtAgAHQUVTLUNCQwAQRW5jcnlwdGlvblBhcmFtcwMACUtleUxlbmd0aABAMAAAAAAAAAAACQAHS2V5SW5mbwMAB1N1YlR5cGUCAA1GbGFzaEFjY2Vzc3YyAAREYXRhAwAITWV0YWRhdGEMAAAZ6E1JSVRhUVlKS29aSWh2Y05BUWNDb0lJVFdqQ0NFMVlDQVFFeEN6QUpCZ1VyRGdNQ0dnVUFNSUlIOWdZSktvWklodmNOQVFjQm9JSUg1d1NDQitNd2dnZmZBZ0VDTUlJQlJ6Q0NBVU1DQVFNQ0FRRUVKRVpGT0VKRlJFWXpMVUl3UlVRdE16ZzROaTFDUVVSRUxVWXlRVEZCTXpVeE5VUkdNekdCdERCQkRDbGpiMjB1WVdSdlltVXVabXhoYzJoaFkyTmxjM011Y21sbmFIUnpMbXhwWTJWdWMyVlZjMkZuWmFBVU1SSXdFQVlKS29aSWh2Y3ZBd1lPb0FNQkFmOHdid3doWTI5dExtRmtiMkpsTG1ac1lYTm9ZV05qWlhOekxuSnBaMmgwY3k1d2JHRjVvRW94U0RBZUJna3Foa2lHOXk4REJnZWdBd0VCLzZFTU1BcWdBd29CQWFFRENnRUJNQ1lHQ1NxR1NJYjNMd01HQzZBREFRSC9vUlF3RXFBRENnRUJvUXN3Q1FvQkFhQUVBd0lHd0tBS0RBZ3pMakF1TURVMU9hRVpEQmRHY21GdVkyVlVWaUJFWldaaGRXeDBJRkJ2YkdsamVhTXlNVEF3TGd3cVkyOXRMbUZrYjJKbExtWnNZWE5vWVdOalpYTnpMbUYwZEhKcFluVjBaWE11WVc1dmJubHRiM1Z6TVFDbUF3RUIvekdDQk80d2dnVHFNU2NNSldoMGRIQTZMeTloWTJObGMzTXVkV0YwTG1SeWJTNTBaV05vYm05c2IyZDVMMnhwZG1Vd2dnUzlNSUlEcGFBREFnRUNBaEF0Mjd3b0VLOWhMK2F3Y1RIZEk5emdNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1HVXhDekFKQmdOVkJBWVRBbFZUTVNNd0lRWURWUVFLRXhwQlpHOWlaU0JUZVhOMFpXMXpJRWx1WTI5eWNHOXlZWFJsWkRFeE1DOEdBMVVFQXhNb1FXUnZZbVVnUm14aGMyZ2dRV05qWlhOeklFTjFjM1J2YldWeUlFSnZiM1J6ZEhKaGNDQkRRVEFlRncweE5EQTFNakl3TURBd01EQmFGdzB4TmpBMU1qRXlNelU1TlRsYU1JR0NNUXN3Q1FZRFZRUUdFd0pWVXpFak1DRUdBMVVFQ2hRYVFXUnZZbVVnVTNsemRHVnRjeUJKYm1OdmNuQnZjbUYwWldReEVqQVFCZ05WQkFzVUNWUnlZVzV6Y0c5eWRERWJNQmtHQTFVRUN4UVNRV1J2WW1VZ1JteGhjMmdnUVdOalpYTnpNUjB3R3dZRFZRUUREQlJXVlVGTVZFOHRWRk5RVkMweU1ERTBNRFV5TWpDQm56QU5CZ2txaGtpRzl3MEJBUUVGQUFPQmpRQXdnWWtDZ1lFQXhMSWFJNFFZbm0xOGFMRm9Ma2F5ZW5IRUVWeG4rdzF0QkdsaFVkQzVEeGl1cDdrWnViRXNXbHhZZzRYQWJWckVRckw1R0h2Qzlha1lwSkxObm9VY0dDVCtDLzFoSWhXTXRMSWRmSmxKTUhHYTQzTVBVTmdqTjJkenMzOTdQb3Z6bWpkS2tnQitkaGJTaCtPZytmNUYyanE0My9pMDE3YjhYYXVXY2JqRnJmMENBd0VBQWFPQ0FjMHdnZ0hKTUdrR0ExVWRId1JpTUdBd1hxQmNvRnFHV0doMGRIQTZMeTlqY213ekxtRmtiMkpsTG1OdmJTOUJaRzlpWlZONWMzUmxiWE5KYm1OdmNuQnZjbUYwWldSR2JHRnphRUZqWTJWemMwTjFjM1J2YldWeVFtOXZkSE4wY21Gd0wweGhkR1Z6ZEVOU1RDNWpjbXd3Q3dZRFZSMFBCQVFEQWdTd01JSGtCZ05WSFNBRWdkd3dnZGt3Z2RZR0NpcUdTSWIzTHdNSkFBQXdnY2N3TWdZSUt3WUJCUVVIQWdFV0ptaDBkSEE2THk5M2QzY3VZV1J2WW1VdVkyOXRMMmR2TDJac1lYTm9ZV05qWlhOelgyTndNSUdRQmdnckJnRUZCUWNDQWpDQmd4cUJnRlJvYVhNZ1kyVnlkR2xtYVdOaGRHVWdhR0Z6SUdKbFpXNGdhWE56ZFdWa0lHbHVJR0ZqWTI5eVpHRnVZMlVnZDJsMGFDQjBhR1VnUVdSdlltVWdSbXhoYzJnZ1FXTmpaWE56SUVOUVV5QnNiMk5oZEdWa0lHRjBJR2gwZEhBNkx5OTNkM2N1WVdSdlltVXVZMjl0TDJkdkwyWnNZWE5vWVdOalpYTnpYMk53TUI4R0ExVWRJd1FZTUJhQUZCb2tadzhrUGlncHNMbmlkWTZGQVYybG45RE1NQjBHQTFVZERnUVdCQlRYMXVCcWxQcFJRZ2hJcGhQUjIxUFV6dDNwc1RBVkJnTlZIU1VFRGpBTUJnb3Foa2lHOXk4RENRRTNNQkVHQ2lxR1NJYjNMd01KQWdVRUF3SUJBREFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBTFRLS212V1pqdTRva3RtRnFockFma0pld1FtRy9vOVU0amJCRkRidjIvZUpIYXhBZHIrZEZ4WXlISDlmSDFocWVxeiswQ2FDT1krRmc1YndGVEVlcStDbFVNa2dpNjg2ZjZ4WVhMZFk2Y0NxYkFnODgrTjk5M0YxWFFOa2J5MEJQOHdoeHR3T3k4NWpGSGdwTUVtUzZ0c0ZmdktJN2ZxMVh4YVh6Mk1oZnJQdEFzeWpxNEJ2ODJHZ25LbDIzZWxOZi8zNmxJSmR6dFVVL1dpbE1vREFUU1A3Z3ZFUnJFYzJwK0xkZkw3R05mMGV1NVh5L3ZqUjVMclAvZE8zWWNvSnprMUowMFFxc2cxRk03UC9NKy9VK3d5cE1kc1V0cUdwWjg2eE5vUmk3MFhuOGtDSWR4SFJNSWpBTHdicVQ1ckRQRnU3SG5yYnBZTllQdGN0ZmhRWTZ6Q0NBUlFFSkVNeE16WkROelpDTFVZNE56RXRNMEU1T0MwNVFUUkdMVUkwTXpZM01FSTRORFZHTUJnUE1qQXhOVEEyTVRZd05qTTRNekphQklHQWtpOThHdXVTVVhsdFplWDB3eEtwcVlabExlOUhWNGY0WnJOOHF1MVJCRmJLb2tnNjdCKzBDeWFzc2N2S1pob2FHRmJaaUlhN2ZzNHp5RmlKaTdtZndrUXk0VGFHYnJqZUNadFZZbG9rTzc0cGUzOXByUmlPM0pEV09DNEg2R2Z3NEJ4Tm10eEx0cFFRRGRhK0dVZUVOeFRIQmJiaVpDQzJyWnhVUzA4S1E5VXdJUVlKS29aSWh2Y3ZBd2dDQkJTd3NhSkwwVnJzeFFlZHM1YS9Eakx3cFpBY0JLQU1CQXB6WVcxd2JHVXViWEEwb1NjTUpXaDBkSEE2THk5aFkyTmxjM011ZFdGMExtUnliUzUwWldOb2JtOXNiMmQ1TDJ4cGRtVXdlVEJsTVFzd0NRWURWUVFHRXdKVlV6RWpNQ0VHQTFVRUNnd2FRV1J2WW1VZ1UzbHpkR1Z0Y3lCSmJtTnZjbkJ2Y21GMFpXUXhNVEF2QmdOVkJBTU1LRUZrYjJKbElFWnNZWE5vSUVGalkyVnpjeUJEZFhOMGIyMWxjaUJDYjI5MGMzUnlZWEFnUTBFQ0VIQzhHc09leVQ5djhKWVRNVnB0dnRtZ0Nnd0lOQzR3TGpBME1UU2dnZ25HTUlJRXZEQ0NBNlNnQXdJQkFnSVFjTHdhdzU3SlAyL3dsaE14V20yKzJUQU5CZ2txaGtpRzl3MEJBUXNGQURCbE1Rc3dDUVlEVlFRR0V3SlZVekVqTUNFR0ExVUVDaE1hUVdSdlltVWdVM2x6ZEdWdGN5QkpibU52Y25CdmNtRjBaV1F4TVRBdkJnTlZCQU1US0VGa2IySmxJRVpzWVhOb0lFRmpZMlZ6Y3lCRGRYTjBiMjFsY2lCQ2IyOTBjM1J5WVhBZ1EwRXdIaGNOTVRRd05USXlNREF3TURBd1doY05NVFl3TlRJeE1qTTFPVFU1V2pDQmdURUxNQWtHQTFVRUJoTUNWVk14SXpBaEJnTlZCQW9VR2tGa2IySmxJRk41YzNSbGJYTWdTVzVqYjNKd2IzSmhkR1ZrTVJFd0R3WURWUVFMRkFoUVlXTnJZV2RsY2pFYk1Ca0dBMVVFQ3hRU1FXUnZZbVVnUm14aGMyZ2dRV05qWlhOek1SMHdHd1lEVlFRRERCUldWVUZNVkU4dFVFdEhVaTB5TURFME1EVXlNakNCbnpBTkJna3Foa2lHOXcwQkFRRUZBQU9CalFBd2dZa0NnWUVBdVpZQjE5THJuSG0zeW5kN0IxQnVzeFZKU2ViWWZaRzVidytOWGUrY0kyYXpsV3FIZDdpZmk4K3NLN1FOTkI4bjhuTmJ3Qi8zRUl2aFlHK3c4RTFMNXF6ZjY0UkdZdk5pSmtPcEJBb1hJU1ZMeHppYmM0TGJJU1RBOTZjM1Nxdi91M0ExNXpUbThQYzgvOVN3TUpQNk5zL1lmTmpHaDFxUUFURDVWZGNGNk04Q0F3RUFBYU9DQWMwd2dnSEpNR2tHQTFVZEh3UmlNR0F3WHFCY29GcUdXR2gwZEhBNkx5OWpjbXd6TG1Ga2IySmxMbU52YlM5QlpHOWlaVk41YzNSbGJYTkpibU52Y25CdmNtRjBaV1JHYkdGemFFRmpZMlZ6YzBOMWMzUnZiV1Z5UW05dmRITjBjbUZ3TDB4aGRHVnpkRU5TVEM1amNtd3dDd1lEVlIwUEJBUURBZ1N3TUlIa0JnTlZIU0FFZ2R3d2dka3dnZFlHQ2lxR1NJYjNMd01KQUFBd2djY3dNZ1lJS3dZQkJRVUhBZ0VXSm1oMGRIQTZMeTkzZDNjdVlXUnZZbVV1WTI5dEwyZHZMMlpzWVhOb1lXTmpaWE56WDJOd01JR1FCZ2dyQmdFRkJRY0NBakNCZ3hxQmdGUm9hWE1nWTJWeWRHbG1hV05oZEdVZ2FHRnpJR0psWlc0Z2FYTnpkV1ZrSUdsdUlHRmpZMjl5WkdGdVkyVWdkMmwwYUNCMGFHVWdRV1J2WW1VZ1JteGhjMmdnUVdOalpYTnpJRU5RVXlCc2IyTmhkR1ZrSUdGMElHaDBkSEE2THk5M2"
}

```

```content_key``` : Provided in Base16

```drm_specific_data``` : This is the DRM ‘blob’ that contains all the relevant DRM data for the Adobe Primetime encryption process. 

### 2.3 Google Widevine 

**Example Response:** 

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

 
```Content Encryption Key``` : 128bit encryption key in Base16 and Base64, we will also provide Big and Little Endean values due to compatability with common encryption.

```key_id``` : Unique ID for the encryption (Base64, Base16 and GUID).

 ```laurl``` : License Server Acquisition URL.
 
 ```drm_specific_dataa``` This is the Widevine PSSH box that contains all the relevant DRM data for widevine encryption. 


### 2.4 CENC (Common Encryption) 

**Example Response :**

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

```Content Encryption Key``` – 128bit encryption key in Base16 and Base64, we will also provide Big and Little Endean values due to compatability with common encryption.

```key_id``` : Unique ID for the encryption (Base64, Base16 and GUID) – due to differences between Playready and Widevine Endianess we return both responses.

```laurl``` : License Server Acquisition URL for both Widevine and Playready.

```key_iv``` : Provides an additional random value to strengthen the encryption.

```widevine_drm_specific_data``` : This is the Widevine PSSH box that contains all the relevant DRM data for widevine encryption. 


### 2.4 AES

**Example Response:**

```JSON

{
    "key_hex":"dd682004123622a99c2a2afcdad6217c",
    "key_url":"http://keyprovider.drm.technology:9293/aes/getkey/vualto/test1"
}
```
 
 
```key_hex``` : AES Content Key : Base16 AES Content key.

```key_url``` : AES Key URL : URL to the AES Key Server 






## Appendix A

The following PHP code snippets provides information to decrypt the Key provider Response – the ClientID and Shared Secret values can be found in the DRM Admin System at ```http://admin.drm.technology``` in the configuration section. 

Sample consists of 3 PHP files: 
Crypto.php 
index.php  
request.php

This is only meant as a simple example. 


### Crypto.php



### index.php

```PHP

<?php
     require 'request.php';
     require 'crypto.php';
     $client = '';
     $contentId = ''; 
     $keyproviderUrl = "http://key-provider.drm.technology:9293/playready/$clientID/$contentID";
     $sharedSecret = '';
     $request = new Request();
     $request->setHeader('content-type', 'utf-8')->setHeader('accept', 'application/json');
     $response = $request->get($keyproviderUrl);
     $responseParts = explode('|', $response);
     $checksum = sha1($sharedSecret . $client . $responseParts[0]); 
     
     if($checksum != $responseParts[1]){ 
        throw new Exception("Checksum has failed");
     } 
     
     $crypto = new Crypto(); 
     $result = $crypto->decrypt($responseParts[0], $sharedSecret);
    
     if(!$result) {
        throw new Exception("Decryption has failed");
      } 
    
      echo $result;
?> 

```

### request.php

```PHP

<?php 
    class Request
    {
        private $_handle;
        private $_headers = array()
        public function __construct() 
        {
            $this->_handle = curl_init();
            curl_setopt($this->_handle, CURLOPT_RETURNTRANSFER, true);
        }
        
        public function get($url)
        {
            curl_setopt($this->_handle, CURLOPT_URL, $url); 
            $this->applyHeaders(); 
            return curl_exec($this->_handle);
        } 
        
        public function setHeader($header, $value) 
        {
            if(!array_key_exists($header, $this->_headers)){
                $this->_headers[$header] = array(); 
            }
            $value = strtolower($value);
            if(!in_array($value, $this->_headers[$header])){
            $this->_headers[$header][] = $value;
        }
        
        return $this;
        }
        
        private function applyHeaders()
        {
            $headers = array();
            foreach($this->_headers as $header => $values){ 
            if(is_array($values)){
                $valueString = implode('; ', $values);
            }else
                $valueString = $values;
            }
            $headers[] = $header . ': ' . $valueString;
        }
        curl_setopt($this->_handle, CURLOPT_HTTPHEADER, $headers);
    }
 }?> 

```