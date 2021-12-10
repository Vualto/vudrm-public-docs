# Legacy

## iOS / tvOS FairPlay SDK

The VUDRMFairPlay SDK is available for iOS and tvOS. This documentation describes the steps to integrate and use the VUDRMFairPlay SDK on these platforms, and how to configure our demo application.

Current release:	iOS v1.0 (254)
					tvOS v1.0 (255)
					
- [Overview](#overview)
- [Requirements](#requirements)
- [Xcode Integration](#xcode-integration)
- [Information about Application Transport Security (ATS)](#information-about-application-transport-security-ats)
- [Preparation](#preparation)
- [Example framework usage](#example-framework-usage)
- [Limitations](#limitations)
- [Known Issues](#known-issues)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

### Overview

VUDRMFairPlay SDK enables Apple’s AVPlayer from AVFoundation to securely request licenses from JW Player's Studio DRM cloud based DRM platform using an extended instance of AssetResourceLoaderDelegate.

The deployment scenario will allow Online Playback / Download and Offline Playback using an associated rental or persist token.

Make use of Apple’s AVPlayer and AVPlayerViewController to present users with the platform default skins, with DRM content. Or create your own video player based on AVPlayer.

This SDK has a number of key benefits:

 - Complete control over the entire AVPlayer lifecycle
 - Numerous optimisations to enable faster playback
 - Bitcode support

The SDK is fully supported in both Objective-C and Swift applications. 

A multiple asset demo application written in Swift, and based on Apple's FairPlay SDK example application, is available on request. Please contact [support@vualto.com](support@vualto.com) to request access.

### Requirements

- Minimum deployment target of iOS 12.0 and tvOS 12.0 or higher
- Xcode 12.2
- Swift 5.3
- Cocoapods

### Xcode Integration
The VUDRMFairPlay SDK is distributed using Cocoapods. Projects set up with Cocoapods use an Xcode workspace, so it is important that projects with Cocoapods are always opened using the workspace file instead of the project file. Our simple demo application demonstrates the required set up for Cocoapods.

If you are integrating the VUDRMFairPlay SDK into your own project, please ensure that the project has been set up correctly for Cocoapods. There are numerous online tutorials demonstrating how to do this. You can then replace or merge the `podfile` with the one in our demo application.

It is important that the latest version of Cocoapods is installed before the demo project or your project can be opened without error.

1. You can ensure that you have the latest version of Cocoapods by opening a new shell in the Terminal application and entering:

	`sudo gem install cocoapods`

2. To pull vudrmFairPlay.framework in to the demo project or your project with Cocoapods, ensure that you quit Xcode if it is running, then open the Terminal application and navigate to the project directory, for example:

	`cd /Users/username/Documents/Dev/vudrm-fairplay-demo`

	Then enter:
	
	`pod install`
	
	or, where the pod has previously been installed:
	
	`pod update`

The project should now be ready to open and run in Xcode.

Please see below for more information on using the demo application.

If you are unable to use Cocoapods in your project, please contact [support@vualto.com](support@vualto.com) to discuss other options.

### Information about application transport security (ATS)

iOS/tvOS 9 introduces Application Transport Security (ATS) which restricts the use of insecure HTTP.
In order to permit playback of content over insecure HTTP exemptions need to be added into your application's Info.plist. For example to disable ATS for any connection you would add the following into your application's Info.plist:

```xml
<dict>
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key> <true/>
    </dict>
</dict>
```

More information about ATS can be found in Apple’s TechNote:
[https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/](https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/)

### Preparation

The demo project is based on Apple's FairPlay SDK example application. This is configured using the projects `Streams.plist` file. 

To provide complete configuration for each `Stream` object in the `Streams.plist`you will require:

 - An `skd://` URI entry in the `content_key_id_list`. The correct URI can be obtained by retrieving the manifest and parsing out the `URI` from the `EXT-X-SESSION-KEY` or `EXT-X-KEY`, this can be done in code (see our demo application for an example) or manually with curl in the Terminal app, e.g. enter: `curl https://eample.cloudfront.net/example-demo/examplecontent/examplecontent.ism/.m3u8`. 
 - The `playlist_url` to correctly prepared content.
 - A `name` for the content. This will be the readable display name and is not used by the SDK. This may differ from the `content_id`.
 - A `content_id` for the content, being the same content ID that the content was prpeared with. This value will always be the last path component of the stream `content_key_id_list` or `skd://` URI entry. Our demo application shows how to retrieve and parse the `content_key_id_list`, however, the `content_id` always needs to be provided for use with offline assets.
 - An `is_protected` boolean value which indicates that the `Stream` object is protected with DRM. This setting would not be needed in scenarios where all content is protected, as all assets can be set as protected by default in the source code. 
 - A `vudrm-token` which must be correctly generated for the type of instance you wish to create. Example Studio DRM token type templates available are:
	- Fairplay Rental `{"type": "r","duration_rental": 3600}`
		- Fairplay Rental tokens may only be used to stream online content to devices running iOS 12 or higher.
	- Fairplay Persist `{"type": "p","duration_persist": 3600}`
 		- Fairplay Persist tokens may be used to stream online content and play offline (downloaded) content to devices running iOS 12 or higher.

For further information about Studio DRM please contact us, or refer to our documentation:
[https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html)

For added convenience, VUDRMFairPlay also bubbles up notifications of errors and progress during the licensing request.

Errors can be intercepted as follows:

    @objc func handleVudrmDidError(_ notification: Notification)
    {
        if let data = notification.userInfo as? [String: String]
        {
            for (function, error) in data
            {
                print("vudrmFairPlay - \(function) reported error \(error)!")
            }
        }
    }
    
And, progress can be monitored using the following:
   
    @objc func handleVudrmProgressUpdate(_ notification: Notification)
    {
        if let data = notification.userInfo as? [String: String]
        {
            for (function, message) in data
            {
                print("vudrmFairPlay - \(function) reported progress \(message).")
            }
        }
    }
        
### Example framework usage
We strongly recommend referring to our iOS / tvOS example multiple asset demo application which uses a `ContentKeyManager`class to initialise an instance of VUDRMFairPlay for each stream, it then attaches the instance to an `AssetResourceLoaderDelegate` which is added to the `AssetResourceLoaderDelegateQueue`. Used with an `AssetPersistenceManager` class, the `ContentKeyManager` can also correctly handle persistence restoration with the `updateResourceLoaderDelegate` function.
	
The ContentKeyManager class should look like this:

	import AVFoundation
 	import vudrmFairPlay
 	
 	class ContentKeyManager {
    
    /// MARK: Types.
    
    /// The singleton for `ContentKeyManager`
    static let shared: ContentKeyManager = ContentKeyManager()
    // MARK: Properties.
    
    /**
     The instance of `AssetResourceLoaderDelegate` which conforms to `AVAssetResourceLoaderDelegate` and is used to respond to content key requests
     from `AVAssetResourceLoader`.
     */
    let assetResourceLoaderDelegate: vudrmFairPlay
    
    /// The DispatchQueue to use for delegate callbacks.
    let assetResourceLoaderDelegateQueue = DispatchQueue(label: "com.vualto.vurdrm.fairplay.AssetResourceLoaderDelegateQueue")
    
    var drm: vudrmFairPlay?
    
	/// MARK: Initialization.
    
    private init() {
        var token = ""
        var contentID = ""
        for stream in StreamListManager.shared.streams {
            if stream.vudrmToken != nil {
                token = stream.vudrmToken!
                contentID = stream.name
                let asset = AVURLAsset(url: URL(string: stream.playlistURL)!)
                self.drm = vudrmFairPlay(asset: asset, contentID: contentID, token: token)!
            }
        }
        assetResourceLoaderDelegate = self.drm!
    }
    
    func updateResourceLoaderDelegate(forAsset asset: AVURLAsset) {
        asset.resourceLoader.setDelegate(self.drm, queue: assetResourceLoaderDelegateQueue)
        }
    }
     
     
###### Online Playback / Download and Offline Playback:

The tvOS platform does not support downloading or offline playback.

Requests for playback or download will result in a license request for the content from the license server, based on the type of token presented for each instance of VUDRMFairPlay. The tokens may be FairPlay Rental or FairPlay Persist. FairPlay Rental token license requests will be provided a streaming content key, which may be used for online streaming only. FairPlay Persist token license requests will be provided a persistent content key, which may be used for both online streaming, and offline playback of downloaded content.

In iOS, when a download of an asset has been completed with a persist token policy, further playback requests will use the downloaded asset and associated content key, even when online.

There are significant limitations using AirPlay to stream any content to an Apple TV that has been downloaded to the users device. Please see the [Limitations](#limitations) section for further information.

To initialise your assets DRM instances, your application should create a DRM instance for each entry in the `Streams.plist` file (or comparable configuration) using a `ContentKeyManager` class. How to do this is demonstrated in our demo application. The `ContentKeyManager` attaches instances of VUDRMFairPlay to an `AssetResourceLoaderDelegate` for each asset using the asset URL, content ID, and valid token. The content ID should be unique to each asset, as it is used along with the `playlist_url` to create a path to, and identify, offline assets and their associated content keys.

When using the same implementation as our demo application, iOS requests for licences for offfine content would reference the framework instance using the`ContentKeyManager`by calling:`ContentKeyManager.shared.assetResourceLoaderDelegate.doRequestPersistableContentKeys(contentKeyIDList: asset.stream.contentKeyIDList!, streamName: asset.stream.contentID, vudrmToken: asset.stream.vudrmToken!, avUrlAsset: avasset)`

###### DEMO

The demo application that we use targets both iOS and tvOS platforms using shared source code. Each target platform references its own version of the framework.

With the VUDRMFairPlay framework installed, the example demo application reads entries in the `Streams.plist` file to configure the content and DRM. You can edit the `Streams.plist` file values, and/or add your own valid values, which should include a `content_id`, `playlist_url`, and `vudrm_token`(see [Preparation](#preparation) above).

- The `content_key_id_list` entry in the `Streams.plist` is retrieved in the demo by parsing the manifest using the `getContentKeyIDList` function in the `AssetListTableViewController`.

- The `name` value is the display name, which may differ from the `content_id`.

- The `is_protected` boolean value indicates whether the content is protected with DRM or not. This value can be overridden in source if all content is protected.

`Stream` class objects are mapped to `Asset` class objects to initialise AVURLAssets using the `ContentKeyManager` class.

In iOS, a call is made upon launch by the `AppDelegate` class to the `AssetPersistenceManager` class to restore any persisted content. The call `AssetPersistenceManager.sharedManager.restorePersistenceManager()` asks the `AssetPersistenceManager` class to check any already mapped asset against any outstanding `AVAssetDownloadTask`.

In both target platforms, the call to initialise content from the `Streams.plist` is then performed by the `ContentKeyManager` class.

In iOS, the call to request an offline license from the framework is:
`ContentKeyManager.shared.assetResourceLoaderDelegate.doRequestPersistableContentKeys(contentKeyIDList: asset.stream.contentKeyIDList!, streamName: asset.stream.contentID, vudrmToken: asset.stream.vudrmToken!, avUrlAsset: avasset)`
and is in the: `override func tableView(_ tableView: UITableView, accessoryButtonTappedForRowWith indexPath: IndexPath)` function of the 
`AssetListTableViewController`. This corresponds to the specific stream Download button.

### Limitations
  
- The SDK does not support AirPlay of protected content after a persist content key has been persisted (content downloaded to device). Apple TV is never able to play protected offline/downloaded content with a persist content key. Attempting to do so may result in unexpected behaviours, crashes referring to the content key type, or the content may not load. Protected content may be transmitted with AirPlay before downloading content (and the content key has been persisted on the users device). Protected content may be transmitted with AirPlay using a streaming content key whenever the Apple TV has a network connection.

- The SDK does not support playback of protected content when running in the iOS Simulator. Attempting to do so will result in an invalid `KEYFORMAT` error and the content will not load.

### Known Issues

- If you believe you have found any issues, please contact us at support@vualto.com

### Troubleshooting

- Most issues are content related. You can use our demo application to test your own content by updating it with your `Stream` configurations. 

- Errors may also arise because the `Stream` configuration is not correct. 
	- Content ID - Please ensure that the `content_id` corresponds to that which was used when preparing your content. The `content_id` should always be unique for each `Stream` instance.
	
	- Tokens - You can easily eliminate token issues by beginning with a default FairPlay token policy. Please ensure your token is formatted correctly and validates. For the avoidance of doubt, where tokens use dates, the dates should always be in the future. For further information about tokens please refer to our token [documentation](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html).
	
If you are not able to play your content after checking it in our demo application please contact support@vualto.com with the demo application logs and the stream configuration used.

### Release notes (iOS / tvOS)

#### v1.0 (build 254/255) on 04/12/2020
- Update to Xcode 12.2, Swift 5.3.1.
- Minor improvements.

#### v1.0 (build 244/212) on 07/10/2020
- Fix issue with iOS simulators.
- Merge iOS and tvOS documentation.

#### v1.0 (build 211) on 02/10/2020
- Update to Xcode 12, iOS 14, Swift 5.3.

#### v1.0 (build 182) on 09/07/2020
- Added error and progress notifications.
- Improved error handling.

#### v1.0 (build 161) on 10/06/2020
- SDK redeveloped to:
	- Resolve issue with persistence restoration after application cache clearance.
	- Enable demonstration of multiple asset DRM based on Apple's HLS Catalog example application.
	- Improve flow and handling using Apple's latest FairPlay SDK methods.

#### v0.0.1 (build 146) on 07/04/2020

- Added retrieval of license server URI from manifest
- Update required for XCode 11.4

#### v0.0.1 (build 131) on 29/01/2020

- Cocoapod distribution integration

#### v0.0.1 (build 105) on 24/09/2019

- Update required for XCode 11 / iOS 13

#### v0.0.1 (build 93) on 29/07/2019

- Update license server DNS
- Replaces instances of assetName and assetID with contentID to be consistent across DRM platform

#### v0.0.1 (build 85) on 27/03/2019

- Update to Swift 5
- Bug fixes and improvements

#### v0.0.1 (build 74) on 21/03/2019

- Fix streaming content key handling
- Bug fixes and improvements

#### v0.0.1 (build 70) on 15/01/2019

- Update Persistable Offline framework to Swift 4.2

#### v0.0.1 (build 65) on 14/12/2018

- Persistable Offline Release

#### v0.0.1 (build 58) on 05/11/2018

- Update framework to Swift 4.2

#### v0.0.1 (build 44) on 09/04/2018

- Bug fixes and improvements

#### v0.0.1 (build 15) on 05/04/2018

- Update framework to Swift 4.1

#### v0.0.1 (build 11) on 27/03/2018

-Bug fixes and improvements

#### v0.0.1 (build 5) on 15/11/2017

- Initial Release
