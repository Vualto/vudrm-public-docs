# Video JS

[Video JS](https://videojs.com) is an industry leading HTML5 video player.

The [vuplay-videojs](https://github.com/vualto/vuplay-videojs) repository demonstrates at a lower level how to integrate [VUDRM](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with Video JS.

## Example of Video JS Integration

You will need to load the following libraries:

```html
    <script src="https://cdnjs.cloudflare.com/ajax/libs/video.js/7.10.2/video.min.js"></script>
    <script src="https://unpkg.com/@videojs/http-streaming@2.3.0/dist/videojs-http-streaming.js"></script>
    <script src="https://unpkg.com/videojs-contrib-eme@3.7.0/dist/videojs-contrib-eme.js"></script>
```

The following code snippet can be found in the repo mentioned above here: [https://github.com/Vualto/vuplay-videojs/blob/master/src/vuplay.js](https://github.com/Vualto/vuplay-videojs/blob/master/src/vuplay.js)

```javascript
const streamURL = "<your-stream-url>"
const contentId = "<content-id>";

const token = encodeURIComponent(
    "<your-vudrm-token>"
);

// playready
const playReadyLicenseServerURL =
    "https://playready-license.vudrm.tech/rightsmanager.asmx?token=" + token;

// widevine
const widevineLicenseServerURL =
    "https://widevine-license.vudrm.tech/proxy?token=" + token;

// fairplay
const fairplayCertificateUri =
    "https://fairplay-license.vudrm.tech/certificate/vualto-demo";

const fairplayLicenseUri =
    "https://fairplay-license.vudrm.tech/license/" +
    contentId +
    "?token=" +
    token;

(function () {
    var player = videojs("my-video", {
        autoplay: true,
    });

    // eme extension must be initialised before source is set
    player.eme();

    player.src({
        src: streamURL,
        type: (streamURL.endsWith('.mpd') ? "application/dash+xml" : "application/x-mpegURL"),
        keySystems: {
            "com.apple.fps.1_0": {
                certificateUri: fairplayCertificateUri,
                licenseUri: fairplayLicenseUri,
            },
            "com.microsoft.playready": playReadyLicenseServerURL,
            "com.widevine.alpha": widevineLicenseServerURL,
        },
    });
})();
```