# JW Player SDK iOS VUDRM FairPlay Integration

Learn how to enable digital rights management (DRM) protected playback for an iOS app using VUDRM FairPlay. This document is an extension of the JW Player SDK iOS [Play DRM-protected content](https://developer.jwplayer.com/jwplayer/docs/ios-play-drm-protected-content-v3) documentation.

## Requirements

- iOS 10+
- JW Player SDK iOS v3+
- DRM license server URL
- VUDRM FairPlay protected content

## Preparation
Ensure playback can be achieved with unprotected content either using your own application, or the updated [JW Player SDK iOS demo application](https://github.com/jwplayer/jwplayer-sdk-ios-demo) which contains all the following source code that will be needed to use VUDRM FairPlay.

## Implementation

In order to play FairPlay encrypted HLS, the class in your app which communicates with your FairPlay Key Server must adhere to the JWDrmDataSource protocol and be set to the JWPlayerController's drmDataSource property, as in: `self.player.drmDataSource = self`. Our Swift example below demonstrates how a view controller conforms to the JWDrmDataSource protocol.

When an FPS-specific tag is included in the playlist of a media stream that the Apple device is asked to play, the JW Player SDK will ask your application for the necessary information to prepare an encrypted key request:

 - the `fetchContentIdentifierForRequest` delegate method gets called, requesting that the content identifier (also known as Asset ID) be passed in through its completion block. Our Swift example below contains a helper method to retrieve this from the playlist/manifest.

 - the `fetchAppIdentifierForRequest` delegate method is called, prompting for an Application Certificate which must get passed via the completion block. The Application Certificate must be encoded with the X.509 standard with distinguished encoding rules (DER). It is obtained when registering an FPS playback app with Apple, by supplying an X.509 Certificate Signing Request linked to your private key. Our Swift example below contains a helper method to retrieve the Application Certificate from the server.

When the key request is ready:

 - the `fetchContentKeyWithRequest` delegate method is called, providing you with the key request data (also known as SPC - Server Playback Context message) which you need to retrieve the CKC (Content Key Context) message from your key server. The CKC message must be returned via the completion block, under the response parameter. When your app sends the request to the server, the FPS code on the server wraps the required key in an encrypted message and sends it to the app. Your app then provides the JW Player SDK with the encrypted message, which is used to unwrap the message and decrypt the stream, so the iOS device can play the media.
For resources that may expire, you can specify a renewal date in the completion block.
We suggest also specifying the content type (the UTI indicating the type of data contained by the response) when a renewal date is set. Our Swift example below contains helper methods to retrieve the license Certificate from the server.

The following example Swift class contains all the modifications required to extend and enable the iOS JW Player SDK demo application to play content protected with VUDRM FairPlay. A content URL, valid VUDRM token, and certificate server URL should be provided:


	class SwiftViewController: UIViewController, JWDrmDataSource {
    
    // Properties
    @IBOutlet weak var playerContainerView: UIView!
    var player: JWPlayerController?
    private var parsedAssetID: NSURL? = nil
    var contentID = "" // Leave empty
    var licenseUrl = "" // Leave empty
    let vudrmTransactionIdHeader = "x-request-id"
    
    // User defined properties - refer to onboarding
    private let contentURL = "INSERT_CONTENT_URL"
    private let vudrmToken = "INSERT_TOKEN"
    private let certificateUrl = "INSERT_CERTIFICATE_URL"

    override func viewDidLoad() {
        super.viewDidLoad()
        
        let config: JWConfig  = JWConfig(contentURL: contentURL)
        player = JWPlayerController(config: config)
        player?.drmDataSource = self
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        if let playerView = player?.view {
            playerContainerView.addSubview(playerView)
            playerView.constrainToSuperview()
        }
    }
    
    // JW Player SDK JWDrmDataSource protocol function to configure the requests ContentIdentifier (`skd://` value). The content 'contentID' and specific 'licenseUrl' may be reliably obtained from this.
    
    func fetchContentIdentifier(forRequest loadingRequestURL: URL, for encryption: JWEncryption, withCompletion completion: @escaping (Data) -> Void) {
        if encryption == .fairPlay {
            // obtain asset Id / Content Key ID here.
            self.getAssetID(videoUrl: NSURL(string:contentURL)!){
                let assetId: String = (self.parsedAssetID?.absoluteString)!
                self.contentID = (self.parsedAssetID?.lastPathComponent)!
                self.licenseUrl = assetId.replacingOccurrences(of: "skd", with: "https")
                let assetIdData = Data(bytes: assetId.cString(using: .utf8)!,
                                       count: assetId.lengthOfBytes(using: .utf8))
                completion(assetIdData)
            }
        }
    }
    
    // JW Player SDK JWDrmDataSource protocol function to acquire the required application Id / Application Certificate.
    
    func fetchAppIdentifier(forRequest loadingRequestURL: URL, for encryption: JWEncryption, withCompletion completion: @escaping (Data) -> Void) {
        if encryption == .fairPlay {
            // obtain application Id / Application Certificate here.
            do {
                
                let applicationCertificate = try requestApplicationCertificate(vudrmToken: vudrmToken)
                
                let applicationId: Data? = applicationCertificate
                
                completion(applicationId!)
            } catch {
                let message = ["fetchAppIdentifier": "Unexpected error: \(error)."]
                print(message)
            }
        }
    }
    
	// JW Player SDK JWDrmDataSource protocol function to acquire the required content key / FairPlay License.
            
    func fetchContentKey(withRequest requestBytes: Data, for encryption: JWEncryption, withCompletion completion: @escaping (Data, Date?, String?) -> Void) {
        if encryption == .fairPlay {
            
            do {
                
                let ckcData = try self.requestContentKeyFromKeySecurityModule(spcData: requestBytes, vudrmToken: vudrmToken, contentID: contentID, licenseURL: licenseUrl)
                
                let key: Data? = ckcData // obtain key here from server by providing the request.
                let renewalDate: Date? // (optional)
                let contentType = "application/octet-stream" // (optional)
                
                renewalDate = nil
                
                completion(key!, renewalDate, contentType)
                
            } catch {
                let message = ["fetchContentKey": "Unexpected error: \(error)."]
                print(message)
            }
        }
    }
    
    // Fetches the required Application Certificate / App Identifier
    
    func requestApplicationCertificate(vudrmToken: String) throws -> Data? {
        var applicationCertificate: Data? = nil
        let uuid = UUID().uuidString
        var components = URLComponents(string: self.certificateUrl)
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
                    let message = ["requestApplicationCertificate": "Unexpected HTTP status code: \(httpResponse.statusCode) from FPS license server with transaction ID: \(unwrappedID)"]
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
    
    // Fetches the required License
    
    func synchFetchSPC(_ spcData:Data, vudrmToken:String, contentID:String, licenseURL:String) throws -> (Data?,URLResponse?,NSError?) {
        var myResponse: URLResponse? = nil
        var myData: Data? = nil
        var myError: NSError? = nil
        let message = ["synchFetchSPC": "Initialised with contentID: \(contentID),token: \(vudrmToken), and url: \(licenseURL)"]
        print(message)
        let components = URLComponents(string: licenseUrl)
        var request = URLRequest(url: (components?.url!)!)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-type")
        let base64Encoded = spcData.base64EncodedData(options: Data.Base64EncodingOptions(rawValue: 0))
        let alpha = NSString(data: base64Encoded, encoding: String.Encoding.utf8.rawValue)!
        let jsonDict : [String: AnyObject] = ["token":vudrmToken as AnyObject,"payload":alpha, "contentId":contentID as AnyObject]
        let stringBody = JSONStringifyCommand(jsonDict)
        request.httpBody = stringBody!.data(using: String.Encoding.utf8)
        
        let session = URLSession.shared
        var gotResp = false
        
        let task = session.dataTask(with: request,
                                    completionHandler: { data, response, error -> Void in
            if let error = error{
                let message = ["synchFetchSPC": "Unable to retrieve FPS license: \(String(describing: error))"]
                print(message)
                return
            }
            myResponse = response!
            myData = data
            myError = error as NSError?
            
            gotResp = true
        })
        task.resume()
        // block thread until completion handler is called
        while !gotResp {
            // wait
        }
        return (myData, myResponse, myError)
    }
    
    // Prepare and handle required License request
    
    public func requestContentKeyFromKeySecurityModule(spcData: Data, vudrmToken: String, contentID: String, licenseURL: String) throws -> Data? {
        
        let message = ["requestContentKeyFromKeySecurityModule": "Initialised for \(contentID) with token: \(vudrmToken) and license URL: \(licenseURL)"]
        print(message)
        
        let ckcResponse = try synchFetchSPC(spcData, vudrmToken: vudrmToken, contentID: contentID, licenseURL: licenseURL)
        guard let urlResponse = ckcResponse.1 as? HTTPURLResponse, urlResponse.statusCode == 200 else {
            return nil
        }
        
        guard let ckc = ckcResponse.0 else {
            return nil
        }
        return ckc
    }
    
    // MARK: - Helper methods
    
    // Prepares the required License request body
    
    func JSONStringifyCommand( _ messageDictionary : Dictionary <String, AnyObject>) -> String? {
        
        let options = JSONSerialization.WritingOptions(rawValue: 0)
        
        if JSONSerialization.isValidJSONObject(messageDictionary) {
            
            do {
                let data =  try JSONSerialization.data(withJSONObject: messageDictionary, options: options)
                if let string = NSString(data: data, encoding: String.Encoding.utf8.rawValue) {
                    return string as String
                }
                
            } catch {
                
                let message = ["JSONStringifyCommand": "Error converting JSON"]
                print(message)
            }
        }
        
        return nil
    }
    
    // Parses the playlist / manifest to retrieve the required Content Key ID, being the 'skd://' value of the "EXT-X-SESSION-KEY" or "EXT-X-KEY". The correct license server URL should be obtained from this value, and the last path component represents the correct Content ID.
    
    func getAssetID (videoUrl: NSURL, completion: @escaping() -> Void) {
        let message = ["getAssetID": "Parsing Content Key ID from manifest with \(videoUrl)"]
        print(message)
        var request = URLRequest(url: videoUrl as URL)
        var gotURI = false
        request.httpMethod = "GET"
        let session = URLSession(configuration: URLSessionConfiguration.default)
        let task = session.dataTask(with: request) { data, response, _ in
            guard let data = data else { return }
            let strData = String(data: data, encoding: .utf8)!
            if strData.contains("EXT-X-SESSION-KEY") || strData.contains("EXT-X-KEY") {
                let start = strData.range(of: "URI=\"")!.upperBound
                let end = strData[start...].range(of: "\"")!.lowerBound
                let keyUrlString = strData[start..<end]
                let keyUrl = URL(string: String(keyUrlString))
                let message = ["getAssetID": "Parsed Content Key ID from manifest: \(keyUrlString)"]
                print(message)
                self.parsedAssetID = keyUrl as NSURL?
                gotURI = true
            } else {
                // This could be HLS content with variants
                if strData.contains("EXT-X-STREAM-INF") {
                    // Prepare the new variant video url last path components
                    let start = strData.range(of: "EXT-X-STREAM-INF")!.upperBound
                    let end = strData[start...].range(of: ".m3u8")!.upperBound
                    let strData2 = strData[start..<end]
                    let start2 = strData2.range(of: "\n")!.lowerBound
                    let end2 = strData2[start...].range(of: ".m3u8")!.upperBound
                    let unparsedVariantUrl = strData[start2..<end2]
                    let variantUrl = unparsedVariantUrl.replacingOccurrences(of: "\n", with: "")
                    // Prepare the new variant video url
                    let videoUrlString = videoUrl.absoluteString
                    let replaceString = String(videoUrl.lastPathComponent!)
                    if let unwrappedVideoUrlString = videoUrlString {
                        let newVideoUrlString = unwrappedVideoUrlString.replacingOccurrences(of: replaceString, with: variantUrl)
                        let pathURL = NSURL(string: newVideoUrlString)!
                        // Push the newly compiled variant video URL through this method
                        self.getAssetID(videoUrl: pathURL){
                        }
                    }
                } else {
                    // Nothing we understand, yet
                    let message = ["getAssetID": "Unable to parse URI from manifest. EXT-X-SESSION-KEY, EXT-X-KEY, or variant not found."]
                    print(message)
                }
            }
            while !gotURI {
                // wait
            }
            completion()
        }
        task.resume()
    }
	}


	// MARK: - Helper extensions

	extension UIView {
    /// Constrains the view to its superview, if it exists, using Autolayout.
    /// - precondition: For player instances, JWP SDK 3.3.0 or higher.
    @objc func constrainToSuperview() {
        translatesAutoresizingMaskIntoConstraints = false
        let horizontalConstraints = NSLayoutConstraint.constraints(withVisualFormat: "H:|[thisView]|",
                                                                   options: [],
                                                                   metrics: nil,
                                                                   views: ["thisView": self])
        
        let verticalConstraints   = NSLayoutConstraint.constraints(withVisualFormat: "V:|[thisView]|",
                                                                   options: [],
                                                                   metrics: nil,
                                                                   views: ["thisView": self])
        
        NSLayoutConstraint.activate(horizontalConstraints + verticalConstraints)
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

## Support and Information

If you require any assistance or further information please contact [support@vualto.com](support@vualto.com). If you are unable to play your content, please include any application logs and the stream configuration used.
