---
name: apple-keychain-passkeys
description: Implement and debug secure small-secret storage and passkey authentication on Apple platforms using Keychain Services, LocalAuthentication, AuthenticationServices, Associated Domains, and the client side of WebAuthn flows. Use for SecItem CRUD/query design, keychain accessibility or synchronization choices, access groups, biometric or passcode-gated retrieval, passkey registration/assertion, AutoFill-assisted passkeys, webcredentials AASA setup, and directly related entitlement failures. Do not use for general cryptography or CryptoKit design, TLS/trust/pinning, OAuth/OIDC, Sign in with Apple, credential-provider extensions, server-side WebAuthn implementation, App Store policy audits, or broad signing/provisioning failures.
---

# Apple Keychain and Passkeys

Treat every storage or authentication change as a security contract. Preserve least privilege, exact query scope,
server verification, and explicit failure handling; never “fix” access by weakening protection or broadening a query.

## Boundaries

- Use `app-store-review` for privacy disclosures, login-policy requirements, account deletion, and export-compliance
  decisions. Secure storage alone does not determine App Store compliance.
- Use `build-macos-apps:signing-entitlements` for certificate, profile, team, distribution signing, Gatekeeper, or
  entitlements-not-authorized-by-profile failures. This skill may identify the narrowly required capability.
- Use a backend security workflow for WebAuthn verification and a separate authentication workflow for OAuth,
  OIDC, `ASWebAuthenticationSession`, or Sign in with Apple.

Read [references/source-map.md](references/source-map.md) before selecting protection values, access-control flags,
passkey fields, or entitlement behavior.

## Define the security contract

Before editing, record:

1. Platform and minimum OS.
2. Secret type, size, owner/account, lifetime, rotation, logout, and deletion behavior.
3. Whether access is foreground-only or must work after first unlock/background execution.
4. Whether data is device-bound, included in backup/migration, or intentionally synchronized through iCloud.
5. Whether one target owns it or an explicit minimal access group shares it.
6. Whether each read requires user presence, passcode fallback, or unchanged biometric enrollment.

Inspect all existing read/write/update/delete paths, account switching, app extensions, upgrade migration,
background work, and logs before replacing storage.

## Keychain workflow

1. Choose the item class by semantics: generic password, internet password, or a real key item. Keychain is for
   small secrets, not arbitrary model data or large blobs.
2. Define a stable identity. For a generic password, use sufficiently specific primary attributes such as service
   plus account. Keep read, update, and delete predicates at least as specific as the add predicate.
3. Choose the least permissive accessibility that satisfies the contract:
   - prefer `kSecAttrAccessibleWhenUnlocked` or its `ThisDeviceOnly` variant for foreground secrets;
   - use `kSecAttrAccessibleAfterFirstUnlock` only for justified locked/background access;
   - use `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` only when device binding and invalidation after passcode
     removal are acceptable.
4. For user-presence protection, create `SecAccessControl` with the protection value and required flags. Do not
   specify a competing `kSecAttrAccessible` in the same add query. Prefer `.userPresence` when passcode fallback is
   acceptable; use `.biometryCurrentSet` only when biometric enrollment changes must invalidate access.
5. Implement typed `SecItemAdd`, `SecItemCopyMatching`, `SecItemUpdate`, and `SecItemDelete` wrappers with narrowly
   scoped queries and explicit `OSStatus` mapping. Treat duplicate-item as an upsert decision, not permission to
   delete broadly. Run potentially blocking Security calls away from the UI thread.
6. Make `kSecAttrSynchronizable` and Keychain Sharing explicit opt-ins. Synchronization propagates updates and
   deletes and cannot be combined with `ThisDeviceOnly`; ordinary private storage does not require an access group.
7. Use `LAContext` and a clear `localizedReason` for protected reads. Add `NSFaceIDUsageDescription` when Face ID
   may be used. Do not use deprecated always-accessible values or deprecated operation-prompt APIs.

Handle at least success, item-not-found, duplicate, authentication failure, interaction-not-allowed, user cancel,
and missing entitlement distinctly. Never log secret bytes, tokens, full queries, passkey response blobs, or
credential identifiers joined with user-identifying context.

## Passkey workflow

1. Configure `webcredentials:<fully-qualified-domain>` in Associated Domains and serve a matching
   `apple-app-site-association` file over HTTPS with no redirect and the correct application identifier.
2. Obtain a fresh single-use challenge and stable opaque user ID from the relying-party server. Do not generate a
   production challenge locally or let the client choose/substitute the relying-party identifier.
3. Create `ASAuthorizationPlatformPublicKeyCredentialProvider` with the exact relying-party domain. Build a
   registration or assertion request and perform it through `ASAuthorizationController`.
4. Forward the returned raw credential fields unchanged to the server. Preserve `rawClientDataJSON` byte-for-byte;
   do not parse and rebuild it. The system retains the private key; never export, store, or transmit it.
5. Establish the app session only after the server verifies challenge freshness, origin/relying party, signature,
   authenticator data, credential, and user binding. A successful controller callback is not completed sign-in.
6. Treat cancellation, no local credential, domain/AASA failure, and server rejection as explicit fallback/error
   paths. Re-registering the same user ID may replace an existing passkey, so make recovery and account intent clear.

## Verify

- Build every affected target and inspect the signed product's effective entitlements, not only source plist files.
- Test keychain add/read/update/delete, upgrade, rotation, logout, account switching, locked/unlocked state,
  background access if claimed, passcode/biometric fallback, enrollment changes, and shared targets on physical
  devices where protection behavior matters.
- Test passkey registration, assertion, cancel, no credential, wrong domain/AASA, replay rejection, server-side
  signature verification, account recovery, and replacement behavior.
- Remove Associated Domains development alternate modes from release entitlements.

## Output

State the security contract, item identity and protection choices, synchronization/access-group decision, handled
`OSStatus` cases, passkey client/server boundary, effective entitlements checked, and exact device/runtime coverage.
Call out unverified server behavior instead of assuming the client proves authentication security.
