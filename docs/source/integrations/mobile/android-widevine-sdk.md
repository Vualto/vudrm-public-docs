

# vudrmWidevine SDK Documentation

vudrmWidevine is an Android Archive (AAR) which can be used to provide a DRM plugin to ExoPlayer 2.2.0 during the media rendering pipeline to work with Vualto DRM workflow.


Current release: v0.1.1 (87)


- Requirements
- Android Studio Integration
- Example Usage
- Error handling
- Devices tested on:
- Known Issues
- Release Notes



## Requirements
- Minimum SDK version is 19 (Android 4.4)
- Android Studio 2.3

## Android Studio Integration
1.	To your project's top level build.gradle at the following to your repositories closure:
```java
allprojects {
    repositories {
        jcenter()
        maven {
            url "https://maven.drm.technology/artifactory/vudrm-widevine"
            credentials {
                username = "mavenUsername"
                password = "mavenPassword"
            }
        }
    }
}		
```
2.	Add your credentials for authentication to the maven repository

3.	Under your module’s build.gradle dependencies add:
___
```java
 compile 'com.vualto.vudrm:widevine:0.2.0'
```

## Example Usage
1.	Instantiate an AssetConfiguration using the fluent interface, minimally you need to provide the content’s KID and DRM token, the default license server in this case will be ‘<https://widevine-proxy.drm.technology/proxy>’. This can also be overridden using the API call ‘licenceUrlWith(<Your license URL>)’.
```java
try {
    assetConfiguration = new AssetConfiguration.Builder()
            .tokenWith(drmToken)
            .KIDWith(KID)
            .build();
} catch (Exception e) {
    // Handle exception
}
```
2. Once you’ve built the object you can construct a plugin by instantiating a WidevineCallback object with the asset configuration.

   WidevineCallback callback = new WidevineCallback(assetConfiguration)

3. Then you can pass it to ExoPlayer as the component required when creating the DrmSessionManager<FrameworkMediaCrypto> object.
```java
try {
    return new DefaultDrmSessionManager<>(vudrm.widevineDRMSchemeUUID,
            FrameworkMediaDrm.newInstance(vudrm.widevineDRMSchemeUUID),
            callback,
            null,
            new Handler(),
            null);
} catch (UnsupportedDrmException e) {
    e.printStackTrace();
    return null;
}
```
References:
<http://google.github.io/ExoPlayer/doc/reference/>

## Error handling
DRM related errors will be bubbled up to ExoPlayer’s event listener system, this should contain the license server's HTTP response code and a transaction ID in the exception.


## Devices tested on:
Internally

- Nexus 6P (7.0.1)
- Samsung Tab 10 A5 (6.0.1)
- Nexus 7 [2013] (6.0.1)
- Nexus 7 [2012] (5.1.1)
- Samsung Galaxy Tab 4 (4.4.2)
- Asus 10 MeMoPad (4.2.2)
- Asus 7 MeMoPad (4.4.2)
- Nexus 4 (5.0.1)
- Samsung S5 (5.0.2)

## Known Issues
- No known issues at this time. If you believe you have found an issue, please contact us at support@vualto.com

## Release Notes
v0.1.1 (build 87) on 20/03/2017




