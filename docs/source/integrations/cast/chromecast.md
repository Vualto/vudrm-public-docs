# Chromecast

To support VUDRM through google cast, you need to implement your own receiver app (see [documentation](https://developers.google.com/cast/docs/caf_receiver/))

## Sender app

The sender app needs to provide the `VUDRM token` to the receiver app.

```javascript
var mediaInfo = new chrome.cast.media.MediaInfo(url);
mediaInfo.customData =
{
    token: "<your-vudrm-token>"
};
```

## Receiver app

### Playback configuration

```javascript
var kid;

var playbackConfig = new cast.framework.PlaybackConfig();

playbackConfig.protectionSystem = cast.framework.ContentProtection.WIDEVINE
playbackConfig.licenseUrl = "https://widevine-proxy.drm.technology/proxy";

playbackConfig.licenseRequestHandler = function licenseRequestHandler(requestInfo) {
    requestInfo.headers["Content-Type"] = "application/json";

    var body = generateBody(requestInfo.content);
    requestInfo.content = generateRequestContent(body);
};

playbackConfig.licenseHandler = function licenseHandler(requestInfo) {
    // Not sure that we need it?
    // https://developers.google.com/cast/docs/reference/caf_receiver/cast.framework.PlaybackConfig#licenseHandler
};

playbackConfig.manifestHandler = function manifestHandler(manifest) {
    // This function set the kid (!side effect!), but it can be passed through the sender app
    var decoded = _UTF8toUTF16LE(manifest);

    var parser = new DOMParser();
    var xmlDoc = parser.parseFromString(decoded, "text/xml");
    kid = xmlDoc.querySelector("ContentProtection").getAttribute("cenc:default_KID");

    return _UTF16toUTF8(decoded);
};

function generateRequestContent(body) {
    var buffer = [];

    for (var i=0; i < body.length; i++) {
        buffer.push(body.charCodeAt(i));
    }

    return new Uint8Array(buffer);
}

function generateBody(requestInfoContent) {
    var context = cast.framework.CastReceiverContext.getInstance();
    var mediaInfo = context.getPlayerManager().getMediaInformation();
    var customData = mediaInfo.customData || {};

    return JSON.stringify({
        token: customData.token,
        drm_info: Array.apply(null, requestInfoContent),
        kid: kid
    })
}

function _UTF16toUTF8(text){
    var cp = [];

    for (var i=0; i<text.length; i+=2) {
        cp.push(text.charCodeAt(i) | ( text.charCodeAt(i+1) << 8 ));
    }

    return String.fromCharCode.apply( String, cp );
}

/*
*   !! little endian conversion !!
*/
function _UTF8toUTF16LE(text){
    var byteArray = new Uint8Array(text.length * 2);

    for (var i = 0; i < text.length; i++) {
        byteArray[i*2] = text.charCodeAt(i); // & 0xff;
        byteArray[i*2+1] = text.charCodeAt(i) >> 8; // & 0xff;
    }

    return String.fromCharCode.apply( String, byteArray );
}
```

<!-- 
```javascript
// Need to check if needed
function stringToUint16Array(string) {
    var buffer = new ArrayBuffer(string.length * 2); // 2 bytes for each char
    var array = new Uint16Array(buffer);
    for (var i = 0, strLen = string.length; i < strLen; i++) {
        array[i] = string.charCodeAt(i);
    }
    return array;
};

function uint16ArrayToString (array) {
    var uint16Array = new Uint16Array(array.buffer);
    return String.fromCharCode.apply(null, uint16Array);
};

function base64EncodeUint8Array (input) {
    var keyStr = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=";
    var output = "";
    var chr1, chr2, chr3, enc1, enc2, enc3, enc4;
    var i = 0;

    while (i < input.length) {
        chr1 = input[i++];
        chr2 = i < input.length ? input[i++] : Number.NaN; // Not sure if the index
        chr3 = i < input.length ? input[i++] : Number.NaN; // checks are needed here

        enc1 = chr1 >> 2;
        enc2 = ((chr1 & 3) << 4) | (chr2 >> 4);
        enc3 = ((chr2 & 15) << 2) | (chr3 >> 6);
        enc4 = chr3 & 63;

        if (isNaN(chr2)) {
            enc3 = enc4 = 64;
        } else if (isNaN(chr3)) {
            enc4 = 64;
        }
        output += keyStr.charAt(enc1) + keyStr.charAt(enc2) +
            keyStr.charAt(enc3) + keyStr.charAt(enc4);
    }
    return output;
};

function base64DecodeUint8Array (input) {
    var raw = window.atob(input);
    var rawLength = raw.length;
    var array = new Uint8Array(new ArrayBuffer(rawLength));

    for (var i = 0; i < rawLength; i++)
        array[i] = raw.charCodeAt(i);

    return array;
}
```
-->