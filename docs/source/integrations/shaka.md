# Shaka

[Shaka](https://shaka-player-demo.appspot.com/docs/api/tutorial-welcome.html) Player is a JavaScript library for adaptive video streaming. It plays DASH content without browser plugins using MediaSource Extensions and Encrypted Media Extensions.

The [vuplay-shaka](https://github.com/Vualto/vuplay-shaka) repository demonstrates at a lower level how to integrate [vudrm](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with Googles' Shaka player.

## Basic setup

```html
<div id="video"></div>
```

```javascript
var mpegdashStreamUrl = "<your-stream-url>";
var vudrmToken = "<your-vudrm-token>";
var shakaPlayer;

shaka.polyfill.installAll();
if (shaka.Player.isBrowserSupported()) {
  var video = document.getElementById("video");
  shakaPlayer = new shaka.Player(video);
  shakaPlayer.addEventListener("error", onErrorEvent);
}
```

## Widevine example

```javascript
shakaPlayer.configure({
  drm: {
    servers: {
      "com.widevine.alpha": "https://widevine-proxy.drm.technology/proxy"
    }
  }
});

shakaPlayer
  .getNetworkingEngine()
  .registerRequestFilter(function(type, request) {
    if (type != shaka.net.NetworkingEngine.RequestType.LICENSE) return;

    var selectedDrmInfo = shakaPlayer.drmInfo();
    if (selectedDrmInfo.keySystem !== "com.widevine.alpha") {
      return;
    }

    var keyId = selectedDrmInfo.keyIds[0].toUpperCase();

    var body = {
      token: vudrmToken,
      drm_info: Array.apply(null, new Uint8Array(request.body)),
      kid: keyId
    };
    body = JSON.stringify(body);
    request.body = body;
    request.headers["Content-Type"] = "application/json";
  });

shakaPlayer
  .load(mpegdashStreamUrl)
  .then(function() {
    console.log("The stream has now been loaded!");
  })
  .catch(onError);
```

## PlayReady example

```javascript
var playReadyLaURL =
  "https://playready-license.drm.technology/rightsmanager.asmx?token=" +
  encodeURIComponent(vudrmToken);
shakaPlayer.configure({
  drm: {
    servers: {
      "com.microsoft.playready": playReadyLaURL
    }
  }
});

shakaPlayer
  .load(mpegdashStreamUrl)
  .then(function() {
    console.log("The stream has now been loaded!");
  })
  .catch(onError);
```

## No FairPlay support

Shaka does not currently support FairPlay.
