---
hide:
  - navigation
  - toc
---

# DNA - vivo Global Search Integration Guide
vivo Global Search 集成指南

**Last audited:** April 23, 2026  
**SDK branch:** `vivo-1.1.0`  
**Current SDK version string:** `vivo-1.3.10`

This guide reflects the current Vivo SDK branch in `dn-sdk`. It replaces the older proposal-style notes and aligns the integration guidance to the actual `vivo-1.3.10` SDK surface.

本文档基于 `dn-sdk` 当前的 Vivo 分支编写，并与实际的 `vivo-1.3.10` SDK 接口保持一致。

## 1. SDK Integration
SDK 集成

See [vivo-sdk-changelog](vivo-sdk-changelog.md) for release history.

版本记录请参见 [vivo-sdk-changelog](vivo-sdk-changelog.md)。

### 1.1 SDK package
SDK 包

The Vivo Global Search SDK is currently distributed as an AAR:

当前 Vivo Global Search SDK 以 AAR 形式分发：

- Latest AAR: [https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.10.aar](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.10.aar)
- `minSdk` is `24`

Place the AAR in your app `libs/` folder and reference it from Gradle:

请将 AAR 放入应用的 `libs/` 目录，并在 Gradle 中引入：

```gradle
dependencies {
    implementation files("libs/com.devicenative.dna-vivo-v1.3.10.aar")
}
```

### 1.2 Manifest entries
Manifest 配置

Register the SDK services in your host app manifest:

请在宿主应用的 manifest 中注册以下服务：

```xml
<service
    android:name="com.devicenative.dna.DNADataOrchestrator"
    android:exported="false" />

<service
    android:name="com.devicenative.dna.utils.DNAConfigBuilder"
    android:process=":dna_config_builder"
    android:exported="false" />
```

Host app permissions:

宿主应用需要以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
```

Notes:

- `QUERY_ALL_PACKAGES` is required for launcher-wide package visibility.
- `PACKAGE_USAGE_STATS` is strongly recommended for the ranking quality expected by this integration.
- The SDK library already declares `android.permission.ACCESS_NETWORK_STATE`.

说明：

- `QUERY_ALL_PACKAGES` 用于完整读取设备应用列表。
- `PACKAGE_USAGE_STATS` 对排序质量非常重要，建议开启。
- SDK 自身已声明 `android.permission.ACCESS_NETWORK_STATE`。

### 1.3 Initialization
初始化

Initialize the SDK from `Application.onCreate()`:

请在 `Application.onCreate()` 中初始化 SDK：

```java
@Override
public void onCreate() {
    super.onCreate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.init("da01177b-1526-4b81-9b9d-4a24e54674ac");
}
```

This Vivo branch currently uses `init(deviceKey)` only. It does not expose the newer Moto-style `DNAConfig` or `PackagesRefreshedListener` API surface.

当前 Vivo 分支仅使用 `init(deviceKey)`，不包含 Moto 分支中的 `DNAConfig` 或 `PackagesRefreshedListener` 接口。

### 1.4 Cleanup
清理

If you have a controlled teardown path, test harness, or equivalent shutdown flow, call:

如果宿主应用存在可控的退出流程、测试环境或等效的关闭逻辑，可以调用：

```java
DeviceNativeAds.getInstance(context).destroy();
```

Do not rely on `Application.onTerminate()` as a normal Android production lifecycle hook.

不要把 `Application.onTerminate()` 当作 Android 生产环境中的常规生命周期回调。

## 2. Global Search Recommended Apps
全局搜索推荐应用

This section covers the current Global Search recommendation integration for:

本节说明 Global Search 推荐应用的当前接入方式：

1. Scenario 1: Slot 3 and 8, CPC
2. Scenario 2: Slot 4 and 9, CPC + CPA
3. Scenario 3: Slot 5 and 10, CPA

### 2.1 Fetch DNA recommendation candidates
获取 DNA 推荐候选结果

Use the cache-safe API because Vivo decides which results become visible and must manually register impressions later.

由于 Vivo 需要自行决定哪些结果真正展示，并在后续手动上报展示，因此这里应使用 cache-safe API。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits = dna.getAdsForCache(6, "gs, recommended apps");
```

Important behavior:

- Results are ordered by relevance and monetization priority.
- This API does **not** auto-fire impressions.
- Installed results (`isInstalled == true`) are re-engagement candidates.
- Uninstalled results (`isInstalled == false`) are install / app-store candidates.

关键行为：

- 结果已按相关性和收益优先级排序。
- 该接口**不会**自动触发展示上报。
- `isInstalled == true` 表示拉活候选。
- `isInstalled == false` 表示安装类候选。

### 2.2 Result fields Vivo should use
推荐使用的结果字段

These fields exist in the current `vivo-1.3.10` branch and are safe to rely on:

