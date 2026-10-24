---
sidebar_position: 1
title: Installation
---

# Android SDK V2 Installation

:::note Java and Kotlin projects are both supported
The SDK is written in Kotlin but is **fully usable from Java**. Projects using Groovy DSL (`build.gradle`) and Java source files work with V2 without any extra configuration. Build script examples below are shown in **Kotlin DSL** (`.kts`) by default, with Groovy DSL equivalents provided for projects that haven't migrated.
:::

## 1. Add Repository

**Kotlin DSL** (`settings.gradle.kts`):

```kotlin title="settings.gradle.kts"
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

**Groovy DSL** (`settings.gradle`):

```groovy title="settings.gradle"
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url "https://jitpack.io" }
    }
}
```

> If your game enables **TikTok login**, also add the ByteDance repo:
>
> Kotlin DSL:
> ```kotlin
> maven { url = uri("https://artifact.bytedance.com/repository/AwemeOpenSDK") }
> ```
> Groovy DSL:
> ```groovy
> maven { url "https://artifact.bytedance.com/repository/AwemeOpenSDK" }
> ```
>
> **Older projects (pre-AGP 7):** If your project doesn't use `dependencyResolutionManagement` in `settings.gradle`, add the repo to `allprojects { repositories {} }` in your project-level `build.gradle` instead.

## 2. Add the SDK Dependency

**Kotlin DSL** (`app/build.gradle.kts`):

```kotlin title="app/build.gradle.kts"
dependencies {
    implementation("com.github.alo-game:alogame-sdk-android:1.5.1")

    // Required for Google login — AGP implementation scope is not transitive,
    // so these must be declared explicitly even though they're in the SDK's POM.
    implementation("androidx.credentials:credentials:1.3.0")
    implementation("androidx.credentials:credentials-play-services-auth:1.3.0")
    implementation("com.google.android.libraries.identity.googleid:googleid:1.1.0")
}
```

**Groovy DSL** (`app/build.gradle`):

```groovy title="app/build.gradle"
dependencies {
    implementation 'com.github.alo-game:alogame-sdk-android:1.5.1'

    // Required for Google login
    implementation 'androidx.credentials:credentials:1.3.0'
    implementation 'androidx.credentials:credentials-play-services-auth:1.3.0'
    implementation 'com.google.android.libraries.identity.googleid:googleid:1.1.0'
}
```

:::info Auto-resolved transitive dependencies
The following are resolved automatically **if your project has `google()` in its repositories**:
- Google Play Billing (`billing-ktx:8.0.0`)
- Firebase Cloud Messaging (`firebase-messaging-ktx`)
- Adjust SDK, Retrofit, OkHttp, and all other internal SDK dependencies
:::

## 3. Optional: Facebook Login

Facebook is **not in the SDK's POM** (`compileOnly`). Add only if your game enables it:

```kotlin title="app/build.gradle.kts (Kotlin DSL)"
dependencies {
    implementation("com.facebook.android:facebook-login:latest.release")
}
```

```groovy title="app/build.gradle (Groovy DSL)"
dependencies {
    implementation 'com.facebook.android:facebook-login:latest.release'
}
```

Add string resources:

```xml title="res/values/strings.xml"
<string name="facebook_app_id">YOUR_FACEBOOK_APP_ID</string>
<string name="facebook_client_token">YOUR_FACEBOOK_CLIENT_TOKEN</string>
<string name="fb_login_protocol_scheme">fbYOUR_FACEBOOK_APP_ID</string>
```

Add manifest entries:

```xml title="AndroidManifest.xml"
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <queries>
        <package android:name="com.facebook.katana" />
    </queries>

    <application ...>
        <meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id" />
        <meta-data android:name="com.facebook.sdk.ClientToken" android:value="@string/facebook_client_token" />

        <activity
            android:name="com.facebook.FacebookActivity"
            android:configChanges="keyboard|keyboardHidden|screenLayout|screenSize|orientation"
            android:label="@string/app_name" />

        <activity android:name="com.facebook.CustomTabActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="@string/fb_login_protocol_scheme" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

## 4. Optional: TikTok Login

TikTok is **not in the SDK's POM** (`compileOnly`). Add only if your game enables it:

```kotlin title="app/build.gradle.kts (Kotlin DSL)"
dependencies {
    implementation("com.tiktok.open.sdk:tiktok-open-sdk-core:latest.release")
    implementation("com.tiktok.open.sdk:tiktok-open-sdk-auth:latest.release")
}
```

```groovy title="app/build.gradle (Groovy DSL)"
dependencies {
    implementation 'com.tiktok.open.sdk:tiktok-open-sdk-core:latest.release'
    implementation 'com.tiktok.open.sdk:tiktok-open-sdk-auth:latest.release'
}
```

And add the ByteDance Maven repository to `settings.gradle` / `settings.gradle.kts` (see Step 1 above).

:::info No manifest changes needed
`TikTokEntryActivity` is declared in the SDK's own `AndroidManifest.xml` and is merged into your app automatically. You do not need to add anything to your manifest for TikTok.

