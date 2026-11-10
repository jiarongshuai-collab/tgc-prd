# TGC OMS（訂單管理系統）MVP 版本產品需求文件 (PRD)

> **所屬體系**：TGC 綜合營運中台  
> **關聯系統代碼**：`cms/frontend/cms.html` · `cms/services/cms_db.js` · `shared/card_db/orders_db.js` · `shared/pricing_engine/`  
> **預設語言體系**：**English (en)** 與 **繁體中文 (zh-TW)** 雙語支援  
> **角色與權限**：**Super Admin（超級管理員）** 單一權限體系  

---

## 1. 文檔定位與業務邊界

### 1.1 系統定位
OMS (Order Management System) 內嵌於 TGC CMS 運營中台（`cms/frontend/cms.html`），作為客戶送評訂單的全流程生命週期調度核心。
* **對上**：接收 Web 門戶（`web/frontend/index.html`）與 移動端 App（`app/frontend/app.html`）提交的送評訂單、申報貨值及在途物流。
* **對下**：與 GMS（評級流水線 `gms_workflow.js`）及 Pricing Engine（公允估值與匯率引擎）緊密聯動，協同處理拆包入庫、進度推進、超額補差價（Upcharge Hold）審核、綜合評級報告交付與回寄發貨。

### 1.2 核心原則
1. **以「訂單（Order）」為全局管控核心**：財務、物流、客戶通知均以 Order 為主體；報告以訂單為單位交付，內部掛載各單卡（Card Item）的獨立評級報告。
2. **多語言架構**：所有狀態碼、介面標籤、通知信件、交付報告全面支援 `en` 與 `zh-TW`。

---

## 2. 角色與權限架構 (RBAC)

依據業務架構規劃，**OMS 在 MVP 階段僅設立單一角色：Super Admin（超級管理員）**。

### 2.1 角色職責定義
* **角色名稱**：`Super Admin`（超級管理員 / 系統總控）
* **代表賬號**：`admin@tgc.com` / `cms-super-admin`
* **管轄邊界**：擁有 OMS 全域最高調度與決策權限，涵蓋：
  1. **訂單全狀態推進**：手動或掃碼執行拆包收件（`PACKAGE_RECEIVED`）、推進狀態至下一裡程碑。
  2. **Upcharge（超額補差價）審核**：審批因卡牌評級後市場公允價值（FMV）超過方案上限而產生的補價差額單。
  3. **異常工單仲裁**：處理拆包數量不符、貨不對板、在途破損等異常，決定退回或扣款。
  4. **物流打單與回寄發貨**：呼叫快遞電子面單、列印寄送裝箱清單（Packing Slip）、輸入/更新回寄追蹤碼。
  5. **報告發布與封裝歸檔**：審核 GMS 回傳的單卡數據，一鍵合成訂單總評級報告並對外開放查驗。

---

## 3. 雙語語言規格 (Bilingual i18n Specification)

系統支援即時切換 `English` 與 `繁體中文`，關鍵業務詞彙與介面定義如下：

| 欄位 / 狀態代碼 | English (en) | 繁體中文 (zh-TW) | 業務說明 |
| :--- | :--- | :--- | :--- |
| `ORDER_CREATED` | Order Created / Pending Intake | 已下單待收件 | 用戶已完成支付，等待快遞送達金庫 |
| `PACKAGE_RECEIVED`| Package Received / Unboxing | 已收件入庫 / 拆包核對 | 倉庫簽收外箱面單，核對卡片數量無誤 |
| `GRADING` | In Grading / Vision Inspection | 評級作業中 | GMS 機台掃描、OCR 辨識與 AI 檢測 |
| `QUALITY_REVIEW` | Quality Review / Human Audit | 複核評審中 | 評級師微觀複核與爭議仲裁 |
| `SLABING` | Slabing & Encapsulation | 封磚封裝中 | 標籤列印與超聲波物理外殼封裝 |
| `SHIPPED` | Outbound Shipped | 已寄出回送 | 順豐/DHL/FedEx 回寄發貨 |
| `COMPLETED` | Order Completed / In Vault | 已結案完成 / 金庫保管 | 用戶簽收或轉入 WMS 實體金庫 |
| `UPCHARGE_HOLD` | Upcharge Hold (Action Required)| 超額補差扣留 | 評級後市場公允價值超出方案上限 |
| `EXCEPTION_HOLD` | Exception Hold | 異常工單掛起 | 實物破損、漏寄、錯發需用戶確認 |

