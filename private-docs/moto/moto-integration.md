# DNA SDK for Moto - Integration Guide

**Last audited:** April 23, 2026  
**Current SDK artifact:** `com.devicenative.dna:moto:1.6.8`  
**Current SDK version string:** `moto-v1.6.8`

This guide reflects the current Moto SDK implementation in `dn-sdk` on the `motorola` branch. It replaces the older `1.3.0` integration notes and calls out the SDK behavior that partners need to handle explicitly.

## 1. SDK Requirements

- Android `minSdk` is `24`
- The SDK is published from Maven Central as `com.devicenative.dna:moto`
- The SDK is designed for launcher and system-surface integrations that can enumerate installed apps and, ideally, read usage stats

Add the dependency to your launcher app:

```kotlin
dependencies {
    implementation("com.devicenative.dna:moto:1.6.8")
}
```

If you are still pinned to an older Moto artifact, coordinate the upgrade with Device Native instead of mixing this guide with an older SDK build.

## 2. Manifest Entries

Register the orchestrator and config builder services in your app manifest:

```xml
<service
    android:name="com.devicenative.dna.DNADataOrchestrator"
    android:exported="false" />

<service
    android:name="com.devicenative.dna.utils.DNAConfigBuilder"
    android:process=":dna_config_builder"
    android:exported="false" />
```

Host-app permissions:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
```

Notes:

- `INTERNET` is required.
- `QUERY_ALL_PACKAGES` is required for launcher-wide app discovery and correct result shaping.
- `PACKAGE_USAGE_STATS` is strongly recommended for the best ranking and freshness. If you intentionally do not use app usage signals, set `config.disableAppUsage = true` so the behavior is explicit.
- The SDK library already declares `android.permission.ACCESS_NETWORK_STATE`; you usually do not need to add it manually.

## 3. Initialize Early and Wait for Data Readiness

Initialize the SDK from your `Application` class. Initial app indexing and sync happen asynchronously, so do not assume the full result set is ready immediately after `init(...)`.

```java
public final class MotoLauncherApp extends Application {
    private static final String DEVICE_KEY = "YOUR_MOTO_DEVICE_KEY";

    @Override
    public void onCreate() {
        super.onCreate();

        DeviceNativeAds dna = DeviceNativeAds.getInstance(this);

        DeviceNativeAds.DNAConfig config = new DeviceNativeAds.DNAConfig();
        config.debugMode = BuildConfig.DEBUG;
        config.countryOverride = BuildConfig.DEBUG ? "US" : null;

        dna.init(DEVICE_KEY, config, new DeviceNativeAds.PackagesRefreshedListener() {
            @Override
            public void onPackagesRefreshed(String reason, long refreshedAtMs) {
                Log.i("DNA", "Packages refreshed: " + reason + " @" + refreshedAtMs);
                // Notify search / recommendation surfaces to refresh their UI.
            }
        });
    }
}
```

Important lifecycle notes:

- `PackagesRefreshedListener` is the current SDK hook for knowing when installed-app data is ready or refreshed.
- You can also register the listener later with `addPackagesRefreshedListener(listener, true)` and remove it with `removePackagesRefreshedListener(listener)`.
- Normal result APIs already trigger debounced background refreshes. Only call `triggerDataRefresh()` when you have a real context change that should force a refresh.
- Do not rely on `Application.onTerminate()` for production cleanup on Android. Call `destroy()` only if you have a deterministic shutdown path such as tests or a controlled process teardown.

### `DNAConfig` behavior

`DNAConfig` is now a set of nullable overrides. Unset fields stay `null` and do not change the current SDK preference for that field.

| Field | Type | Behavior |
|------|------|----------|
| `debugMode` | `Boolean` | Enables verbose SDK logging when set |
| `disableGAID` | `Boolean` | Disables GAID collection when set |
| `disableAppUsage` | `Boolean` | Disables app usage signal collection when set |
| `countryOverride` | `String` | Overrides country for testing or controlled rollouts |
| `disableAds` | `Boolean` | Disables ads globally when set |
| `disableSearchInstallAds` | `Boolean` | Disables search install ads when set |
| `disableSearchAds` | `Boolean` | Disables all search ads when set |
| `disableRecomInstallAds` | `Boolean` | Disables recommendation install ads when set |
| `disableRecomAds` | `Boolean` | Disables all recommendation ads when set |
| `carrierValue` | `String` | Supplies a carrier value when your surface already knows it |

Two practical consequences:

- `new DNAConfig()` does not mean "all flags false"; it means "override only the fields I set."
- `updateConfig(config)` also only changes the fields you populate.

If you need the plain default behavior, call `dna.init(deviceKey)` with no config, or explicitly set every field you care about.

## 4. Search Integration

For Moto search surfaces, use the immediate-display API and pass the number of results you will actually render:

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> results = dna.getOrganicResultsForSearch(
        query,
        "moto-search",
        visibleResultCount
);
```

Notes:

- This API is synchronous and safe to call on the main thread.
- It automatically fires impressions for eligible items when they are returned.
- Only request as many items as you intend to render immediately. If you need prefetch or caching behavior, coordinate with Device Native and use the cache APIs plus manual impression handling.
- The SDK already debounces refresh work internally; you do not need to add a second debounce layer just to protect the SDK.

