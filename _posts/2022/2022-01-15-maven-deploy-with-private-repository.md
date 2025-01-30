---
layout: post
title: maven deploy 搭配私有 nexus 服務
categories: java maven nexus 假裝自己是系統工程師
---

 接續  [在Ubuntu 21.10 上用 podman 架設 nexus 服務]({% link _posts/2022-01-14-setup-podman-and-nexus-on-ubuntu.md %}) ，裝好 nexus 之後，當然是需要設定 maven 環境，把個人專案放到私有的 nexus 服務上頭了。


nexus 3 安裝後，預設會有 maven-central, maven-public, maven-releases, maven-snapshots 這四個 maven2 的儲存庫，分別是：

maven-central
: maven 官方儲存庫的中介

maven-public
: 聚合了另外三個 maven 儲存庫

maven-releases
: 放置私有發佈版本套件的位置

maven-snapshots
: 放置私有快照（snapshot）版本套件的位置 

## 全域 .m2/settings.xml 設定

首先是定義連結各 repository 時使用的帳號密碼
```xml
  <servers>
    <server>
      <id>maven-public</id>
      <username>...</username>
      <password>...</password>
    </server>
    <server>
      <id>maven-releases</id>
      <username>...</username>
      <password>.../password>
    </server>
    <server>
      <id>maven-snapshots</id>
      <username>...</username>
      <password>...</password>
    </server>
  </servers>
```

接著是設定 mirror 以及對應的儲存庫（repository）：
```xml
  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*</mirrorOf>
      <url>http://grave:8081/repository/maven-public</url>
    </mirror>
  </mirrors>
  <profiles>
    <profile>
      <id>nexus</id>
      <!--Enable snapshots for the built in central repo to direct -->
      <!--all requests to nexus via the mirror -->
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
      
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
```

最後再把 profile 設為 active
```xml
  <activeProfiles>
    <activeProfile>nexus</activeProfile>
  </activeProfiles>
```

全域設定部份，這樣就算完成了。

# 個別專案設定

個簣專案部份需要設定散佈的位置：

```xml
    <distributionManagement>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <name>my internal repository</name>
            <url>maven-snapshots url</url>
        </snapshotRepository>
        <repository>
            <id>my-releases</id>
            <name>My internal repository</name>
            <url>maven-releases url</url>
        </repository>
    </distributionManagement>
```

要注意的是 distributionManagment 裡的 id 要跟 `setting.xml` `<server>` 的 id 相同。

如此就能夠透過 `mvn deploy` 就能夠把 jar 檔放到私有的儲存庫上頭了。