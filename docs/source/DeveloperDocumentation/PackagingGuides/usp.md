# Unified Streaming

The recommended approach to integrate with USP will depend on your exact use case. If you will be requesting CPIX documents regularly, e.g. for use with live content, it is recommended to use our [CPIX Edge Reverse Proxy](https://hub.docker.com/r/vualto/cpix-proxy) to retrieve CPIX documents. If you will be making less frequent calls for CPIX documents, e.g. for one time packaging, you can use either our [CPIX Edge Reverse Proxy](https://hub.docker.com/r/vualto/cpix-proxy) or request CPIX documents from our CPIX Key Provider API directly.

If you would prefer to not use a CPIX document at all you can also use the [JSON formatted response](#json-keys).

## mp4split

### With a CPIX document

A CPIX document can be used in the below mp4split commands to encrypt content. For a full list CPIX mp4split options see Unified Streamings full [documentation](https://docs.unified-streaming.com/documentation/drm/cpix_cmdline_options.html).

You can encrypt content with a CPIX document stored locally by doing the following commands.

```bash
curl -XGET -H "API-KEY: <api-key>" \
  "https://cpix.vudrm.tech/v1/cpix/<client-name>/<content-id>" \
  > drm.cpix

mp4split --license-key=$USP_LICENSE_KEY -o $ismName \
  --cpix=drm.cpix \
  video.ismv audio.isma
```

Or you can reference our [CPIX Edge Reverse Proxy](https://hub.docker.com/r/vualto/cpix-proxy) directly in the mp4split command as follows.

```bash
mp4split --license-key=$USP_LICENSE_KEY -o $ismName \
  --cpix="http://local-cpix-proxy/v1/cpix/<client-name>/<content-id>" \
  video.ismv audio.isma
```

### With JSON keys

Keys requested in JSON format can be used in the below mp4split to encrypt content. For more information about using mp4split you can vist Unified Streamings full [documentation](https://docs.unified-streaming.com/documentation/drm/index.html).

```bash
mp4split --license-key=$USP_LICENSE_KEY -o $ismName \
     --iss.key=$key_id_hex:$content_key_hex \
     --iss.license_server_url="https://playready-license.vudrm.tech/rightsmanager.asmx" \
     --widevine.key=$key_id_hex:$content_key_hex \
     --widevine.license_server_url="https://widevine-license.vudrm.tech/proxy" \
     --widevine.drm_specific_data=$widevine_drm_specific_data \
     --hls.client_manifest_version=4 \
     --hls.key=:$content_key_hex \
     --hls.key_iv=$iv_hex \
     --hls.license_server_url=$fairplay_laurl \
     --hls.playout=sample_aes_streamingkeydelivery \
     video.ismv audio.isma
```
