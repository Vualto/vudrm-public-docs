# Tizen

Samsung TV's use the Tizen SDK extension for providing applications to their Smart TV platforms.

## Basic setup

Samsung provide extensive documentation showing how to get a development environment up and running, including this [quick start guide](https://developer.samsung.com/smarttv/develop/getting-started/quick-start-guide.html).

## DRM support

PlayReady DRM is currently the only DRM supported by Samsung.

## Player Partners support

All of our player partners provide mechanisms to make use of VUDRM, and as Tizen only supports PlayReady, the documentation for each can be followed to implement.  

## Native support

Tizen has an inbuilt player within the SDK which can be used to provide encrypted playback utilising PlayReady DRM.

For further information on Samsungs PlayReady support please visit the [Tizen PlayReady documentation](https://developer.samsung.com/smarttv/develop/guides/multimedia/media-playback/using-avplay.html#drm-contents-playback-sequence).

Implementing a PlayReady solution can be done with the following [demo repo](https://github.com/Vualto/vudrm-tizen).

```javascript
// create listeners for your player
const listeners = {};
const source = "<YOUR DASH STREAM>";
const drmParam = {
    DeleteLicenseAfterUse: true,
    LicenseServer: "https://playready-license.vudrm.tech/rightsmanager.asmx",
    CustomData: "<YOUR_VUDRM_TOKEN>"
};

webapis.avplay.open(source);
webapis.avplay.setDisplayRect(0, 0, 1920, 1080);
webapis.avplay.setListener(listeners);
webapis.avplay.setDrm("PLAYREADY", "SetProperties", JSON.stringify(drmParam));
webapis.avplay.prepareAsync(function () {
    // do something
});
```
