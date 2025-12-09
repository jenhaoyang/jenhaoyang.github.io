---
title: azure-blod作為硬碟儲存訓練照片
date: 2025-12-08 19:03:47
categories:
tags:
---
建立 Azure Blob Storage 其實包含兩個主要步驟：首先要建立一個 **「儲存體帳戶 (Storage Account)」**，然後在該帳戶底下建立 **「容器 (Container)」**（這個容器才是真正用來放檔案的地方）。

以下是透過 **Azure Portal (網頁介面)** 最簡單的建立流程：

### 第一階段：建立儲存體帳戶 (Storage Account)

1.  登入 [Azure Portal](https://portal.azure.com)。
2.  在上方搜尋列輸入 **"Storage accounts"** (或是 "儲存體帳戶") 並點擊進入。
3.  點擊左上角的 **「+ Create (+ 建立)」**。
4.  **填寫基本資訊 (Basics)：**
    *   **Subscription (訂用帳戶)**：選擇您的付費帳戶。
    *   **Resource Group (資源群組)**：點擊 "Create new"，取個名字（例如 `rg-my-training-data`）。這是用來分類資源的群組。
    *   **Storage account name (帳戶名稱)**：**最重要的一步**。
        *   必須是**全球唯一**（不能跟別人重複）。
        *   只能用**小寫英文**和**數字**。
        *   長度 3~24 字元。
    *   **Region (區域)**：選擇離您（或您的 VM）最近的地方，例如 **Japan East (日本東部)** 或 **East Asia (香港)**。*(注意：台灣目前可能沒有公開的區域可選，除非您有特殊權限，通常選日本或香港速度最快)*。
    *   **Performance (效能)**：選 **Standard (標準)** 即可。
        *   *Premium (進階)* 適合極低延遲需求，但較貴，存照片用 Standard 就夠了。
    *   **Redundancy (備援)**：
        *   **LRS (本地備援)**：最便宜，資料在同一機房複製 3 份。
        *   **GRS (異地備援)**：較貴，資料會複製到另一個國家（防地震）。
        *   *建議：測試用選 LRS，正式重要資料選 GRS。*

5.  點擊下方的 **「Review + create (檢閱 + 建立)」**，確認無誤後點擊 **「Create (建立)」**。
6.  等待幾分鐘部署完成，點擊 **「Go to resource (前往資源)」**。

---

### 第二階段：建立容器 (Container)

有了帳戶後，您需要一個「容器」來分類檔案（這就像是 Windows 的資料夾，或者是 AWS S3 的 Bucket）。

1.  在剛剛建立好的儲存體帳戶頁面，左側選單找到 **「Data storage (資料儲存體)」** -> **「Containers (容器)」**。
2.  點擊上方的 **「+ Container」**。
3.  **Name (名稱)**：取個名字，例如 `photos` 或 `training-data`（只能小寫、數字、連字號）。
4.  **Public access level (公用存取層級)**：
    *   **Private (no anonymous access)**：**強烈建議選這個**。這代表只有擁有金鑰或登入的人才能存取（最安全）。
    *   *Blob / Container*：這會讓檔案變成公開網址，任何人拿到網址都能下載（除非您要做公開網站，否則不要選）。
5.  點擊 **「Create (建立)」**。

恭喜！您現在已經有一個可以上傳照片的地方了。

---

### 第三階段：取得連線資訊 (給 BlobFuse2 或程式使用)

為了讓您的 Docker、BlobFuse2 或 Python 程式能連線到這裡，您需要 **金鑰 (Key)**。

1.  在儲存體帳戶頁面，左側選單找到 **「Security + networking (安全性 + 網路)」** -> **「Access keys (存取金鑰)」**。
2.  您會看到 `key1` 和 `key2`（兩個都可以用）。
3.  點擊 **「Show (顯示)」**。
4.  複製 **Connection string (連接字串)** 或 **Key (金鑰)**。

**在 BlobFuse2 的 `config.yaml` 設定中：**
*   `account-name`: 您在第一階段設定的帳戶名稱。
*   `account-key`: 剛剛複製的 Key 內容。
*   `container`: 您在第二階段建立的容器名稱 (例如 `photos`)。

---

### 進階小提示 (省錢設定)

既然您是要存「訓練用的照片」，通常是寫入一次，讀取多次。您可以在建立帳戶時（或建立後），注意 **Access Tier (存取層)** 的設定：

*   **Hot (熱)**：預設值。儲存費稍高，讀寫費較低。適合**正在訓練中**的資料。
*   **Cool (冷)**：儲存費較低，讀寫費稍高。適合**存著備用**，幾個月才讀一次的資料。

如果您的照片是長期封存，建議設定 Lifecycle Management (生命週期管理)，讓超過 30 天沒用的檔案自動轉成 Cool Tier 以節省費用。



將 BlobFuse2 掛載為 Linux 檔案系統主要分為三個步驟：**安裝**、**設定設定檔 (config.yaml)**、以及**執行掛載**。

相比舊版 (v1)，BlobFuse2 效能更好且功能更完整，以下是詳細操作流程：

### 1. 安裝 BlobFuse2
BlobFuse2 需要 `fuse3` 函式庫支援。以下以 Ubuntu 為例 (其他發行版請參考微軟官方 Repo)：

```bash
# 1. 設定微軟的 Package Repository (以 Ubuntu 22.04 為例)
wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update

# 2. 安裝 BlobFuse2 與 Fuse3
sudo apt-get install fuse3 libfuse3-dev blobfuse2
```

### 2. 準備設定檔 (config.yaml)
BlobFuse2 推薦使用 YAML 格式的設定檔。請在任意位置（例如 `~/blobfuse2-config.yaml`）建立檔案，並貼上以下內容：

```yaml
allow-other: true  # 允許其他使用者（包含 Docker）存取，非常重要
logging:
  type: syslog
  level: log_warning

components:
  - libfuse
  - file_cache
  - attr_cache
  - azstorage

libfuse:
  attribute-expiration-sec: 120
  entry-expiration-sec: 120
  negative-entry-expiration-sec: 240

file_cache:
  path: /mnt/blobfuse_cache  # 本地快取路徑 (建議使用 SSD)
  timeout-sec: 120
  max-size-mb: 10240         # 快取最大空間 (MB)

attr_cache:
  timeout-sec: 7200

azstorage:
  type: block
  account-name: <你的儲存帳戶名稱>
  account-key: <你的儲存帳戶金鑰>
  endpoint: https://<你的儲存帳戶名稱>.blob.core.windows.net
  mode: key
  container: <你要掛載的容器名稱>
```

**關鍵設定說明：**
*   **path**: 快取資料夾，請確保該目錄存在 (`sudo mkdir -p /mnt/blobfuse_cache`) 且有寫入權限。
*   **allow-other**: 如果你要給 Docker 或非 root 使用者讀取，此項必須為 `true`。

### 3. 執行掛載
建立掛載點目錄並執行掛載指令：

```bash
# 1. 建立掛載點
sudo mkdir -p /mnt/my_blob_container

# 2. 執行掛載
# 語法：sudo blobfuse2 mount <掛載點> --config-file=<設定檔路徑>
sudo blobfuse2 mount /mnt/my_blob_container --config-file=./blobfuse2-config.yaml
```

### 4. 常見問題與權限設定 (重要)

如果您遇到 **"Permission denied"** 或 Docker 無法讀取的問題，請檢查以下兩點：

1.  **修改 `/etc/fuse.conf`**：
    打開該檔案，找到 `#user_allow_other`，將前面的 `#` 拿掉並存檔。這允許非 root 使用者使用 `allow_other` 參數。

2.  **確保 `file_cache` 權限**：
    BlobFuse2 需要在快取目錄寫入暫存檔。如果該目錄權限不足，掛載會失敗或運作異常。
    ```bash
    sudo chmod 777 /mnt/blobfuse_cache
    ```

### 5. 如何設定開機自動掛載 (/etc/fstab)
若要開機自動掛載，請編輯 `/etc/fstab` 加入以下一行：

```fstab
blobfuse2 /mnt/my_blob_container fuse3 defaults,_netdev,allow_other,--config-file=/path/to/blobfuse2-config.yaml 0 0
```
*注意：請確保 config.yaml 的路徑是絕對路徑。*

### 6. 卸載方式
如果不使用了，請使用標準的 Linux 卸載指令：

```bash
sudo umount /mnt/my_blob_container
# 或者 (如果卡住的話)
sudo umount -l /mnt/my_blob_container
```