以下字段存在于当前 `vivo-1.3.10` 分支中，可直接使用：

| Field | Use |
|------|-----|
| `id` | Tracking identifier |
| `resultType` | Distinguishes `TYPE_AD`, `TYPE_APP`, `TYPE_SHORTCUT`, etc. |
| `packageName` | App package |
| `className` | Activity / component when present |
| `userHandle` / `uId` | Multi-user / cloned-app identity |
| `isInstalled` | Installed vs. uninstalled split |
| `title` / `description` | Rendered creative text |
| `iconUrl` | Remote icon for promoted / uninstalled results |
| `clickUrl` | SDK-managed tracking URL |
| `destinationUrl` | Expected terminal routing target |
| `impressionUrl` | Impression endpoint used by the SDK |
| `source` | Result source |
| `placementTag` | Original request tag |

Do not rely on older doc-only fields such as `eCPM` or `learningMode`; they are not part of the current branch’s `DNAResultItem` class.

请不要依赖旧文档中的 `eCPM`、`learningMode` 等字段；这些字段并不存在于当前分支的 `DNAResultItem` 中。

### 2.3 Split the results into the 3 recommendation scenarios
将结果拆分到 3 个推荐场景中

The SDK returns one ranked list. Vivo should split it into the three recommendation scenarios.

SDK 返回的是一个统一排序后的列表，Vivo 需要按业务场景自行拆分。

```java
List<DNAResultItem> scenario1Ads = new ArrayList<>(); // slot 3 and 8, CPC
List<DNAResultItem> scenario2Ads = new ArrayList<>(); // slot 4 and 9, CPC + CPA
List<DNAResultItem> scenario3Ads = new ArrayList<>(); // slot 5 and 10, CPA

for (DNAResultItem resultItem : adUnits) {
    if (resultItem.isInstalled) {
        if (scenario1Ads.size() < 2) {
            scenario1Ads.add(resultItem);
        } else if (scenario2Ads.size() < 2) {
            scenario2Ads.add(resultItem);
        }
    } else {
        if (scenario2Ads.size() < 2) {
            scenario2Ads.add(resultItem);
        } else if (scenario3Ads.size() < 2) {
            scenario3Ads.add(resultItem);
        }
    }
}
```

For installed results, dedupe and replacement logic should preserve the full installed-app identity. Use a key like:

对于已安装结果，去重和替换时应保留完整的安装实例身份。建议使用如下 key：

```text
packageName + "|" + className + "|" + userHandle
```

Do not collapse installed results by `packageName` alone, because cloned apps, work-profile apps, or multi-activity variants can legitimately share the same package.

不要仅按 `packageName` 去重已安装结果，因为分身应用、工作资料夹应用以及多 Activity 变体可能共享同一个包名。

For uninstalled app-store candidates, package-level dedupe is still reasonable.

对于未安装的商店候选结果，按包名去重仍然是合理的。

### 2.4 Render icons and text
渲染图标和文本

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);
TextView itemDescription = itemView.findViewById(R.id.item_description);

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
            Log.e("GlobalSearchActivity", "Error loading icon: " + message);
        }
    });
}

itemTitle.setText(resultItem.title);
if (resultItem.description == null || resultItem.description.isEmpty()) {
    itemDescription.setVisibility(View.GONE);
} else {
    itemDescription.setText(resultItem.description);
}
```

### 2.5 Register impressions manually
手动上报展示

Because `getAdsForCache(...)` does not auto-register impressions, Vivo must register impressions for the results that actually became visible.

由于 `getAdsForCache(...)` 不会自动上报展示，所以 Vivo 需要对真正展示给用户的结果手动上报。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
dna.fireImpressions(scenario1Ads, "gs, scenario 1, slot 3 and 8, CPC", null);
dna.fireImpressions(scenario2Ads, "gs, scenario 2, slot 4 and 9, CPC+CPA", null);
dna.fireImpressions(scenario3Ads, "gs, scenario 3, slot 5 and 10, CPA", null);
```

The third argument is the optional `DeviceNativeClickHandler`; pass `null` if you do not need callback handling for the impression batch.

第三个参数是可选的 `DeviceNativeClickHandler`；如果不需要回调，可以传 `null`。

### 2.6 Route clicks through DNA
通过 DNA 处理点击路由

Use the SDK click router so tracking, redirect handling, and deep linking stay aligned with the Vivo branch behavior.

请使用 SDK 自带的点击路由逻辑，以保证点击跟踪、跳转链路和 deeplink 行为与当前 Vivo 分支保持一致。

If Vivo needs to override the store destination URL, use the overload with `destinationUrlOverride`:

如果 Vivo 需要覆盖商店目标地址，请使用带 `destinationUrlOverride` 的重载接口：