Useful `DNAResultItem` fields for Moto search:

| Field | Description |
|------|-------------|
| `id` | Stable item ID used for tracking |
| `resultType` | Result kind such as `TYPE_APP`, `TYPE_AD`, `TYPE_SHORTCUT`, or `TYPE_NOTIFICATION` |
| `packageName` | Advertiser or app package |
| `className` | Activity / component name when available |
| `userHandle` / `uId` | Multi-user and work-profile context |
| `isInstalled` | Whether the app is already installed |
| `title` / `description` | UI copy to render |
| `iconUrl` | Remote creative icon for non-installed or promoted results |
| `clickUrl` | Tracking URL used by the SDK click flow |
| `destinationUrl` | Final destination domain the click router expects |
| `source` | Result source, useful for analytics/debugging |
| `placementTag` | Placement tag returned with the result |

For icon handling:

- Use the package manager icon for installed apps.
- Use `loadCreativeDrawableAsync(context, callback)` for promoted results or other non-installed results.

```java
if (resultItem.isInstalled) {
    Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
    itemIcon.setImageDrawable(icon);
} else {
    resultItem.loadCreativeDrawableAsync(this, new DNAResultItem.ImageCallback() {
        @Override
        public void onImageLoaded(Drawable icon) {
            runOnUiThread(() -> itemIcon.setImageDrawable(icon));
        }

        @Override
        public void onError(String message) {
            Log.e("MotoSearch", "Icon load failed: " + message);
        }
    });
}
```

## 5. Recommendation Surfaces

The current Moto SDK exposes three distinct recommendation-style result surfaces:

### App suggestions

```java
List<DNAResultItem> appSuggestions =
        DeviceNativeAds.getInstance(this).getOrganicAppSuggestions(6, "moto-suggested-apps");
```

Use this for app recommendation rows where organic app results and ads can be mixed together.

### Link suggestions

```java
List<DNAResultItem> linkSuggestions =
        DeviceNativeAds.getInstance(this).getOrganicLinkSuggestions(3, "moto-suggested-links");
```

Use this for deep links, shortcuts, notification-derived entries, and promoted content that should appear in a suggestions surface.

### Hot apps

```java
List<DNAResultItem> hotApps =
        DeviceNativeAds.getInstance(this).getHotAppsList(10, "moto-hot-apps");
```

Use this for download-focused app discovery sections.

All three APIs:

- are synchronous and safe on the main thread,
- can auto-fire impressions for eligible items,
- return results ordered for relevance / monetization,
- accept an optional placement tag for attribution.

## 6. Route Clicks Through the SDK

Use SDK click routing for both ads and organic results. This keeps click tracking, redirect handling, deep linking, and fallback launch behavior aligned with the current SDK implementation.

```java
DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem, new DeviceNativeClickHandler() {
    @Override
    public void onClickServerCompleted() {
        Log.i("DNA", "Click tracking completed");
    }

    @Override
    public void onClickRouterCompleted(boolean didRoute) {
        if (!didRoute) {
            Log.e("DNA", "No route target was opened");
        }
    }

    @Override
    public void onFailure(int errorCode, String errorMessage) {
        Log.e("DNA", "Click failed: " + errorMessage);
    }
});
```

Important click notes:

- Show a short loading state while routing is in progress.
- Use the SDK callback names exactly as defined above. The older `onAdClickRouterCompleted(...)` callback name is not the current Moto SDK interface.
- If Motorola needs to override the destination store URL for a surface, use the overload `fireClickAndRoute(resultItem, destinationUrlOverride, clickHandler)`.

## 7. Optional Advanced Controls

Hide a package from future suggestion results:

```java
DeviceNativeAds.getInstance(this).setPackageSuggestionVisibility(
        "com.example.app",
        android.os.Process.myUserHandle(),
        false
);
```

Hide a specific component from future suggestion results:

```java
DeviceNativeAds.getInstance(this).setComponentSuggestionVisibility(
        "com.example.app/.MainActivity",
        android.os.Process.myUserHandle(),
        false
);
```

If you need a lightweight snapshot of what the SDK currently thinks is installed, use:

- `getInstalledAppsLite()`
- `getInstalledAppsLiteJson()`

Those are primarily useful for debugging and diagnostics rather than normal partner UI flows.

## 8. Optional Notification Signal Integration

If your Moto surface already runs a `NotificationListenerService`, forward posted notifications to the SDK so it can use that signal for more relevant suggestions and creatives.

Manifest entry:

```xml
<service
    android:name=".notification.NotificationListener"
    android:enabled="true"
    android:exported="true"
    android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
    <intent-filter>
        <action android:name="android.service.notification.NotificationListenerService" />
    </intent-filter>
</service>
```

Forward notifications:

```java
@Override
public void onNotificationPosted(StatusBarNotification sbn) {
    DeviceNativeAds.getInstance(getApplicationContext()).onNotificationPosted(sbn);
}
```

## 9. Reference Docs

- [Moto EULA](moto-eula.md)
- [General SDK Integration](integrate-sdk.md)
- [Organic/Ads - Recommendations](rec-organic.md)
- [Organic/Ads - Search](search-organic.md)
- [Ads Only - Hot Apps](hot-app-suggestions.md)

For integration support, email [help@devicenative.com](mailto:help@devicenative.com).
