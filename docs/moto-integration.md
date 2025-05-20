---
hide:
  - navigation
  - toc
---

# DNA - Motorola Integration Guide

**Date:** 2025-05-20

Below are the proposed integration steps for Motorola's implementation of the DNA SDK.

1. [DNA SDK Integration](#dna-sdk-integration)
2. [Search Integration](#search-integration)
    1. [User enters character in search bar](#1-user-enters-character-in-search-bar)
    2. [Call the DNA SDK to get search results](#2-call-the-dna-sdk-to-get-search-results)
    3. [Load ad creative in the UI](#3-load-ad-creative-in-the-ui)
    4. [Send user click to DNA for routing](#4-send-user-click-to-dna-for-routing)
3. [Recommended Apps Integration](#recommended-apps-integration)
    1. [User opens app recommender](#1-user-opens-app-recommender)
    2. [Call the DNA SDK to get recommended apps](#2-call-the-dna-sdk-to-get-recommended-apps)
    3. [Load ad creative in the UI](#3-load-ad-creative-in-the-ui-1)
    4. [Send user click to DNA for routing](#4-send-user-click-to-dna-for-routing-1)
    5. [Do not cache the results](#5-do-not-cache-the-results)

# DNA SDK Integration

### 1. Add Maven Central Dependency
Add the following dependency to your app's `build.gradle` file:

```gradle
dependencies {
    implementation 'com.devicenative.dna:moto:1.2.7'
}
```

### 2. Register the Data Orchestrator Service
In your AndroidManifest.xml, register the DNADataOrchestrator service and the DNAConfigBuilder service. The DNADataOrchestrator is the main service which coordinates data fetching and processing to deliver fresh advertising results. It will run in your application's process and persist. The DNAConfigBuilder is a one-time use service which runs in a separate process to retrieve the user agent for the device. It will run for approximately 1 second at startup, and not again.

```xml
<service android:name="com.devicenative.dna.DNADataOrchestrator" />
<service
    android:name="com.devicenative.dna.utils.DNAConfigBuilder"
    android:process=":dna_config_builder"
    android:exported="false"/>
```

### 3. Verify Required Permissions
Make sure to include the following permissions in your AndroidManifest.xml:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"/>
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS"/>
```

### 4. Initialize DNA SDK
Initialize the SDK in your Application class's `onCreate` method:

```java
@Override
public void onCreate() {
    super.onCreate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    
    // Basic initialization
    // dna.init("2d6e21f3-1213-4aac-9cd9-06e8ec91fa4e");
    
    // Advanced initialization with configuration
    DeviceNativeAds.DNAConfig config = new DeviceNativeAds.DNAConfig();
    config.debugMode = true; // Enable verbose logging
    config.disableGAID = false; // Keep Google Advertising ID enabled
    config.disableAppUsage = false; // Keep app usage tracking enabled
    config.countryOverride = null; // Use device's country
    config.disableAds = false; // Keep ads enabled
    
    dna.init("2d6e21f3-1213-4aac-9cd9-06e8ec91fa4e", config);

    // any other code you have
}
```

#### Configuration Options

The `DNAConfig` class provides several options to customize the behavior of the SDK:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `debugMode` | boolean | false | Enables verbose logging for debugging purposes |
| `disableGAID` | boolean | false | Disables the collection of Google Advertising ID |
| `disableAppUsage` | boolean | false | Disables app usage tracking functionality |
| `countryOverride` | String | null | Overrides the country code (ISO 3166-1 alpha-2 format, e.g., "US") |
| `disableAds` | boolean | false | Completely disables ads functionality |
| `disableSearchInstallAds` | boolean | false | Disables search install ads |
| `disableSearchAds` | boolean | false | Disables all search ads |
| `disableRecomInstallAds` | boolean | false | Disables recommended install ads |
| `disableRecomAds` | boolean | false | Disables all recommended ads |
| `carrierValue` | String | null | Sets the carrier value for the device |

#### Example: Disabling Ads for Certain Users

You can conditionally disable ads for certain user segments:

```java
DeviceNativeAds.DNAConfig config = new DeviceNativeAds.DNAConfig();

// Example: Disable ads for premium users
if (isPremiumUser()) {
    config.disableAds = true;
}

dna.init("2d6e21f3-1213-4aac-9cd9-06e8ec91fa4e", config);
```

#### Example: Testing with Debug Mode

For development and testing:

```java
DeviceNativeAds.DNAConfig config = new DeviceNativeAds.DNAConfig();
config.debugMode = BuildConfig.DEBUG;
config.countryOverride = BuildConfig.DEBUG ? "US" : null; // Override country in debug builds

dna.init("2d6e21f3-1213-4aac-9cd9-06e8ec91fa4e", config);
```

#### Update configuration after initialization

```java
DeviceNativeAds.DNAConfig config = new DeviceNativeAds.DNAConfig();
config.debugMode = BuildConfig.DEBUG;
config.countryOverride = BuildConfig.DEBUG ? "US" : null; // Override country in debug builds
DeviceNativeAds.getInstance(this).updateConfig(config);
```

### 5. Clean Up DNA Resources
In the Application class's `onTerminate` method, clean up SDK resources:

```java
@Override
public void onTerminate() {
    super.onTerminate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.destroy();

    // any other code you have
}
```

# Search Integration

### 1. User enters character in search bar
When the user begins typing in your search interface, you should call the DNA SDK to get search results.

### 2. Call the DNA SDK to get search results
This method call will return a list of DNAResultItem objects which can be shown to the user.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
// Replace 10 with the actual number of results you intend to display/trigger impressions for
List<DNAResultItem> adUnits = dna.getOrganicResultsForSearch(query, "organic search", 10);
```

#### Key Fields of DNAResultItem Class

| Parameter | Description |
|----------|-------------|
| `id` | Unique identifier for the ad. Just a UUID for reference if you need |
| `packageName` | The package name of the advertiser's app |
| `isInstalled` | A convenient boolean indicating whether the advertiser's app is installed, derived from package manager |
| `appName` | The name of the advertiser's app |
| `className` | The class name of the activity to be shown to the user. Can be null! |
| `userHandle` | The user handle of the app. Can be null! |
| `title` | The ad creative title to be shown to the user |
| `description` | The ad creative description to be shown to the user. Can be null! |
| `iconUrl` | The ad creative icon URL to be shown to the user. Can be null! |
| `ratings` | The number of ratings of the advertiser's app from Google Play |
| `downloads` | The number of downloads of the advertiser's app from Google Play |
| `rating` | The average rating of the advertiser's app from Google Play |
| `category` | The category of the advertiser's app from Google Play |

#### Helper Methods and Implementation Notes

- `public ComponentName getComponentName()` - Returns a ComponentName constructed from the packageName and className. This is useful for launching the app via an Intent.
- DNAResultItem implements the Parcelable interface, making it easy to transfer between Activities or Services.

```java
// Example of using getComponentName to launch an app
Intent launchIntent = new Intent();
launchIntent.setComponent(resultItem.getComponentName());
startActivity(launchIntent);

// Example of passing a DNAResultItem between activities
Intent intent = new Intent(this, DetailActivity.class);
intent.putExtra("result_item", resultItem); // Parcelable implementation
startActivity(intent);
```

### 3. Load ad creative in the UI
Below shows an example implementation of loading the ad creative in the UI, with DNA method calls.

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);
TextView itemDescription = itemView.findViewById(R.id.item_description);

// handle loading app icon async if app uninstalled
if (!resultItem.isInstalled) {
  resultItem.loadCreativeDrawableAsync(this, new DNAResultItem.ImageCallback() {
    @Override
    public void onImageLoaded(Drawable icon) {
      new Thread(() -> {
        runOnUiThread(() -> {
          if (icon == null) {
            try {
              Drawable backupIcon = getPackageManager().getApplicationIcon(resultItem.packageName);
              itemIcon.setImageDrawable(backupIcon);
            } catch (Exception e) {
              Log.e("SearchActivity", "Error loading app icon: " + e.getMessage());
            }
          } else {
            itemIcon.setImageDrawable(icon);
          }
        });
      }).start();
    }

    @Override
    public void onError(String message) {
      try {
        Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
        itemIcon.setImageDrawable(icon);
      } catch (Exception e) {
        Log.e("SearchActivity", "Error loading app icon: " + e.getMessage());
      }
    }
  });

  // Note, if convenient, there is a sync method to get the icon but not recommended for UI thread
  // Drawable icon = resultItem.loadCreativeDrawable();
} else {
  // if app is installed, show the app icon
  Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
  itemIcon.setImageDrawable(icon);
}

itemTitle.setText(resultItem.title);

if (resultItem.description == null || resultItem.description.isEmpty()) {
  itemDescription.setVisibility(View.GONE);
} else {
  itemDescription.setText(resultItem.description);
}
```

### 4. Send user click to DNA for routing
After the user clicks on a DNA result, send the click to DNA for routing. DNA should handle the click routing because it is important to deep link the user to the advertiser's app with the appropriate parameters.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem, new DeviceNativeClickHandler() {
  @Override
  public void onClickServerCompleted() {
    Log.i("SearchActivity", "Click tracking completed successfully.");
  }

  @Override
  public void onClickRouterCompleted(boolean didRoute) {
    if (!didRoute) {
      Log.e("SearchActivity", "Error routing click: No activity found to handle the click.");
    } else {
      Log.i("SearchActivity", "Click routed successfully.");
    }
  }

  @Override
  public void onFailure(int errorCode, String errorMessage) {
    Log.e("SearchActivity", "Error clicking ad: " + errorMessage);
  }
});
```

#### Important click handling notes
1. We recommend that you add some sort of "loading" UI after the user clicks a result. In some cases, there are delays in the click tracking and routing due to network latency.
2. Make sure to handle the failure cases, such as no activity found to handle the click, and errors in the click tracking and routing. This should be very rare, but it's important to handle it gracefully.

# Recommended Apps Integration

### 1. User opens app recommender
Adding this step to indicate that the user has opened the app recommender view, which is the trigger for the following logic.

### 2. Call the DNA SDK to get recommended apps
This method call will return a list of DNAResultItem objects which can be used for app recommendations.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getOrganicAppSuggestions(6);
```

#### Note
1. These are ordered by relevance and monetization potential, so the first app will be the most relevant.
2. This method returns results in milliseconds, so it's safe to run on the main thread.
3. You can separate re-engagement and install apps using the boolean `isInstalled` field.

#### Key Fields of DNAResultItem Class

| Parameter | Description |
|----------|-------------|
| `id` | Unique identifier for the ad. Just a UUID for reference if you need |
| `packageName` | The package name of the advertiser's app |
| `isInstalled` | A convenient boolean indicating whether the advertiser's app is installed, derived from package manager |
| `appName` | The name of the advertiser's app |
| `className` | The class name of the activity to be shown to the user. Can be null! |
| `userHandle` | The user handle of the app. Can be null! |
| `title` | The ad creative title to be shown to the user |
| `description` | The ad creative description to be shown to the user. Can be null! |
| `iconUrl` | The ad creative icon URL to be shown to the user. Can be null! |
| `ratings` | The number of ratings of the advertiser's app from Google Play |
| `downloads` | The number of downloads of the advertiser's app from Google Play |
| `rating` | The average rating of the advertiser's app from Google Play |
| `category` | The category of the advertiser's app from Google Play |

#### Helper Methods and Implementation Notes

- `public ComponentName getComponentName()` - Returns a ComponentName constructed from the packageName and className. This is useful for launching the app via an Intent.
- DNAResultItem implements the Parcelable interface, making it easy to transfer between Activities or Services.

```java
// Example of using getComponentName to launch an app
Intent launchIntent = new Intent();
launchIntent.setComponent(resultItem.getComponentName());
startActivity(launchIntent);

// Example of passing a DNAResultItem between activities
Intent intent = new Intent(this, DetailActivity.class);
intent.putExtra("result_item", resultItem); // Parcelable implementation
startActivity(intent);
```

### 3. Load ad creative in the UI
Below shows an example implementation of loading the ad creative in the UI, with DNA method calls.

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);
TextView itemDescription = itemView.findViewById(R.id.item_description);

// handle loading app icon async if app uninstalled
if (!resultItem.isInstalled) {
  resultItem.loadCreativeDrawableAsync(this, new DNAResultItem.ImageCallback() {
    @Override
    public void onImageLoaded(Drawable icon) {
      new Thread(() -> {
        runOnUiThread(() -> {
          if (icon == null) {
            try {
              Drawable backupIcon = getPackageManager().getApplicationIcon(resultItem.packageName);
              itemIcon.setImageDrawable(backupIcon);
            } catch (Exception e) {
              Log.e("RecommendedAppsActivity", "Error loading app icon: " + e.getMessage());
            }
          } else {
            itemIcon.setImageDrawable(icon);
          }
        });
      }).start();
    }

    @Override
    public void onError(String message) {
      try {
        Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
        itemIcon.setImageDrawable(icon);
      } catch (Exception e) {
        Log.e("RecommendedAppsActivity", "Error loading app icon: " + e.getMessage());
      }
    }
  });
} else {
  // if app is installed, show the app icon
  Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
  itemIcon.setImageDrawable(icon);
}

itemTitle.setText(resultItem.title);

if (resultItem.description == null || resultItem.description.isEmpty()) {
  itemDescription.setVisibility(View.GONE);
} else {
  itemDescription.setText(resultItem.description);
}
```

### 4. Send user click to DNA for routing
After the user clicks on a DNA result, send the click to DNA for routing. DNA should handle the click routing because it is important to deep link the user to the advertiser's app with the appropriate parameters.

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem, new DeviceNativeClickHandler() {
  @Override
  public void onClickServerCompleted() {
    Log.i("RecommendedAppsActivity", "Click tracking completed successfully.");
  }

  @Override
  public void onClickRouterCompleted(boolean didRoute) {
    if (!didRoute) {
      Log.e("RecommendedAppsActivity", "Error routing click: No activity found to handle the click.");
    } else {
      Log.i("RecommendedAppsActivity", "Click routed successfully.");
    }
  }

  @Override
  public void onFailure(int errorCode, String errorMessage) {
    Log.e("RecommendedAppsActivity", "Error clicking ad: " + errorMessage);
  }
});
```

#### Important click handling notes
1. We recommend that you add some sort of "loading" UI after the user clicks a result. In some cases, there are delays in the click tracking and routing due to network latency.
2. Make sure to handle the failure cases, such as no activity found to handle the click, and errors in the click tracking and routing. This should be very rare, but it's important to handle it gracefully.

### 5. Do not cache the results
Fresh app suggestion retrieval is very low latency and not resource intensive, so there is no need to cache the results. Results can become stale very quickly, and it's important that the user sees the most relevant results for optimum engagement.
