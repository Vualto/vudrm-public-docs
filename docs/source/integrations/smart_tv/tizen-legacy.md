# Tizen

Samsung TV's use the Tizen SDK extension for providing applications to their Smart TV platforms.

## Basic setup

Samsung provide extensive documentation showing how to get a development environment up and running, including this [quick start guide](https://developer.samsung.com/tv/develop/getting-started/quick-start-guide).

## DRM support

PlayReady DRM is currently the only DRM supported by Samsung.

## Player Partners support

All of our player partners provide mechanisms to make use of VUDRM, and as Tizen only supports PlayReady, the documentation for each can be followed to implement.  

## Native support

Tizen has an inbuilt player within the SDK which can be used to provide encrypted playback utilising PlayReady DRM.

For further information on Samsungs PlayReady support please visit the [Tizen PlayReady documentation](https://developer.samsung.com/tv/develop/legacy-platform-library/tut00064/index).

Implementing a PlayReady solution can be done with the following code sample.

For Samsungs full documentation on PlayReady support please visit the [Tizen PlayRead documentation](https://developer.samsung.com/tv/develop/legacy-platform-library/tut00064/index).

```javascript
avplayObj.show();
avplayObj.open(
    url,
    {
        drm : {
            type : "Playready",
            company : 'Microsoft Corporation',
            deviceID : '1'
        }
    }
);
avplayObj.play(
    self.playSuccess,
    function (error) {
        console.error(error.message);
    },
    0
);

let customData = "<YOUR VUDRM TOKEN>"
avplayObj.setPlayerProperty(3, customData, customData.length);

let laUrl ="https://playready-license.drm.technology/rightsmanager.asmx";
avplayObj.setPlayerProperty(4, laUrl, laUrl.length);

SefPluginPlayReady = document.getElementById('PluginSefPlayReady');
SefPluginPlayReady.Open("PlayReadyDrm",  "1.000",  "PlayReadyDrm");
SefPluginPlayReady.Execute("ProcessInitiatorsFromUrl",  laUrl);
SefPluginPlayReady.Close();
```
