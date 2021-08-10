# Shaka Packager

## Setup using Docker

- First download and install Docker - https://www.docker.com/get-started

- Pull the latest Shaka Packager Docker image by entering ```docker pull google/shaka-packager``` in your terminal

- Now run the Docker image by entering ```docker run -v /<file_path_to_content>/:/media -it --rm google/shaka-packager``` in your terminal. This will replicate your directory and create a ```media``` file path within the Docker container.

- Test that setup is correct, by analyising a file within the filepath. You can do this by entering the command ```packager input=/media/<name_of_content>.mp4 --dump_stream_info```

If successful you should then get a dump of the stream information which should look something like this:

```
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

### Using VUDRM Admin site

- Log on to the VUDRM Admin site - https://admin.vudrm.tech

- On the left hand side of the page select `Configuration`

- Go to the `VUDRM Encryption Keys` section

- Select which `DRM Providers` are required

- Add a `Content ID`

- Click `Generate`

This will then generate a CPIX Document and CPIX keys as JSON - you can copy and paste these to your favourite text editor to refer to them for future use.

### Using CURL Requests

Alternatively, you can also retrieve these documents with a CURL Request. 

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
        <cpix:ContentKey kid="3e4a4bfb-a252-a4b4-c839-6c8954d986f5" explicitIV="MjlkMzNlYzVjYWIyZmRmZg==" commonEncryptionScheme="cenc">
            <cpix:Data>
                <pskc:Secret>
                    <pskc:PlainValue>3wFFjPspsydY/t2uSPeE6w==</pskc:PlainValue>
                </pskc:Secret>
            </cpix:Data>
        </cpix:ContentKey>
    </cpix:ContentKeyList>
    <cpix:DRMSystemList>
        <cpix:DRMSystem kid="3e4a4bfb-a252-a4b4-c839-6c8954d986f5" systemId="9a04f079-9840-4286-ab92-e65be0885f95">
            <cpix:PSSH><PlayReady_PSSH></cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="3e4a4bfb-a252-a4b4-c839-6c8954d986f5" systemId="edef8ba9-79d6-4ace-a3c8-27dcd51d21ed">
            <cpix:PSSH><Widevine_PSSH></cpix:PSSH>
        </cpix:DRMSystem>
        <cpix:DRMSystem kid="3e4a4bfb-a252-a4b4-c839-6c8954d986f5" systemId="94ce86fb-07ff-4f43-adb8-93d2fa968ca2">
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

Now we have retrieved the relevant documentation, we can now package content.

Below is an example of how we can send a request to Shaka Packager to encrypt our content with Widevine and Playready DRM and package it:

```
  packager \
  in=media/audio.mp4,stream=audio,output=media/encrypted-test-audio.mp4,drm_label=AUDIO \
  in=media/source.mp4,stream=video,output=media/encrypted-test-video.mp4,drm_label=VIDEO \
  --enable_raw_key_encryption \
  --keys label=AUDIO:key_id=<key_id_hex_value>:key=<content_key_hex_value>,label=VIDEO:key_id=<key_id_hex_value>:key=<content_key_hex_value> \
  --pssh <Widevine_PSSH(converted to Hex format)> \
  --protection_systems Widevine,PlayReady \
  --mpd_output media/encrypted-test.mpd
```

### Line 1

`packager` is used to start our request.

### Line 2

We map which media we would like to be packaged. Notice the prefix of `media/` before the mp4 file. This is because Docker has mapped the file path you gave it when running the image to `media`. 

In this instance, we have selected an audio file, therefore `stream=audio` is entered.

The output is what you would like the packaged mp4 file to be named. In this case, `output=media/encrypted-test-audio.mp4`.

Finally, we give a label to the content - this is used for issuing keys later. We have entered `drm_label=AUDIO`.

### Line 3

Same as line 2, but this time we are selecting a video file, therefore `stream=video` is entered.

The output is `output=media/encrypted-test-video.mp4`. 

The label will be `drm_label=VIDEO`.

### Line 3

`enable_raw_key_encryption` as we are encrypting our content using the keys generated by the "CPIX keys as JSON" document.

### Line 4

The data needed for this line comes from the "CPIX keys as JSON" document, specifically the `key_id_hex_value` and `content_key_hex_value`. 

We assign the key values to the same label we labelled our media in line 2 and 3. In this instance, `label=AUDIO` and `label=VIDEO`.

The same key values are used for both AUDIO and VIDEO labels.

### Line 5

For the line where we input the `pssh` data, the value `Widevine_PSSH` comes from the CPIX Document.

Please be aware that the `pssh` data from the CPIX Document is in Base64 format. Shaka Packager requires the `pssh` value to be in Hex format. You can use any converter, such as https://base64.guru/converter/decode/hex

NOTE - COULD TALK ABOUT THE SHAKA PSSH BOX CONVERSION ALSO WITHIN THE SHAKA PACKAGER DOCKER IMAGE??????.......

PlayReady does not require a `pssh` value in order for it to be encrypted. This is handled by the key values set in Line 4.

### Line 6 

`protection_systems` is where you input which DRM you would like to encrypt. In this case, `Widevine, PlayReady` is entered.

### Line 7

`mpd_output` is where you name the manifest and where it is saved once packaged. In this case, `media/encrypted-test.mpd` is entered.

## Packaging Content with Fairplay DRM

Below is an example of how we can send a request to Shaka Packager to encrypt our content with Fairplay DRM and package it: 

```
  packager \
  in=media/audio.mp4,stream=audio,output=media/encrypted-test-hls-audio.mp4,drm_label=AUDIO \
  in=media/test.mp4,stream=video,output=media/encrypted-test-hls-video.mp4,drm_label=VIDEO \
  --protection_scheme cbcs \
  --enable_raw_key_encryption \
  --keys label=AUDIO:key_id=<key_id_hex_value>:key=<content_key_hex_value>,label=VIDEO:key_id=<key_id_hex_value>:key=<content_key_hex_value> \
  --protection_systems Fairplay \
  --iv <iv_hex_value> \
  --hls_master_playlist_output media/encrypted-test-hls.m3u8 \
  --hls_key_uri skd://fairplay-license.vudrm.tech/v2/license/<content_id>
