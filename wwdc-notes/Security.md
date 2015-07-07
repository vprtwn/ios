### Network Security
* TLS doesn't just provide secrecy
    * also protects integrity of connection
* TLS is not enough
    * Many servers default to TLSv1.0 from 1999
    * TLSv1.2 is not enough either
    * TLS needs to be configured to support forward secrecy
* App Transport Security
    * By default, apps linked against iOS9 can't make unprottected HTTP connections
    * TLS connections require compliance with best practices
        * TLSv1.2 with forward secrecy
        * no known-insecure cryptographic primitives
        * key size requirements
    * Exceptions can be declared in Info.plist

### System Integrity Protection
* Defense in depth
    * security is all about layers
    * one layer failing shouldn't defeat all security
    * multiple layers with different properties
* New layer - new policy, applies to all processes
* Potential impact for non-Mac app store apps

### Keychain
* specialized database
* optimized for small secrets
* `SecItemCreate`, `SecItemDelete`, etc.
* factor keychain code into a simple, testable, unit
* use the highest data protection level you can
* `kSecAttrAccessibleWhenUnlocked` is default and best
* `kSecAttrAccessibleAfterFirstUnlock` for background apps
* Shared credentials between safari & apps (new in iOS9)
    * need to add associated domain entitlement
    * also need to add apple-app-site-association JSON file to server
        * no need to sign in iOS9

```swift
// Save to safari

let username = "j@icloud.com"
let password = SecCreateSharedWebCredentialPassword().takeRetainedValue()
SecAddSharedWebCredential("www.bla.com", username, password) { error in 
    // handle error
}

SecRequestSharedWebCredential("www.bla.com", .None) { ... }
```


* For passwords that can be used on multiple devices
    * add `kSecAttrSynchronizable` to all SecItem calls

* Device specific credentials
    * limited use tokens and cookies
    * encrypted messaging keys
    * keys with specific protection requirements
    * `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`
    * `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`

### Security architecture
* User space
    * application
    * security framework
* Kernel
    * process separation
* Secure Enclave
    * KeyStore
    * TouchID

### Touch ID
* Convenience -> stronger security (longer pw, lock immediately)
* LocalAuthentication
    * TouchID success sent to your app in user space
    * can replace existing security barriers
    * can add more convenient barriers
* (new) `touchIDAuthenticationAllowableReuseDuration`
    * don't need touch ID if user has recently unlocked their device
* (new) `evaluatedPolicyDomainState`
    * get a representation of current set of enrolled fingers
* (new) `cancel`
    * cancel a user prompt from code
* (new) `evaluateAccessControl()`
    * use LocalAuthentication with Access Control lists

#### Access control lists
* Add additional protection to a saved credential
* Take advantage of the secure enclave
* Examples:
    * don't require username and password on launch
    * protect local encryption keys

```swift
let acl = SecAccessControlCreateWithFlags(kCFAllocatorDefault,
    [data protection class], [ACL policy], &error).takeRetainedValue()
```

* ACL policy types
    * `.UserPresence` -> TouchID, fallback to PW
    * `.TouchIDAny` -> TouchID, no fallback
    * `.TouchIDCurrentSet` -> item only released if enrolled fingers hasn't changed
* Touch ID and multi factor authentication
    * Something you know (pw) and something you have (touch ID)
    * `SecAccessControlCreateFlags.TouchIDCurrentSet`
        * no fallback, and adversary can't enroll a new finger in settings
* `ApplicationPassword`
    * application provides a password in addition to user's passcode
    * server can send password
* Asymmetric cryptography
    * `SecKeyGeneratePair` -> stores private key in secure enclave
    * `SecKeyRawSign()` -> signs data with private key

#### Strengthening Touch ID as a second factor
* Enrollment
    * Generate keypair
    * Send public key to server
    * Server record public key
* Verification
    * server sends challenge
    * app calls `SecKeyRawSign()``
    * user presents finger
    * app sends signed data back to server
    * server verifies signature against stored public key

