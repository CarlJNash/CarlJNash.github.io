---
layout: post
title: iOS App Security
date: 2023-05-04 15:18 +0100
---

<!-- ![headerImage](https://developer.apple.com/news/images/og/security-og.jpg) -->
![headerImage](https://9to5mac.com/wp-content/uploads/sites/6/2022/10/apple-security.jpg)

Apple provides many built-in security features for iOS apps.

## App Distribution

To provide a level of security and safety, Apple do not allow side-loading apps on iOS devices. That is, apps can only be distributed via the [App Store](#app-store) and the Apple Developer [Enterprise Program](#enterprise-program).

There are ways around this, ie. [Jailbreaking](#jailbreaking) a device, but this is not something most general users will do.

### App Store

For an app to be made available on the App Store, it must be signed with a valid code signing identity (profile & certificate) by an authorised Apple Developer account.

The app must also be reviewed and approved by Apple.

### Enterprise Program

The [Apple Developer Enterprise Program](https://developer.apple.com/programs/enterprise/) is a service allows organisations to distribute apps in-house to employees, avoiding the App Store.

For a device to run an enterprise app, the user must first install and authorise the relevant enterprise profile. Enterprise apps must still be signed using a valid profile and certificate by an authorised Apple Developer Enterprise Account.

## iOS Security

Once an app is installed iOS provides further security measures.

### Code signing

iOS only allows executable code signed by a valid profile/certificate to run on the device.

Code signing also ensures that the code hasn't changed since being installed/last updated.

Code signing requires:

- Authorised Apple developer account
- Valid distribution certificate
- Valid distribution profile

[support.apple.com/en-gb/guide/security/sec7c917bf14/1/web/1](https://support.apple.com/en-gb/guide/security/sec7c917bf14/1/web/1)

### Runtime Processes

> iOS and iPadOS help ensure runtime security by using a “sandbox”, declared entitlements and Address Space Layout Randomisation (ASLR).

[support.apple.com/en-gb/guide/security/sec15bfe098e/web](https://support.apple.com/en-gb/guide/security/sec15bfe098e/web)

#### Sandbox

Apps are "sandboxed" to restrict access outside of their designated directory.

[support.apple.com/en-gb/guide/security/sec15bfe098e/1/web/1](https://support.apple.com/en-gb/guide/security/sec15bfe098e/1/web/1)

#### Entitlements

Apple's "permission" system for which systems apps can access and actions they can perform on the system, eg.:

- Background tasks
- Keychain
- Location
- Microphone
- Camera
- Photo library

### ATS (App Transport Security)

ATS ensures that all network connections are secure using HTTPS/TLS.

This can be overridden if needed but a suitable reason must given when submitting the app to App Store review.

> App Transport Security provides default connection requirements so that apps adhere to best practices for secure connections when using NSURLConnection, CFURL or NSURLSession APIs.

[support.apple.com/en-gb/guide/security/sec100a75d12/1/web/1](https://support.apple.com/en-gb/guide/security/sec100a75d12/1/web/1)

## Security Tools

### Apple Security Tools

Apple provide tools and features that the developer can use to further enhance security.

Apple's [Security](https://developer.apple.com/documentation/security) framework provides tools such as:

#### iOS Keychain

> The infrastructure and a set of APIs used by Apple operating systems and third-party apps to store and retrieve passwords, keys and other sensitive credentials.

This is a specialised database for storing confidential data.

If storing passwords in the Keychain, these should be salted and hashed rather than stored in plain-text.

<details markdown=1>
<summary>Code sample: Hash password:</summary>

```swift
import CryptoKit
import CommonCrypto

func hashPassword(password: String) -> String? {
    guard let salt = generateSalt() else {
        return nil
    }
    
    let passwordData = Data(password.utf8)
    let saltData = Data(salt.utf8)
    var hashData = Data(count: Int(CC_SHA256_DIGEST_LENGTH))
    
    guard CCKeyDerivationPBKDF(
            CCPBKDFAlgorithm(kCCPBKDF2),
            password, passwordData.count,
            salt, saltData.count,
            CCPseudoRandomAlgorithm(kCCPRFHmacAlgSHA256),
            10000, // iterations
            &hashData, hashData.count) == kCCSuccess else {
        return nil
    }
    
    let saltString = saltData.base64EncodedString()
    let hashString = hashData.base64EncodedString()
    return "\(saltString):\(hashString)"
}

private func generateSalt() -> String? {
    let count = 32
    var bytes = [UInt8](repeating: 0, count: count)

    let result = SecRandomCopyBytes(kSecRandomDefault, count, &bytes)
    guard result == errSecSuccess else {
        return nil
    }
    
    return Data(bytes).base64EncodedString()
}
```

</details>

<details markdown=1>
<summary>Code sample: Save password to keychain</summary>

```swift
import Foundation
import Security

func savePasswordToKeychain(password: String) -> Bool {
    let service = "com.example.app"
    let account = "user123"
    
    guard let passwordData = password.data(using: .utf8) else {
        return false
    }
    
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: service,
        kSecAttrAccount as String: account,
        kSecValueData as String: passwordData
    ]
    
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess
}
```

</details>

---

NOTE: The code for interacting with the Apple Keychain is written in C, so to make this a little easier for iOS developers to work with Apple have a [sample project](https://developer.apple.com/library/archive/samplecode/GenericKeychain/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007797-Intro-DontLinkElementID_2) which uses a Swift wrapper.

#### iOS Secure Enclave

> The Secure Enclave is a system on chip (SoC) that is included on all recent iPhone, iPad, Apple Watch, Apple TV and HomePod devices, and on a Mac with Apple silicon as well as those with the Apple T2 Security Chip. The Secure Enclave itself follows the same principle of design as the SoC does, containing its own discrete Boot ROM and AES engine. The Secure Enclave also provides the foundation for the secure generation and storage of the keys necessary for encrypting data at rest, and it protects and evaluates the biometric data for Face ID and Touch ID.

<!--
#### Apple Cryptography

SOMETHING ABOUT CRYPTOGRAPHY
-->

### Third Party Security Tools

Aside from the Apple security tools, there are many third party tools and services that can be used to improve security in various areas, such as:

#### Certificate pinning

- [TrustKit](https://github.com/datatheorem/TrustKit)

#### Obfuscation

[Security Through Obscurity](https://en.wikipedia.org/wiki/Security_through_obscurity)

#### Cryptography

<!-- Description of what cryptography is and how it can be used -->

If your app minimum SDK is iOS 13 or newer then you can use Apple's [CryptoKit](https://developer.apple.com/documentation/cryptokit/).
> Use Apple CryptoKit to perform common cryptographic operations:
Compute and compare cryptographically secure digests.
Use public-key cryptography to create and evaluate digital signatures, and to perform key exchange. In addition to working with keys stored in memory, you can also use private keys stored in and managed by the Secure Enclave.
Generate symmetric keys, and use them in operations like message authentication and encryption.
Prefer CryptoKit over lower-level interfaces. CryptoKit frees your app from managing raw pointers, and automatically handles tasks that make your app more secure, like overwriting sensitive data during memory deallocation.

If your app supports older than iOS 13 then you can use the open-source library [CryptoSwift](https://github.com/krzyzanowskim/CryptoSwift).

## Data Security

When it comes to app security that the developer should be concerned with, these can be grouped into [local data](#local-data) and [networking data](#network-data).

### Local Data

Local data is anything that is stored on the device, this could be in the compiled code, User Defaults, Keychain, on-disk, etc.

#### Compiled Data

Compiled data can include the app code, but it can also include any other files such as the `Info.plist` or other files that are included with the app archive.

The app code is obfuscated when it is packaged into a `.ipa` file. This obfuscation can provide some level of protection, but it can be easily de-obfuscated.
<!-- MORE INFO NEEDED -->

##### Secrets

Secrets can be any confidential values that the app uses for communication/encryption, etc.

- [nshipster.com/secrets](https://nshipster.com/secrets/)
- [nshipster.com/swift-gyb](https://nshipster.com/swift-gyb/)

###### API Key

What are API keys?

###### Access token

Is an access token the same as an API key?

#### Runtime Data

- User data
- Refresh token

The most secure way to store data is to not store data.

If the app must store data, then the following options are available to store this data securely:

- Data storage
  - Keychain
  - Secure enclave
  - [Encryption](#encryption)
  - Obfuscation

### Network Data

All network data to and from the app should be [authenticated](#authentication), [authorised](#authorisation) and [encrypted](#encryption).

#### Authentication vs Authorisation

##### Authentication

Authentication is proving who you are.

We can authenticate the user, and also the app.

##### Authorisation

Authorisation defines what you are allowed to do.

- [Certificate Pinning](#certificate-pinning)
  - [OAuth 2.0](#oauth-20)
  - [JWT](#jwt)

#### Encryption

SOMETHING ABOUT ENCRYPTION

#### ATS

App Transport Security.

Only allows secure network connections via HTTPS protocol.

#### Certificate Pinning

Preventing MITM (man in the middle) attacks.

#### OAuth 2.0

Authorisation rather than authentication.

#### OWASP

[owasp.org](https://owasp.org)

#### JWT

> JSON Web Token (JWT) is an open standard (RFC 7519) that defines a way to securely transmit information.

## Source Control

- If you commit sensitive information to a source control repository then there's a possibility it could be leaked or accessed by unauthorised person(s).

---

## Useful Links

- [Apple's Platform Security Guide](https://help.apple.com/pdf/security/en_GB/apple-platform-security-guide-b.pdf)
