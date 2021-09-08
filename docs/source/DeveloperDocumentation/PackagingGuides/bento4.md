# Bento4

## Installing Bento4

## Get CPIX Documents

### Using VUDRM Admin site

- Log on to the VUDRM Admin site - https://admin.vudrm.tech

- On the left hand side of the page select `Configuration`.

- Go to the `VUDRM Encryption Keys` section.

- Select which `DRM Providers` are required.

- Add a `Content ID`.

- Click `Generate`.

This will then generate a CPIX Document and CPIX keys as JSON document - you can copy and paste these to your favourite text editor to refer to them for future use.

### Using CURL Requests

Alternatively, you can also retrieve these documents with a CURL Request. 

Below is the CURL request for the CPIX Document:

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>"
```

Below is the CURL request for the CPIX keys as JSON document:

```bash
curl -XGET -H "API-KEY: <api-key>" \
"https://cpix.vudrm.tech/v1/keys/<client-name>/<content-id>"
```

An in-depth guide is detailed in the following link: https://docs.vualto.com/projects/vudrm/en/latest/DeveloperDocumentation/VUDRM-key-provision.html

### Examples of Each Document

#### CPIX Document

Each DRM Provider has a unique `systemId`. This helps to distinguish which values belong to which DRM Provider in a CPIX Document. 

```
System IDs:
Widevine: edef8ba9-79d6-4ace-a3c8-27dcd51d21ed
PlayReady: 9a04f079-9840-4286-ab92-e65be0885f95
FairPlay: 94ce86fb-07ff-4f43-adb8-93d2fa968ca2
```

Below is an example of a CPIX Document with all `DRM Providers` selected. 

```
<?xml version="1.0" encoding="UTF-8"?>
<cpix:CPIX xmlns:pskc="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:xsi="urn:ietf:params:xml:ns:keyprov:pskc" xmlns:cpix="urn:dashif:org:cpix" xsi:schemaLocation="urn:dashif:org:cpix cpix.xsd">
    <cpix:ContentKeyList>
        <cpix:ContentKey kid="11111111-1111-1111-1111-111111111111" explicitIV="MjlkMzNlYzVjYWIyZmRmZg==" commonEncryptionScheme="cenc">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>3wFFjPspsydY/t2uSPeE6w==</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="11111111-1111-1111-1111-111111111111" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH><PlayReady_PSSH></cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="22222222-2222-2222-22222-22222222222" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH><Widevine_PSSH></cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="33333333-3333-3333-33333-33333333333" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
            <cpix:URIExtXKey><FairPlay_URIExtXKey</cpix:URIExtXKey>
            <cpix:HLSSignalingData playlist="master"><master_playlist_value></cpix:HLSSignalingData>
            <cpix:HLSSignalingData playlist="media"><media_playlist_value></cpix:HLSSignalingData>
        </cpix:DRMSystem>
    </cpix:DRMSystemList>
    <cpix:ContentKeyUsageRuleList></cpix:ContentKeyUsageRuleList>
