# JW Player SDK Android VUDRM Widevine Integration

Learn how to enable digital rights management (DRM) protected playback for an Android app using VUDRM Widevine. This document is an extension of the JW Player SDK Android [Play DRM-protected content](https://developer.jwplayer.com/jwplayer/docs/android-play-drm-protected-content#supported-drm-provider) documentation.

## Requirements

- Android 4.3+
- JW Player SDK Android v4+
- DRM license server URL
- VUDRM Widevine protected content

## Preparation
Ensure playback can be achieved with unprotected content either using your own application, or the updated [JW Player SDK Android demo application](https://github.com/vualto/vudrm-jw-sdk-android) which contains all the following source code that will be needed to use VUDRM Widevine.

## Implementation

If you require any assistance or further information with regard to the following implementation, please contact [support@vualto.com](support@vualto.com). If you are unable to play your content, please include any application logs and the stream configuration used.

1. Add a **Util.java** to your project. This utility is used by the `MediaDrmCallback` class to download data.

		public class Util {
    	public static byte[] executePost(String url, byte[] data, Map<String, String> requestProperties)
            throws IOException {
        HttpURLConnection urlConnection = null;
        try {
            urlConnection = (HttpURLConnection) new URL(url).openConnection();
            urlConnection.setRequestMethod("POST");
            urlConnection.setDoOutput(data != null);
            urlConnection.setDoInput(true);
            if (requestProperties != null) {
                for (Map.Entry<String, String> requestProperty : requestProperties.entrySet()) {
                    urlConnection.setRequestProperty(requestProperty.getKey(), requestProperty.getValue());
                }
            }
            // Write the request body, if there is one.
            if (data != null) {
                OutputStream out = urlConnection.getOutputStream();
                try {
                    out.write(data);
                } finally {
                    out.close();
                }
            }
            // Read and return the response body.
            InputStream inputStream = urlConnection.getInputStream();
            try {
                return toByteArray(inputStream);
            } finally {
                inputStream.close();
            }
        } finally {
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
        }
    	}
		}

2. Add **VUDRMCallback.java** to your project. This is the VUDRM `MediaDrmCallback` class that prepares and handles the VUDRM Widevine license request. Be sure to replace `INSERT_CERTIFICATE_URL` with a certificate server URL.

		@TargetApi(18)
		public class VUDRMCallback implements MediaDrmCallback {

    	private static final String WIDEVINE_GTS_DEFAULT_BASE_URI =
            "INSERT_CERTIFICATE_URL";

    	private final String defaultUri;

    	public VUDRMCallback(String token) {
        String params = "?token=" +  Uri.encode(token);
        defaultUri = WIDEVINE_GTS_DEFAULT_BASE_URI + params;
    	}

    	protected VUDRMCallback(Parcel in) {
        defaultUri = in.readString();
    	}

    	public static final Creator<VUDRMCallback> CREATOR = new Creator<VUDRMCallback>() {
        @Override
        public VUDRMCallback createFromParcel(Parcel in) {
            return new VUDRMCallback(in);
        }

        @Override
        public VUDRMCallback[] newArray(int size) {
            return new VUDRMCallback[size];
        }
    	};

    	@Override
    	public byte[] executeProvisionRequest(UUID uuid, ExoMediaDrm.ProvisionRequest request) throws IOException {
        String url = request.getDefaultUrl() + "&signedRequest=" + new String(request.getData());
        return Util.executePost(url, null, null);
    	}

    	@Override
    	public byte[] executeKeyRequest(UUID uuid, ExoMediaDrm.KeyRequest request) throws IOException {
        String url = request.getLicenseServerUrl();
        if (TextUtils.isEmpty(url)) {
            url = defaultUri;
        }
        return Util.executePost(url, request.getData(), null);
    	}

    	@Override
    	public int describeContents() {
        return 0;
    	}

    	@Override
    	public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(defaultUri);
    	}
		}

3. In the player view class, create a `PlaylistItem` and set the video URL (`file`) of the DRM-protected content.

 4.  Create a new instance of your `MediaDrmCallback` implementation and set it to the PlaylistItem. Be sure to replace `{INSERT_TOKEN}` with a valid VUDRM token.


		// Create a list to contain the PlaylistItems
		List<PlaylistItem> playlist = new ArrayList<>();

		// Add a PlaylistItem pointing to the first piece of content
		PlaylistItem content = new PlaylistItem.Builder()
				.file("INSERT_CONTENT_URL")
				.mediaDrmCallback(new VUDRMCallback("INSERT_TOKEN"))
				.build();

		// Add the content to the playlist
		playlist.add(content);

		// Load the media source
		PlayerConfig config = new PlayerConfig.Builder()
				.playlist(playlist)
				.build();

		mPlayer.setup(config);