---

## 4. 核心功能規格詳細定義

### 4.0 前端客戶建卡與清單錄入機制 (Web & App Intake Mechanisms)

為兼顧藏家在不同終端的錄入習慣與大批量送評效率，系統支援多模態建卡與快速清單構建：

* **4.0.1 Web 端官方卡庫多維搜索錄入 (Search & Add)**：
  * **檢索接口**：Web 前端串接 `/api/search-cards?q={query}` 專屬卡庫 API，支援多幣種即時公允市價回傳。
  * **建議檢索格式 (Suggested Format Tags)**：支援一鍵追加檢索標籤（遊戲 `Game`、發行年份 `Release Year`、卡牌語言 `Language`、系列名稱 `Set Name`、卡牌名稱 `Card Name`、卡號/編號 `Serial No.`）。
  * **聯想下拉與一鍵加單**：輸入關鍵字（如 `pikachu`、`024`）毫秒級展開候選列表，點擊任一卡片即可將名稱、版次、稀有度、參考市值自動帶入送評清單。

* **4.0.2 Web 端行內一鍵複製功能 (Card Item Duplicate)**：
  * **高頻業務場景**：專為藏家一次性送評多張同款或同版次卡牌沖評（如 5 張同一版皮卡丘拼 Gem Mint 10 分）設計，避免重複檢索。
  * **操作交互**：送評清單表格中每行均配置 `⎘ 複製 (Duplicate)` 操作按鈕。
  * **數據處理**：點選後在當前行下方瞬間插入副本，完整繼承該卡牌的全部元數據（卡名、版次、稀有度、申報價值、縮略圖），並自動分配唯一的臨時草稿編碼（Draft Cert ID），同時彈出操作反饋 Toast。
  * **清單輔助調序**：支援行內 `▲ 上移`、`▼ 下移` 與 `✕ 移除`，方便藏家依實物擺放順序整理清單。

* **4.0.3 移動端 App 拍照掃卡建冊同步 (Mobile Camera Scan & Binder Sync)**：
  * **App 掃描建冊**：用戶在手機 App 端使用相機進行 OCR 光學比對，卡牌直接收入個人數位卡冊（Binders）。
  * **Web 聯動拉取**：Web 端「送評物料錄入」提供「手機掃卡模擬抽屜 / 跨端卡冊同步」，一鍵將 App 端掃描的卡牌整批拉取至 Web 購物車送評清單中。

* **4.0.4 申報價值動態校驗與超額預警 (Declared Value Real-time Validation)**：
  * 用戶在清單中可自由手動修改申報貨值（Declared Value）。
  * 若任一卡牌申報貨值大於當前所選方案的最高保額上限（Max Insured Value），系統即時高亮顯示超額警告（Notice Over Value），提示用戶升級至更高方案（如 `Express` 或 `Super Express`），防範後續 GMS 判定引發的 Upcharge 扣留。

---

### 4.1 訂單進件與收件調度看板（結合現有 `tab-oms`）

* **4.1.1 頂部 5 大全景指標卡（Metric Cards）**：
  * **送評訂單總量 (Total Orders)**：全渠道提交累計訂單總數。
  * **待拆包簽收 (Pending Intake)**：處於 `ORDER_CREATED` 且有物流追蹤號的待處理包件。
  * **評級流轉中 (In Progress)**：處於 `GRADING`、`QUALITY_REVIEW`、`SLABING` 的作業中訂單。
  * **已結案完成 (Completed)**：已出庫簽收或入金庫的訂單。
  * **超額補差扣留 (Upcharge Hold)**：觸發補款扣留待審核或待用戶支付的單量。

