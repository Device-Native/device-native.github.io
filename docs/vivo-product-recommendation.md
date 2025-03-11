---
hide:
  - navigation
  - toc
---

# DNA - vivo Product Recommendation Integration Proposals

**Date:** 2025-03-11

Below are the proposed integration steps to accommodate vivo's Product Recommendation integration product spec.

1. [API Overview](#api-overview)
2. [Integration Steps](#integration-steps)
    1. [Make API Request](#1-make-api-request)
    2. [Process Response Data](#2-process-response-data)
    3. [Display Product Recommendations](#3-display-product-recommendations)
    4. [Track Impressions](#4-track-impressions)
    5. [Handle User Clicks](#5-handle-user-clicks)

以下是适应vivo产品推荐集成产品规格的建议集成步骤。

1. [API概述](#api-overview)
2. [集成步骤](#integration-steps)
    1. [发起API请求](#1-make-api-request)
    2. [处理响应数据](#2-process-response-data)
    3. [显示产品推荐](#3-display-product-recommendations)
    4. [跟踪展示](#4-track-impressions)
    5. [处理用户点击](#5-handle-user-clicks)

# API Overview
API概述

The DNA Product Recommendation API provides personalized product recommendations based on user preferences and behavior. This API allows vivo to integrate product recommendations into their applications, enhancing user experience and engagement.

DNA产品推荐API根据用户偏好和行为提供个性化产品推荐。此API允许vivo将产品推荐集成到其应用程序中，提升用户体验和参与度。

## Endpoint
端点

```
POST https://trending.devicenative.com/products
```

## Request Parameters
请求参数

The following table describes the parameters that should be included in the API request:

下表描述了应包含在API请求中的参数：

| Parameter | Type | Required | Description | 描述 |
|-----------|------|----------|-------------|------|
| `gaid` | String | Yes | User GAID identifier | 用户GAID标识符 |
| `os_version` | String | Yes | System version | 系统版本 |
| `os` | String | Yes | System type (e.g., android) | 系统类型（例如，android） |
| `language` | String | Yes | Language (e.g., en_US) | 语言（例如，en_US） |
| `app_version` | String | No | Report for installed, none for uninstalled | 已安装的报告，未安装则为空 |
| `region` | String | Yes | Region (corresponding to country code, e.g., IN) | 地区（对应国家代码，例如，IN） |
| `count` | Integer | No | Return the number of products (default: 10) | 返回产品数量（默认：10） |
| `source` | Integer | Yes | Request source, internal enumeration value | 请求来源，内部枚举值 |
| `device_model` | String | Yes | Device model name (e.g., M2002J9E) | 设备型号名称（例如，M2002J9E） |
| `device_resolution` | String | No | Resolution (e.g., 1080×1980) | 分辨率（例如，1080×1980） |
| `device_type` | String | No | Device type (phone, pad, flip, fold) | 设备类型（手机、平板、翻盖、折叠） |
| `device_brand` | String | No | Device brand (e.g., VIVO) | 设备品牌（例如，VIVO） |

## Response Parameters
响应参数

The API response is an array of product recommendation objects with the following parameters:

API响应是一个产品推荐对象数组，具有以下参数：

| Parameter | Type | Required | Description | 描述 |
|-----------|------|----------|-------------|------|
| `name` | String | Yes | Product name | 产品名称 |
| `icon_url` | String | Yes | Product image URL | 产品图片URL |
| `package_name` | String | Yes | App package name | 应用包名 |
| `description` | String | No | Product description | 产品描述 |
| `deep_link` | String | Yes | Deep link to the product | 产品深度链接 |
| `h5` | String | Yes | H5 link to the product | 产品H5链接 |
| `click_link` | String | Yes | Click tracking URL | 点击跟踪URL |
| `expose_link` | String | Yes | Impression tracking URL | 展示跟踪URL |
| `current_price` | String | Yes | Product price after discount | 折扣后的产品价格 |
| `original_price` | String | Yes | Original price of product | 产品原价 |
| `discount` | String | Yes | Product discount percentage | 产品折扣百分比 |
| `currency_mark` | String | Yes | Currency symbol | 货币符号 |

# Integration Steps
集成步骤

### 1. Make API Request
1. 发起API请求

To retrieve product recommendations, make a POST request to the API endpoint with the required parameters:

要获取产品推荐，请使用所需参数向API端点发出POST请求：

```java
// Example using Android's native HttpURLConnection
new Thread(() -> {
    HttpURLConnection connection = null;
    try {
        // Create request body
        JSONObject requestBody = new JSONObject();
        requestBody.put("gaid", "test-gaid-1234");
        requestBody.put("os_version", "19");
        requestBody.put("os", "android");
        requestBody.put("language", "en_US");
        requestBody.put("app_version", "1.0.0");
        requestBody.put("region", "IN");
        requestBody.put("source", 0);
        requestBody.put("device_model", "M2002J9E");
        requestBody.put("device_resolution", "1080×1980");
        requestBody.put("device_type", "phone");
        requestBody.put("device_brand", "VIVO");
        
        // Set up connection
        URL url = new URL("https://trending.devicenative.com/products");
        connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("POST");
        connection.setRequestProperty("Content-Type", "application/json");
        connection.setDoOutput(true);
        connection.setConnectTimeout(5000);
        connection.setReadTimeout(5000);
        
        // Write request body
        try (OutputStream os = connection.getOutputStream()) {
            byte[] input = requestBody.toString().getBytes("utf-8");
            os.write(input, 0, input.length);
        }
        
        // Read response
        int responseCode = connection.getResponseCode();
        if (responseCode == HttpURLConnection.HTTP_OK) {
            try (BufferedReader br = new BufferedReader(
                    new InputStreamReader(connection.getInputStream(), "utf-8"))) {
                StringBuilder response = new StringBuilder();
                String responseLine;
                while ((responseLine = br.readLine()) != null) {
                    response.append(responseLine.trim());
                }
                
                // Process the response on the main thread
                String finalResponse = response.toString();
                new Handler(Looper.getMainLooper()).post(() -> {
                    processResponseData(finalResponse);
                });
            }
        } else {
            Log.e("ProductRecommendation", "API request failed with code: " + responseCode);
        }
    } catch (Exception e) {
        Log.e("ProductRecommendation", "API request failed: " + e.getMessage());
    } finally {
        if (connection != null) {
            connection.disconnect();
        }
    }
}).start();
```

### 2. Process Response Data
2. 处理响应数据

Parse the JSON response and extract the product recommendation data:

解析JSON响应并提取产品推荐数据：

```java
private void processResponseData(String responseData) {
    try {
        JSONArray productsArray = new JSONArray(responseData);
        List<ProductRecommendation> recommendations = new ArrayList<>();
        
        for (int i = 0; i < productsArray.length(); i++) {
            JSONObject productObj = productsArray.getJSONObject(i);
            
            ProductRecommendation product = new ProductRecommendation();
            product.setName(productObj.getString("name"));
            product.setIconUrl(productObj.getString("icon_url"));
            product.setPackageName(productObj.getString("package_name"));
            product.setDeepLink(productObj.getString("deep_link"));
            product.setH5Link(productObj.getString("h5"));
            product.setClickLink(productObj.getString("click_link"));
            product.setExposeLink(productObj.getString("expose_link"));
            product.setCurrentPrice(productObj.getString("current_price"));
            product.setOriginalPrice(productObj.getString("original_price"));
            product.setDiscount(productObj.getString("discount"));
            product.setCurrencyMark(productObj.getString("currency_mark"));
            
            // Optional fields
            if (productObj.has("description") && !productObj.isNull("description")) {
                product.setDescription(productObj.getString("description"));
            }
            
            recommendations.add(product);
        }
        
        // Update UI with recommendations
        updateUIWithRecommendations(recommendations);
        
    } catch (JSONException e) {
        Log.e("ProductRecommendation", "Error parsing response: " + e.getMessage());
    }
}
```

### 3. Display Product Recommendations
3. 显示产品推荐

Display the product recommendations in your UI:

在您的UI中显示产品推荐：

```java
private void updateUIWithRecommendations(List<ProductRecommendation> recommendations) {
    runOnUiThread(() -> {
        // Assuming you have a RecyclerView adapter
        productAdapter.setProducts(recommendations);
        productAdapter.notifyDataSetChanged();
        
        // Track impressions for visible products
        trackImpressions(recommendations);
    });
}
```

Example of a product item layout:

产品项布局示例：

```xml
<!-- product_item.xml -->
<androidx.cardview.widget.CardView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp">

        <ImageView
            android:id="@+id/product_image"
            android:layout_width="80dp"
            android:layout_height="80dp"
            android:scaleType="centerCrop" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="12dp"
            android:orientation="vertical">

            <TextView
                android:id="@+id/product_name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:textSize="16sp"
                android:textStyle="bold" />

            <TextView
                android:id="@+id/product_description"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="4dp"
                android:textSize="14sp" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:orientation="horizontal">

                <TextView
                    android:id="@+id/product_current_price"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:textColor="@color/colorAccent"
                    android:textSize="16sp"
                    android:textStyle="bold" />

                <TextView
                    android:id="@+id/product_original_price"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="8dp"
                    android:textColor="@color/colorGray"
                    android:textSize="14sp"
                    android:textDecoration="line-through" />

                <TextView
                    android:id="@+id/product_discount"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="8dp"
                    android:textColor="@color/colorRed"
                    android:textSize="14sp" />
            </LinearLayout>
        </LinearLayout>
    </LinearLayout>
</androidx.cardview.widget.CardView>
```

### 4. Track Impressions
4. 跟踪展示

Track impressions for the displayed products:

跟踪显示的产品的展示：

```java
private void trackImpressions(List<ProductRecommendation> visibleProducts) {
    ExecutorService executor = Executors.newFixedThreadPool(5);
    
    for (ProductRecommendation product : visibleProducts) {
        executor.execute(() -> {
            try {
                URL url = new URL(product.getExposeLink());
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                connection.setRequestMethod("GET");
                connection.setConnectTimeout(5000);
                connection.setReadTimeout(5000);
                
                int responseCode = connection.getResponseCode();
                if (responseCode == HttpURLConnection.HTTP_OK) {
                    Log.d("ProductRecommendation", "Impression tracked for: " + product.getName());
                } else {
                    Log.e("ProductRecommendation", "Failed to track impression: " + responseCode);
                }
                
                connection.disconnect();
            } catch (Exception e) {
                Log.e("ProductRecommendation", "Error tracking impression: " + e.getMessage());
            }
        });
    }
    
    executor.shutdown();
}
```

### 5. Handle User Clicks
5. 处理用户点击

Handle user clicks on product recommendations:

处理用户对产品推荐的点击：

```java
private void handleProductClick(ProductRecommendation product) {
    // Track click
    trackClick(product.getClickLink());
    
    // Open deep link or H5 link
    try {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        
        // Try to use deep link first
        if (product.getDeepLink() != null && !product.getDeepLink().isEmpty()) {
            intent.setData(Uri.parse(product.getDeepLink()));
            if (intent.resolveActivity(getPackageManager()) != null) {
                startActivity(intent);
                return;
            }
        }
        
        // Fall back to H5 link if deep link fails
        if (product.getH5Link() != null && !product.getH5Link().isEmpty()) {
            intent.setData(Uri.parse(product.getH5Link()));
            startActivity(intent);
        }
    } catch (Exception e) {
        Log.e("ProductRecommendation", "Error opening product link: " + e.getMessage());
    }
}

private void trackClick(String clickLink) {
    new Thread(() -> {
        try {
            URL url = new URL(clickLink);
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setConnectTimeout(5000);
            connection.setReadTimeout(5000);
            
            int responseCode = connection.getResponseCode();
            if (responseCode == HttpURLConnection.HTTP_OK) {
                Log.d("ProductRecommendation", "Click tracked successfully");
            } else {
                Log.e("ProductRecommendation", "Failed to track click: " + responseCode);
            }
            
            connection.disconnect();
        } catch (Exception e) {
            Log.e("ProductRecommendation", "Error tracking click: " + e.getMessage());
        }
    }).start();
}
```

## Example Request
请求示例

### Using curl
使用curl

```bash
curl -X POST https://trending.devicenative.com/products \
  -H "Content-Type: application/json" \
  -d '{
    "gaid": "test-gaid-1234",
    "os_version": "19",
    "os": "android",
    "language": "en_US",
    "app_version": "1.0.0",
    "region": "IN",
    "source": 0,
    "device_model": "M2002J9E",
    "device_resolution": "1080×1980",
    "device_type": "phone",
    "device_brand": "VIVO"
}'
```

### Using Android
使用Android

```json
{
    "gaid": "test-gaid-1234",
    "os_version": "19",
    "os": "android",
    "language": "en_US",
    "app_version": "1.0.0",
    "region": "IN",
    "source": 0,
    "device_model": "M2002J9E",
    "device_resolution": "1080×1980",
    "device_type": "phone",
    "device_brand": "VIVO"
}
```

## Example Response
响应示例

```json
[
    {
        "icon_url": "https://assets.myntassets.com/dpr_2,q_60,w_210,c_limit,fl_progressive/assets/images/29664078/2024/6/4/29476fa6-287f-40b9-b77c-d79a8ca6ed731717493672307VByVerageTokyoTexturedHard-SidedCabin-SizedTrolleyBag4704L11.jpg",
        "original_price": "5999.0",
        "currency_mark": "INR",
        "name": "V By Verage Hard Cabin-Sized Trolley Bag",
        "package_name": "com.myntra.android",
        "description": "",
        "discount": "84%",
        "expose_link": "https://trending.devicenative.com/imp?pid=1a986682-b54a-424b-8b01-f179e1a9edd4&pck=com.myntra.android&referrer=0&uid=test-gaid-1234",
        "current_price": "949.0",
        "click_link": "https://trending.devicenative.com/click?url=https%3A%2F%2Fwww.myntra.com%2Fv-by-verage-trolley-bag%3FrawQuery%3DV%2520By%2520Verage%2520Trolley%2520Bag%26sort%3Dprice_asc&pid=1a986682-b54a-424b-8b01-f179e1a9edd4&pck=com.myntra.android&referrer=0&uid=test-gaid-1234",
        "h5": "https://www.myntra.com/v-by-verage-trolley-bag?rawQuery=V%20By%20Verage%20Trolley%20Bag&sort=price_asc&utm_source=dms_trafficaffiliates&utm_medium=dms_veve_cpv&utm_campaign=dms_trafficaffiliates_veve_cpv_INV320",
        "deep_link": "https://www.myntra.com/v-by-verage-trolley-bag?rawQuery=V%20By%20Verage%20Trolley%20Bag&sort=price_asc&utm_source=dms_trafficaffiliates&utm_medium=dms_veve_cpv&utm_campaign=dms_trafficaffiliates_veve_cpv_INV320"
    }
]
```

## ProductRecommendation Class
ProductRecommendation类

Here's a sample ProductRecommendation class that can be used to store the data:

以下是可用于存储数据的示例ProductRecommendation类：

```java
public class ProductRecommendation {
    private String name;
    private String iconUrl;
    private String packageName;
    private String description;
    private String deepLink;
    private String h5Link;
    private String clickLink;
    private String exposeLink;
    private String currentPrice;
    private String originalPrice;
    private String discount;
    private String currencyMark;

    // Getters and setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getIconUrl() {
        return iconUrl;
    }

    public void setIconUrl(String iconUrl) {
        this.iconUrl = iconUrl;
    }

    public String getPackageName() {
        return packageName;
    }

    public void setPackageName(String packageName) {
        this.packageName = packageName;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getDeepLink() {
        return deepLink;
    }

    public void setDeepLink(String deepLink) {
        this.deepLink = deepLink;
    }

    public String getH5Link() {
        return h5Link;
    }

    public void setH5Link(String h5Link) {
        this.h5Link = h5Link;
    }

    public String getClickLink() {
        return clickLink;
    }

    public void setClickLink(String clickLink) {
        this.clickLink = clickLink;
    }

    public String getExposeLink() {
        return exposeLink;
    }

    public void setExposeLink(String exposeLink) {
        this.exposeLink = exposeLink;
    }

    public String getCurrentPrice() {
        return currentPrice;
    }

    public void setCurrentPrice(String currentPrice) {
        this.currentPrice = currentPrice;
    }

    public String getOriginalPrice() {
        return originalPrice;
    }

    public void setOriginalPrice(String originalPrice) {
        this.originalPrice = originalPrice;
    }

    public String getDiscount() {
        return discount;
    }

    public void setDiscount(String discount) {
        this.discount = discount;
    }

    public String getCurrencyMark() {
        return currencyMark;
    }

    public void setCurrencyMark(String currencyMark) {
        this.currencyMark = currencyMark;
    }
}
```

