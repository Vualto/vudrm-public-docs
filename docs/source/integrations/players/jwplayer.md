# JWPlayer

[JWPlayer](https://jwplayer.com/) is an industry leading HTML5 video player.

The [vuplay-jwplayer](https://github.com/Vualto/vuplay-jwplayer) repository demonstrates at a lower level how to integrate [VUDRM](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with JWPlayer.

## Example of JWplayer Integration

This file can be found in the repo mentioned above here: []()

```javascript
(function() {
    // Set the mpeg-dash stream URL.
    var dashStreamURL = "<dash-stream-url>";
    // Set the hls stream URL.
    var hlsStreamURL = "<hls-stream-url>";

    // Set the URL to retrieve the fairplay certificate from.
    var fairplayCertURL = "<fairplay-cert-url>";

    // Please login to https://admin.vudrm.tech to generate a VUDRM token.
    var vudrmToken = "<your-vudrm-token>";

    // setup jwplayer, passing the stream URLs and DRM configurations.  
    jwplayer("vuplay-container").setup({
        "playlist": [{
            "sources": [{
                "file": dashStreamURL,
                "drm": {
                    "widevine": {
                        "url": "https://widevine-license.vudrm.tech/proxy",
                        "headers": [{
                            "name": "X-VUDRM-TOKEN",
                            "value": vudrmToken
                        }]
                    },
                    "playready": {
                        "url": "https://playready-license.vudrm.tech/rightsmanager.asmx",
                        "headers": [{
                            "name": "X-VUDRM-TOKEN",
                            "value": vudrmToken
                        }]
                    }
                }
            }, 
            {
                "file": hlsStreamURL,
                "drm": {
                    "fairplay": {
                        "certificateUrl": fairplayCertURL,
                        "processSpcUrl": function (initData) {
                            return "https://" + initData.split("skd://").pop();
                        },
                        "licenseRequestHeaders": [
                            {
                                "name": "Content-type",
                                "value": "arraybuffer"
                            },
                            {
                                "name": "X-VUDRM-TOKEN",
                                "value": vudrmToken
                            }
                        ]
                    }
                }
            }]
        }]
    });
})();
```
