---
hide:
  - navigation
  - toc
---

# DNA - vivo App Drawer Integration Guide
vivo App Drawer 集成指南

**Last audited:** April 23, 2026  
**SDK branch:** `vivo-1.1.0`  
**Current SDK version string:** `vivo-1.3.10`

This guide reflects the current Vivo App Drawer integration expectations for the `vivo-1.1.0` branch in `dn-sdk`.

本文档基于 `dn-sdk` 中当前的 `vivo-1.1.0` 分支编写，适用于最新的 Vivo App Drawer SDK 行为。

## 1. SDK Integration
SDK 集成

See [vivo-sdk-changelog](vivo-sdk-changelog.md) for the generic Vivo release history.

版本记录请参见 [vivo-sdk-changelog](vivo-sdk-changelog.md)。

### 1.1 SDK package
SDK 包

The App Drawer integration currently uses the Vivo ad AAR:

当前 App Drawer 集成使用 Vivo ad AAR：

- Latest AAR: [https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-ad-v1.3.10.aar](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-ad-v1.3.10.aar)
- `minSdk` is `24`

Gradle example:

Gradle 示例：

```gradle
dependencies {
    implementation files("libs/com.devicenative.dna-vivo-ad-v1.3.10.aar")
}
```

### 1.2 Manifest entries
Manifest 配置

```xml
<service
    android:name="com.devicenative.dna.DNADataOrchestrator"
    android:exported="false" />

<service
    android:name="com.devicenative.dna.utils.DNAConfigBuilder"
    android:process=":dna_config_builder"
    android:exported="false" />
```

Required host-app permissions:

宿主应用需要以下权限：

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.QUERY_ALL_PACKAGES" />
<uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
```

Notes:

- `QUERY_ALL_PACKAGES` is required for a launcher/App Drawer surface.
- `PACKAGE_USAGE_STATS` should remain enabled for expected ranking quality.
- The SDK library already declares `android.permission.ACCESS_NETWORK_STATE`.

### 1.3 Initialization
初始化

```java
@Override
public void onCreate() {
    super.onCreate();

    DeviceNativeAds dna = DeviceNativeAds.getInstance(this);
    dna.init("6a8c19b4-452a-485a-ab8e-e056efd568de");
}
```

This Vivo branch uses the simpler `init(deviceKey)` API surface.

当前 Vivo 分支使用简化的 `init(deviceKey)` 接口。

### 1.4 Cleanup
清理

If you have a controlled teardown path, you may call:

如果存在可控的销毁流程，可以调用：

```java
DeviceNativeAds.getInstance(context).destroy();
```

Do not rely on `Application.onTerminate()` during normal Android production execution.

不要依赖 `Application.onTerminate()` 作为 Android 生产环境中的常规退出回调。

## 2. App Drawer Recommended Apps
App Drawer 推荐应用

This section covers the current App Drawer recommended-apps integration where DNA results are merged into Vivo’s own recommendation list.

本节说明当前 App Drawer 推荐应用集成方式，即将 DNA 结果混入 Vivo 自有推荐结果。

### 2.1 Fetch DNA candidates
获取 DNA 候选结果

Use `getAdsForCache(...)` because Vivo decides which candidates will actually be shown and must manually fire impressions.

由于最终展示项由 Vivo 自行决定，并需要手动上报展示，因此这里应使用 `getAdsForCache(...)`。

```java
DeviceNativeAds dna = DeviceNativeAds.getInstance(getApplicationContext());
List<DNAResultItem> dnaResults = dna.getAdsForCache(6, "app drawer, recommended apps");
```

Behavior:

- The results are ranked by relevance and monetization priority.
- This API does **not** auto-fire impressions.
- For App Drawer, these are expected to be installed / re-engagement-oriented candidates.

### 2.2 Result fields to rely on
推荐使用的结果字段

The current Vivo branch exposes these fields on `DNAResultItem`:

当前 Vivo 分支中的 `DNAResultItem` 可使用以下字段：

| Field | Use |
|------|-----|
| `id` | Tracking identifier |
| `resultType` | Result type |
| `packageName` | Package name |
| `className` | Activity / component |
| `userHandle` / `uId` | Multi-user / cloned-app identity |
| `isInstalled` | Installed state |
| `title` / `description` | UI text |
| `iconUrl` | Remote icon when needed |
| `clickUrl` | SDK routing / tracking URL |
| `destinationUrl` | Final route target |
| `impressionUrl` | Impression endpoint |
| `source` | Result source |
| `placementTag` | Original placement tag |

Do not rely on older doc-only fields such as `eCPM` or `learningMode`.

请不要依赖旧文档中的 `eCPM`、`learningMode` 等字段。

### 2.3 Merge DNA results into Vivo’s list
将 DNA 结果合并到 Vivo 列表

The current slotting rule remains:

当前坑位规则仍然是：

- Prefer replacing duplicates in positions 1–3 and 5–7
- Use positions 4 and 8 as fallback placements

Translated to zero-based indexes:

换成从 0 开始的索引：

```text
priorityIdx = [0, 1, 2, 4, 5, 6]
fallbackIdx = [3, 7]
```

Recommended merge logic:

建议的合并逻辑：

```text
1. Compare each DNA installed result against Vivo’s existing visible candidates.
2. For installed results, dedupe by:
   packageName + "|" + className + "|" + userHandle
