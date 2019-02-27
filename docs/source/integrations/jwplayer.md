# JWPlayer

[JWPlayer](https://jwplayer.com/) is an industry leading HTML5 video player.

The [vuplay-jwplayer](https://github.com/Vualto/vuplay-jwplayer) repository demonstrates at a lower level how to integrate [vudrm](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with JWPlayer.

## Basic set up

```javascript
var vudrmToken = "<vudrm-token>";

// where `player` is the id of a div
jwplayer("player").setup({
  playlist: [
    {
      sources: [
        {
          file: mpegDashStreamUrl,
          drm: {
            playready: playReadyDrmConfig,
            widevine: widevineDrmConfig
          }
        },
        {
          file: hlsStreamUrl,
          drm: {
            fairplay: fairplayDrmConfig
          }
        }
      ]
    }
  ]
});
```

Each `source` object within the `sources` array, has a `drm` property where a drm type specific configuration object can be passed. Examples of each type of configuration are provided below.

## Widevine example

```javascript
var widevineDrmConfig = {
  url: "https://widevine-proxy.drm.technology/proxy",
  licenseRequestFilter: function(request, drmInfo) {
    var keyId = drmInfo.keyIds[0].toUpperCase();
    var body = JSON.stringify({
      token: vudrmToken,
      drm_info: Array.apply(null, new Uint8Array(request.body)),
      kid: keyId
    });
    request.body = body;
    request.headers["Content-Type"] = "application/json";
  }
};
```

## PlayReady example

```javascript
var playReadyDrmConfig = {
  url:
    "https://playready-license.drm.technology/rightsmanager.asmx?token=" +
    encodeURIComponent(vudrmToken)
};
```

## Fairplay example

```javascript
var fairplayDrmConfig = {
  certificateUrl: fairplayCertUrl,
  processSpcUrl: "https://fairplay-license.drm.technology/license",
  extractContentId: extractContentId,
  licenseRequestHeaders: [
    {
      name: "Content-Type",
      value: "application/json"
    }
  ],
  licenseResponseType: "arraybuffer",
  licenseRequestMessage: createLicenseRequestMessage
};

var extractContentId = function(initData) {
  var laurlAsArray = initData.split("/");
  contentId = laurlAsArray[laurlAsArray.length - 1];
  return contentId;
};

var createLicenseRequestMessage = function(keyMessage) {
  var body = {
    token: vudrmToken,
    contentId: contentId,
    payload: base64EncodeUint8Array(keyMessage)
  };
  return JSON.stringify(body);
};

var base64EncodeUint8Array = function(input) {
  var keyStr =
    "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
  var output = "";
  var chr1, chr2, chr3, enc1, enc2, enc3, enc4;
  var i = 0;

  while (i < input.length) {
    chr1 = input[i++];
    chr2 = i < input.length ? input[i++] : Number.NaN;
    chr3 = i < input.length ? input[i++] : Number.NaN;

    enc1 = chr1 >> 2;
    enc2 = ((chr1 & 3) << 4) | (chr2 >> 4);
    enc3 = ((chr2 & 15) << 2) | (chr3 >> 6);
    enc4 = chr3 & 63;

    if (isNaN(chr2)) {
      enc3 = enc4 = 64;
    } else if (isNaN(chr3)) {
      enc4 = 64;
    }
    output +=
      keyStr.charAt(enc1) +
      keyStr.charAt(enc2) +
      keyStr.charAt(enc3) +
      keyStr.charAt(enc4);
  }
  return output;
};
```
