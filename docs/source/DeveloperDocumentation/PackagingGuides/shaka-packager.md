# Shaka Packager

## Setup using Docker

Setup Shaka Packager using the commands below:

```bash
# Pull and run shaka packager 
docker pull google/shaka-packager
docker run -v /<file_path_to_content>/:/media -it --rm google/shaka-packager

# Test that setup is correct by getting stream info
packager input=/media/<name_of_content>.mp4 --dump_stream_info
```

If successful, the expected output of the stream dump should look like this:

```text
File "/media/test.mp4":
Found 1 stream(s).
Stream [0] type: Video
 codec_string: avc1.640028
 time_scale: 12288
 duration: 17078784 (1389.9 seconds)
 is_encrypted: false
 codec: H264
 width: 1920
 height: 810
 pixel_aspect_ratio: 1:1
 trick_play_factor: 0
 nalu_length_size: 4

Packaging completed successfully.
```

## Get CPIX Documents

CPIX Documents are required to give Shaka Packager the necessary information in order to encrypt your media.

You can retrieve these documents from the VUDRM Admin site. 

A guide can be found at the following link - [VUDRM Admin - CPIX Document Request](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#vudrm-encryption-keys)

More information on our CPIX Key Provider API can be found at the following link - [CPIX Key Provider API](https://docs.vualto.com/projects/vudrm/en/latest/DeveloperDocumentation/VUDRM-key-provision.html#cpix-key-provider-api)

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

Now you have retrieved the relevant documentation, you can package content with Shaka Packager.

The values from the CPIX documents will need to be copied and pasted to the corresponding section in the examples below.

For more information, please refer to the Shaka Packager Documentation which can be found at the following link - [Shaka Packager Documentation](https://google.github.io/shaka-packager/html/)

### Packaging Content with Widevine and Playready DRM

Below is an example of how we can send a request to Shaka Packager to encrypt our content with Widevine and Playready DRM and package it.

Please note, the `Widevine_PSSH` from the CPIX Document is in Base64 format. Shaka Packager requires this to be in Hex format which can be achieved by running the following command `echo '<Widevine_PSSH>' | od -A n -t x1 | sed 's/ *//g'`:

```text
  packager \
   in=/media/<name-of-content>.mp4,stream=audio,output=/media/<name-of-output-content>.mp4,drm_label=AUDIO \
   in=/media/<name-of-content>.mp4,stream=video,output=/media/<name-of-output-content>.mp4,drm_label=VIDEO \
   --enable_raw_key_encryption \
   --keys label=AUDIO:key_id=<key_id_hex_value>:key=<content_key_hex_value>,label=VIDEO:key_id=<key_id_hex_value>:key=<content_key_hex_value> \
   --pssh <Widevine_PSSH> \
   --protection_systems Widevine,PlayReady \
   --mpd_output /media/<name-of-manifest>.mpd
```

### Packaging Content with Fairplay DRM

Below is an example of how we can send a request to Shaka Packager to encrypt our content with Fairplay DRM and package it: 

```text
  packager \
   in=media/<name-of-content>.mp4,stream=audio,output=media/<name-of-output-content>.mp4,drm_label=AUDIO \
   in=media/<name-of-content>.mp4,stream=video,output=media/<name-of-output-content>.mp4,drm_label=VIDEO \
   --protection_scheme cbcs \
   --enable_raw_key_encryption \
   --keys label=AUDIO:key_id=<key_id_hex_value>:key=<content_key_hex_value>,label=VIDEO:key_id=<key_id_hex_value>:key=<content_key_hex_value> \
   --protection_systems Fairplay \
   --iv <iv_hex_value> \
   --hls_master_playlist_output media/<name-of-manifest>.m3u8 \
   --hls_key_uri <fairplay_laurl_value>
```

## Testing packaged content

Upload your packaged files and manifest to AWS S3 or an equivalent hosting service. You can now test your newly packaged content with our player on the VUDRM Admin site.

A guide can be found at the following link - [Test Your Stream](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#test-your-stream)

You can confirm a relevant license request has been made by checking the network tab of the browsers dev tools.
