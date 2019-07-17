# Chromecast

Chromecast is a casting device developed by Google, to enable streaming from web apps upon large screen display devices.

Google have extensive [documentation](https://developers.google.com/cast/docs/developers) about their SDK. If you have never developed a Chromecast application before we would strongly advise you start here.

## Basic setup

In order to stream DRM protected content on a Chromecast using Googles' web SDK you will need to create a Custom Receiver app. The app will utilise two parts a `Sender` and a `Receiver`.

Typically the `Sender` would reside either as part of the web page or your chosen player.

The `Receiver` is an application that you host and register with Google.

Below is an example of basic `Sender` & `Receiver` side code using the Google SDK.  

## Sender

Setup of the load function within your `sender` within your site or player.

```javascript
load(url, token, laUrl) {
    if (!this.isConnected) { return };
    let mediaInfo, request;
    mediaInfo = new chrome.cast.media.MediaInfo(url);
    mediaInfo.metadata = new chrome.cast.media.GenericMediaMetadata();
    mediaInfo.metadata.metadataType = chrome.cast.media.MetadataType.GENERIC;
    mediaInfo.metadata.title = 'VUDRM Demo';
    mediaInfo.customData = { laUrl, token };

    request = new chrome.cast.media.LoadRequest(mediaInfo);
    request.autoplay = true;
    request.currentTime = 0;
    this._castContext.getCurrentSession().loadMedia(request).catch((error) => {
        console.error(error);
    });
}
```

## Receiver

Setup of the host within your `receiver` application.

```javascript
let videoElement = document.getElementById("your-video-element");
if (event.data.media && event.data.media.contentId){
    let url = event.data.media.contentId;
    let initStart = event.data.media.currentTime || 0;
    let autoplay = !!event.data.autoplay;
    let customData = event.data.media.customData || {};

    let host = new cast.player.api.Host({mediaElement:videoElement, url});
    let protocol = cast.player.api.CreateDashStreamingProtocol(host);
    let player = new cast.player.api.Player(host);
    player.load(protocol, initStart);
    host.onError = (errorCode) => {
        console.error('Fatal Error ' + errorCode);
        if (player) {
            player.unload();
            player = null;
        }
    }
```

### MPEG-DASH overrides

```javascript
let kid;
host.protectionSystem = cast.player.api.ContentProtection.WIDEVINE;
host.licenseUrl = 'https://widevine-proxy.drm.technology/proxy';

host.processManifest = (manifest) => {
    let parser = new DOMParser();
    let xmlDoc = parser.parseFromString(manifest, 'text/xml');
    let el = xmlDoc.querySelector('ContentProtection');
    kid = el ? el.getAttribute('cenc:default_KID') : '';
    return manifest;
};

host.updateLicenseRequestInfo = (requestInfo) => {
    // customData is the message object that was sent to the receiver.
    let token = customData.token;
    let body = JSON.stringify({
        token: token,
        drm_info: Array.apply(null, requestInfo.content),
        kid: kid
    });

    let buffer = [];

    for (let i=0; i<body.length; i++) {
        buffer.push(body.charCodeAt(i));
    }

    requestInfo.headers['Content-Type'] = 'application/json';
    requestInfo.content = new Uint8Array(buffer);
}

protocol = cast.player.api.CreateDashStreamingProtocol(host);
```
