## Day 27 - 漫步在雲端：在 GCP 中建立 k8s 叢集

### 本日共賞

* 啟用 GCP
* 建立 k8s

### 希望你知道

* [k8s 不自賞 - 基礎篇](https://ithelp.ithome.com.tw/articles/10192193)
* [gcloud](https://cloud.google.com/sdk/gcloud/?hl=zh-TW)

今天我們來看看如何在 [GCP](https://cloud.google.com) (Google Cloud Platform) 部署 k8s

<br/>

#### 啟用 GCP

任何一個有效的 `Google` 帳號 + 信用卡都可以啟用 [GCP](https://cloud.google.com)，目前 `Google` 提供一年 [300 美金的免費方案](https://cloud.google.com/free/?hl=zh-tw)，不論你使用 `GCP` 上任何資源皆可抵扣。而 `GCP` 底下的 `Kubernetes Engine` (GKE, 之前叫 Google Container Engine) 就是提供 k8s 叢集管理的服務。

> `GCP` 申請過程很容易，這邊就不帶著大家一步一步做。[申請網址在這](https://console.cloud.google.com/freetrial)
> 
> 之後可以連到 [Console 網址](https://console.cloud.google.com)，就可以操作 `GCP` 
>

完成申請後進入 [Console](https://console.cloud.google.com)，你會發現 GCP 會自動幫你建立一個新的專案。

> 你也可以自己建立一個新專案

![](https://ithelp.ithome.com.tw/upload/images/20171226/201070627LCeIpyw0S.png)


點開 `Cloud Shell`，初次啟動會花點時間，開啟後畫面就像 Mac/Linux 的 Terminal 一樣，可以直接打指令。

![](https://ithelp.ithome.com.tw/upload/images/20171226/20107062ZB8icdME7s.png)

> 為了讓大家更習慣使用 command line，這裡我們使用 gcloud 這個工具來建立 k8s


#### 建立 k8s 步驟

1. 指定專案與區域
2. 建立 k8s
3. 設定本機操作

*1. 指定專案與區域*

首先先使用 gcloud 查詢可用專案並指定使用

> gcloud 與 GCP 的關係類似於 kubectl 與 k8s，我們可以使用 gcloud 來對 GCP 進行操作，例如：新增、查詢與刪除等等。在 Cloud Shell 裡面你可以直接使用 gcloud 來操作 GCP 的雲端功能，不僅僅只是 k8s，其他的功能 (例如 GCE) 也可以透過 gcloud 操作。

```bash
$ gcloud projects list  <=== 查詢專案
PROJECT_ID             NAME              PROJECT_NUMBER
stately-magpie-188902  My First Project  772112589157
<=== stately-magpie-188902 即為專案名稱 

$ gcloud config set project stately-magpie-188902  <=== 指定使用 stately-magpie-188902 專案 
Updated property [core/project].
```

接著指定區域 (compute zone)，我們使用一個近一點的區域 `asia-east1-b`。

```bash
$ gcloud config set compute/zone asia-east1-b  <=== 指定區域為 asia-east1-b
Updated property [compute/zone].
```

> 你可以使用 `gcloud compute zones list` 查詢可用區域

*2. 建立 k8s*

設定好專案跟區域後，就可以透過 gcloud 建立 k8s

```bash
gcloud container clusters create [Cluster Name]
```

這裡我們建立一個名稱為 `ithome` 的叢集

```bash
$ gcloud container clusters create ithome \
--cluster-version=1.8.4-gke.1 \
--machine-type=n1-standard-1 \
--num-nodes=2 \
--disk-size=50 \
--scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
Creating cluster ithome.../
Creating cluster ithome...done.
Created [https://container.googleapis.com/v1/projects/stately-magpie-188902/zones/asia-east1-b/clusters/ithome].
kubeconfig entry generated for ithome.
NAME    LOCATION      MASTER_VERSION  MASTER_IP       MACHINE_TYPE   NODE_VERSION  NUM_NODES  STATUS
ithome  asia-east1-b  1.8.4-gke.1     35.201.165.94  n1-standard-1  1.8.4-gke.1   2          RUNNING
```

> 第一次使用 `gcloud.container.clusters.create` 你可能會看到底下的錯誤訊息
> 
> ```bash
>ERROR: (gcloud.container.clusters.create) ResponseError: code=403, message=The Kubernetes Engine API is not enabled for project stately-magpie-188902. Please ensure it is enabled in the Google Cloud Console at https://console.cloud.google.com/apis/api/container.googleapis.com/overview?project=stately-magpie-188902 and try again.
> ```
>
> 因為 API 還沒有啟用，你可以按照提示到指定的網址啟用 API。上面提示我們可以到 "https://console.cloud.google.com/apis/api/container.googleapis.com/overview?project=stately-magpie-188902" 啟用 API，啟用後就再執行一次上面的指令建立叢集。

* `--cluster-version`：指定叢集版本，[k8s 不自賞](https://ithelp.ithome.com.tw/users/20107062/ironman/1244) 使用的版本為 1.8.4-gke.1

> GKE 支援的 k8s 版本可能會隨著時間改變，你可以在 [Release Notes](https://cloud.google.com/kubernetes-engine/release-notes) 找到目前支援的版本號來建立叢集

* `--machine-type`：指定機器種類，預設會使用 n1-standard-1 (1 vCPU, 3.75 GB Memory) 的機型
* `--num-nodes`：指定 Node 數，預設會啟動三台機器。為什麼我們只開啟兩台機器呢？也沒為什麼就只是示範可以指定 Node 數而已，不過要記得開越多要扣的錢就越多就是了。
* `--disk-size`：指定磁碟大小，預設 100 GB
* `--scopes`： `storage-rw` 啟用 Google Container Registry (類似 Docker Hub) 的存取權限, `projecthosting` 如果要使用 Google Cloud Source Repositories (類似 github) 則可以使用這個選項

這裡會花一些時間建立叢集，基本上跟你選擇的機器種類與叢集大小有關。

接著你可以直接在 `Cloud Shell` 內使用 `kubectl`

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.4", GitCommit:"9befc2b8928a9426501d3bf62f72849d5cbcd5a3", GitTreeState:"clean", BuildDate:"2017-11-20T05:28:34Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"8+", GitVersion:"v1.8.4-gke.1", GitCommit:"04502ae78d522a3d410de3710e1550cfb16dad4a", GitTreeState:"clean", BuildDate:"2017-12-08T17:24:53Z", GoVersion:"go1.8.3b4", Compiler:"gc", Platform:"linux/amd64"}
```

> 在這裡操作 `kubectl` 就與之前文章提到過的操作內容一樣

到這裡，k8s 叢集算是已經建立成功可以直接使用了！你可以在 [Console](https://console.cloud.google.com) 選擇左邊選單內的 `Kubernetes Engine` > `Kubernetes Clusters` 找到新建立的 k8s 叢集。

![](https://ithelp.ithome.com.tw/upload/images/20171226/20107062sPD3auWMk8.png)

*3. 設定本機操作*

如果想要從本機端的 `kubectl` 連到剛剛建立的 `ithome` 叢集，你需要在本機端安裝 [gcloud](https://cloud.google.com/sdk/gcloud/?hl=zh-TW)，完成後先登入

```bash
$ gcloud auth login
```

它會開啟瀏覽器讓你登入 Google 並要求允許存取，接下來取得與 k8s 連線的認證

```bash
$ gcloud container clusters get-credentials ithome \
--zone asia-east1-b \
--project stately-magpie-188902
Fetching cluster endpoint and auth data.
kubeconfig entry generated for ithome.
```

> --zone 與 --project 即建立時指定的專案與地區

你也可以在左邊選單選擇 `Kubernetes Engine` > `Kubernetes Clusters` 點擊 `連線` 查詢連線的指令。 

之後你就不需要再登入 `Google` 使用 `Cloud Shell`，而是在本機端進行操作。

![](https://ithelp.ithome.com.tw/upload/images/20171226/201070624LodqNOlpB.png)

> 這時候你的 kubectl 就可以操作多個環境，還記得怎麼切換嗎？看看 [Day 8 - 與 k8s 溝通: kubectl - 管理多個 k8s 叢集](https://ithelp.ithome.com.tw/articles/10193502) 的內容吧

本文同步發表於 [https://jlptf.github.io/ironman2018-day27/](https://jlptf.github.io/ironman2018-day27/)