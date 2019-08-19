# tvOS FairPlay SDK

The VUDRMFairPlay SDK is available for tvOS. This documentation describes the steps to integrate and use the VUDRMFairPlay SDK on this platform.

Current release: v0.0.1 (14)

- Overview
- Requirements
- Xcode integration
- Information about Application Transport Security (ATS)
- Example framework usage
- Limitations
- Known Issues
- Release Notes

## Overview

VUDRMFairPlay SDK enables Apple’s AVPlayer from AVFoundation to securely request licenses from Vualto’s VUDRM cloud based DRM platform using an extended instance of AssetResourceLoaderDelegate.

Make use of Apple’s AVPlayer and AVPlayerViewController to present users with the platform default skins, with DRM content. Or create your own video player based on AVPlayer.

As an evolution of the the hugely successful VUPLAY SDK, this SDK has a number of key benefits:

 - Complete control over the entire AVPlayer lifecycle
 - Numerous optimisations to enable faster playback
 - Bitcode support
 - Support for building/linking against the simulators

The SDK is fully supported in both Objective-C and Swift applications. 

A demo application written in Swift is available on request. Please contact [support@vualto.com](support@vualto.com) to request a download of the SDK.

## Requirements

- Minimum tvOS deployment target of 11.x or higher
- Xcode 10.2
- Swift 5

## Xcode Integration

1. Copy the vudrmFairPlay.framework into your application's project directory
2. Navigate to your target and add vudrmFairPlay.framework to your ‘Embedded Binaries’. You should see that vudrmFairPlay.framework also now appears under ‘Linked Frameworks and Libraries’:

## Information about application transport security (ATS)

tvOS 9 introduces Application Transport Security (ATS) which restricts the use of insecure HTTP.
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

Before you can use the demo project or framework in your own project, for each instance you will require a URL to correctly prepared content and a VUDRM token which must be correctly generated for the type of instance you wish to create. An example VUDRM token type template available for use with tvOS is:

- Fairplay Rental `{"type": "r","duration_rental": 3600}`

For further information about VUDRM please contact us, or refer to our documentation:
[https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html](https://docs.vualto.com/projects/vudrm/en/latest/VUDRM-token.html)

## Example framework usage

* Import the VUDRMFairPlay module into your view controller:

	`import vudrmFairPlay`

* Initialise an instance of VUDRMFairPlay:

Pass in a reference to your stream URL and the token to be used when retrieving a license. An AVAsset will be created by VUDRMFairPlay and this can be used to create your player.

```swift
self.drm = vudrmFairPlay(url:assetURL! as NSURL, token: token)
let playerItem = AVPlayerItem(asset: (drm?.asset)!)
let player = AVPlayer(playerItem: playerItem)
```

Finally, present your player:

```swift
self.player = player
let playerController = AVPlayerViewController() 
playerController.delegate = self as? AVPlayerViewControllerDelegate
self.present(playerController, animated: true, completion: nil)
playerController.view.frame = self.view.frame
playerController.player = self.player player.play()
```

##### DEMO

With the VUDRMFairPlay framework installed, the example demo application can be initialised either via source code in the IDE, or using field entry on device.

- To initialise via code in IDE, before running, simply replace the empty ViewController.swift values with your own valid values.

```swift
private var assetURL = URL(string: "")
private var token = ""
```

- To initialise via view controller on device, populate the URL and Token fields with valid values.

## Limitations

- The SDK does not support playback of protected content when running in the tvOS Simulator. Attempting to do so will result in an invalid `KEYFORMAT` error and the content will not load.

## Known Issues

- There are no known issues at this time

- If you believe you have found any issues, please contact us at support@vualto.com

## Release notes

### v0.0.1 (build 14) on 13/08/2019

- Update license server DNS

### v0.0.1 (build 10) on 08/04/2019

- Initial release