* **4.1.2 訂單管理主列表**：
  * 列表欄位：訂單號 (`Order ID`)、客戶姓名 (`Customer Name`)、送評方案 (`Service Tier`)、卡牌數量 (`Card Count`)、申報總價值 (`Declared Value`)、支付狀態 (`Payment Status`)、工單流轉狀態 (`Status`)、承運物流商 (`Carrier & Tracking`)、下單時間 (`Created At`)、操作項 (`Actions`)。
  * **篩選矩陣**：
    * 支付狀態：`ALL` / `PAID` / `UNPAID` / `UPCHARGE`。
    * 工單狀態：支援按 7 大標準狀態及 2 個掛起狀態過濾。
    * 快速搜尋：支援模糊搜尋「訂單號 / 姓名 / 物流單號 / Email」。

---

### 4.2 服務方案與定價體系規格（對齊 `orders_db.js`）

系統原生支援多幣種自動換算（USD, HKD, TWD, JPY, EUR），預設展示標準：

| 方案代碼 (`id`) | 方案名稱 (Name) | 基準價格 (USD) | 申報保價上限 (Max Insured) | 預估週轉週期 (Turnaround) | 包含增值權益 (Features) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `STANDARD` | Standard (標準送評) | $19 (約 HK$148) | $200 (約 HK$1,564) | 40 - 50 工作日 | 數位報告、UV防護、外殼封裝 |
| `EXPRESS` | Express (加急送評) | $45 (約 HK$352) | $1,000 (約 HK$7,820) | 10 - 15 工作日 | 優先排期、360°高清微距視頻 |
| `SUPER_EXPRESS` | Super Express (特快) | $120 (約 HK$938) | $5,000 (約 HK$39,100) | 3 - 5 工作日 | 專人跟進、金色專屬封磚標籤 |

---

### 4.3 Upcharge（超額估值補差價）核心機制

* **4.3.1 觸發條件**：
  * 當單卡在 GMS 評級完畢後，GMS 呼叫 `Pricing Engine` 計算該卡牌在特定評級分下的**公允市場價值（FMV）**。
  * 若單卡 $\text{FMV} > \text{方案最高保價上限 (Max Insured Value)}$，系統自動觸發 Upcharge 計算。
* **4.3.2 補差公式**：
  $$\text{Upcharge Fee} = \text{適合該 FMV 的目標方案價格} - \text{原已付方案價格} + \text{超額保險費}$$
* **4.3.3 調度與阻斷規則**：
  1. 訂單狀態自動變更為 `UPCHARGE_HOLD`，阻斷 GMS 進入最後出庫發貨。
  2. Super Admin 在後台審核補差明細，點選「批准並發送補繳帳單」。
  3. Web/App 客戶端彈窗提醒並發送雙語郵件，用戶線上完成補款後，訂單自動解除掛起，推進至 `SLABING` / `SHIPPED`。
  4. 若用戶拒絕補繳，系統執行**降級保價出庫**（需用戶線上手動簽署《超額免責保證書》）。

---

### 4.4 異常攔截與調度處置規則

| 異常類型 | 系統識別路徑 | Super Admin 處置動作 | 用戶端 (Web/App) 互動與結果 |
| :--- | :--- | :--- | :--- |
| **入庫數量短少** | 拆包核驗卡數 < 預報數量 | 點選「數量異常」，上傳拆箱開箱視頻，發起核驗工單 | 用戶收到確認通知：選擇「確認漏發按實收評級（自動退差價）」或「整包暫扣暫停」。 |
| **入庫卡種不符** | 實物版次與申報不一致 | 修正單卡關聯至 CMS 正確卡片 ID | 若價值落差大，系統提示補繳或退還保價差額。 |
| **在途物流受損** | 拆包前外箱已嚴重破損 | 拍攝外觀留證，標記 `CARRIER_DAMAGED` | 系統生成《物流受損存證單》，協助用戶向承運快遞索賠。 |
| **假卡/改卡拒絕封裝** | GMS 判定 `COUNTERFEIT` | 審核 GMS 提交的微觀防偽存證 | 扣除基礎鑑定工本費，退還封磚費；隨單出具《不予評級存證報告》原卡退回。 |

---

### 4.5 報告聚合與雙語交付規格

