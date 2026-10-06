# OTP SMS auto-fill (Android)

When a parent signs in with their phone number, the app reads the 6-digit code
out of the incoming SMS and submits it, so they never leave the app to go and
copy it from Messages.

## How it works

Android will never let an app read the SMS inbox without the `READ_SMS`
permission — and Google Play effectively bans that permission for apps that only
want an OTP. Instead, Google Play Services offers two APIs that hand over **one
specific message**, and nothing else:

| | SMS Retriever | SMS User Consent |
|---|---|---|
| User sees | nothing — silent | one dialog: "Allow DreamSpace Academy to read this message?" |
| Permission | none | none |
| Message must contain | an 11-char hash of our package + signing cert | a 4–10 char code, sender not in contacts |
| Server must change the SMS | **yes** — `<#>` prefix + hash | no |
| Window | 5 minutes from `startSmsRetriever()` | 5 minutes from `startSmsUserConsent()` |

The hash is what makes SMS Retriever safe: Play Services computes
`sha256(packageName + " " + signingCertificate)`, truncates it, and only releases
messages ending in that string. An app can therefore only ever read texts that
were addressed to it on purpose.

We start **both** at once and take whichever fires first:

- Hash configured and matching → silent auto-fill, nothing to tap.
- Hash missing or from a different signing key → the consent dialog appears,
  one tap, then auto-fill.
- Neither (no Play Services, iOS, browser) → nothing changes; the user types the
  code. On iOS the OS does this itself: the keyboard offers the code above any
  field marked `autocomplete="one-time-code"`, which ours already is.

## Where the code lives

| Piece | File |
|---|---|
| Native plugin (both APIs, code extraction, hash calc) | `mobile/android/app/src/main/java/academy/dreamspace/app/SmsOtpPlugin.java` |
| Plugin registration | `mobile/android/app/src/main/java/academy/dreamspace/app/MainActivity.java` |
| Play Services dependency | `mobile/android/app/build.gradle` + `variables.gradle` |
| JS wrapper | `web/src/lib/smsOtp.ts` |
| Sign-in OTP screen | `web/src/pages/mobile/MobileLoginPage.tsx` |
| Change-my-number OTP screen | `web/src/pages/mobile/MobileProfilePage.tsx` |
| SMS body (single source) | `api/src/lib/otp.ts` → `otpSmsMessage()` |
| Hash config | `ANDROID_SMS_APP_HASH` in `api/src/config/env.ts` |

`mobile/android/` is otherwise generated and gitignored; the two Java files above
are re-included by name in `.gitignore` because they are hand-written app code.

## Configuring `ANDROID_SMS_APP_HASH`

**The hash depends on the signing certificate, so every signing key has its own.**
A locally-signed test build and the Play Store build are different apps to this
API. Only one hash can be in a given SMS, so pick the one matching the build your
users actually run (Play), and let test builds fall back to the consent prompt —
or point a local API at the test build's hash while testing.

Current values:

| Build | Signed with | Hash |
|---|---|---|
| Local release APK (`assembleRelease` + `keystore.properties`) | `dreamspace-release.jks` (upload key) | `iyjfIRB6jKv` |
| Google Play | Play app-signing key | read it from the installed app (below) |

To read the hash of any installed build, call `getSmsAppHash()` from
`web/src/lib/smsOtp.ts` and log it, or compute it from the certificate:

```bash
# From a keystore you hold (upload key):
keytool -exportcert -keystore dreamspace-release.jks -alias dreamspace-release \
  -storepass "$PW" -file cert.der
python3 - <<'EOF'
import hashlib, base64
der = open('cert.der','rb').read()
d = hashlib.sha256(("academy.dreamspace.app " + der.hex()).encode()).digest()
print(base64.urlsafe_b64encode(d[:9]).decode().rstrip('=')[:11])
EOF
```

For the Play-signed build, download the app signing certificate from
**Play Console → Test and release → App integrity → App signing key
certificate** and run the same snippet against it.

Leaving the variable unset is a safe default: the SMS is exactly what it is
today and users get the one-tap consent prompt instead.

## Message length

With the hash set, the SMS is:

```
<#> Your DreamSpace verification code is 196919. It expires in 10 minutes.
iyjfIRB6jKv
```

86 characters — one GSM-7 segment, and under the 140-byte limit SMS Retriever
enforces. Do not add to it without re-checking both.
