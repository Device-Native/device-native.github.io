---
hide:
  - navigation
  - toc
---

# vivo DNA SDK Changelog

## [**vivo-1.3.10**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.10.aar) released 12/7/2025

- Replaced and updated the Android wrapper on QuickJS
- Ad serving cache improvements

**中文翻译：**
- 替换并更新了 QuickJS 的 Android 包装层
- 改进广告投放缓存

## [**vivo-1.3.9**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.9.aar) released 11/24/2025

- Improved search query cache
- Smarter cache invalidation
- Improved data service connectivity and refresh logic

**中文翻译：**
- 改进搜索查询缓存
- 更智能的缓存失效机制
- 改进数据服务的连接性与刷新逻辑

## [**vivo-1.3.8**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.8.aar) released 11/14/2025

- Reduced overhead of init() call and data service restart
- Further improved from 1.3.7 cache for latency improvements
- GZIP on data uploads for reduced bandwidth usage

**中文翻译：**
- 降低 init() 调用和数据服务重启的开销
- 在 1.3.7 的缓存优化基础上进一步降低延迟
- 对数据上传启用 GZIP 以减少带宽使用

## [**vivo-1.3.7**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.7.aar) released 11/3/2025

- Dramatic improvement in CPU usage, and reduction in latency for result serving
- Smarter timing of local data refresh to reduce battery consumption

**中文翻译：**
- 显著改善 CPU 使用率并降低结果提供延迟
- 更智能地安排本地数据刷新时间以降低电量消耗

## [**vivo-1.3.6**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.6.aar) released 10/30/2025

- Overhaul cache refresh logic to reduce battery consumption
- Improved usage refresh logic to reduce battery consumption
- DB hardening to handle case of corrupt file
- Less frequent agg stats sync to minimize network usage

**中文翻译：**
- 全面改造缓存刷新逻辑以降低电量消耗
- 优化使用数据刷新逻辑以降低电量消耗
- 强化数据库以应对文件损坏场景
- 降低聚合统计同步频率以减少网络占用

## [**vivo-1.3.5**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.5.aar) released 10/15/2025

- Improved network connection handling for reduced bandwidth usage and improved latency
- Fixed a bug introduced with the shortcut logic in 1.3.4
- Disabled shortcuts and notifications by default.

**中文翻译：**
- 改进网络连接处理以降低带宽占用并缩短延迟
- 修复 1.3.4 中快捷方式逻辑引入的缺陷
- 默认禁用快捷方式和通知

## [**vivo-1.3.4**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.4.aar) released 9/26/2025

- Try/catch around shortcut calls that crash in device-lock scenarios

**中文翻译：**
- 在可能因设备锁定而崩溃的快捷方式调用外部添加 try/catch

## [**vivo-1.3.3**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.3.aar) released 9/18/2025

- Removed some unnecessary logging
- Upgraded the QuickJS library to support modern Android requirements

**中文翻译：**
- 移除部分不必要的日志
- 升级 QuickJS 库以满足现代 Android 要求

## [**vivo-1.3.2**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.2.aar) released 9/1/2025

- Better handling in case of local database corruption
- Improved local database performance
- Significant improvement in data refresh memory efficiency
- Better handling of ad link latency (timeouts, network detection)
- Improved service connectivity management
- Addressed risk in cross-service file communication (user agent sync)

**中文翻译：**
- 更好地处理本地数据库损坏情况
- 提升本地数据库性能
- 显著提升数据刷新时的内存效率
- 改进广告链接延迟处理（超时、网络检测）
- 改进服务连接管理
- 解决跨服务文件通信（User Agent 同步）中的风险

## [**vivo-1.3.1**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.1.aar) released 7/25/2025

- Better support for upgrade path for pre-1.3.0 SDK versions
- Minor improvements to the isInstalled logic
- Minor bug fixes on new features for 1.3.0

**中文翻译：**
- 更好地支持 1.3.0 之前版本的升级路径
- 对 isInstalled 逻辑进行小幅改进
- 修复 1.3.0 新特性相关的一些小问题

