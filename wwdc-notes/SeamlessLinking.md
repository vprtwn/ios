### Have your app handle link to your website
* web content often mirrors app content
* pitfalls of custom URL schemes
    * don't always map to your app
    * don't work without your app installed
    * don't protect user's privacy
    * (apps can figure out what apps are installed)
* In a universal link, app will match
    * scheme
    * domain
    * path or path prefix
    * can use `NSURLQueryItem` to get components of query string
* Your app needs to claim that it can handle links from your domain
    * iOS contacts your server
* Need to create an `apple-app-site-association` file
    * paths array = which paths are accessible on your website
    * `/wwdc/news`, `/videos/wwdc/2015/*`
* Generate an SSL certificate from a certificate authority
* Sign JSON file with your SSL certificate using `openssl`
* Upload signed json file to the root of your server
    * `https://www.example.com/apple-app-site-association`
    * each domain needs its own file
* In iOS9, no longer need to sign the file
    * still required for iOS8

#### Getting your app ready
* `application:continueUserActivity:...`
* userActivity has a `webpageURL` and `activityType`
* update entitlements
    * add associated domains (`applinks.example.com`, `applinks.www.example.com`)


```swift
// Nice URL matching in Swift
if pathComponents.count >= 4 {
    switch (pathcOmponents[0], pathComponents[1], pathComponents[2], pathComponents[3]) {
        case ("/", "videos", "wwdc", let yearString):
        if let year = Int(yearString), let session = findSession(components) {
            switch (year, session) {
                case (2011...2015, 100...1000):
                return presentVideoController(...)
                default:
                return false
            }
        }
    }
}
```

### Smart App Banners
* (new in iOS9) indexed by Apple to improve discovery
* just add the "apple-itunes-app" meta tag
* app-argument is passed to `applciation:openURL`
* see "Introducing Search APIs" session

### Shared Web Credentials
* allow your users to use safari saved passwords
* need to add "webcredentials" key to JSON file on server
* app needs to add new entry to Associated Domains entitlement
* `SecRequestSharedWebCredential`
    * remember to dispatch back to the main queue
* `SecAddSharedWebCredential`
    * add to shared passwords
* Refer to last year's session "Your app, your website, and safari"



