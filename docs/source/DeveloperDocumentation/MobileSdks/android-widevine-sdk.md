

# Android Widevine SDK

VUDRMWidevine is an Android Archive (AAR) which can be used during the media rendering pipeline to provide a DRM plugin to ExoPlayer 2.12.2 which will work with Studio DRM workflow. VUDRMWidevine has been developed to specifically manage the session DRM, allowing complete asset and player management.

Current release: v0.3.5 (277)

- [Requirements](#requirements)
- [Android Studio Integration](#android-studio-integration)
- [Example Usage](#example-usage)
- [Error handling](#error-handling)
- [Devices tested on:](#devices-tested-on:)
- [Known Issues](#known-issues)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

## Requirements
- Minimum SDK version is 19 (Android 4.4)
- Android Studio 4.0.1

## Android Studio Integration

1.	VUDRMWidevine is distributed from our maven repository, access to which is made possible by adding the 
following to the repositories closure of your apps top level build.gradle file:

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
2.	Map the username and password to your credentials for authentication to the maven repository

3.	Under your module’s build.gradle dependencies add:

	```java
    implementation 'com.vualto.vudrm:widevine:0.3.5'
	```

4.	Access to our KID plugin is enabled similarly using the same maven repository configuration, but with the url:

	<https://maven.drm.technology/artifactory/kid-plugin>

	and dependency:

	```java 
	implementation 'com.vualto.vudrm:kidplugin:0.3.5'
	```
	
## Example Usage

Instances of VUDRMWidevine are associated with specific assets. Offline assets can be tracked using Exoplayers OfflineLicenseHelper, DownloadService, and DownloadTracker classes. Two types of session are available, determined by the current session's Studio DRM token policy, and the asset configuration.

For further information about Studio DRM please contact us, or refer to our [documentation](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html).

### Online and Offline Streaming Sessions

Exoplayer 12 introduces more 'under the hood' management of DRM assets, which simplifies the flow for online and offline streaming sessions. For an example of implementation we recommend referring to our multiple asset Studio DRM Widevine demo application, which is directly adapted from Exoplayers demo application. To manage the required modification of license calls we have added a simple `VudrmHelper` class to the demo application.
	
Asset configurations should contain at least a stream `name`,`uri`,`drm_scheme`("widevine"), and a `drm_license_url`. Tokens may be presented using either Exoplayers license header `drm_key_request_properties`, or a URL encoded token may instead be added to the license server URI, eg: `"drm_license_uri": "https://widevine-license.staging.vudrm.tech/proxy?token=vualto-token-value"`.


1.	To implement Studio DRM Widevine, instantiate an AssetConfiguration using the fluent interface for both online and offline sessions. Minimally you need to provide the contents KID, DRM token, and license server URI.

	```java
	assetConfiguration = new AssetConfiguration.Builder()
		.tokenWith(drmToken)
		.KIDWith(KID)
		.build();
	```

2. Once you’ve built the object you can construct a plugin by instantiating a WidevineCallback object with the asset configuration.

	```java
	MediaDrmCallback callback = new WidevineCallback (assetConfiguration);
	```

3.	Then you can pass it to ExoPlayer as the component required when creating the DefaultDrmSessionManager object.

	```java
	return new DefaultDrmSessionManager.Builder()
    	.setUuidAndExoMediaDrmProvider(C.WIDEVINE_UUID, FrameworkMediaDrm.DEFAULT_PROVIDER)
	  	.setMultiSession(false)
		.build(mediaDrmCallback);
	```

5.	For offline sessions, Exoplayers DownloadTracker will create a `WidevineOfflineLicenseFetchTask` to acquire and download the license. After this the download will commence. When the download is complete, Exoplayer will manage the asset for playback.

References:
<http://google.github.io/ExoPlayer/doc/reference/>

## Error handling
DRM related errors will be bubbled up to ExoPlayer’s event listener system, this should contain the license server's HTTP response code and a transaction ID in the exception if applicable.


## Devices tested on:
Internally

- Xiaomi Redmi 5 Plus (8.1.0)
- Nexus 10 (5.1.1)
- Samsung S5 (6.0.1)
- Samsung Tab 10 A5 (6.0.1)
- HP 8 Tablet (4.4.2)
- Huawei Honor 9 (8.0.0)

## Known Issues

- Exoplayer 12 for AndroidX introduced a requirement that the PSSH box be present in the main manifest. If the PSSH box is not present in the main manifest then Exoplayers `DefaultDrmSessionManager` will throw this error `MissingSchemeDataException: Media does not support uuid: edef8ba9-79d6-4ace-a3c8-27dcd51d21ed`.

- 32-bit devices displayed issues where a license expiry time (secs) is too large to be handled, it therefore returns 0 seconds remaining and considers the license expired. To work around this, always set the license expiry time in your Studio DRM token policy. For further information about Studio DRM please contact us, or refer to our [documentation](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html).

If you believe you have found any other issue, please contact us at support@vualto.com

## Troubleshooting

- Most issues are content related. You can use our demo application to test your own content by updating the `media.exolist.json` with your configurations.

- Errors may also arise because the stream or asset configuration is not correct. 
	- Tokens - You can easily eliminate token issues by beginning with an empty policy in the Studio DRM token. Please ensure your token is formatted correctly and validates. For the avoidance of doubt, where tokens use dates, the dates should always be in the future. For further information about tokens please refer to our [documentation](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html).

	
If you are not able to play your content after checking it in our demo application please contact support@vualto.com with the demo application logs and the stream configuration used.

## Release Notes

v0.3.5 (build 277) on 07/08/2019

- Update license server DNS
- Replaces instances of assetName and assetID with contentID to be consistent across DRM platform

v0.3.4 (build 272) on 22/05/2019

- Widevine Offline Implementation – Download asset and license, and play downloaded asset offline with saved license.

v0.3.3 (build 243) on 15/05/2019

- Build automation updates

v0.3.2 (build 233) on 17/04/2019

- Update dependencies
- Resolve deprecations
- Update to Android Studio 3.3.2 and SDK 28

v0.3.1 (build 216) on 21/02/2019

- Add Widevine provision callback

v0.3.0 (build 204) on 12/01/2018

- Add Widevine provision callback
- Bug fixes and improvements

v0.2.0 on 12/04/2017

- New major exoplanet release
- Bug fixes and improvements

v0.1.0 on 10/03/2017

- Initial release