The SDK manifest also declares **Android App Links** (`android:autoVerify="true"`) for the Alogame callback domains:
- `api-sdk.dev.alogame.vn` (dev)
- `api-sdk.alogame.vn` (production)

The OS verifies these at install time via `/.well-known/assetlinks.json` served by the Alogame backend. When TikTok redirects to the callback URL after authorization, the OS intercepts it and opens your app directly — no browser involved.

Your app's **SHA-256 signing fingerprint** must be registered in the Alogame CMS (App Links section) for verification to succeed.
:::

## 5. Manifest Setup {#manifest-setup}

Minimal required entries — internet permission only:

```xml title="AndroidManifest.xml"
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```


### Push notifications (optional)

The SDK uses **Firebase Cloud Messaging (FCM)** for push notifications — this is the only Firebase service used. `google-services.json` is only required if your game enables push notifications.

Firebase is already resolved transitively. **No Firebase dependency declaration is needed.** You only need:

1. Place `google-services.json` in your `app/` folder.
2. Apply the Google Services plugin in your **app-level** build file:

```kotlin title="app/build.gradle.kts (Kotlin DSL)"
plugins {
    id("com.google.gms.google-services")
}
```

```groovy title="app/build.gradle (Groovy DSL)"
apply plugin: 'com.google.gms.google-services'
```

3. Add the plugin classpath to your **root** build file:

```kotlin title="build.gradle.kts (Kotlin DSL, root)"
plugins {
    id("com.google.gms.google-services") version "4.4.1" apply false
}
```

```groovy title="build.gradle (Groovy DSL, root)"
buildscript {
    dependencies {
        classpath 'com.google.gms:google-services:4.4.1'
    }
}
```

## 6. Initialize the SDK

Initialize once in your `Application` class — just a Game ID, no config file needed. All other behaviour (user info fields, feature flags, tracking config) is loaded from the server at runtime.

```kotlin title="App.kt (Kotlin)"
import vn.oeg.sdk.v2.legacy.AlogameSdk

class App : Application() {
    override fun onCreate() {
        super.onCreate()
        AlogameSdk.init(this, YOUR_GAME_ID)
    }
}
```

```java title="App.java (Java)"
import vn.oeg.sdk.v2.legacy.AlogameSdk;

public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        AlogameSdk.init(this, YOUR_GAME_ID);
    }
}
```

### Pointing at the dev backend

Pass `debugEnv = true` (since **1.4.1**) to use the dev API host instead of production. Omit it — or pass `false` — for production. This mirrors iOS's `Builder().setDebugEnv(true)`.

```kotlin title="Kotlin"
AlogameSdk.init(this, YOUR_GAME_ID, debugEnv = true)  // dev backend
AlogameSdk.init(this, YOUR_GAME_ID)                   // production (default)
```

```java title="Java"
AlogameSdk.init(this, YOUR_GAME_ID, true);  // dev backend
AlogameSdk.init(this, YOUR_GAME_ID);        // production (default)
```

:::warning
`debugEnv = true` also enables verbose HTTP request/response logging. Never ship a release build with it on.
:::

### Alternative: `oeg_config.json`

Prefer a config file (e.g. to vary `game_id` per build flavor, or to set advanced options like `debug_env`, Sentry, or local SNS fallbacks without a code change)? Place one at `app/src/main/assets/oeg_config.json` and call the file-based overload instead:

```kotlin title="Kotlin"
AlogameSdk.init(this) // reads app/src/main/assets/oeg_config.json
```

```java title="Java"
AlogameSdk.init(this); // reads app/src/main/assets/oeg_config.json
```

```json title="app/src/main/assets/oeg_config.json"
{
  "core": {
    "game_id": YOUR_GAME_ID
  }
}
```

<details>
<summary>Full reference (all optional local keys)</summary>

```json
{
  "core": {
    "game_id": YOUR_GAME_ID,
    "debug_env": true
  },
  "logging": {
    "enable_timber": true,
    "sentry_dsn": ""
  },
  "sns": {
    "google_web_client_id": "YOUR_GOOGLE_WEB_CLIENT_ID",
    "facebook_app_id": "YOUR_FACEBOOK_APP_ID",
    "facebook_client_token": "YOUR_FACEBOOK_CLIENT_TOKEN",
    "fb_login_protocol_scheme": "fbYOUR_FACEBOOK_APP_ID",
    "tiktok_client_key": "YOUR_TIKTOK_CLIENT_KEY"
  }
}
```

> The `sns` block supplements server-delivered config. Values in `strings.xml`/`AndroidManifest.xml` (required by Facebook SDK) take precedence over this block.

</details>

## 7. Quick Smoke Test

```kotlin title="Kotlin"
// Show login UI
AlogameSdk.showLogin(callback)

// Show full dashboard (profile, change password, logout…)
AlogameSdk.showDashboard()
```

```java title="Java"
// Show login UI
AlogameSdk.showLogin(callback);

// Show full dashboard (profile, change password, logout…)
AlogameSdk.showDashboard();
```

