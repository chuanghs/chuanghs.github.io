---
layout: post
title: Spring-boot 多個 datasource 設定
categories: spring-boot
---

Spring-boot 預設有一個 datasource，若程式需要多個 datasource，可採用以下設定方式：

1. 在 application.properties 中加上第二個 datasource 的相關設定

{% highlight properties %}
domain2.datasource.url=<jdbc url>
domain2.datasource.username=
domain2.datasource.password=
domain2.datasource.driver-class-name=
{% endhighlight %}

2. 宣告對應的 DataSourceProperties，就能夠接到上一步設定的 properties 值

{% highlight java %}
@Bean
@ConfigurationProperties("domain2.datasource")
public DataSourceProperties domain2DataSourceProperties() {
  return new DataSourceProperties();
}
{% endhighlight %}

3. 宣告對應的 DataSource
{% highlight java %}
@Bean(name="hrmsDataSource")
public DataSource hrmsDataSource() {
  return hrmsDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
}
{% endhighlight %}

接下來就可以利用 @Qualifier，透過 name 指定要連結的 DataSource 了。


