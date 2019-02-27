# Bitmovin

[Bitmovin](https://bitmovin.com) is an industry leading HTML5 video player.

The [vuplay-bitmovin](https://github.com/Vualto/vuplay-bitmovin) repository demonstrates at a lower level how to integrate [vudrm](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with Bitmovin.

## Basic set up

```javascript
var container = document.getElementById("my-player");
var VUDRM_TOKEN = "<your-vudrm-token>";

var playerConfig = {
  key: "<your-bitmovin-player-key>"
};

var source = {
  title: "Getting Started with the Bitmovin Player",
  description:
    "Now you are ready to embed the Bitmovin Player into your own website :)",
  dash: "<your-mpeg-dash-stream-url>",
  hls: "<your-hls-stream-url>",
  poster: "https://bitmovin-a.akamaihd.net/content/MI201109210084_1/poster.jpg"
};
```

The source object above has a `drm` property, within this you can add the appropriate DRM configuration.

## Widevine example

```javascript
source.drm = {
  widevine: {
    LA_URL: "https://widevine-proxy.drm.technology/proxy",
    prepareMessage: function(keyMessage) {
      return JSON.stringify({
        token: VUDRM_TOKEN,
        drm_info: Array.apply(null, new Uint8Array(keyMessage.message)),
        kid: "<CONTENT-KEY-ID>"
      });
    }
  }
};
```

## PlayReady example

```javascript
source.drm = {
    playready: {
        LA_URL: `https://playready-license.drm.technology/rightsmanager.asmx?token=" + encodeURIComponent(VUDRM_TOKEN);
    }
}
```

## Fairplay example

```javascript
source.drm = {
  fairplay: {
    certificateURL: "<YOUR-FAIRPLAY-CERT>",
    LA_URL: "<FAIRPLAY-LICENSE-SERVER-URL>",
    certificateHeaders: {
      "x-vudrm-token": [VUDRM_TOKEN]
    },
    headers: {
      "Content-Type": "application/json"
    },
    prepareMessage: function(keyMessageEvent, keySession) {
      return JSON.stringify({
        token: VUDRM_TOKEN,
        contentId: keySession.contentId,
        payload: keyMessageEvent.messageBase64Encoded
      });
    },
    prepareContentId: function(rawContentId) {
      var tmp = rawContentId.split("/");
      return tmp[tmp.length - 1];
    },
    prepareCertificate: function(cert) {
      return new Uint8Array(cert);
    },
    prepareLicense: function(license) {
      return new Uint8Array(license);
    },
    licenseResponseType: "arraybuffer"
  }
};
```
