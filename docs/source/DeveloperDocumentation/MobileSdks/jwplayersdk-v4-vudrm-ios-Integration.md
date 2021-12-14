# JW Player SDK v4 - VUDRM - iOS - Integration

_Learn how to enable digital rights management (DRM) protected playback for an iOS app using VUDRM._

In order to play a FairPlay encrypted HLS stream, the class in your app that communicates with your FairPlay Key Server must satisfy the following points:

* Adhere to the `JWDRMContentKeyDataSource` protocol
* Be set to the `contentKeyDataSource` property of the `JWPlayerViewController`, for example, `player.contentKeyDataSource = self`

When an Apple device needs to play a stream that includes an FPS-specific tag, the JW Player SDK requests the content identifier and Application Certificate from your application for the information to prepare an encrypted key request.

The following table explains each requested item.

| Item | Notes |
| --- | --- |
| **Content Identifier** | The `contentIdentifierForURL` delegate requests that the content identifier be passed in through its completion block.<br /><br />The content identifier is also known as Asset ID and the `skd://` value. The content `contentID` and specific `licenseUrl` may be reliably obtained from this. |
| **Application Certificate** |  The `appIdentifierForURL` delegate method prompts for an Application Certificate which must get passed via the completion block.<br /><br />When registering your FPS playback app with Apple, be sure to supply an X.509 Certificate Signing Request linked to your private key. This ensures that the Application Certificate is encoded with the X.509 standard with distinguished encoding rules (DER). |
| **Server Playback Context (SPC)** | The `contentKeyWithSPCData` delegate method provides the key request data (SPC) needed to retrieve the Content Key Context (CKC) message.<br /><br />The CKC message must be returned via the completion block under the response parameter. When your app sends the request to the server, the following actions take place:<br /><br />&bull; The FPS code on the server wraps the required key in an encrypted message and sends it to your app.<br />&bull; Your app provides the JW Player SDK with an encrypted message. The encrypted message enables unwrapping the message and decrypting the stream. This allows the Apple device to play the media. |
 
 
## Requirements

- iOS 10+
- JW Player SDK iOS v4+
- DRM license server URL
- VUDRM FairPlay protected content URL
- VUDRM token
- Application capable of unprotected content playback