```

Packaging content with Fairplay DRM is much of the same as packaging with Widevine and PlayReady DRM. However, there are differences detailed below.

### Line 3

`protection_scheme` must be set to `cbcs`

### Line 6

`protection_systems` is now set to Fairplay.

### Line 7

The `iv_hex_value` from the "CPIX keys as JSON" document is needed.

### Line 8

`hls_master_playlist_output` acts similar to the `mpd_output`. The file name must have the suffix `.m3u8`.

### Line 9

`hls_key_uri` is set to the value of the `fairplay_laurl` in the "CPIX keys as JSON" document.

## Testing packaged content

Upload your packaged files and manifest to AWS S3 or an equivalent hosting service.

You can now test your newly packaged content with our player on the VUDRM Admin site.

- Log on to the VUDRM Admin site - https://admin.vudrm.tech

- On the left hand side of the page select `Configuration`

- Click on `Test your Stream` 

- A token is automatically generated for use with your content
 
- If you are testing Widevine / PlayReady content, enter the URL for where your manifest has been uploaded in the `DASH Stream URL` text box

- If you are testing Fairplay content, ensure you are using a Mac and using Safari as your browser, enter the URL for where your manifest has been uploaded in the `HLS Stream URL` text box

- Click `Load Player`

- The player should now appear, click play to play the content


NOTE - TALK ABOUT HOW TO SEE IF LICENSES HAVE BEEN AQUIRED IN THE NETWORK TABS FOR EACH BROWSER???....







THIS IS FOR MY REFERENCE ONLY - THIS WILL NOT BE IN THE FINAL PRODUCT...


The below works for Widevine encryption only:

  packager \
  in=media/source.mp4,stream=video,output=media/encrypted-test-video.mp4,drm_label=VIDEO \
  --enable_raw_key_encryption \
  --keys label=VIDEO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB \
  --pssh 000000427073736801000000edef8ba979d64acea3c827dcd51d21ed000000013e4a4bfba252a4b4c8396c8954d986f50000000e2206736f6d65496448e3dc959b06 \
  --protection_systems Widevine \
  --mpd_output media/encrypted-test.mpd

Below is for Widevine / PlayReady encryption

  packager \
  in=media/tomorrowland/test.mp4,stream=video,output=media/tomorrowland/encrypted-test4-video.mp4,drm_label=VIDEO \
  --enable_raw_key_encryption \
  --keys label=VIDEO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB \
  --pssh 000000427073736801000000edef8ba979d64acea3c827dcd51d21ed000000013e4a4bfba252a4b4c8396c8954d986f50000000e2206736f6d65496448e3dc959b06  \
  --protection_systems Widevine,PlayReady \
  --mpd_output media/tomorrowland/encrypted-test4.mpd

Below is for Widevine / PlayReady encryption with sound...

  packager \
  in=media/tomorrowland/audio.mp4,stream=audio,output=media/tomorrowland/encrypted-test5-audio.mp4,drm_label=AUDIO \
  in=media/tomorrowland/test.mp4,stream=video,output=media/tomorrowland/encrypted-test5-video.mp4,drm_label=VIDEO \
  --enable_raw_key_encryption \
  --keys label=AUDIO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB,label=VIDEO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB \
  --pssh 000000427073736801000000edef8ba979d64acea3c827dcd51d21ed000000013e4a4bfba252a4b4c8396c8954d986f50000000e2206736f6d65496448e3dc959b06  \
  --protection_systems Widevine,PlayReady \
  --mpd_output media/tomorrowland/encrypted-test5.mpd

  https://shaka-packager-test.s3.eu-west-1.amazonaws.com/encrypted-test5.mpd

Below is for Fairplay encryption with sound

  packager \
  in=media/tomorrowland/audio.mp4,stream=audio,output=media/tomorrowland/encrypted-test-hls-audio.mp4,drm_label=AUDIO \
  in=media/tomorrowland/test.mp4,stream=video,output=media/tomorrowland/encrypted-test-hls-video.mp4,drm_label=VIDEO \
  --protection_scheme cbcs \
  --enable_raw_key_encryption \
  --keys label=AUDIO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB,label=VIDEO:key_id=3E4A4BFBA252A4B4C8396C8954D986F5:key=DF01458CFB29B32758FEDDAE48F784EB \
  --protection_systems Fairplay \
  --iv 32396433336563356361623266646666 \
  --hls_master_playlist_output media/tomorrowland/encrypted-test-hls.m3u8 \
  --hls_key_uri skd://fairplay-license.vudrm.tech/v2/license/someId

  
  https://shaka-packager-test.s3.eu-west-1.amazonaws.com/encrypted-test-hls.m3u8


Below are the system IDs for their respective encryption service:

```
System IDs:
Widevine: edef8ba9-79d6-4ace-a3c8-27dcd51d21ed
PlayReady: 9a04f079-9840-4286-ab92-e65be0885f95
FairPlay: 94ce86fb-07ff-4f43-adb8-93d2fa968ca2
```



./usr/bin/pssh-box.py

./usr/bin/pssh-box.py --human --from-base64 <pssh-box details>

