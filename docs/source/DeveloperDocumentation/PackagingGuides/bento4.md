# Bento4

## Installing Bento4

You can download the binaries for your preferred OS from [Bento4](https://www.bento4.com/downloads/).

Add the filepath for the binaries to the Bento4 `bin` folder as a value to your `Path` environemnt variables on your computer.

## Get CPIX Documents

CPIX Documents are required to give Bento4 the necessary information in order to encrypt your media.

You can retrieve these documents from the VUDRM Admin site. 

For more information, please view our documentation on [CPIX Document Requests](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#vudrm-encryption-keys) and our [CPIX Key Provider API](https://docs.vualto.com/projects/vudrm/en/latest/DeveloperDocumentation/VUDRM-key-provision.html#cpix-key-provider-api).

### Examples of Each Document

#### CPIX Document

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

## Package your content

Now you have retrieved the relevant documentation, you can package content with Bento4.

The values from the CPIX documents will need to be copied and pasted to the corresponding section in the examples below.

For more information, please refer to the documentation from [Bento4](https://www.bento4.com/documentation/mp4dash/).

### Packaging Content with Widevine and Playready DRM

Below is an example of how we can send a request to Bento4 to encrypt our content with Widevine and PlayReady DRM and package it:

```text
mp4dash --encryption-key=<key_id_hex_value>:<content_key_hex_value> --playready --widevine-header=#<Widevine_PSSH> --mpd-name=<name_of_manifest> <name_of_video_file>.mp4 <name_of_audio_file>.mp4
```

### Packaging Content with Fairplay DRM

Below is an example of how we can send a request to Bento4 to encrypt our content with Fairplay DRM and package it:

```text
mp4dash --hls --encryption-key=<key_id_hex>:<content_key_hex_value>:<iv_hex_value> --encryption-cenc-scheme=cbcs --fairplay-key-uri=<fairplay_laurl_value> --hls-master-playlist-name=<name_of_manifest> <name_of_video_file>.mp4 <name_of_audio_file>.mp4
```

## Testing packaged content

Upload your packaged files and manifest to AWS S3 or an equivalent hosting service. You can now test your newly packaged content with our player on the VUDRM Admin site.

For more information, please view our documentation on how to [Test Your Stream](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#test-your-stream).

You can confirm a relevant license request has been made by checking the network tab of the browsers dev tools.