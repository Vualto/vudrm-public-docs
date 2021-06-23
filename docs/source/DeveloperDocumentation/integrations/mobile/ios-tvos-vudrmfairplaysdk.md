# iOS / tvOS FairPlay SDK

VUDRMFairPlaySDK is available for iOS and tvOS. This documentation describes the steps to integrate and use VUDRMFairPlaySDK on these platforms, and how to configure our demo applications.

Current release:	iOS v1.0 (22)
					tvOS v1.0 (23)
					
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

## Overview

VUDRMFairPlaySDK enables Apple’s AVPlayer from AVFoundation to securely request licenses from Vualto’s VUDRM cloud based DRM platform.

The deployment scenario will allow Online Playback / Download and Offline Playback using an associated rental or persist token.

Make use of Apple’s AVPlayer and AVPlayerViewController to present users with the platform default skins, with DRM content, or create your own video player based on AVPlayer.

This SDK has a number of key benefits:

 - Complete control over the entire AVPlayer lifecycle
 - Numerous optimisations to enable faster playback
 - Bitcode support

The SDK is fully supported in both Objective-C and Swift applications. 

Multiple asset demo applications written in Swift, and based on Apple's FairPlay SDK example applications, are available on request. Please contact [support@vualto.com](support@vualto.com) to request access.

## Requirements

- Minimum deployment target of iOS 10.0 and tvOS 9.0 or higher
- Xcode 12.4
- Swift 5.3
- Cocoapods

## Xcode Integration
VUDRMFairPlaySDK is distributed using Cocoapods. Projects set up with Cocoapods use an Xcode workspace, so it is important that projects with Cocoapods are always opened using the workspace file instead of the project file. Our simple demo applications demonstrate the required set up for Cocoapods.

If you are integrating VUDRMFairPlaySDK into your own project, please ensure that the project has been set up correctly for Cocoapods. There are numerous online tutorials demonstrating how to do this. You can then replace or merge the `podfile` with the one in our demo application.

It is important that the latest version of Cocoapods is installed before the demo project or your project can be opened without error.

1. You can ensure that you have the latest version of Cocoapods by opening a new shell in the Terminal application and entering:

	`sudo gem install cocoapods`

2. To pull vudrmFairPlaySDK.framework in to the demo project or your project with Cocoapods, ensure that you quit Xcode if it is running, then open the Terminal application and navigate to the project directory, for example:

	`cd /Users/username/Documents/Dev/vudrm-fairplay-demo`

	Then enter:
	
	`pod install`
	
	or, where the pod has previously been installed:
	
	`pod update`

	If you have installed our earlier SDK's and receive this CocoaPod error: `[!] Unable to find a specification for 'vudrmFairPlaySDK'`

	you may also need to enter the following:

	`pod install --repo-update`

The project should now be ready to open and run in Xcode.

Please see below for more information on using the demo application.

If you are unable to use Cocoapods in your project, please contact [support@vualto.com](support@vualto.com) to discuss other options.

## Information about application transport security (ATS)

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

## Preparation

Instances of VUDRMFairPlaySDK require some configuration which would normally be populated with an API. Our demo projects, which are based on Apple's FairPlay SDK example applications, are configured using the projects `Streams.plist` file.

