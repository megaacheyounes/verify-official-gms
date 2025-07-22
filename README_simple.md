# Quick Fix: Check if Google Play Services is Official

### Problem

Apps relying on sensitive Google services, like Google Wallet or Google Billing, might crash or have issues on Android devices where unofficial Google services are in use. For example, many Huawei users choose to rely on MicroG, which is an unofficial alternative to Google Mobile Services

### Fix

Before using sensitive Google SDKs, **check if Google Play Services installed on the device is official** (signed by Google). If not, skip the feature or show a message.

---

### Simple Kotlin Check

```kotlin
val isOfficial = isOfficialGooglePlayServicesInstalled(packageManager)
if (isOfficial) {
    initializeGooglePay() // or other sensitive GMS SDK
} else {
    disableGooglePayOrShowWarning()
}
```

```kotlin
    fun isOfficialGooglePlayServicesInstalled(packageManager: PackageManager): Boolean {
        val officialSHA1 = listOf(
            "BD32424203E0FB25F36B57E5AA356F9BDD1DA998",
            "38918A453D07199354F8B19AF05EC6562CED5788",
            "58E1C4133F7441EC3D2C270270A14802DA47BA0E",
            "2169EDDB5FBB1FDF241C262681024692C4FC1ECB",
            "4F87463A1AE6F7D71B2C0B0658845790236DBA42",
            "ECD58F7F5413491021BD78EFE82F3B8206BD5FAF"
        )
        return try {
            val info = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P)
                packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNING_CERTIFICATES)
            else
                packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNATURES)

            val signatures = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P)
                info.signingInfo?.apkContentsSigners ?: emptyArray()
            else
                info.signatures ?: emptyArray()

            signatures.any { sig ->
                val sha1 = MessageDigest.getInstance("SHA-1").digest(sig.toByteArray())
                val hex = sha1.joinToString("") { "%02X".format(it) }
                hex in officialSHA1
            }
        } catch (e: Exception) {
            false
        }
    }
```
---

### For Huawei HMS devices

If you're releasing your app to **Huawei AppGallery**, add this metadata to `AndroidManifest.xml`:

```xml
<manifest>
<!--  [...]  -->
<application>
    <!--  [...]  -->

    <meta-data
        android:name="com.huawei.hms.client.service.name:hwid"
        android:value="hwid:6.3.0.300" />

</application>
</manifest>
```

---

### 🛡️ Optional: Fallback with Try-Catch

Use try-catch to gracefully handle crashes in case above solution fail or can't be implemented:

```kotlin
try {
    initializeGooglePay() // or other sensitive GMS SDK
} catch (e: Exception) {
    Log.w("MyApp", "GMS not available or not official")
    // Hide button or use another SDK
    disableGooglePayOrShowWarning() 
}
```

---

### Disclaimer

This code is provided **without warranty**. SHA-1 values may change in the future. **Google may add or change signatures.** Please **test well** before using in production.

For more details, read the full gist: [https://github.com/megaacheyounes/verify-official-gms](https://github.com/megaacheyounes/verify-official-gms)
