# LG

LG TV's use the WebOS SDK to provide applications to their Smart TV platforms.

## Basic setup

WebOS applications are HTML5, Javascript applications which incorporate a WebOS Javascript library which allows access to TV functionality.

Being a HTML5, Javascript friendly environment there are two options for you when writing an application and integrating with VUDRM.

- Use one of our partner players with existing integration (Bitmovin, JWPlayer, THEOplayer).
- Use the inbuilt `luna` services with a `video` element.

## Recommendations

Below are a list of choices we recommend based on our experience using the LG WebOS platform.

### Player choice

Using a partner player will give your viewers and your dev team the best experience at getting your application up and running as a robust understanding of PlayReady or Widevine is required.

Links to the documentation for integrating VUDRM with our partner players can be found below:

- [Bitmovin](../players/bitmovin.md)
- [JWPlayer](../players/jwplayer.md)
- [THEOplayer](../players/theo-player.md)

If you choose to use the luna services please follow LG's own documentation describing [how to play DRM content](http://webostv.developer.lge.com/develop/app-developer-guide/playing-drm-content/).

### PlayReady XML message retrieval

Within the LG documentation you are shown a snippet of XML you are to provide to the luna DRM service.

```javascript

function getPlayReadyMessage(manifest, token) {

        return '<?xml version="1.0" encoding="utf-8"?>'
        + '<PlayReadyInitiator xmlns= "http://schemas.microsoft.com/DRM/2007/03/protocols/">'
        + '<SetCustomData>'
        + '<CustomData>' + token + '</CustomData>'
        + '</SetCustomData>'
        + '</PlayReadyInitiator>';
    });
}

let XMLMessages = getPlayReadyMessage(manifestString, "VUDRM TOKEN");
```
