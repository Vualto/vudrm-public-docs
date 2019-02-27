# iOS FAIRPLAY SDK

The VUDRMFairPlay SDK is available for iOS. This documentation describes the steps to integrate and use the VUDRMFairPlay SDK on this platform.
Current release: v0.0.1 (70)

- Overview
- Requirements
- Xcode integration
- Information about Application Transport Security (ATS)
- Example usage
- Limitations
- Known Issues
- Release Notes

### OVERVIEW

VUDRMFairPlay SDK enables Apple’s AVPlayer from AVFoundation to securely request licenses from Vualto’s VUDRM cloud based DRM platform using an extended instance of AssetResourceLoaderDelegate.
There are three deployment scenarios:

1.      Online Playback / Download and Offline Playback using a persisted token and license.
    And legacy methods:
2.      Make use of Apple’s AVPlayer and AVPlayerViewController to present users with the platform
    default skins, with DRM content.
3.      Create your own video player based on AVPlayer.
    As an evolution of the the hugely successful VUPLAY SDK, this SDK has a number of key benefits:
    ● Complete control over the entire AVPlayer lifecycle
    ● Numerous optimisations to enable faster playback
    ● Bitcode support
    ● Support for building/linking against the simulators
    The SDK is fully supported in both Objective-C and Swift applications. A demo application written in Swift is available on request.
    Please contact support@vualto.com to request a download of the SDK.

### REQUIREMENTS

● Minimum iOS deployment target of 9.x or higher
● Xcode 8.3.3
● Swift 3.1

### XCODE INTEGRATION

1. Copy the vudrmFairPlay.framework into your application's project directory
2. Navigate to your target and add vudrmFairPlay.framework to your ‘Embedded Binaries’. You should see that vudrmFairPlay.framework also now appears under ‘Linked Frameworks and Libraries’:

### INFORMATION ABOUT APPLICATION TRANSPORT SECURITY (ATS)

iOS/tvOS 9 introduces Application Transport Security (ATS) which restricts the use of insecure HTTP.
In order to permit playback of content over insecure HTTP exemptions need to be added into your application's Info.plist. For example to disable ATS for any connection you would add the following into your application's Info.plist:
<dict>
<key>NSAppTransportSecurity</key> <dict>
<key>NSAllowsArbitraryLoads</key> <true/>
</dict>
</dict>
More information about ATS can be found in Apple’s TechNote https://developer.apple.com/library/prerelease/ios/technotes/App-Transport-Security-Technote/

### EXAMPLE FRAMEWORK USAGE

1. Import the VUDRMFairPlay module into your player import vudrmFairPlay

2. Initialise an instance of VUDRMFairPlay:

Online Playback / Download and Offline Playback:
Pass in a reference to your AVPlayer instance, the token to be used, and the asset ID when retrieving a license.
The example uses a single dynamically assigned instance. It would be expected that your application will manage the applications offline assets, including their ID’s, locations and download status. This method uses AVAssetDownloadURLSession with a URLSessionConfiguration to create an AVAssetDownloadTask for each configured asset.

The current example initialises an AVAssetDownloadURLSession with a URLSessionConfiguration upon
loading the ViewController:

backgroundConfiguration = URLSessionConfiguration.background(withIdentifier: "AssetDownloadConfigurationIdentifier")
assetUrlSession = AVAssetDownloadURLSession(configuration: backgroundConfiguration!,
assetDownloadDelegate: self, delegateQueue: .main)

The asset configuration is then assigned upon use of the download or
play buttons. To play an asset we create the assetDownloadTask and then assign the associated urlAsset to a player:

let asset = AVURLAsset(url: assetURL!)
self.drm = vudrmFairPlay(asset: asset, assetName: assetName, token: token)
assetDownloadTask = assetUrlSession.makeAssetDownloadTask(asset: asset, assetTitle: assetName,
assetArtworkData: nil, options: nil)!
assetDownloadTask.taskDescription = assetName

let playerItem = AVPlayerItem(asset: assetDownloadTask.urlAsset)
let player = AVPlayer(playerItem: playerItem)
let playerViewController = AVPlayerViewController()
playerViewController.player = player
present(playerViewController, animated: true) {
player.play()
}
And to download, instead of assigning the asset to a player, call:

assetDownloadTask.resume().

To cancel a download, call:

assetDownloadTask.cancel().

Legacy methods:

These are the originally provided methods which are for online playback use only, and are provided for ease of upgrade for legacy users. These methods now also require an asset ID in the initialiser to be compatible with the new framework.

Legacy method 1: Pass in a reference to your AVPlayer instance, the token to be used, and the asset ID when retrieving a license.

let player = AVPlayer(url: assetURL!)
self.drm = vudrmFairPlay(player: player, token: token, assetName: assetName)
  
Legacy method 2: Pass in a reference to your stream URL and the token to be used when retrieving a license. An AVAsset will be created by VUDRMFairPlay and this can be used to create your player.

self.drm = vudrmFairPlay(url:assetURL! as NSURL, token: token, assetName: assetName)
let playerItem = AVPlayerItem(asset: (drm?.asset)!)
let player = AVPlayer(playerItem: playerItem)

3. Finally, present your player:
   self.player = player let playerController = AVPlayerViewController() playerController.delegate = self as? AVPlayerViewControllerDelegate self.present(playerController, animated: true, completion: nil) playerController.view.frame = self.view.frame playerController.player = self.player player.play()
   DEMO
   With the VUDRMFairPlay framework installed, the example demo application can be initialised either via source code in the IDE, or using field entry on device.
   ● To initialise via code in IDE, before running, simply replace the empty ViewController.swift values with your own valid values.
   private var assetURL = URL(string: "")
   private var token = ""
   private var assetName = ""

● To initialise via view controller on device, populate the URL, Token, and Asset Name fields with valid values. For ease of use, pasting into the URL field in the form URL + space + Token will auto populate both fields correctly. At runtime, any values in the URL and Token fields will override any hard coded values.

### Limitations

● The current example application demonstrates a single instance of offline playback using an AVAssetDownloadURLSession with a URLSessionConfiguration to create an AVAssetDownloadTask for that asset. The example is not able to handle multiple assets. Users are expected to develop a method of mapping instances to assets within their application which will avoid multiple downloads of the same asset and will enable management of assets.
The example includes legacy methods for online playback only. The use of these methods in conjunction with the default method may cause unexpected results and is not recommended.
● The SDK does not support playback of protected content when running in the iOS Simulator. Attempting to do so will result in an invalid KEYFORMAT error and the content will not load.

### Known Issues

● No known issues at this time. If you believe you have found an issue, please contact us at support@vualto.com

### RELEASE NOTES

v0.0.1 (build 70) on 15/01/2019
● Update Persistable Offline framework to Swift 4.2
v0.0.1 (build 65) on 14/12/2018
● Persistable Offline Release
v0.0.1 (build 58) on 05/11/2018
● Update framework to Swift 4.2
v0.0.1 (build 44) on 09/04/2018
● Bug fixes and improvements
v0.0.1 (build 15) on 05/04/2018
● Update framework to Swift 4.1
v0.0.1 (build 11) on 27/03/2018
● Bug fixes and improvements
v0.0.1 (build 5) on 15/11/2017
● Initial Release
