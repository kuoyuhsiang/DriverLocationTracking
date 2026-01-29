## Init version
**A. 司機端**
* 大約有 3,000 名司機在線 
* 每 2 秒會發送一次 GPS 數據 
* 任務：告訴伺服器「司機是誰，司機人在哪」

**B. 乘客端**
* 他們會向伺服器送請求：「給我目前位置附近有哪些司機」
* 沒有使用者人數

**C. 伺服器**
* 它必須同時支援寫入與查詢
* 負責把司機傳來的座標存進資料庫 
* 乘客需要去讀取司機位置資料

```mermaid
graph LR
    Driver[司機端 Driver App] -- "更新座標 (每2秒)" --> Service[後端服務 Tracking Service]
    Rider[乘客端 Rider App] -- "查詢附近司機" --> Service
    Service -- "讀取/寫入" --> DB[(資料庫 Database)]
```
## 2th version
**A. 司機端**
* 會有大流量進入， 1,500 次請求/秒 (RPS)
* 快速上傳定位數據給伺服器

**B. 乘客端**
* 偏向「一般功能查詢」，乘客查詢「附近司機」單純種商業邏輯請求。

### 細部元件
* Load Balancer (負載平衡器)：它是系統的「分流器」。當司機們發送 GPS 數據時，負載平衡器會把這些請求平均分配給後端多台 Service 。

* API Gateway (API 閘道)：在處理乘客端的請求時，它負責檢查乘客身份（Auth）並限制流量（Rate Limiting）。

### 細部設計
**A. 司機端 - 追求高吞吐量，就適合選 Load Balance**

**B. 乘客端 - 一般 API 請求且不考慮量大，可以選 API Gateway 即可**

```mermaid
graph LR
    subgraph Clients
        Driver[司機端 Driver App]
        Rider[乘客端 Rider App]
    end

    %% 司機走 LB 追求高吞吐量
    Driver -- "高頻更新" --> LB[Load Balancer]
    LB --> S1[ Service A]
    LB --> S2[ Service B]

    %% 乘客走 Gateway 追求安全控管
    Rider -- "查詢請求 (HTTPS)" --> GW[API Gateway]
    GW --> S1[ Service A]
    GW --> S2[ Service B]

    S1 & S2 --- DB[(資料庫 Database)]
```