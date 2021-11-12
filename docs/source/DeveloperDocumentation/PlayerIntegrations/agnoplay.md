# Agnoplay

[Agnoplay](https://www.agnoplay.com/) is a fully agnostic, cloudbased player service.

The [agnoplay](https://github.com/Vualto/agnoplay) repository demonstrates at a lower level how to integrate [VUDRM](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with Agnoplay.

## Example of Agnoplay Integration

This files below can be found in the repo mentioned above here: 

* [https://github.com/Vualto/agnoplay/blob/master/index.html](https://github.com/Vualto/agnoplay/blob/master/index.html)
* [https://github.com/Vualto/agnoplay/blob/master/agnoplay_vudrm.js](https://github.com/Vualto/agnoplay/blob/master/agnoplay_vudrm.js)

### index.html

```javascript
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Agnoplay example</title>
        <meta name="description" content="Agnoplay Player Example" />
        <meta name="author" content="Vualto" />
        <style>
            #agnoplayer {
                width: 50%;
            }
        </style>
    </head>
    <body>
        <div id="agnoplayer"></div>
        <script
            type="text/javascript"
            crossorigin="anonymous"
            // Please replace "yourdomain" with your brand name.
            src="https://player.yourdomain.com/static/agnoplay/js/agnoplay.js"
        ></script>
        <script type="text/javascript" src={agnoplayjs}></script>
    </body>
</html>
```

### agnoplay_vudrm.js
```javascript
var element = document.getElementById("agnoplayer");

// Please login to https://admin.vudrm.tech to generate a VUDRM token.
var vudrmToken = "<your-vudrm-token>"

var config = {
    //Enter your brand name.
    brand: "<brand-name>",
    url: window.location.href,
    streaming_protocol: 'auto',
    //Enter the Agnoplay Player License Key.
    license_key: "<agnoplay-license-key>",
    stream_source: "custom",
    custom_source: {
        //Set your stream URL.
        source: "<video-file-url>",
        //Set the title name.
        title: "<video-title>", // Required
        //Set the file path for the thumbnail.
        thumbnail: "<video-poster>", // Required
        type: "application/dash+xml",
        drm: {
            widevine: {
                licenseUrl: "https://widevine-license.vudrm.tech/proxy",
                licenseHeaders: {
                    "x-vudrm-token": vudrmToken
                }
            },
            playready: {
                licenseUrl: "https://playready-license.vudrm.tech/rightsmanager.asmx",
                licenseHeaders: {
                    "x-vudrm-token": vudrmToken
                }
            },
            fairplay: {
                licenseUrl: "https://fairplay-license.vudrm.tech/v2/license/",
                licenseHeaders: {
                    "x-vudrm-token": vudrmToken
                },
                certificateUrl: "https://fairplay-license.vudrm.tech/certificate/",
                certificateHeaders: {
                    "x-vudrm-token": vudrmToken
                }
            }
        }
    }
}
var player = window.AGNO.insertPlayer(config, element);
```