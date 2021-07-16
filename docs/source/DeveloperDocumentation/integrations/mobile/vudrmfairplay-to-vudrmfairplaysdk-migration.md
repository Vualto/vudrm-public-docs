# vudrmFairPlay to vudrmFairPlaySDK Migration

####vudrmFairPlaySDK - Current release:	iOS v1.0 (22) tvOS v1.0 (23)

Vualto are pleased to announce the release of the new`vudrmFairPlaySDK` for iOS and tvOS.

Our earlier `vudrmFairPlay` iOS and tvOS SDK was originally based on Apple's 2015 example of FairPlay implementation using the AVAssetResourceLoader API and `AssetResourceLoaderDelegate` class. Apple later announced the AVContentKeySession API using the `ContentKeyDelegate` class, which introduced new features. Vualto has developed our new `vudrmFairPlaySDK` for iOS and tvOS to allow a single SDK to be used with both AVAssetResourceLoader  - `AssetResourceLoaderDelegate` and AVContentKeySession  - `ContentKeyDelegate` based projects, and to fully expose all the classes and methods that may be required to take advantage of the latest features.

All projects developed using our original `vudrmFairPlay` framework will have been based on the AVAssetResourceLoader API and `AssetResourceLoaderDelegate` class. Only these projects would need to be migrated.

Assuming the architecture of our demo application and Apple's example implementation, to migrate from our `vudrmFairPlay` framework to `vudrmFairPlaySDK` framework:

1. The new `vudrmFairPlaySDK` framework will need to replace the `vudrmFairPlay` framework at project level, and class import declaratons or build scripts will need to be updated.

2. An `AssetResourceLoaderDelegate` class should be added for iOS and tvOS targets, and an `AssetResourceLoaderDelegate+Persistable` class should be added for the iOS target only. These classes were previously wrapped up in the `vudrmFairPlay` framework.

3. The new `vudrmFairPlaySDK` framework does not need to be initialised, instead the `AssetResourceLoaderDelegate` does. Your app should map and manage the assets, and the new `vudrmFairPlaySDK` framework is called as required (upon playback or download) to obtain an application certificate, and a license. Therefore, any use of a `ContentKeyManager` class might need to be amended to initialise the `AssetResourceLoaderDelegate` instead of the `vudrmFairPlay` framework. The new `vudrmFairPlaySDK` framework is called only in the `prepareAndSendContentKeyRequest` method of the `AssetResourceLoaderDelegate` class for online requests, and in the `prepareAndSendPersistableContentKeyRequest` method of the `AssetResourceLoaderDelegate+Persistable` class for offline requests. 

	An example of the required calls for both online and offline scenarios is:

	// Invoke Framework

	`self.drm = vudrmFairPlaySDK()`
	
	// Acquire Application Certificate

	`let applicationCertificate = try drm!.requestApplicationCertificate(token: self.vuToken, contentID: contentID)`
	
	// Acquire License
	
	`let spcData = try resourceLoadingRequest.streamingContentKeyRequestData(forApp: applicationCertificate!, contentIdentifier: assetIDData, options: nil)`
	`let ckcData = try self.drm!.requestContentKeyFromKeySecurityModule(spcData: spcData, token: self.vuToken, assetID: self.contentID, licenseURL: self.licenseUrl)`

4. Calls to the `vudrmFairPlay` framework to request persistable content keys will change from:
`ContentKeyManager.shared.assetResourceLoaderDelegate.doRequestPersistableContentKeys`

 to:	
	
	`ContentKeyManager.shared.assetResourceLoaderDelegate.requestPersistableContentKeys`

5.  Calls to the vudrmFairPlay framework to delete persistable content keys will change from:
`ContentKeyManager.shared.assetResourceLoaderDelegate.deletePersistableContentKey`

	to:
	
	`ContentKeyManager.shared.assetResourceLoaderDelegate.deleteAllPersistableContentKeys`
	
6. Instances of `AVAssetDownloadTask` in the `AssetPersistenceManager` class of our `vudrmFairPlay` framework demo app should be updated to use `AVAggregateAssetDownloadTask`.
	

Our latest `vudrmFairPlaySDK` framework demo app contains examples of the two classes that would need to be added, the additional changes required to other classes, and changes at project level. The demo app also demonstrates where and how to implement the calls to the new `vudrmFairPlaySDK` framework.

If you require any further information, or are not able to play your content after checking it in our demo application, please contact [support@vualto.com](support@vualto.com) with any demo application logs and the stream configuration used.
