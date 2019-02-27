# THEOplayer

[THEOplayer](https://www.theoplayer.com) is an industry leading HTML5 video player.

The [vuplay-theoplayer](https://github.com/vualto/vuplay-theoplayer) repository demonstrates at a lower level how to integrate [vudrm](https://docs.vualto.com/projects/vudrm/en/latest/index.html) with THEOplayer.

## Basic setup

```html
<div
  id="vuplay-container"
  class="video-js theoplayer-skin theo-seekbar-above-controls"
></div>
```

```javascript
var streamUrl = "<your-stream-url>";
var vudrmToken = "<your-vudrm-token>";
var containerElement = document.getElementById("vuplay-container");

var player = new THEOplayer.Player(containerElement, {
  libraryLocation: "<your-theoplayerjs-scripts-path>",
  ui: {
    fluid: true
  }
});
```

## Widevine example

```javascript
player.source = {
  sources: [
    {
      src: streamUrl,
      type: "application/dash+xml",
      contentProtection: {
        integration: "vudrm",
        token: vudrmToken,
        widevine: {
          licenseAcquisitionURL: "https://widevine-proxy.drm.technology/proxy"
        }
      }
    }
  ]
};
```

## PlayReady example

```javascript
player.source = {
  sources: [
    {
      src: streamUrl,
      type: "application/dash+xml",
      contentProtection: {
        integration: "vudrm",
        token: vudrmToken,
        playready: {
          licenseAcquisitionURL:
            "https://playready-license.drm.technology/rightsmanager.asmx"
        }
      }
    }
  ]
};
```

## FairPlay example

```javascript
player.source = {
  sources: [
    {
      src: streamUrl,
      type: "application/x-mpegurl",
      contentProtection: {
        integration: "vudrm",
        token: vudrmToken
      }
    }
  ]
};
```
