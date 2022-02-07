# Video JS

[Video JS](https://videojs.com) is an industry leading HTML5 video player.

The [vuplay-videojs](https://github.com/vualto/vuplay-videojs) repository demonstrates at a lower level how to integrate [Studio DRM](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with Video JS.

## Example of Video JS Integration

You will need to load the following libraries:

```html
    <script src="https://cdnjs.cloudflare.com/ajax/libs/video.js/7.10.2/video.min.js"></script>
    <script src="https://unpkg.com/@videojs/http-streaming@2.3.0/dist/videojs-http-streaming.js"></script>
    <script src="https://unpkg.com/videojs-contrib-eme@3.7.0/dist/videojs-contrib-eme.js"></script>
```

The following code snippet can be found in our [vuplay videojs repo](https://github.com/Vualto/vuplay-videojs/blob/master/src/vuplay.js).

```javascript
const streamURL = "<your-stream-url>";

const token = "<your-vudrm-token>";

// playready
const playReadyLicenseServerURL =
    "https://playready-license.vudrm.tech/rightsmanager.asmx?token=" + encodeURIComponent(token);

// widevine
const widevineLicenseServerURL =
    "https://widevine-license.vudrm.tech/proxy?token=" + encodeURIComponent(token);

// fairplay
const fairplayCertificateUri =
    "https://fairplay-license.vudrm.tech/certificate/<your-client-name>";

var fairplayLicenseUri;

var Utils = {
    uint16ArrayToString: function(array) {
        var uint16Array = new Uint16Array(array.buffer);
        return String.fromCharCode.apply(String, uint16Array);
    }
};

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
                getContentId: function (emeOptions, initData, eme) {
                    fairplayLicenseUri = "https://" + Utils.uint16ArrayToString(initData).split("skd://").pop();
                    var contentId = fairplayLicenseUri.split("/").pop();
                    return contentId;
                },
                getLicense: function (emeOptions, contentId, keyMessage, callback) {
                    videojs.xhr({
                        url: fairplayLicenseUri,
                        method: 'POST',
                        responseType: 'arraybuffer',
                        body: keyMessage,
                        headers: {
                            'Content-type': 'application/octet-stream',
                            'x-vudrm-token': token
                        }
                    }, (err, response, responseBody) => {
                        if (err) {
                            callback(err)
                            return
                        }
                        callback(null, responseBody)
                    })
                },
            },
            "com.microsoft.playready": playReadyLicenseServerURL,
            "com.widevine.alpha": widevineLicenseServerURL,
        },
    });
})();
```