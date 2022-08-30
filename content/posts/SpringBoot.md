---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "SpringBoot整合elasticsearch和redis的问题"
subtitle: ""
summary: ""
authors: []
tags: []
categories: []
date: 2020-05-22T12:38:34+08:00
lastmod: 2020-05-22T12:38:34+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

## Spring Boot

#### HTTPS

1. 生成证书

   ```
   keytool -genkeypair -alias alias_name -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore alias_name.p12 -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -validity 3650
   ```

2. 将证书放到resources目录下

3. spring boot 配置

   ```
   server:
     port: 8443
     ssl:
       enabled: true
       key-alias: alias_name
       key-store-password: ******
       key-store: classpath:alias_name.p12
       key-store-type: PKCS12
   ```

4. 客户端配置

   ```
   // 自定义 RestTemplate
   RestTemplate serverRestTemplateWith() {
           SSLContext sslContext = getSSLContext();
           SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(sslContext);
           CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(socketFactory).build();
           HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
           factory.setConnectTimeout(30000);
           factory.setReadTimeout(30000);
           return new RestTemplate(factory);
       }
   
   // 构建 SSLContext
   SSLContext getSSLContext() {
           try {
               AppProperties appProperties = appContext.getAppProperties();
               return SSLContexts.custom()
                       .loadTrustMaterial(ResourceUtils.getFile(appProperties.getServerKeyStore()), appProperties.getServerKeyStorePassword().toCharArray())
                       // 跳过证书检查
   //                    .loadTrustMaterial(null, (X509Certificate[] chain, String authType) -> true)
                       .build();
           } catch (Exception e) {
               throw new RuntimeException(e);
           }
       }
   ```


#### 生成证书

```
keytool -genkeypair -alias bc -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore bc.p12 -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -validity 3650
```

### 问题

1. 问题描述：availableProcessors is already set to [8], rejecting [8]

解决：https://blog.csdn.net/busishenren/article/details/90478928