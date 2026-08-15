# JustSpot — Legal

Privacy Policy and Terms of Use for **JustSpot — Trading Journal**, published by Fazogirapps.

Served via GitHub Pages from the **`justspot-privacy`** repository:

| Document | URL |
|---|---|
| Privacy Policy | https://elyorberdiev.github.io/justspot-privacy/ |
| Terms of Use | https://elyorberdiev.github.io/justspot-privacy/terms.html |

**Why `index.html` is the Privacy Policy itself** and not a landing page: the root URL was already
submitted to App Store Connect and the Google Play Console as the Privacy Policy URL. Making the root
a landing page would leave the registered link pointing at something that is not the policy.
**Do not rename these files** — changing a path breaks the links already registered with the stores.

## Updating

This folder is the source of truth. To publish:

```bash
git clone https://github.com/elyorberdiev/justspot-privacy.git /tmp/justspot-privacy
cp -R legal/. /tmp/justspot-privacy/
cd /tmp/justspot-privacy && git add -A && git commit -m "Update legal documents" && git push
```

Bump the "Last Updated" date in the file header before publishing. GitHub Pages redeploys within a
minute or two. For material changes, also update the effective date and give users notice in-app.

## Claims that depend on the app's actual behaviour

These documents describe what the app does today. If a feature changes, the corresponding text must
change too — an inaccurate privacy policy is worse than none.

| Text says | True only while |
|---|---|
| "never connects to a broker or exchange", "no API keys" | No broker/exchange integration is added. This is the central promise of §3 of the Privacy Policy and §4 of the Terms. |
| "signed in anonymously … may sign in with Google or Apple" | Matches `subscription_bloc.dart` (`signInAnonymously`, Google Sign-In, Sign in with Apple). Adding email/password or another provider means updating §2. |
| The trade-field list in §4.1 | Matches the `Trades` table in `lib/features/trade_journal/data/datasources/trade_database.dart` (symbol, assetType, entry/exitPrice, quantity, date, strategy, emotion, mistake, shariahScreened, notes, imagePath). Adding a column means adding it here. |
| "synced to Cloud Firestore … images to Cloud Storage" | `firestore_remote_data_source.dart` keeps writing to Firestore/Storage under the user's uid. |
| "Firebase Crashlytics, Analytics, Messaging, Remote Config, App Check" and "PostHog" | All are actually initialised in the released build. Remove any row that is dropped. |
| Analytics events `sign_up`, `login`, `account_deleted` | Matches `lib/core/analytics/analytics_helper.dart`. Update §5 when events are added. |
| Deletion wording in §11 | `_deleteAccount()` in `auth_profile_sheet.dart` calls `TradeRepository.deleteAccountData()` — which erases the Storage images, the `trades` subcollection and the `users/{uid}` document — *before* `user.delete()`. If that order is ever changed, or the cascade removed, §11 becomes a false promise. |
| "Camera / Photo library" permissions in §9 | `Info.plist` keeps `NSCameraUsageDescription` and `NSPhotoLibraryUsageDescription`. |

## How account deletion works

Both issues that once made §11 inaccurate are fixed (2026-08-15):

1. **Cascade delete.** `FirestoreRemoteDataSource.deleteAllUserData()` removes the Storage images
   (listing the folder, with per-document deletion as a fallback), then every document in the
   `trades` subcollection in batches, then the `users/{uid}` document. `TradeRepositoryImpl
   .deleteAccountData()` calls it and then clears the local Drift database. The UI runs all of that
   **before** `user.delete()`, because the security rules need a valid auth token.
2. **Recent-login pre-check.** Firebase rejects `user.delete()` when the last sign-in is stale.
   `_deleteAccount()` checks `user.metadata.lastSignInTime` first and aborts *before* deleting
   anything, so a refusal can never leave the account alive with an already-erased journal. If the
   check is passed but deletion still fails, the user is told the data is gone and to sign in again
   (`delete_account_partial`).
3. **Storage rule.** `storage.rules` allows the owner to `list` `users/{userId}/trades`, which the
   image cleanup needs. **Deploy it** — `firebase deploy --only storage` — or the cleanup silently
   falls back to per-document deletion and leaves orphaned images behind.
4. **iOS usage descriptions.** `NSCameraUsageDescription` and `NSPhotoLibraryUsageDescription` are
   now in `ios/Runner/Info.plist`. They are English only; add `InfoPlist.strings` per language if you
   want them localized like the rest of the app.

## Publishing checklist

1. Copy this folder into the **public** repo `justspot-privacy` (see "Updating" above).
2. Settings → Pages → Source: `main` branch, `/` root — already configured.
3. Open both URLs and check them on a phone.
4. Put the URLs into:
   - the profile/settings screen of the app (Privacy Policy, Terms of Use)
   - App Store Connect → App Privacy → Privacy Policy URL
   - App Store Connect → App Information → EULA (or link the Terms)
   - Google Play Console → Store listing → Privacy Policy
