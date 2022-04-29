# dash.js

[dash.js](https://github.com/Dash-Industry-Forum/dash.js/wiki) is an initiative of the DASH Industry Forum to establish a production quality framework for building video and audio players that play back MPEG-DASH content using client-side JavaScript libraries leveraging the Media Source Extensions API set as defined by the W3C.

The [vuplay-dashjs](https://github.com/Vualto/vuplay-dashjs) repository demonstrates at a lower level how to integrate [Studio DRM](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with dash.js.

## Basic setup

```javascript
var streamUrl = "<your-stream-url>";
var vudrmToken = "<your-vudrm-token>";

var player = dashjs.MediaPlayer().create();
```

## Widevine example

```javascript
var overrideKeySystemWidevine = function() {
  return {
    getInitData: function(cpData, kid) {
      this.kid = kid;
      if ("pssh" in cpData) {
        return BASE64.decodeArray(cpData.pssh.__text).buffer;
      }
      return null;
    },

    getLicenseRequestFromMessage: function(message) {
      var body = {
        token: vudrmToken,
        drm_info: Array.apply(null, new Uint8Array(message)),
        kid: this.kid
      };
      body = JSON.stringify(body);
      return body;
    }
  };
};

var overrideProtectionKeyController = function() {
  var parent = this.parent;

  return {
    getSupportedKeySystemsFromContentProtection: function(cps) {
      var cp, ks, ksIdx, cpIdx;
      var supportedKS = [];
      var keySystems = parent.getKeySystems();
      var cpsWithKeyId = cps.find(function(element) {
        return element.KID !== null;
      });

      if (cps) {
        for (ksIdx = 0; ksIdx < keySystems.length; ++ksIdx) {
          ks = keySystems[ksIdx];
          for (cpIdx = 0; cpIdx < cps.length; ++cpIdx) {
            cp = cps[cpIdx];
            if (cp.schemeIdUri.toLowerCase() === ks.schemeIdURI) {
              var initData = ks.getInitData(cp, cpsWithKeyId.KID);
              if (initData) {
                supportedKS.push({
                  ks: keySystems[ksIdx],
                  initData: initData
                });
              }
            }
          }
        }
      }
      return supportedKS;
    }
  };
};

var widevineLaUrl = "https://widevine-license.vudrm.tech/proxy";

player.setProtectionData({
  "com.widevine.alpha": {
    serverURL: widevineLaUrl,
    httpRequestHeaders: {}
  }
});

player.attachSource(streamUrl);
```

## PlayReady example

```javascript
var playReadyLaUrl =
  "https://playready-license.vudrm.tech/rightsmanager.asmx?token=" +
  encodeURIComponent(vudrmToken);

player.setProtectionData({
  "com.microsoft.playready": {
    serverURL: playReadyLaUrl,
    httpRequestHeaders: {}
  }
});

player.attachSource(streamUrl);
```

## No FairPlay support

dash.js does not currently support FairPlay.
