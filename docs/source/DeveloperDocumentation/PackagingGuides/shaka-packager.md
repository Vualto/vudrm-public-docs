# Shaka Packager

## Setup using Docker

- First download and install Docker - https://www.docker.com/get-started

- Pull the latest Shaka Packager Docker image by entering ```docker pull google/shaka-packager``` in your terminal.

- Now run the Docker image by entering ```docker run -v /<file_path_to_content>/:/media -it --rm google/shaka-packager``` in your terminal. This will replicate your directory and create a ```media``` file path within the Docker container.

- Test that setup is correct, by analyising a file within the filepath. You can do this by entering the command ```packager input=/media/<name_of_content>.mp4 --dump_stream_info```.

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

CPIX Documents are required to give Shaka Packager the necessary information in order to encrypt your media.

### Using VUDRM Admin site

You can retrieve these documents from the VUDRM Admin site. 

A guide can be found at the following link - [VUDRM Admin - CPIX Document Request](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#vudrm-encryption-keys)

### Using CURL Requests

Alternatively, you can also retrieve these documents with a CURL Request. 

Guides can be found at the following links:

* [VUDRM Key Provision - CPIX Document Request](https://docs.vualto.com/projects/vudrm/en/latest/DeveloperDocumentation/VUDRM-key-provision.html#request)

* [VUDRM Key Provision - JSON Keys Request](https://docs.vualto.com/projects/vudrm/en/latest/DeveloperDocumentation/VUDRM-key-provision.html#id16)

## Package your content

Now you have retrieved the relevant documentation, you can package content with Shaka Packager.

For more information on the examples below, please refer to the Shaka Packager Documentation which can be found at the following link - [Shaka Packager Documentation](https://google.github.io/shaka-packager/html/)

### Packaging Content with Widevine and Playready DRM

Below is an example of how we can send a request to Shaka Packager to encrypt our content with Widevine and Playready DRM and package it:

```
  packager \
   in=media/audio.mp4,stream=audio,output=media/encrypted-test-audio.mp4,drm_label=AUDIO \
   in=media/source.mp4,stream=video,output=media/encrypted-test-video.mp4,drm_label=VIDEO \
   --enable_raw_key_encryption \
   --keys label=AUDIO:key_id=<key_id_hex_value>:key=<content_key_hex_value>,label=VIDEO:key_id=<key_id_hex_value>:key=<content_key_hex_value> \
   --pssh <Widevine_PSSH(this needs to be converted from Base64 to Hex format)> \
   --protection_systems Widevine,PlayReady \
   --mpd_output media/encrypted-test.mpd
```

### Packaging Content with Fairplay DRM

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

## Testing packaged content

Upload your packaged files and manifest to AWS S3 or an equivalent hosting service. You can now test your newly packaged content with our player on the VUDRM Admin site.

A guide can be found at the following link - [Test Your Stream](https://docs.vualto.com/projects/vudrm/en/latest/UserGuide/VUDRM-Admin.html#test-your-stream)

You can confirm a relevant license request has been made by checking the network tab of the browsers dev tools.