## Preparation
Ensure playback can be achieved with unprotected content either using your own application, or the updated [JW Player SDK v4 iOS demo application](https://github.com/Vualto/vudrm-jw-sdk-ios/tree/main-v4) which contains all the following source code that will be needed to use VUDRM with the JW Player SDK.

The easiest way to start integration of the JW Player SDK and VUDRM is to use Cocoapods. This can be achieved by adding VUDRM FairPlay SDK spec to the project Cocoapod Podfile, so your podfile would look like this (note the addition of `PRIVATE_SOURCE` URL and `pod 'vudrmFairPlaySDK'`).

```
# Uncomment the next line to define a global platform for your project
platform :ios, '10.0'

target 'JWPlayer-SDK-iOS-Demo' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for JWPlayer-SDK-iOS-Demo
	pod 'JWPlayerKit', '>= 4.0.0'
end
```

Then using the Terminal app, navigate to the project directory, and enter:

`pod update`

If you receive an error reporting that a framework cannot be found, enter:

`pod install --repo-update`

## Implementation

The following example Swift class contains all the modifications required to extend and enable the [JW Player SDK iOS demo application](https://github.com/jwplayer/jwplayer-sdk-ios-demo) to play content protected with the VUDRM FairPlay.

_**NOTE**: If you require any assistance or further information regarding the following implementation, please contact [support@vualto.com](support@vualto.com).  If you are unable to play your content, please include any application logs and the stream configuration used._

	import UIKit
	import JWPlayerKit
	
	private let VUDRMCertificateEndpoint = "INSERT_CERTIFICATE_URL"
	private let VUDRMVideoEndpoint = "INSERT_CONTENT_URL"
	private let VUDRMToken = "INSERT_TOKEN"

	class ViewController: JWPlayerViewController,
                      JWDRMContentKeyDataSource {
    
    // Properties
    var contentUUID: String?
    var contentID: String?
    var licenseUrl: String?
    let vudrmTransactionIdHeader = "x-request-id"
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Set up the player.
        setUpPlayer()
    }
    
    /**
     Sets up the player with a DRM configuration.
     */
    private func setUpPlayer() {
        // Open a do-catch block to catch possible errors with the builders.
        do {
            let videoUrl = URL(string:VUDRMVideoEndpoint)!
            
            // First, use the JWPlayerItemBuilder to create a JWPlayerItem that will be used by the player configuration.
            let playerItem = try JWPlayerItemBuilder()
                .file(videoUrl)
                .build()
            
            // Second, create a player config with the created JWPlayerItem.
            let config = try JWPlayerConfigurationBuilder()
                .playlist([playerItem])
                .autostart(true)
                .build()
            
            // Third, set the data source class. This class conforms to JWDRMContentKeyDataSource, and defines methods which affect DRM.
            player.contentKeyDataSource = self
            
            // Lastly, use the created JWPlayerConfiguration to set up the player.
            player.configurePlayer(with: config)
        } catch {
            // Builders can throw, so be sure to handle the build failures.
            print(error.localizedDescription)
            return
        }
    }
    
    
    // MARK: JWDRMContentKeyDataSource
    
    /**
     When called, this delegate method requests the identifier for the protected content to be passed through the delegate method's completion block.
     */
    func contentIdentifierForURL(_ url: URL, completionHandler handler: @escaping (Data?) -> Void) {
        guard let uuid = url.absoluteString.split(separator: ";").last,
              let uuidData = uuid.data(using: .utf8) else {
                  handler(nil)
                  return
              }
        self.contentUUID = String(uuid)
        handler(uuidData)
    }
    
    
    /**
     When called, this delegate method requests an Application Certificate binary which must be passed through the completion block.
     - note: The Application Certificate must be encoded with the X.509 standard with distinguished encoding rules (DER). It is obtained when registering an FPS playback app with Apple, by supplying an X.509 Certificate Signing Request linked to your private key.
     */
    func appIdentifierForURL(_ url: URL, completionHandler handler: @escaping (Data?) -> Void) {

        guard let certUrl = URL(string: VUDRMCertificateEndpoint),
              let appIdData = try? requestApplicationCertificate(vudrmToken: VUDRMToken) else {
                  handler(nil)
                  return
              }
        handler(appIdData)
    }
    
    /**
     When the key request is ready, this delegate method provides the key request data (SPC - Server Playback Context message) needed to retrieve the Content Key Context (CKC) message from your key server. The CKC message must be returned via the completion block under the response parameter.
     
     After your app sends the request to the server, the FPS code on the server sends the required key to the app. This key is wrapped in an encrypted message. Your app provides the encrypted message to the JWPlayerKit. The JWPlayerKit unwraps the message and decrypts the stream to enable playback on an iOS device.
     
     - note: For resources that may expire, specify a renewal date and the content-type in the completion block.
     */
    func contentKeyWithSPCData(_ spcData: Data, completionHandler handler: @escaping (Data?, Date?, String?) -> Void) {
        
        guard let contentUUID = self.contentUUID else {
            handler(nil, nil, nil)
            return
        }
        
        let licenseApiPath = contentUUID.replacingOccurrences(of: "skd", with: "https")
        
        var ckcRequest = URLRequest(url: URL(string: licenseApiPath)!)
        ckcRequest.httpMethod = "POST"
        ckcRequest.httpBody = spcData
        ckcRequest.addValue("application/octet-stream", forHTTPHeaderField: "Content-Type")
        ckcRequest.addValue(VUDRMToken, forHTTPHeaderField: "x-vudrm-token")
        
        URLSession.shared.dataTask(with: ckcRequest) { (data, response, error) in
            guard error == nil, let httpResponse = response as? HTTPURLResponse, (200...299).contains(httpResponse.statusCode) else {
                handler(nil, nil, nil)
                return
            }
            handler(data, nil, nil)
        }.resume()
    }
    
    func requestApplicationCertificate(vudrmToken: String) throws -> Data? {
        
        var applicationCertificate: Data? = nil
        let uuid = UUID().uuidString
        var components = URLComponents(string: VUDRMCertificateEndpoint)
        components?.query = uuid
        let urlString = NSURL(string: (components?.url?.absoluteString)!)
        let urlMinusQueryString  = urlString?.absoluteStringByTrimmingQuery()
        var request = URLRequest(url: URL(string: urlMinusQueryString!)!)
        request.setValue(vudrmToken, forHTTPHeaderField: "x-vudrm-token")
        request.setValue("max-age=0", forHTTPHeaderField: "Cache-Control")
        let session = URLSession.shared
        var gotResp = false
        let task = session.dataTask(with: request,
                                    completionHandler: { data, response, error -> Void in
            if let error = error{
                let message = ["requestApplicationCertificate": "Unable to retrieve FPS certificate: \(String(describing: error))."]
                print(message)
                
                return
            }
            let httpResponse = response as! HTTPURLResponse
            let transactionID = httpResponse.allHeaderFields[self.vudrmTransactionIdHeader] as? String
            if let unwrappedID = transactionID {
                if (httpResponse.statusCode >= 400){
                    let message = ["requestApplicationCertificate": "Unexpected HTTP status code: \(httpResponse.statusCode) from FPS certificate server with transaction ID: \(unwrappedID)"]
                    print(message)
                    
                    return
                }
            }
            applicationCertificate = data
            gotResp = true
        })
        task.resume()
        
        while !gotResp {
            // wait
        }
        return applicationCertificate
    }
    }

	extension NSURL {
    func absoluteStringByTrimmingQuery() -> String? {
        if let urlcomponents = NSURLComponents(url: self as URL, resolvingAgainstBaseURL: false) {
            urlcomponents.query = nil
            return urlcomponents.string
        }
        return nil
    }
	}

