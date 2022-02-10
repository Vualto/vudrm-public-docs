# NGINX VOD Module

Our CPIX Key Provider API has an endpoint that is dedicated to providing the key information needed to integrate with Kaltura's [NGINX VOD Module](https://github.com/kaltura/nginx-vod-module). 

You can request encryption keys in the format required by the NGINX VOD Module by performing the following command.

```bash
  curl --location --request GET 'https://cpix.vudrm.tech/v1/keys/<client-name>/nginx/<content-id>' \
    --header 'api-key: <api-key>'
```

This will return the following response:
```json
[
    {
        "pssh": [
            {
                "data": "<base64 encoded PlayReady PSSH data>",
                "uuid": "9a04f079-9840-4286-ab92-e65be0885f95"
            },
            {
                "data": "<base64 encoded Widevine PSSH data>",
                "uuid": "edef8ba9-79d6-4ace-a3c8-27dcd51d21ed"
            }
        ],
        "key": "<base64 encoded encryption key>",
        "key_id": "<base64 encoded key id>",
        "iv": "<base64 encoded iv>"
    }
]
```

Below you can find a full example of an NGINX config file that will integrate the VOD module with VUDRM. This example is a modified version of the example given by [The New York Times](https://github.com/nytimes/nginx-vod-module-docker) in their repo containing the Docker files necessary to run the VOD module as a Docker container. 

This modifications to the example config are as follows:
You will need the addition of this to activate just in time DRM on all endpoints.
```
vod_drm_enabled on;
vod_drm_request_uri "/<client-name>/nginx/<content-id>";
vod_drm_upstream_location /drm;
```

The addition of this proxy pass inorder to add your api key to the request made to our CPIX Key Provider API.
```
location /drm {
  internal;
  proxy_pass "https://cpix.vudrm.tech/v1/keys";
  proxy_set_header API-KEY <api-key>;
} 
```

This will give you support for DASH with Widevine and PlayReady.
To support HLS with FairPlay you will need to add the following to your hls endpoint.
```
vod_hls_encryption_method sample-aes;
vod_hls_encryption_key_uri "skd://fairplay-license.vudrm.tech/v2/license/<content-id>";
vod_hls_encryption_key_format "com.apple.streamingkeydelivery";
vod_hls_encryption_key_format_versions "1";
```

## NGINX Config - Full Example

```
worker_processes  auto;

events {
	use epoll;
}

http {
	log_format  main  '$remote_addr $remote_user [$time_local] "$request" '
		'$status "$http_referer" "$http_user_agent"';

	access_log  /dev/stdout  main;
	error_log   stderr debug;

	default_type  application/octet-stream;
	include       /usr/local/nginx/conf/mime.types;

	sendfile    on;
	tcp_nopush  on;
	tcp_nodelay on;

	vod_mode                           local;
	vod_metadata_cache                 metadata_cache 16m;
	vod_response_cache                 response_cache 512m;
	vod_last_modified_types            *;
	vod_segment_duration               9000;
	vod_align_segments_to_key_frames   on;
	vod_dash_fragment_file_name_prefix "segment";
	vod_hls_segment_file_name_prefix   "segment";

	vod_drm_enabled on;
	vod_drm_request_uri "/vualto-demo/nginx/test-nginx";
	vod_drm_upstream_location /drm;

	vod_manifest_segment_durations_mode accurate;

	open_file_cache          max=1000 inactive=5m;
	open_file_cache_valid    2m;
	open_file_cache_min_uses 1;
	open_file_cache_errors   on;

	aio on;

	server {
		listen 80;
		server_name localhost;
		root /opt/static;

		location ~ ^/videos/.+$ {
			autoindex on;
		}

		location /hls/ {
			vod hls;
			vod_hls_encryption_method sample-aes;
			vod_hls_encryption_key_uri "skd://fairplay-license.vudrm.tech/v2/license/test-nginx";
			vod_hls_encryption_key_format "com.apple.streamingkeydelivery";
			vod_hls_encryption_key_format_versions "1";

			alias /opt/static/videos/;
			add_header Access-Control-Allow-Headers '*';
			add_header Access-Control-Allow-Origin '*';
			add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
		}

		location /thumb/ {
			vod thumb;
			alias /opt/static/videos/;
			add_header Access-Control-Allow-Headers '*';
			add_header Access-Control-Allow-Origin '*';
			add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
		}

		location /dash/ {
			vod dash;
			alias /opt/static/videos/;
			add_header Access-Control-Allow-Headers '*';
			add_header Access-Control-Allow-Origin '*';
			add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
		}

		location /drm {
			internal;
			proxy_pass "https://cpix.vudrm.tech/v1/keys";
			proxy_set_header API-KEY <api-key>;
		} 
	}
}
```