</cpix:CPIX>
```

#### CPIX keys as JSON

Below is an example of a CPIX keys as JSON document with all `DRM Providers` selected. 

```
{
    "key_id_hex": <key_id_hex_value>,
    "content_key_hex": <content_key_hex_value>,
    "iv_hex": <iv_hex_value>,
    "playready_pssh_data": <playready_pssh_data_value>,
    "playready_system_id": "9a04f079-9840-4286-ab92-e65be0885f95",
    "widevine_drm_specific_data": <widevine_drm_specific_data_value>,
    "widevine_drm_specific_data_with_key_id": <widevine_drm_specific_data_with_key_id_value>,
    "widevine_system_id": "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed",
    "fairplay_system_id": "94ce86fb-07ff-4f43-adb8-93d2fa968ca2",
    "fairplay_laurl": "skd://fairplay-license.vudrm.tech/v2/license/<content_id>"
}
```




## Packaging Content with Widevine and Playready DRM

widevine pssh hex = 000000427073736801000000edef8ba979d64acea3c827dcd51d21ed000000013e4a4bfba252a4b4c8396c8954d986f50000000e2206736f6d65496448e3dc959b06

widevine pssh base64 = AAAAQnBzc2gBAAAA7e+LqXnWSs6jyCfc1R0h7QAAAAGK8pQz9EeCAvSVMqHo0EsfAAAADiIGc29tZUlkSOPclZsG

mp4dash --encryption-key=3E4A4BFBA252A4B4C8396C8954D986F5:DF01458CFB29B32758FEDDAE48F784EB --playready --widevine-header=#AAAAQnBzc2gBAAAA7e+LqXnWSs6jyCfc1R0h7QAAAAGK8pQz9EeCAvSVMqHo0EsfAAAADiIGc29tZUlkSOPclZsG test.mp4 audio.mp4

https://vualto-demo.s3.eu-west-1.amazonaws.com/test/shaka-packager/bento4/output-vid-audio/stream.mpd

https://widevine-license.staging.vudrm.tech/proxy?token=vualto-demo%7c2021-08-31T11%3a19%3a49Z%7cRAQrLiTYv%2bZ8U9LrxO0JDw%3d%3d%7c88837fd431ea01691d32a76c3c17b3fb91c7f84b

## Packaging Content with Fairplay DRM


mp4hls --hls-version 7 --output-single-file --encryption-mode=SAMPLE-AES --encryption-key=3E4A4BFBA252A4B4C8396C8954D986F5:DF01458CFB29B32758FEDDAE48F784EB --encryption-iv-mode=fps --encryption-key-format=com.apple.streamingkeydelivery --encryption-key-uri=skd://fairplay-license.vudrm.tech/v2/license/someId --encryption-key-format-versions=1 test.mp4 audio.mp4

mp4hls --hls-version 7 --output-single-file --encryption-mode=SAMPLE-AES --encryption-key=3E4A4BFBA252A4B4C8396C8954D986F5:32396433336563356361623266646666 --encryption-iv-mode=fps --encryption-key-format=com.apple.streamingkeydelivery --encryption-key-uri=skd://fairplay-license.vudrm.tech/v2/license/someId --encryption-key-format-versions=1 test.mp4 audio.mp4

mp4hls --hls-version 7 --output-single-file --encryption-mode=SAMPLE-AES --encryption-key=3E4A4BFBA252A4B4C8396C8954D986F5:32396433336563356361623266646666 --encryption-iv-mode=sequence --encryption-key-format=com.apple.streamingkeydelivery --encryption-key-uri=skd://fairplay-license.vudrm.tech/v2/license/someId --encryption-key-format-versions=1 test.mp4 audio.mp4

mp4hls --hls-version 7 --output-single-file --encryption-mode=SAMPLE-AES --encryption-key=3E4A4BFBA252A4B4C8396C8954D986F532396433336563356361623266646666 --encryption-iv-mode=fps --encryption-key-format=com.apple.streamingkeydelivery --encryption-key-uri=skd://fairplay-license.vudrm.tech/v2/license/someId --encryption-key-format-versions=1 test.mp4 audio.mp4

USE MP4 DASH INSTEAD OF MP4HLS???????????????????


iv hex: 32396433336563356361623266646666

## Testing packaged content

Upload your packaged files and manifest to AWS S3 or an equivalent hosting service.

You can now test your newly packaged content with our player on the VUDRM Admin site.

- Log on to the VUDRM Admin site - https://admin.vudrm.tech

- On the left hand side of the page select `Configuration`.

- Click on `Test your Stream`.

- A token is automatically generated for use with your content.
 
- If you are testing Widevine / PlayReady content, enter the URL for where your manifest has been uploaded in the `DASH Stream URL` text box.

- If you are testing Fairplay content, ensure you are using a Mac and using Safari as your browser, enter the URL for where your manifest has been uploaded in the `HLS Stream URL` text box.

- Click `Load Player`.
 
- The player should now appear, click play to play the content.

You can confirm a relevant license request has been made by checking the network tab of the browsers dev tools.