To provide complete configuration for each `Stream` object you will require:

 - An `skd://` URI entry in the `content_key_id_list`. The correct URI can be obtained by retrieving the manifest and parsing out the `URI` from the `EXT-X-SESSION-KEY` or `EXT-X-KEY`, this can be done in code (see our demo application for an example) or manually with curl in the Terminal app, e.g. enter: `curl https://eample.cloudfront.net/example-demo/examplecontent/examplecontent.ism/.m3u8`. 
 - The `playlist_url` to correctly prepared content.
 - A `name` for the content. This will be the readable display name and is not used by the SDK. This may differ from the `content_id`.
 - A `content_id` for the content, being the same content ID that the content was prepared with. This value will always be the last path component of the stream `content_key_id_list` or `skd://` URI entry. Our demo application shows how to retrieve and parse the `content_key_id_list`, however, the `content_id` always needs to be provided for use with offline assets.
 - An `is_protected` boolean value which indicates that the `Stream` object is protected with DRM. This setting would not be needed in scenarios where all content is protected, as all assets can be set as protected by default in the source code. 
 - A `vudrm-token` which must be correctly generated for the type of instance you wish to create. Please see the [Tokens and Licensing](#tokens-and-licensing) section for further information.

For added convenience, VUDRMFairPlaySDK also bubbles up notifications of errors and progress during the licensing request. Errors can be intercepted as follows:

    @objc func handleVudrmDidError(_ notification: Notification)
    {
        if let data = notification.userInfo as? [String: String]
        {
            for (function, error) in data
            {
                print("vudrmFairPlaySDK - \(function) reported error \(error)!")
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
                print("vudrmFairPlaySDK - \(function) reported progress \(message).")
            }
        }
    }
        
## Example framework usage

We strongly recommend referring to our example iOS / tvOS multiple asset demo application.

Using either of Apples recommended implentations, being AssetResourceLoaderDelegate or ContentKeyDelegate, VUDRMFairPlaySDK performs the two required licensing network tasks, being acquisition of an application certificate, followed by a FairPlay license either for online or offline use.

An application certificate is always required to initialise a request for a FairPlay license. 

To invoke an instance of vudrmFairPlaySDK and call for an application certificate using either AssetResourceLoaderDelegate or ContentKeyDelegate, use:

	self.drm = vudrmFairPlaySDK()
	let applicationCertificate = try drm!.requestApplicationCertificate(token: self.vuToken, contentID: contentID)

Using the application certificate, a call for a license may then be prepared on device using the same call but passing different streamingContentKeyRequestData options for online and offline requests. The resulting data is then passed back to the vudrmFairPlaySDK to request a license from the KSM for either online or offline use. The implementation of the next steps differ for each method of handling FairPlay requests, they are as follows:

#### AssetResourceLoaderDelegate:

##### Online Playback

	let spcData = try resourceLoadingRequest.streamingContentKeyRequestData(forApp: applicationCertificate!,contentIdentifier: assetIDData, options: nil)

	let ckcData = try self.drm!.requestContentKeyFromKeySecurityModule(spcData: spcData, token: vuToken, assetID: self.contentID, licenseURL: licenseUrl)
                
	if ckcData != nil {
		resourceLoadingRequest.dataRequest?.respond(with: ckcData!)
	} else {
		print("Failed to get CKC for the request object of the resource.")
		return
	}

You should always set the contentType before calling finishLoading() on the resourceLoadingRequest to make sure you have a contentType that matches the key response.
	
	resourceLoadingRequest.contentInformationRequest?.contentType = AVStreamingKeyDeliveryContentKeyType
	resourceLoadingRequest.finishLoading()
                    
##### Download and Offline Playback:
	
	let spcData = try resourceLoadingRequest.streamingContentKeyRequestData(forApp: applicationCertificate!, contentIdentifier: assetIDData, options: [AVAssetResourceLoadingRequestStreamingContentKeyRequestRequiresPersistentKey: true])

	let ckcData = try self.drm!.requestContentKeyFromKeySecurityModule(spcData: spcData, token: self.vuToken, assetID: self.contentID, licenseURL: self.licenseUrl)
            
	let persistentKey = try resourceLoadingRequest.persistentContentKey(fromKeyVendorResponse: ckcData!, options: nil)
	
You can now write the persistent content key to disk:

	try writePersistableContentKey(contentKey: persistentKey, withContentKeyIdentifier: contentID)
 
and then provide the content key response to make protected content available for processing before calling finishLoading() on the resourceLoadingRequest.
 
	resourceLoadingRequest.dataRequest?.respond(with: persistentKey)
	resourceLoadingRequest.finishLoading()
	
The call for an offline license should be made upon download of the asset.

			
#### ContentKeyDelegate:
			
##### Online Playback

    let completionHandler = { [weak self] (spcData: Data?, error: Error?) in
    	guard let strongSelf = self else { return }
	 		if let error = error {
    			keyRequest.processContentKeyResponseError(error)
					return
			} 
		guard let spcData = spcData else { return }      
		do {
			guard let ckcData = try strongSelf.drm!.requestContentKeyFromKeySecurityModule(spcData: spcData, token: vuToken, assetID: self!.contentID, licenseURL: licenseUrl) else { return }	
			let keyResponse = AVContentKeyResponse(fairPlayStreamingKeyResponseData: ckcData)
			keyRequest.processContentKeyResponse(keyResponse)
				} catch {
			keyRequest.processContentKeyResponseError(error)
				}
			}
		keyRequest.makeStreamingContentKeyRequestData(forApp: applicationCertificate!, contentIdentifier: assetIDData, options: [AVContentKeyRequestProtocolVersionsKey: [1]], completionHandler: completionHandler)	                    
                    
##### Download and Offline Playback:
	
	let ckcData = try self!.drm!.requestContentKeyFromKeySecurityModule(spcData: spcData, token: self!.vuToken, assetID: self!.contentID, licenseURL: self!.licenseUrl)
	
	let persistentKey = try keyRequest.persistableContentKey(fromKeyVendorResponse: ckcData!, options: nil)
	
You can now write the persistent content key to disk:
	
	try strongSelf.writePersistableContentKey(contentKey: persistentKey, withContentKeyIdentifier: self!.contentID)

Finally, provide the content key response to make protected content available for processing.

	let keyResponse = AVContentKeyResponse(fairPlayStreamingKeyResponseData: persistentKey)
	keyRequest.processContentKeyResponse(keyResponse)

##### Tokens and Licensing:

Requests for playback or download will result in a license request for the content from the license server, based on the type of token presented for each VUDRMFairPlaySDK request. The tokens may be FairPlay Rental or FairPlay Persist. FairPlay Rental token license requests will be provided a streaming content key, which may be used for online streaming only. FairPlay Persist token license requests will be provided a persistent content key, which may be written to device and subsequently used for both online streaming, and offline playback of downloaded content.

In iOS, when a download of an asset has been completed with a persist token policy, further playback requests will use the downloaded asset and associated license (or content key), even when online.

Example VUDRM token type templates available are:

- Fairplay Rental `{"type": "r","duration_rental": 3600}`
	- Fairplay Rental tokens may only be used to stream online content.
- Fairplay Persist `{"type": "p","duration_persist": 3600}`
 - Fairplay Persist tokens may be used to stream online content and play offline (downloaded) content.

For further information about VUDRM please contact us, or refer to our documentation:
[https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html)

##### Note:

The content ID should be unique to each asset, as it is used along with the `playlist_url` to create a path to, and identify, offline assets and their associated content keys.
The tvOS platform does not support downloading or offline playback.

There are significant limitations using AirPlay to stream any content to an Apple TV that has been downloaded to the users device. Please see the [Limitations](#limitations) section for further information.


##### Demo Applications

We have developed demo applications based on Apples Fairplay SDK examples, using `AssetResourceLoaderDelegate` for iOS 10 and above, and the newer `ContentKeyDelegate` for iOS 11.2 and above.

The demo applications target both iOS and tvOS platforms using shared source code. Each target platform references its own version of the framework. The lowest version of iOS supported in the demo applications is iOS 11.2, however it is possible to add the required support for iOS 10 and above.

With the VUDRMFairPlaySDK framework installed, both the example demo applications read entries in the `Streams.plist` file to configure the content and DRM. You can edit the `Streams.plist` file values, and/or add your own valid values, which should include a `content_id`, `playlist_url`, and `vudrm_token`(see [Preparation](#preparation) above).

- The `content_key_id_list` entry in the `Streams.plist` is retrieved in the demo by parsing the manifest using the `getContentKeyIDList` function in the `AssetListTableViewController`.

- The `name` value is the display name, which may differ from the `content_id`.

- The `is_protected` boolean value indicates whether the content is protected with DRM or not. This value can be overridden in source if all content is protected.


In both target platforms, the `AssetResourceLoaderDelegate` or `ContentKeyDelegate` class handles license requests for online streaming.

In iOS only the `AssetResourceLoaderDelegate+Persistable` or `ContentKeyDelegate+Persistable` extension handles license requests for iOS offline playback.

The call to make a request for an online license from the framework is made by iOS when a protected asset is requested for playback in the `override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)` function of the 
`AssetListTableViewController`. This corresponds to the specific stream, and an `Asset` object (with associated `AVURLAsset` is initialised for the stream by the returned table cell.

In iOS, the call used to make a request for an offline license from the framework in the AssetResourceLoaderDelegate demo is
`ContentKeyManager.shared.assetResourceLoaderDelegate.requestPersistableContentKeys(forAsset: asset)`, or in the ContentKeyDelegate demo `ContentKeyManager.shared.contentKeyDelegate.requestPersistableContentKeys(forAsset: asset)`
and this call is in the: `override func tableView(_ tableView: UITableView, accessoryButtonTappedForRowWith indexPath: IndexPath)` function of the 
`AssetListTableViewController`. This corresponds to the specific stream Download button.

In iOS, a call is made upon launch by the `AppDelegate` class to the `AssetPersistenceManager` class to restore any persisted content. The call `AssetPersistenceManager.sharedManager.restorePersistenceManager()` asks the `AssetPersistenceManager` class to check any already mapped asset against any outstanding `AVAssetDownloadTask`.

## Limitations
  
- The SDK does not support AirPlay of protected content after a persist content key has been persisted (content downloaded to device). Apple TV is never able to play protected offline/downloaded content with a persist content key because the key cannot be written to the playback device. Attempting to do so may result in unexpected behaviours, crashes referring to the content key type, or the content may not load. Protected content may be transmitted with AirPlay using a streaming content key whenever the Apple TV has a network connection.

- The SDK does not support playback of protected content when running in the iOS Simulator. Attempting to do so will result in an invalid `KEYFORMAT` error and the content will not load.

## Known Issues

- If you believe you have found any issues, please contact us at support@vualto.com

## Troubleshooting

- Most issues are content related. You can use our demo application to test your own content by updating it with your `Stream` configurations. 

- Errors may also arise because the `Stream` configuration is not correct. 
	- Content ID - Please ensure that the `content_id` corresponds to that which was used when preparing your content. The `content_id` should always be unique for each `Stream` instance.
	
	- Tokens - You can easily eliminate token issues by beginning with a default FairPlay token policy. Please ensure your token is formatted correctly and validates. For the avoidance of doubt, where tokens use dates, the dates should always be in the future. For further information about tokens please refer to our token [documentation](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html).
	
If you are not able to play your content after checking it in our demo application please contact support@vualto.com with the demo application logs and the stream configuration used.

## Release notes (iOS / tvOS)

### v1.0 (build 22/23) on 19/03/2021

- Initial Release
