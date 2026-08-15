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
| "Camera / Photo library" permissions in §9 | The app really does offer both `ImageSource.camera` and `ImageSource.gallery` in `add_trade_tab.dart`. See the open issue below. |
| Deletion wording in §11 | **Read this before changing it.** `auth_profile_sheet.dart` calls `user.delete()`, which removes the Firebase Auth account only — it does **not** delete the Firestore documents or Storage images. The policy therefore promises exactly that, plus an email route for erasing the cloud copy within 30 days. |

## Open issues to fix in the app

1. **Account deletion does not erase cloud data.** `user.delete()` removes the auth account but leaves
   `users/{uid}` documents, the `trades` subcollection, and the uploaded images in Cloud Storage.
   Apple requires that in-app account deletion actually removes the user's data. Until a Cloud
   Function or client-side cascade delete is implemented, the email route in §11 must stay in the
   policy and must be honoured within 30 days.
2. **Missing iOS usage descriptions.** `ios/Runner/Info.plist` has no `NSCameraUsageDescription` or
   `NSPhotoLibraryUsageDescription`, yet the app calls both `ImageSource.camera` and
   `ImageSource.gallery`. On iOS this crashes at the moment of the request and is an App Review
   rejection. Add both keys before the next submission.

## Publishing checklist

1. Copy this folder into the **public** repo `justspot-privacy` (see "Updating" above).
2. Settings → Pages → Source: `main` branch, `/` root — already configured.
3. Open both URLs and check them on a phone.
4. Put the URLs into:
   - the profile/settings screen of the app (Privacy Policy, Terms of Use)
   - App Store Connect → App Privacy → Privacy Policy URL
   - App Store Connect → App Information → EULA (or link the Terms)
   - Google Play Console → Store listing → Privacy Policy
