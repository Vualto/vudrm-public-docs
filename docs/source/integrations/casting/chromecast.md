# Chromecast

Chromecast is a casting device developed by Google.

## Basic setup

In order to stream DRM protected content on a Chromecast using Google' web SDK you will need to create a Custom Receiver app. The app will utilise two parts a `Sender` and a `Receiver`.

Typically the `Sender` would reside either as part of the web page or your chosen player.

The `Receiver` is an application that you host and register with Google.

```javascript
if (event.data.media && event.data.media.contentId){
    let host, protocol;
    let url = event.data.media.contentId;
    let initStart = event.data.media.currentTime || 0;
    let autoplay = event.data.autoplay !== false;
    let customData = event.data.media.customData || {};

    host = new cast.player.api.Host({mediaElement:this._video, url});

    host.onError = (errorCode:string) => {
        console.error('Fatal Error ' + errorCode);
        this._unloadPlayer();
    }

    switch (true) {
        // MPEG-DASH
        case url.lastIndexOf('.mpd') >= 0:
            let kid:string;
            host.protectionSystem = cast.player.api.ContentProtection.WIDEVINE;
            host.licenseUrl = customData.laUrl || 'https://widevine-proxy.drm.technology/proxy';

            host.processManifest = (manifest:string):string => {
                let parser = new DOMParser();
                let xmlDoc = parser.parseFromString(manifest, 'text/xml');
                let el = xmlDoc.querySelector('ContentProtection');
                kid = el ? el.getAttribute('cenc:default_KID') : '';
                return manifest;
            };

            host.updateLicenseRequestInfo = (requestInfo:any) => {
                let token = customData.token;
                let body = JSON.stringify({
                                token: token,
                                drm_info: Array.apply(null, requestInfo.content),
                                kid: kid
                            });

                let buffer:any[] = [];

                for (let i=0; i<body.length; i++) {
                    buffer.push(body.charCodeAt(i));
                }

                requestInfo.headers['Content-Type'] = 'application/json';
                requestInfo.content = new Uint8Array(buffer);
            }

            protocol = cast.player.api.CreateDashStreamingProtocol(host);
            break;

            // HLS
            case url.lastIndexOf('.m3u8') >= 0:
                protocol = cast.player.api.CreateHlsStreamingProtocol(host);
                break;

            // Smooth Streaming
            case url.indexOf('.ism/') >= 0:
                protocol = cast.player.api.CreateSmoothStreamingProtocol(host);
                break;
            }

            if (protocol)
            {
                this._player = new cast.player.api.Player(host);
                this._player.load(protocol, initStart);
            }
            else
            {
                // MP4, MOV, etc
                this._defaultOnLoad(event);
            }
        }
```