## [**vivo-1.3.0**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.3.0.aar) released 6/25/2025

- Improved support for app enabling/disabling
- Improved support for work profile apps, removing and enabling profiles
- Better non-English language support for search query matching
- App entities are now for launch activities in apps, rather than just package names aligning better with most OEM launcher UX
- 1 character length query search cache delivering instant first results
- Fixed some search query cache bugs, where a cache could be stale when apps were enabled/disabled

**中文翻译：**
- 提升应用启用/禁用的支持能力
- 改进工作配置文件应用的管理，包括移除与启用
- 增强非英语语言的搜索匹配支持
- 应用实体现在指向应用内的启动 Activity，而不再仅对应包名，更契合大多数 OEM 启动器体验
- 针对单字符查询的搜索缓存，提供即时首个结果
- 修复当应用启用/禁用时可能导致缓存过期的搜索缓存缺陷

## [**vivo-1.2.4**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.2.4.aar) released 5/3/2025

- Improve wake broadcast for better data synchronization
- Reduce unnecessary event logging
- Improve debug mode for first session

**中文翻译：**
- 改进唤醒广播以提升数据同步
- 减少不必要的事件日志
- 优化首次会话的调试模式

## [**vivo-1.2.3**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.2.3.aar) released 3/14/2025

- Latency improvements for ad serving through warm ups of key service
- Support new macro for passing product source to URL
- Security improvement for parseUri on link redirection

**中文翻译：**
- 通过预热关键服务改善广告投放延迟
- 支持用于将产品来源传递到 URL 的新宏
- 提升链接重定向中 parseUri 的安全性

## [**vivo-1.2.2**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.2.2.aar) released 3/14/2025

- Minor memory optimizations

**中文翻译：**
- 进行少量内存优化

## [**vivo-1.2.1**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.2.1.aar) released 2/20/2025

- Implemented native fallback support for search and suggestions

**中文翻译：**
- 实现搜索与建议的原生回退支持

## [**vivo-1.2.0**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.2.0.aar) released 2/4/2025

- Added cloned and multi-user app support
- Restructured network sync to improve performance and reduce data transfer

**中文翻译：**
- 新增克隆与多用户应用支持
- 重构网络同步以提升性能并减少数据传输

## [**vivo-1.1.17**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.17.aar) released 12/12/2024

- More robust coverage of initialization to handle unpredictable calls to destroy()

**中文翻译：**
- 加强初始化覆盖范围，以应对不可预测的 destroy() 调用

## [**vivo-1.1.16**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.16.aar) released 12/10/2024

- Added support for destination URL override

**中文翻译：**
- 新增目标 URL 覆盖支持

## [**vivo-1.1.15**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.15.aar) released 12/4/2024

- Added support for gzip for reduced bandwidth usage

**中文翻译：**
- 新增 gzip 支持以减少带宽使用

## [**vivo-1.1.14**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.14.aar) released 11/24/2024

- Changed back to market:// URLs for app install campaigns
- Fixed a bug for routing duplicate clicks
- Removed logging when debug mode is off
- Fast open app store when link is cached
- Query parameters for market URL rewrites

**中文翻译：**
- 将应用安装推广的链接改回使用 market:// URL
- 修复处理重复点击的缺陷
- 在非调试模式下移除日志
- 当链接已缓存时快速打开应用商店
- 为市场 URL 重写增加查询参数

## [**vivo-1.1.13**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.13.aar) released 11/7/2024

- Fixed an issue where new apps were not being assigned a user profile
- Improved the latency of link redirects in some cases
- Added support for intent-based deep linking

**中文翻译：**
- 修复新应用未分配用户画像的问题
- 在部分场景下改善链接重定向延迟
- 新增基于 Intent 的深度链接支持

## [**vivo-1.1.12**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.12.aar) released 11/5/2024

- Better management of the local GAID
- Change of direct linking for app install campaigns

**中文翻译：**
- 更好地管理本地 GAID
- 调整应用安装推广的直接链接策略

## [**vivo-1.1.11**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.11.aar) released 11/1/2024

- Reduced the time taken to retrieve the Chrome User Agent in the separate process to minimize the chance of collisions.