3. Replace duplicates first in positions 1–3 and 5–7.
4. Fill positions 4 and 8 with any remaining DNA results.
5. Re-check the first 8 visible results for duplicate installed-app identities.
```

For App Drawer, do not collapse installed results by package name alone. Cloned apps, work-profile apps, and different launcher activities can legitimately share the same package.

对于 App Drawer，请不要只按包名去重。分身应用、工作资料夹应用以及不同启动 Activity 可能共享同一个包名。

### 2.4 Render icons and labels
渲染图标和标题

These App Drawer candidates are normally installed apps, so load the icon from `PackageManager`:

这些 App Drawer 候选结果通常是已安装应用，因此直接从 `PackageManager` 加载图标即可：

```java
ImageView itemIcon = itemView.findViewById(R.id.item_icon);
TextView itemTitle = itemView.findViewById(R.id.item_title);

Drawable icon = getPackageManager().getApplicationIcon(resultItem.packageName);
itemIcon.setImageDrawable(icon);
itemTitle.setText(resultItem.title);
```

### 2.5 Register impressions manually
手动上报展示

Because `getAdsForCache(...)` does not auto-register impressions, Vivo must register impressions for the results actually shown in the App Drawer surface.

由于 `getAdsForCache(...)` 不会自动上报展示，因此 Vivo 需要对真正展示给用户的结果手动上报展示。

```java
DeviceNativeAds.getInstance(getApplicationContext())
        .fireImpressions(dnaResultsShown, "app drawer, visible recommended apps", null);
```

If you do not need callbacks for the impression batch, pass `null` as the third argument.

如果不需要展示批次的回调处理，第三个参数直接传 `null`。

### 2.6 Route clicks through DNA
通过 DNA 处理点击路由

```java
DeviceNativeAds.getInstance(this).fireClickAndRoute(resultItem, new DeviceNativeClickHandler() {
    @Override
    public void onClickServerCompleted() {
        Log.i("AppDrawerActivity", "Click tracking completed.");
    }

    @Override
    public void onClickRouterCompleted(boolean didRoute) {
        if (!didRoute) {
            Log.e("AppDrawerActivity", "No route target was opened.");
        }
    }

    @Override
    public void onFailure(int errorCode, String errorMessage) {
        Log.e("AppDrawerActivity", "Click failed: " + errorMessage);
    }
});
```

Use the SDK click router so tracking, redirect handling, and deep linking stay consistent with the current branch behavior.

请使用 SDK 自带点击路由，以保证跟踪、跳转和 deeplink 行为与当前分支一致。

## 3. Practical Notes
实践注意事项

- `getAdsForCache(...)` is the correct API here because Vivo controls which candidates are finally shown.
- These results should be treated as short-lived render candidates, not long-lived persistent cache entries.
- The current Vivo branch already uses the callback names `onClickServerCompleted`, `onClickRouterCompleted`, and `onFailure`.

说明：

- 这里使用 `getAdsForCache(...)` 是正确的，因为最终展示结果由 Vivo 决定。
- 这些结果应视为短生命周期的渲染候选，不建议长期持久化。
- 当前 Vivo 分支的点击回调名称是 `onClickServerCompleted`、`onClickRouterCompleted` 和 `onFailure`。

For support, email [help@devicenative.com](mailto:help@devicenative.com).