```java
String overrideAppStoreUrl =
        "vivoMarket://mobile/detail?package_name=com.whatsapp&direct_download=false&...";

DeviceNativeAds.getInstance(this).fireClickAndRoute(
        resultItem,
        overrideAppStoreUrl,
        new DeviceNativeClickHandler() {
            @Override
            public void onClickServerCompleted() {
                Log.i("GlobalSearchActivity", "Click tracking completed.");
            }

            @Override
            public void onClickRouterCompleted(boolean didRoute) {
                if (!didRoute) {
                    Log.e("GlobalSearchActivity", "No route target was opened.");
                }
            }

            @Override
            public void onFailure(int errorCode, String errorMessage) {
                Log.e("GlobalSearchActivity", "Click failed: " + errorMessage);
            }
        }
);
```

Show a short loading state while the route is in progress.

点击处理中建议显示短暂的 loading 状态。

## 3. Global Search Search Results
全局搜索搜索结果

This section covers:

本节覆盖以下场景：

1. Scenario 5: Search / Local Apps Result, CPC
2. Scenario 6: Search / App Store Result, CPA

### 3.1 Fetch all search results
获取搜索结果

Use the cache-safe search API because Vivo mixes and filters the result set before rendering.

由于 Vivo 会在渲染前自行混排和过滤结果，因此这里应使用 cache-safe 搜索接口。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> adUnits =
        dna.getOrganicResultsForSearchForCache(query, "gs, search ads");
```

This returns a mixed list of installed/local results and uninstalled/store results.

该接口返回一个混合列表，其中包含本地已安装结果和商店未安装结果。

### 3.2 Split local-app vs app-store scenarios
拆分本地应用结果与商店结果

```java
List<DNAResultItem> scenario5Results = new ArrayList<>(); // local / installed
List<DNAResultItem> scenario6Ads = new ArrayList<>();     // store / uninstalled

for (DNAResultItem resultItem : adUnits) {
    if (resultItem.isInstalled) {
        scenario5Results.add(resultItem);
    } else {
        scenario6Ads.add(resultItem);
    }
}
```

For Scenario 5, replace Vivo’s local-app list with DNA’s ranked installed results. Preserve the full variant identity with `(packageName, className, userHandle)` when deduping or reconciling the local list.

对于场景 5，建议使用 DNA 的已安装结果替换 Vivo 原有的本地应用结果。去重和合并时请保留 `(packageName, className, userHandle)` 这组完整身份。

### 3.3 Register impressions for visible Scenario 5 results
为可见的场景 5 结果注册展示

Only register impressions for the results that were actually shown.

只对真正展示给用户的结果上报展示。

```java
List<DNAResultItem> visibleScenario5Results =
        scenario5Results.subList(0, Math.min(3, scenario5Results.size()));

DeviceNativeAds.getInstance(getApplicationContext())
        .fireImpressions(visibleScenario5Results, "gs, scenario 5, local apps", null);
```

### 3.4 Mix Scenario 6 with Vivo app-store results
将场景 6 与 Vivo 商店结果混排

For Scenario 6, Vivo can mix the uninstalled DNA results into the app-store result set according to its slotting rules.

对于场景 6，Vivo 可以根据自己的坑位规则，将未安装的 DNA 结果混入应用商店结果流中。

When deduping Scenario 6 candidates, package-level dedupe is sufficient.

场景 6 的候选结果按包名去重即可。

### 3.5 Register impressions for visible Scenario 6 results
为可见的场景 6 结果注册展示

```java
DeviceNativeAds.getInstance(getApplicationContext())
        .fireImpressions(visibleScenario6Results, "gs, scenario 6, app store", null);
```

### 3.6 Render icons and route clicks
渲染图标与处理点击

Rendering and click handling should use the same rules as the recommendation section:

渲染和点击处理逻辑与推荐区域保持一致：

- Installed results: use `PackageManager` icons
- Uninstalled results: use `loadCreativeDrawableAsync(...)`
- Route clicks through `fireClickAndRoute(...)`

## 4. Practical Notes
实践注意事项

- These Vivo integrations intentionally use the `*ForCache(...)` APIs so that Vivo controls the final visible set and manually fires impressions.
- Do not persist these cached results across long-lived sessions; treat them as short-lived render candidates.
- The current Vivo branch already uses the newer callback names: `onClickServerCompleted`, `onClickRouterCompleted`, and `onFailure`.

说明：

- 这里使用 `*ForCache(...)` 接口是有意为之，因为最终展示列表和展示上报由 Vivo 侧控制。
- 不建议将这些缓存结果长时间持久化，只应作为一次渲染周期内的候选结果。
- 当前 Vivo 分支使用的点击回调名称是 `onClickServerCompleted`、`onClickRouterCompleted` 和 `onFailure`。

For support, email [help@devicenative.com](mailto:help@devicenative.com).
