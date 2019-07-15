# Roku

Roku is a streaming stick.

## Basic setup

Follow the instructions laid out in the [Roku getting started guide](https://developer.roku.com/en-gb/docs/developer-program/getting-started/roku-dev-prog.md) to set up your environment. Roku uses it's own scripting language called BrightScript to set up channels/apps.

We recommend using DASH with PlayReady on Roku devices.

the documentation for which can be found in the [content protection](https://developer.roku.com/en-gb/docs/specs/content-protection.md)

```javascript
customData = "<VUDRM-TOKEN>";
contentNode = createObject("roSGNode", "contentNode")
contentNode.streamFormat = "dash"
contentNode.url = "wwww.myvideo.com/content.mpd"
contentNode.encodingType = "PlayReadyLicenseAcquisitionAndChallenge"
contentNode.encodingKey = "PlayReadyLicenseServerUrl" + "%%%" + customData
```