**中文翻译：**
- 缩短在独立进程中获取 Chrome User Agent 的时间，降低冲突概率

## [**vivo-1.1.10**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.10.aar) released 10/20/2024

- Fixed a bug where the app name was being used instead of the title
- Implemented better launching of User Agent collection service to handle background restrictions

**中文翻译：**
- 修复使用应用名称替代标题的缺陷
- 改进 User Agent 收集服务的启动方式以适应后台限制

## [**vivo-1.1.9**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.9.aar) released 10/8/2024

- Added DNAResultItem.className and DNAResultItem.getComponentName to support alternative filtering/deduping
- SDK now supports work profile management
- Some performance improvements to ad serving

**中文翻译：**
- 新增 DNAResultItem.className 和 DNAResultItem.getComponentName 以支持替代的过滤/去重策略
- SDK 现已支持工作配置文件管理
- 提升广告投放性能

## [**vivo-1.1.8**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.8.aar) released 9/30/2024

- Fixed potential null pointer exception in profile change monitor

**中文翻译：**
- 修复画像变更监视器中潜在的空指针异常

## [**vivo-1.1.7**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.7.aar) released 9/26/2024

- Improved the robustness of inter-process communication for user agent
- Lower resource consumption when reading user agent
- Added code to prevent duplicate data orchestrator services from running

**中文翻译：**
- 增强 User Agent 相关的进程间通信可靠性
- 降低读取 User Agent 时的资源消耗
- 新增逻辑以防止数据编排服务重复运行

## [**vivo-1.1.6**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.6.aar) released 9/9/2024

- Made DNAResultItem Parceable

**中文翻译：**
- 使 DNAResultItem 实现 Parcelable 接口

## [**vivo-1.1.5**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.5.aar) released 9/5/2024

- Initialization ongoing memory reduction of ~ 20MB
- Isolated Chrome User Agent fetch to process in order to constrain memory consumption

**中文翻译：**
- 初始化过程的持续内存占用减少约 20MB
- 将 Chrome User Agent 的获取隔离到单独进程以限制内存消耗

## [**vivo-1.1.4**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.4.aar) released 8/26/2024

- Most app-install clicks will now be instantaneous
- Parallel process tracking link and Play Store redirect when possible
- Improved performance of Google Play to Custom Store remapping

**中文翻译：**
- 大多数应用安装点击现在可即时响应
- 尽可能并行处理跟踪链接与 Play 商店重定向
- 改进从 Google Play 到自定义商店的重映射性能

## [**vivo-1.1.3**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.3.aar) released 8/23/2024

- Redirect prefetching for high likelihood clicks
- Cache and reused redirect outcomes for improved performance on repeat clicks
- Improved thread performance for click speedup
- Implemented deep linking for install ads that are already installed

**中文翻译：**
- 为高概率点击预取重定向
- 缓存并复用重定向结果以提升重复点击性能
- 改进线程性能以加速点击响应
- 为已安装的安装广告实现深度链接

## [**vivo-1.1.2**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.2.aar) released 8/22/2024

- Remote control to disable app usage stats collection
- Improved poor connection handling in link routing use case
- Exposed app ratings, downloads, and reviews in DNAResultItem

**中文翻译：**
- 新增远程控制以禁用应用使用统计收集
- 在链接路由场景下改进弱网处理
- 在 DNAResultItem 中暴露应用评分、下载量与评价

## [**vivo-1.1.1**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.1.aar) released 8/20/2024

- Added ability to register impression manually for DNA search results (already existed for recommendations)

**中文翻译：**
- 为 DNA 搜索结果新增手动注册曝光的能力（推荐结果已支持）

## [**vivo-1.1.0**](https://dna-hosting.s3.amazonaws.com/public/com.devicenative.dna-vivo-v1.1.0.aar) released 8/19/2024

- First release of custom vivo DNA SDK
- Performance optimizations for vivo Global Search use case

**中文翻译：**
- 定制版 vivo DNA SDK 的首次发布
- 针对 vivo 全局搜索场景进行性能优化