* **訂單綜合報告包含兩大層次**：
  1. **第一層：訂單級綜合履約總覽 (Order Consolidated Report)**
     * 訂單識別碼、客戶編號、方案等級、總送評卡數、滿分率 (Gem Mint 10 %)、整體估值增幅 ROI。
  2. **第二層：單卡獨立評級證書 (Individual Card Specimen Report)**
     * 證書唯一號（`Cert Number: TGC-2026-XXXXX`）、標準卡名、編號、稀有度。
     * 四大微觀子項小分（Centering 居中度、Corners 四角、Edges 邊緣、Surface 表面）。
     * 高清正反面 4K 實拍圖、微距網點防偽圖、防偽動態 QR Code。
* **語言切換機制**：
  * 用戶在線上查驗報告時，點選頂部 `[EN / 繁中]` 即可無縫切換報告語言，PDF 下載時可自選輸出語言版本。

---

## 5. 資料庫實體規格 (對齊現有 `customer_orders`)

### 5.1 訂單主表 (`customer_orders`)
```sql
CREATE TABLE IF NOT EXISTS customer_orders (
    id TEXT PRIMARY KEY,                       -- e.g. 'ORD-2026-8801'
    user_id TEXT NOT NULL,                     -- e.g. 'USER-8801'
    user_name TEXT NOT NULL,                   -- 用戶姓名
    user_email TEXT NOT NULL,                  -- 聯繫郵箱
    service_tier TEXT NOT NULL,                -- 'STANDARD', 'EXPRESS', 'SUPER_EXPRESS'
    status TEXT NOT NULL,                      -- 7大里程碑 + 'UPCHARGE_HOLD' + 'EXCEPTION_HOLD'
    currency TEXT NOT NULL DEFAULT 'HKD',      -- 結算幣種
    service_fee REAL NOT NULL DEFAULT 0.0,     -- 基礎評級費
    shipping_fee REAL NOT NULL DEFAULT 0.0,    -- 回寄運費
    insurance_fee REAL NOT NULL DEFAULT 0.0,   -- 保價費
    upcharge_fee REAL DEFAULT 0.0,             -- 超額補差金額
    total_amount REAL NOT NULL DEFAULT 0.0,    -- 實收總金額
    payment_status TEXT NOT NULL DEFAULT 'PAID',-- 'PAID', 'UNPAID', 'UPCHARGE'
    carrier TEXT,                              -- 承運商 (SF Express / DHL)
    tracking_number TEXT,                      -- 回寄單號
    inbound_tracking_no TEXT,                  -- 寄入單號
    dispatch_date TEXT,                        -- 寄出時間
    timeline_json TEXT,                        -- 履約節點時間戳記 JSON
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 5.2 訂單明細項表 (`customer_order_items`)
```sql
CREATE TABLE IF NOT EXISTS customer_order_items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    order_id TEXT NOT NULL,                    -- 關聯主訂單
    card_id TEXT NOT NULL,                     -- 關聯 Card DB 標準卡牌
    card_name TEXT,                            -- 官方卡名 (雙語)
    set_name TEXT,                             -- 系列名稱
    card_number TEXT,                          -- 卡號 (e.g. '025/165')
    image_url TEXT,                            -- 卡片預覽圖
    declared_value REAL NOT NULL DEFAULT 0.0,  -- 用戶申報價值
    fmv_calculated REAL DEFAULT 0.0,           -- 評級後市場公允價值
    mission_id TEXT,                           -- 關聯 GMS 工單號
    cert_number TEXT,                          -- 證書編號 (e.g. 'TGC-2026-88912')
    grade REAL,                                -- 最終總分 (1.0~10.0)
    grade_name TEXT,                           -- 評級名稱 (Gem Mint, Mint, etc.)
    subgrade_centering REAL,                   -- 居中度小分
    subgrade_corners REAL,                     -- 四角小分
    subgrade_edges REAL,                       -- 邊緣小分
    subgrade_surface REAL,                     -- 表面小分
    status TEXT DEFAULT 'PENDING',             -- 'PENDING', 'IN_GMS', 'COMPLETED', 'REJECTED'
    report_url TEXT,                           -- 單卡報告 CDN 鏈接
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(order_id) REFERENCES customer_orders(id)
);
```
