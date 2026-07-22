# Keychain and passkey source map

Use current Apple documentation and verify deployment availability in the installed SDK.

## Keychain Services

- [Keychain Services](https://developer.apple.com/documentation/security/keychain-services): secure storage model
  and item APIs.
- [`SecItemAdd`](https://developer.apple.com/documentation/security/secitemadd(_:_:)),
  [`SecItemCopyMatching`](https://developer.apple.com/documentation/security/secitemcopymatching(_:_:)),
  [`SecItemUpdate`](https://developer.apple.com/documentation/security/secitemupdate(_:_:)), and
  [`SecItemDelete`](https://developer.apple.com/documentation/security/secitemdelete(_:)): CRUD behavior,
  result types, and blocking/thread considerations.
- [Updating and deleting keychain items](https://developer.apple.com/documentation/security/updating-and-deleting-keychain-items):
  query matching and the risk of affecting every matching item.
- [`kSecAttrSynchronizable`](https://developer.apple.com/documentation/security/ksecattrsynchronizable): iCloud
  synchronization behavior and compatibility constraints.
- [Sharing keychain access](https://developer.apple.com/documentation/security/sharing-access-to-keychain-items-among-a-collection-of-apps):
  intentional access groups across targets/apps.
- [`SecAccessControlCreateWithFlags`](https://developer.apple.com/documentation/security/secaccesscontrolcreatewithflags(_:_:_:_:)):
  accessibility plus user-presence/biometric/passcode/private-key policies.
- [`LAContext`](https://developer.apple.com/documentation/localauthentication/lacontext): capability checks,
  localized reason, authentication context, and invalidation.

### Safe API vocabulary

- Item identity/result: `kSecClass`, `kSecClassGenericPassword`, `kSecClassInternetPassword`,
  `kSecAttrService`, `kSecAttrAccount`, `kSecAttrServer`, `kSecValueData`, `kSecReturnData`,
  `kSecReturnAttributes`, `kSecMatchLimitOne`.
- Protection: `kSecAttrAccessibleWhenUnlocked`, `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`,
  `kSecAttrAccessibleAfterFirstUnlock`, `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`,
  `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`, `kSecAttrAccessControl`, `kSecUseAuthenticationContext`.
- Sharing/sync/macOS: `kSecAttrSynchronizable`, `kSecAttrSynchronizableAny`, `kSecAttrAccessGroup`,
  `kSecUseDataProtectionKeychain` when current macOS documentation and the target require it.
- Errors: `errSecSuccess`, `errSecItemNotFound`, `errSecDuplicateItem`, `errSecAuthFailed`,
  `errSecInteractionNotAllowed`, `errSecMissingEntitlement`.

Avoid deprecated `kSecAttrAccessibleAlways*`, deprecated Touch ID-named access-control flags, deprecated
`kSecUseOperationPrompt`, and legacy `SecKeychain*` APIs for modern cross-platform app storage.

## Passkeys and associated domains

- [Supporting passkeys](https://developer.apple.com/documentation/authenticationservices/supporting-passkeys):
  registration/assertion, account identity, replacement, and platform credentials.
- [Meet passkeys](https://developer.apple.com/videos/play/wwdc2022/10092/): client/server registration and
  assertion fields and verification boundary.
- [What's new in passkeys](https://developer.apple.com/videos/play/wwdc2025/279/): fresh single-use challenge,
  stable user ID, and current AuthenticationServices behavior.
- [`rawClientDataJSON`](https://developer.apple.com/documentation/authenticationservices/aspublickeycredential/rawclientdatajson):
  opaque bytes supplied to the relying party for verification.
- [Supporting Associated Domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains)
  and [Configuring an associated domain](https://developer.apple.com/documentation/xcode/configuring-an-associated-domain):
  `webcredentials`, AASA delivery, developer alternate mode, and entitlement setup.

Core APIs include `ASAuthorizationPlatformPublicKeyCredentialProvider`,
`createCredentialRegistrationRequest(challenge:name:userID:)`, `createCredentialAssertionRequest(challenge:)`,
`ASAuthorizationController`, `performRequests()`, `performAutoFillAssistedRequests()`, registration/assertion
credential types, `credentialID`, `rawClientDataJSON`, `rawAttestationObject`, `rawAuthenticatorData`, `signature`,
and `userID`. Confirm exact availability and request options in the installed SDK.

## Signing and compliance boundaries

- [Associated Domains entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.developer.associated-domains):
  passkey relying-party association.
- [TN3125: Inside code signing provisioning profiles](https://developer.apple.com/documentation/technotes/tn3125-inside-code-signing-provisioning-profiles):
  effective entitlements must be authorized by the profile.
- [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) and
  [Export compliance overview](https://developer.apple.com/help/app-store-connect/manage-app-information/overview-of-export-compliance):
  separate policy/privacy/encryption determinations that this implementation skill must not guess.
