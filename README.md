# Verifying Official Google Play Services on Android Devices

## Problem Overview

On some Android devices that lack Google Play Services, particularly **Huawei phones**, users often install unofficial alternatives to Google Play Services, such as [**MicroG**](https://microg.org/). MicroG is an open-source project that provides a replacement for Google Play Services and attempts to replicate its functionality without relying on proprietary software from Google.

However, **MicroG** does not fully replicate all Google services, and some services (e.g. **Google Pay**) are not supported. This can lead to **crashesor**  or **errors** in apps that rely on the full functionality of the official **Google Play Services** (GMS Core).

To ensure that users are running the official, unmodified Google Play Services and prevent issues with unsupported services, we can check the cryptographic signature of the installed Google Play Services APK. This ensures that only the version signed by Google is allowed.

## Contents

- [Verifying Official Google Play Services on Android Devices](#verifying-official-google-play-services-on-android-devices)
  - [Problem Overview](#problem-overview)
  - [Contents](#contents)
  - [What is MicroG?](#what-is-microg)
    - [Supported Services in MicroG:](#supported-services-in-microg)
    - [Unsupported or Incomplete Services in MicroG:](#unsupported-or-incomplete-services-in-microg)
  - [Solution: Verifying Official Google Play Services](#solution-verifying-official-google-play-services)
    - [How to Implement](#how-to-implement)
      - [Kotlin Example](#kotlin-example)
      - [Java Example](#java-example)
    - [HMS SDK support](#hms-sdk-support)
    - [Handle exception gracefully](#handle-exception-gracefully)
  - [Disclaimer](#disclaimer)


## What is MicroG?

**MicroG** is an open-source implementation that aims to provide similar functionality to Google Play Services using free and open-source software (FOSS). It’s used mainly on devices where official Google Play Services are not available, such as on Huawei devices without access to the Google ecosystem.

### Supported Services in MicroG:

- **GCM/FCM (Google Cloud Messaging / Firebase Cloud Messaging)**
- **Location Services** (using open-source location databases)
- **Maps API** (basic features only)
- **Google Account Login** (limited support)
- **SafetyNet** (partial support, spoofing mechanisms required)
- **Google Play Store** (partial compatibility through third-party apps)
- ...

### Unsupported or Incomplete Services in MicroG:

- **Google Pay** (Not supported, cannot handle payments)
- **Widevine DRM** (No support for DRM-protected content)
- **Google Play Games Services** (Limited or no support) 
- **Nearby Share** (No support)
- ...

Learn more about MicroG Implementation Status here: [MicroG implementation status](https://github.com/microg/GmsCore/wiki/Implementation-Status)

** Due to these limitations, apps that rely on unsupported services may **crash** or behave unpredictably when running with MicroG or other unofficial replacements.**

## Solution: Verifying Official Google Play Services

To prevent users from running your app with unofficial Google Play Services (like MicroG) or partial replacements, you can verify that the installed Google Play Services (GMS Core) on the device is the **official version signed by Google**.

There are currently **4 known official Google Play Services SHA-1 signatures**:

- `BD32424203E0FB25F36B57E5AA356F9BDD1DA998`
- `38918A453D07199354F8B19AF05EC6562CED5788`
- `58E1C4133F7441EC3D2C270270A14802DA47BA0E`
- `2169EDDB5FBB1FDF241C262681024692C4FC1ECB`

Below are Kotlin and Java functions that check if the installed Google Play Services APK matches one of these official certificates.

### How to Implement

Before initializing or invoking any Google service that is dependent on the official GMS Core, verify that the GMS Core available on the user's device is the **official version signed by Google**.
  
#### Kotlin Example

```kotlin
val isOfficial = isOfficialGooglePlayServicesInstalled(packageManager)
if (isOfficial) {
    // Proceed with the app's functionality
    initializeGooglePay()
} else {
    // Show a warning or disable the service
    disableGooglePayOrShowWarning()
}
```

 Kotlin function to Verify Official Google Play Services

```kotlin
import android.content.pm.PackageManager
import android.content.pm.PackageInfo
import android.os.Build
import java.security.MessageDigest

fun isOfficialGooglePlayServicesInstalled(packageManager: PackageManager): Boolean {
    // List of official SHA-1 fingerprints of Google Play Services
    val officialGooglePlaySHA1 = listOf(
        "BD32424203E0FB25F36B57E5AA356F9BDD1DA998",
        "38918A453D07199354F8B19AF05EC6562CED5788",
        "58E1C4133F7441EC3D2C270270A14802DA47BA0E",
        "2169EDDB5FBB1FDF241C262681024692C4FC1ECB"
    )
    
    try {
        // Get the package info of Google Play Services (GMS Core)
        val packageInfo: PackageInfo = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
            // Use the getPackageInfo with flags for signatures for API 28 and above
            packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNING_CERTIFICATES)
        } else {
            // Use the legacy method for API 27 and below
            packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNATURES)
        }
        
        // Get the signature(s) of the package
        val signatures = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
            packageInfo.signingInfo.apkContentsSigners
        } else {
            packageInfo.signatures
        }

        // Iterate through the signatures and verify
        for (signature in signatures) {
            // Compute the SHA-1 hash of the signature
            val sha1 = MessageDigest.getInstance("SHA-1").digest(signature.toByteArray())
            val sha1Hex = sha1.joinToString("") { "%02X".format(it) }
            
            // Check if the SHA-1 matches any of the official Google Play Services signatures
            if (officialGooglePlaySHA1.contains(sha1Hex)) {
                return true // Official GMS Core installed
            }
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
    
    // If no match, GMS Core is NOT official or missing
    return false
}
```

#### Java Example

<details> 
<summary> show JAVA code </summary>

```java
boolean isOfficial = isOfficialGooglePlayServicesInstalled(getPackageManager());
if (isOfficial) {
    // Proceed with the app's functionality
    initializeGooglePay()
} else {
    // Show a warning or disable the service
    disableGooglePayOrShowWarning()
}
```

Java Method to Verify Official Google Play Services
 
```java
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.os.Build;
import java.security.MessageDigest;
import java.util.Arrays;
import java.util.List;

public boolean isOfficialGooglePlayServicesInstalled(PackageManager packageManager) {
    // List of official SHA-1 fingerprints of Google Play Services
    List<String> officialGooglePlaySHA1 = Arrays.asList(
        "BD32424203E0FB25F36B57E5AA356F9BDD1DA998",
        "38918A453D07199354F8B19AF05EC6562CED5788",
        "58E1C4133F7441EC3D2C270270A14802DA47BA0E",
        "2169EDDB5FBB1FDF241C262681024692C4FC1ECB"
    );
    
    try {
        // Get the package info of Google Play Services (GMS Core)
        PackageInfo packageInfo;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
            // Use getPackageInfo with flags for signatures for API 28 and above
            packageInfo = packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNING_CERTIFICATES);
        } else {
            // Use the legacy method for API 27 and below
            packageInfo = packageManager.getPackageInfo("com.google.android.gms", PackageManager.GET_SIGNATURES);
        }

        // Get the signature(s) of the package
        android.content.pm.Signature[] signatures;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
            signatures = packageInfo.signingInfo.getApkContentsSigners();
        } else {
            signatures = packageInfo.signatures;
        }

        // Iterate through the signatures and verify
        for (android.content.pm.Signature signature : signatures) {
            // Compute the SHA-1 hash of the signature
            MessageDigest md = MessageDigest.getInstance("SHA-1");
            byte[] sha1 = md.digest(signature.toByteArray());
            StringBuilder sha1Hex = new StringBuilder();
            for (byte b : sha1) {
                sha1Hex.append(String.format("%02X", b));
            }

            // Check if the SHA-1 matches any of the official Google Play Services signatures
            if (officialGooglePlaySHA1.contains(sha1Hex.toString())) {
                return true; // Official GMS Core installed
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    
    // If no match, GMS Core is NOT official or missing
    return false;
}
```

</details>

---

### HMS SDK support

If the app is published to Huawei AppGallery, you must add the folllowing metadata to your `AndroidManifest.xml` file:

```xml
<manifest>
<!--  ...  -->
    <application>
        <!--  ...  -->
        
        <meta-data
            android:name="com.huawei.hms.client.service.name:hwid"
            android:value="hwid:6.3.0.300" />

    </application>
</manifest>
```

## Handle exception gracefully

If the above does not work and could not be implmentated, you can simply wrap the GMS service initializatin in try-catch block to prevent crashing

```kotlin
try{
  googlePayButton.initialize()
} catch (e:Exception){
  Log.w("MyApp","Google Wallet module not available or not official")
  //TODO: hide GooglePay button or use alternative payment service/SDK
}
```

## Disclaimer 

This code is provided **without warranty of any kind**. It may not be updated or maintained in the future.

The SHA-1 signatures listed here were collected from various sources on the internet and may be subject to change. **Google may introduce new signatures or modify existing ones without notice** .

It is your responsiblity to carefully review and test the implementation before deploying it in production environments.






 
