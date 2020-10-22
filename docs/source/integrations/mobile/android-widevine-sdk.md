

# Android Widevine SDK

VUDRMWidevine is an Android Archive (AAR) which can be used during the media rendering pipeline to provide a DRM plugin to ExoPlayer 2.9.6 which will work with Vualto DRM workflow. VUDRMWidevine has been developed to specifically manage the session DRM, allowing complete asset and player management.

Current release: v0.3.5 (277)


- Requirements
- Android Studio Integration
- Example Usage
- Error handling
- Devices tested on:
- Known Issues
- Release Notes



## Requirements
- Minimum SDK version is 19 (Android 4.4)
- Android Studio 3.3.2

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
2.	Change the username and password to your credentials for authentication to the maven repository

3.	Under your module’s build.gradle dependencies add:

	```java
    implementation 'com.vualto.vudrm:widevine:0.3.4'
	```

4.	Access to our KID plugin is enabled similarly using the same maven repository configuration, but with the url:

	<https://maven.drm.technology/artifactory/kid-plugin>

	and dependency:

	```java 
	implementation 'com.vualto.vudrm:kidplugin:0.3.4'
	```
	
## Example Usage

Instances of VUDRMWidevine are associated with specific assets. Offline assets can be tracked using Exoplayers OfflineLicenseHelper, DownloadAction, DownloadService, and DownloadTracker classes. Two types of session are available, determined by the session VUDRM token policy, and the asset configuration.

For further information about VUDRM please contact us, or refer to our documentation:
<https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html>

### Online Streaming Sessions

1.	Instantiate an AssetConfiguration using the fluent interface, minimally you need to provide the contents KID and DRM token, the default license server in this case will be ‘<https://widevine-license.vudrm.tech/proxy>’. 

	The default license server can also be overridden using the API call ‘licenceUrlWith(<Your license URL>)’. An Asset ID is required only for offline streaming sessions storage and management.

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

	```
	WidevineCallback callback = new WidevineCallback(assetConfiguration);
	```

3.	Then you can pass it to ExoPlayer as the component required when creating the DefaultDrmSessionManager<FrameworkMediaCrypto> object.

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


### Offline Streaming Sessions

Offline sessions require a method of management that will enable the storage and recall of offline assets, with an associated content URL, and the contents offline license. 

The management system will ensure that when offline the offline asset will not need to be loaded from the file path where it was stored on device, instead the system will call the original online URL, and the stored offline asset will load with the correct license. As demonstrated in our example app, Exoplayer offers us classes that act as such a management system.

It is important to consider that while online, VUDRMWidevine handles generating the Widevine license only for the purposes of online playback or acquisition of an offline license. Offline licenses are persisted as standard Widevine licenses which means that the asset configuration (using ```OfflineAssetConfiguration```) required for offline playback is constructed with minor differences to the online configuration (using ```AssetConfiguration```).

1.	Initially identical to online sessions, start by instantiating an (online) AssetConfiguration using the fluent interface. This configuration is used to acquire the offline license for later use when offline.

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

2.	Once you’ve built the object you can construct a plugin by instantiating a WidevineCallback object with the asset configuration.

		
		WidevineCallback callback = new WidevineCallback(assetConfiguration);

3.	When creating an ExoPlayer OfflineLicenseHelper you can pass this callback as the component required when creating the DefaultDrmSessionManager<FrameworkMediaCrypto> object. 

	The OfflineLicenseHelper requires DrmInfo as ```drmInitData``` which can be collated using ExoPlayers DashUtil class. 

4.	An offline license can then be acquired using Exoplayers OfflineLicenseHelper call:

	```
	byte[] offlineLicenseKeySetId = mOfflineLicenseHelper.downloadLicense(drmInitData);
	```

	The ```offlineLicenseKeySetId``` is the key binder between the offline asset and the associated offline license.

5.	Depending on the chosen set up, when the license has been acquired, the user may then download the asset to their device by calling the Exoplayer DownloadTracker with the content URL. This call will automatically create an assoicated Exoplayer DownloadAction and Exoplayer DownloadService for the given asset. Use these classes with the Exoplayer DownloadTracker to manage the assets and their current status. The DownloadTracker will detect whether the asset for the URL already exists.

	When the download is complete, offline playback can then be achieved by calling the asset using a VUDRM OfflineAssetConfiguration and a standard Widevine DefaultDrmSessionManager<FrameworkMediaCrypto> object using an HttpMediaDrmCallback that can be passed to the player.

	```java
	try {
		OfflineAssetConfiguration assetConfiguration = new OfflineAssetConfiguration.Builder()
			.kidProviderWith(
				new HttpKidSource(
					new URL(streamUrl)
					)
			)
		.build();
		} catch (Exception e) {
			// Handle exception
		}
	```


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

- 32-bit devices displayed issues where a license expiry time (secs) is too large to be handled, it therefore returns 0 seconds remaining and considers the license expired. To work around this, always set the license expiry time in your VUDRM token policy. For further information about VUDRM please contact us, or refer to our documentation:
<https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html>

If you believe you have found any other issue, please contact us at support@vualto.com

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