# Roku

 The Roku OS was purpose-built for streaming and runs across all Roku devices, including streaming players and Roku TVs. On the Roku platform the applications that stream your media are called channels.

## Basic setup

Follow the getting started instructions laid out in the [Roku documentation](https://developer.roku.com/en-gb/overview) to set up your environment. Roku uses it's own scripting language called BrightScript to set up channels/apps.

We recommend using DASH with PlayReady on Roku devices.

The documentation for which can be found in the [content protection](https://developer.roku.com/en-gb/docs/specs/content-protection.md).

```javascript

customData = { "<VUDRM-TOKEN>" };
contentNode = createObject("roSGNode", "contentNode")
contentNode.streamFormat = "dash"
contentNode.url = "<MPEG-DASH-URL>"
contentNode.encodingType = "PlayReadyLicenseAcquisitionAndChallenge"
contentNode.encodingKey = "PlayReadyLicenseServerUrl" + "%%%" + customData
```

## Testing your stream

You can also test your streams outside of the Roku app using the [Stream testing tool](https://developer.roku.com/en-gb/docs/developer-program/dev-tools/tools-overview.md#stream-testing-tool).

- Add your device `IP` and `Model` to the device manager.
- Ensure that `Mode` is set to `Video`.
- Ensure that `Format` is set to either `Auto` or `DASH`.
- Within the DRM section choose `encoding type` : `PlayReady: LA And Challenge`.
- Put your newly generated VUDRM-token in to the custom data input box.
- In `License server Url(optional)` put `https://playready-license.vudrm.tech/rightsmanager.asmx`.
- Click `Play`.
