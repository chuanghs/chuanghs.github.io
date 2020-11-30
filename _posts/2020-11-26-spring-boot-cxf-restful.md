---
layout: post
title: Spring-boot 2.3.5 + CXF RESTful 實作設定
categories: spring-boot cxf restful
---

因未知原因使得系統原有的 AXIS2 SOAP 服務造成資料庫負載異常過高，因原有服務是用 Java 1.4 時代開發，且沒有使用 connection pool 等機制，除了先將服務後端轉換到 dataguard 資料庫之外，利用機會將使用率最高的兩個服務先轉換為 RESTful 服務，逐步將系統現代化。

轉換過程十分平順，使用 autoscan 的方式帶入需要的 EndPoint，相關設定及重點分列如下。

## application.properties
{% highlight properties %}
cxf.jaxrs.component-scan=true
cxf.jaxrs.classes-scan-packages=<package 名稱>
{% endhighlight %}

classes-scan-pacakges 會自動搜尋下層 package，只需設定適當的上層 package 名稱即可。

## JSON 格式轉換

JSON 格式轉換需要引入 jackson 並特別定義轉換用的 class，先在 pom.xml 中加上

{% highlight xml %}
<dependency>
  <groupId>com.fasterxml.jackson.jaxrs</groupId>
    <artifactId>jackson-jaxrs-json-provider</artifactId>
  <version>2.11.3</version>
</dependency>
{% endhighlight %}

接著在 config class (小型程式可直接宣告在 application class) 中宣告

{% highlight java %}
@Bean
public JacksonJaxbJsonProvider jacksonJaxbJsonProvider() {
  return new JacksonJaxbJsonProvider();
}
{% endhighlight %}

就可能夠正常取得

## EndPoint 注意事項
EndPoint 會對應到某個 RESTful Path，需要注意的事項如下
- 因搭配 Spring-boot，需標上 @Component
- class 層需標上 jax-rs 的 @Path

