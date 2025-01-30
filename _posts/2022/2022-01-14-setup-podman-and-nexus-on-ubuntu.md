---
layout: post
title: 在 Ubnutu 21.10 上用 podman 架設 nexus 服務
categories: ubuntu, podman, nexus, 假裝自己是系統工程師
---

最近因為轉換工作，寫程式從本業變成興趣，自然需要架設一些個人開發的時候需要的東西，像是放個人函式庫用的 maven 儲存庫，順便玩一下 podman（為什麼不用 docker ? 這個嘛，自然有一些原因，本來還打算直上 singularity 的，不過，那個等之後再說了)。

## Ubuntu 21.10  安裝 podman

podman 套件自 21.10 之後就進入了官方儲存庫，不介意用的版本稍舊的話，直接執行
`
suto apt install podman
`

就行了，如果想用比較新的版本，就得用 [Kubic project](https://build.opensuse.org/package/show/devel:kubic:libcontainers:stable/podman) 的套件，使用的命令如下：

```bash
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_21.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_21.04/Release.key" | sudo apt-key add -
sudo apt update
sudo apt upgrade
sudo apt install podman
```

(因為目前 Kubic project 還沒有支援 Ubuntu 21.10，所以得使用 21.04 的儲存庫)。


安裝過程應該會十分順利，安裝完畢後，可以執行

```bash
podman --version
```

或

```bash
podman info
```

確認是否安裝成功

要特別注意的是，千萬不要混用 ubuntu 官方儲存庫的套件跟 Kubic project 的套件，否則，輕則安裝過程中出現一堆套件衝突，重則安裝完畢後 podman 發生神秘行為。


## 用 podman 安裝 nexus 服務

因為只是內網中的個人用服務，檔案權限的設定上採取比較放鬆的方式。

先建立儲存資料用的空間，並設定對應的權限
```bash
mkdir /opt/nexus
chown -R <username>.<usergroup> /opt/nexus
mkdir /opt/nexus/nexus-data
chown -R 200 /opt/nexus/nexus-data
```

`<username>' 與 '<usergroup>' 請依自身主機實際狀況填入，因為我的 /opt 所有者不是執行 podman 的帳號，所以一定要建兩層，不然啟動  nexus 的時候會發生問題。

確認目錄權限正常後，就可以啟動 nexus 服務了：

```bash
podman run -d --user <userid> -p <外部port>:8081 --name nexus -v /opt/nexus/nexus-data:/nexus-data sonatype/docker-nexus3
```

由於啟動需要一些時間，可以用以下命令檢查 log，確認服務是否開啟：
```bash
podman logs -f nexus
```

服務開啟後，就可以從網址 http://<主機>:<外部 port> 連上 nexus 服務了。

第一次登入的時候，會需要修改 admin 密碼，預設密碼可以用以下命令取得：
```bash
podman exec -it nexus more /nexus-data/admin.password
```

修改完密碼後就可以正常使用了。

## 用 podman play 儲存容器設定

podman 原生就提供了類似 docker-compose 的功能，還能夠儲存執行中的容器設定，在上述的 nexus 容器執行中的狀態下，執行：

```bash
podman play kube nexus >> nexus.yaml
```

就可以將名為 nexus 容器的設定狀態儲存到 nexus.yaml 檔中，檔案內容是 k8s 使用的 yaml 格式。

接下來，停止先前手寫命令啟動的容器：
```bash
podman stop nexus
podman rm nexus
```

就可以用`podman play` 與儲存下來的 nexus.yaml 檔啟動與停止服務了：

啟動：
```bash
podman play kube nexus.yaml
```

停止：
```bash
podman play kube --down nexus.yaml
```


現在，就有一個可正常使用的 nexus 服務了。