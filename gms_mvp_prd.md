# TGC GMS（評級實驗室管理系統）MVP 版本產品需求文件 (PRD)

> **所屬體系**：TGC 智能質檢評級流水線  
> **關聯系統代碼**：`gms/frontend/gms.html` · `gms/system/gms_workflow.js` · `gms/system/authenticator.js` · `gms/system/ai_grader.js` · `gms/system/schema.sql`  
> **標準規範手冊**：`gms/docs/TGC_卡牌评级核心三大支柱体系规范手册.md`  
> **預設語言體系**：**English (en)** 與 **繁體中文 (zh-TW)** 雙語支援  
> **角色與權限**：**Super Admin**、**Grader**、**Ops Manager** 三級權限體系  

---

## 1. 文檔定位與業務邊界

### 1.1 系統定位
GMS (Grading Management System) 是面向 TGC 評級實驗室內部的**工業級智能質檢評級操作系統**（前端適配 iPad Pro 11 吋觸控與桌面工作站 `gms/frontend/gms.html`）。
* **核心邊界**：接收 OMS 下發的訂單預報任務，在實驗室內完成單卡賦碼、定制相機光學採圖、OCR 卡庫匹配、真偽一票否決判定、品相四大維度量化打分、人工複核、防偽封磚入殼及出具「單卡獨立評級報告（DIF Certificate）」。
* **硬件現狀**：定制相機硬件已就位，當前處於**「設備通信傳動調試、微距成像素質校準、算法數據標定」**階段。

---

## 2. 角色與權限架構 (RBAC 三大角色體系)

依據現有 `gms.html` 與實驗室運營架構，GMS 嚴格劃分三大角色：

```
+-----------------------------------------------------------------------------------+
|                                  Super Admin                                      |
|            (系統總控：相機調校、AI權重/閾值配置、人員賬號授權、應急解鎖)           |
+-----------------------------------------+-----------------------------------------+
                                          │
                  ┌───────────────────────┴───────────────────────┐
                  ▼                                               ▼
+-----------------------------------+           +-----------------------------------+
|            Ops Manager            |           |              Grader               |
|          (運營質檢經理)            |           |           (認證評級師)            |
| - 工單批次分配與隊列調度           |           | - 微觀光學圖質檢與真偽複核         |
| - 評級差異仲裁 (Grader vs AI)     |           | - 四大子項評分審查與微調理由標註   |
| - 封磚入殼外觀瑕疵質檢 (QC)       |           | - 假卡/篡改卡一票否決標記         |
| - 終審放行與單卡報告批准          |           | - 簽字確認評級結果                |
+-----------------------------------+           +-----------------------------------+
```

### 2.1 角色職責與介面權限矩陣

| 功能模組 / 操作權限 | Grader (認證評級師)<br/>*代表: Tom Lee* | Ops Manager (運營質檢經理)<br/>*代表: Sara Chang* | Super Admin (系統超級管理員)<br/>*代表: Admin* |
| :--- | :---: | :---: | :---: |
| **頂部切換與工作台視圖** | 專屬評級師工作台 (`list_grader`, 評級明細) | 經理看板 (`list_manager`, 任務派發, 隊列監控) | 全域視圖 (`account_access`, 硬件配置, 審計) |
| **相機掃描控制與重拍** | 支援觸發採圖、標註重拍原因 | 支援強制放行機掃任務 | 完整相機參數調試、傳動驅動控制 |
| **手動翻拍兜底上傳** | 允許翻拍上傳 | 允許審批翻拍圖像 | 允許設置翻拍標準參數 |
| **AI 打分確認與手動調分** | **核心權限**：可微調分數並填寫改分原因 | 複核評級師調整記錄，仲裁分歧 | 具備全域修改權限 |
| **假卡/改卡拒絕判定** | 標記疑似假卡，提交複審 | **核心權限**：審批假卡判定，出具拒評書 | 終審歸檔與法律存證管理 |
| **任務分配與調度派工** | 僅能領取已指派工單 | **核心權限**：批量指派工單給特定評級師 | 具備最高排班與指派權限 |
| **外殼封磚質檢 (QC)** | 查看封裝狀態 | **核心權限**：掃碼質檢，點選通過/返工 | 封磚耗材統計與廢品核銷 |
| **報告合成放行至 OMS** | 提交個人簽署分值 | **核心權限**：終審批准，發送完工事件 | 強制推送/重新生成報告 |
| **帳號權限與審計日誌** | 無 | 查看班組工作績效報表 | 賬號增刪、權限指派、核心日誌導出 |

---

## 3. 雙語語言規格 (Bilingual i18n Specification)

GMS 系統原生支援即時切換 `English (en)` 與 `繁體中文 (zh-TW)`：

| 介面項目 / 評級術語 | English (en) | 繁體中文 (zh-TW) | 備註說明 |
| :--- | :--- | :--- | :--- |
| **工位/狀態** | Inbound & Tagging | 拆包核驗與打標 | 工位一 |
| | Camera Vision Scan | 定制相機光學掃描 | 工位二 (硬件聯調核心) |
| | OCR Catalog Match | OCR 卡庫特徵比對 | 工位三 |
| | AI & Expert Grading | AI 初檢與專家評級 | 工位四 |
| | Slab Encapsulation | 物理封磚與貼標 | 工位五 |
| | QC & Final Handover | 終檢放行與訂單合流 | 工位六 |
| **真偽結論** | AUTHENTIC | 正品認證通過 | 符合各道防線標準 |
| | COUNTERFEIT | 判定為贗品/假卡 | 一票否決，不予封裝 |
| | ALTERED | 人工改造/篡改卡 | 修邊(Trimming)、上色、壓痕修復 |
| **四大品相小分** | Centering | 居中度 | 計算左右 (L/R)、上下 (T/B) 比例 |
| | Corners | 四角完整度 | 磨損、白邊、圓鈍檢測 |
| | Edges | 邊緣平整度 | 切邊毛刺、邊緣掉漆檢測 |
| | Surface | 表面品相 | 划痕、印刷線、凹陷、指紋檢測 |
| **品相評級名稱** | Gem Mint (10.0) | 寶石級完美品相 | 最高評級榮譽 |
| | Mint (9.0 ~ 9.5) | 近完美薄荷品相 | 微觀極輕微瑕疵 |
| | Near Mint-Mint (8.0~8.5) | 近薄荷品相 | 輕微白邊或居中偏差 |
| | Excellent (5.0 ~ 7.5) | 優良收藏品相 | 存在肉眼可見輕度瑕疵 |
| | Poor / Fair (1.0 ~ 4.5) | 磨損破損品相 | 重度折痕、污損、水漬 |

---

## 4. 評級三大支柱核心規格（對齊規范手冊）

### 4.1 第一支柱：公允市場估值（Pricing & Market Valuation）
* 評級完成後，系統呼叫 `shared/pricing_engine`：
  $$\text{FMV}(\text{Card}, \text{Grade}) = \text{BaseRawPrice} \times \text{Multiplier}(\text{Grade}, \text{Scarcity})$$
* **價值懸崖提示**：若為古董卡（如 2000 年前寶可夢初版），系統自動標註「9分與10分存在巨大價值階梯」，提示評級師加倍審慎。
* **超額保價警報**：若計算出的 FMV 超出訂單所屬方案上限，自動生成 Upcharge 預警數據回傳給 OMS。

---

### 4.2 第二支柱：真偽鑑定體系（Authentication & Anti-Counterfeiting）
真偽鑑定採用「一票否決制」，由 AI 初檢提取特徵，評級師人工複核，必須通過以下四道防線：

```
[原始卡牌入機]
      │
      ▼
【第一道防線：物理規格】 ──(尺寸公差 ±0.15mm, 厚度 0.28~0.32mm, 重量公差 ±0.05g)──> 異常則報警
      │
      ▼
【第二道防線：夾層黑芯】 ──(透光反應 + 截面光學檢測：BLACK_CORE 黑芯層)────────> 白芯/透光不均直接判定假卡
      │
      ▼
【第三道防線：印刷網點】 ──(高倍微距比對膠印 Rosette Pattern 玫瑰網點)───────────> 噴墨/數碼打印顆粒直接判定假卡
      │
      ▼
【第四道防線：微縮文字】 ──(高精度檢測版權字輪廓與特殊光學油墨)──────────────────> 筆畫糊連判定篡改
```

* **判定為假卡/篡改卡時的處理**：
  1. GMS 立即終止後續評級打分，狀態標記為 `REJECTED_FAKE`。
  2. 生成《不予評級存證書》（標註微觀對比不合格圖例與專家簽名）。
  3. 扣留封裝外殼，卡片保留原始卡套，隨訂單一同退回。

---

### 4.3 第三支柱：品相定級與量化打分（Grading Standards & Rules）

#### 4.3.1 四大維度權重加權公式
$$\text{Grade}_{\text{Raw}} = 0.35 \times \text{Centering} + 0.25 \times \text{Surface} + 0.20 \times \text{Corners} + 0.20 \times \text{Edges}$$

#### 4.3.2 居中度（Centering）比例容差表
* **正面 (Front Ratio)**：
  * Gem Mint 10: 允許在 $50/50 \sim 55/45$ 範圍內。
  * Mint 9: 允許在 $60/40$ 範圍內。
  * NM 8: 允許在 $65/35$ 範圍內。
* **背面 (Back Ratio)**：
  * Gem Mint 10: 允許在 $60/40 \sim 75/25$ 範圍內。

#### 4.3.3 硬性約束與封頂規則（Hard Override Constraints）
為保證公信力，打分設有一票封頂機制：
1. **折痕封頂 (Crease Cap)**：卡面若檢測到微小表面折痕，總分**強制封頂最高 6.0 分**；貫穿性深折痕**強制封頂 4.0 分**。
2. **缺肉/孔洞封頂 (Paper Loss / Pinholes)**：強制封頂 **1.5 分**。
3. **單項最差拖累規則 (Worst-Subgrade Penalty)**：最終總分不得高於「四項子分中最低分 $+ 1.0$ 分」（例如 Corners 得 7.0 分，總分最高不得超過 8.0 分）。

---

## 5. 核心工位詳細規格與異常兜底

### 5.1 定制相機機檢工位規格（硬件聯調專項）
* **5.1.1 硬件通信架構**：
  * GMS 前端透過本地代理服務（WebSocket / Local HTTP `127.0.0.1:8989`）與定制相機主板驅動通信。
  * 實時反饋狀態：頂部狀態欄綠燈展示 `Hardware Online (Active 120 FPS / 4K UHD)`。
* **5.1.2 採圖流水線規格**：
  * 一鍵觸發步進電機傳動進卡，依次切換光源採集：
    1. 白光漫反射（正反面 4K 高保真無反光原圖）；
    2. 45° 側射光（表面微觀壓痕、指紋、划痕立體浮雕圖）；
    3. UV 紫外線光（全息防偽層、熒光劑殘留、人工補色辨識）；
    4. 背光透射光（黑芯透光度測量）。
* **5.1.3 硬件聯調期「手動翻拍兜底機制」**：
  * 若傳動卡機、相機連線中斷或遇到超厚特殊卡磚，系統在 3 秒內提示「切換手動翻拍」。
  * 操作員在翻拍台手動拍照上傳，GMS 調用軟體級算法自動進行透視變形校正與邊緣裁剪，確保業務不中斷。

---

### 5.2 OCR 特徵識別與 CMS 卡庫比對規格
* **5.2.1 識別字段**：
  * 提取卡牌名稱、卡號（如 `OP01-001`、`025/165`）、版權年份、語言（日文/英文/中文）。
* **5.2.2 比對與人工校驗**：
  * 比對置信度 $\ge 95\%$：自動鎖定 CMS 標準卡，調出正品標準數據。
  * 比對置信度 $< 95\%$ 或查無此卡：工作台右側彈出「候選卡庫匹配區」，評級師支援手動快速檢索選擇，或點選「一鍵登記未收錄新卡」。
* **5.2.3 客戶端前置申報清單自動對齊機制 (Pre-declared Intake Alignment)**：
  * **元數據繼承**：客戶在 Web 端（透過卡庫搜尋與行內「一鍵複製」）或 App 端（掃描建冊同步）提交時，訂單中每張卡已綁定預報卡名、卡號及申報貨值。
  * **流水線自動位次咬合**：定制相機 OCR 辨識出的卡號第一時間與訂單預報清單進行加權比對。若同一訂單內包含多張相同卡牌（用戶使用 Web 端複製功能批量送評），GMS 自動依序關聯卡位並分別生成獨立評級任務流水號（Item Sequence ID），評級師無需重新鍵入任何卡牌元數據，直接進入光學防偽與瑕疵標註。

---

### 5.3 物理外殼封裝（Slabing）與防偽貼標規格
* **5.3.1 專屬標籤數據注入**：
  * 終審批准後，系統生成唯一證書號碼（`TGC-2026-XXXXX`）。
  * 驅動工業級熱轉印/激光打標機，列印雙語專屬標籤（正面：卡名、卡號、總分、防偽二維碼；背面：四項子項小分及認證防偽徽章）。
* **5.3.2 封磚外觀質檢 (QC)**：
  * 超聲波封合完畢後，Ops Manager 進行外觀質檢：
    * 檢查外殼超聲波熔接線是否均勻；
    * 檢查殼內是否有肉眼可見粉塵、指紋；
    * 質檢合格點選 `[QC Approved]`；有瑕疵點選 `[Slab Defect / 返工拆殼]`。

---

### 5.4 單卡獨立報告合成與 OMS 訂單合流
* **5.4.1 單卡報告生成**：
  * 每一張通過評級的卡牌，GMS 渲染引擎即時輸出單卡高清 `DIF Digital Certificate`。
  * 包含：正反面高解析度大圖、微距網點對比存證、四項子分比例、防偽溯源鏈。
* **5.4.2 訂單完工判定**：
  * GMS 定時輪詢該訂單下的所有 Mission ID。
  * **合流條件**：當訂單內全部單卡均達到 `PACKED` 或 `REJECTED_FAKE` 終態時，GMS 觸發 `GMS_ORDER_COMPLETED` Webhook，將整包數據推給 OMS，正式進入回寄物流包裝工序。

---

## 6. 資料庫實體規格 (對齊現有 `gms/system/schema.sql`)

### 6.1 評級任務工單表 (`grading_missions`)
```sql
CREATE TABLE IF NOT EXISTS grading_missions (
    id TEXT PRIMARY KEY,                       -- 工單號，如 'MSN-2026-0001'
    order_id TEXT NOT NULL,                    -- 關聯 OMS 訂單號
    card_id TEXT NOT NULL,                     -- 關聯 CMS 卡庫 ID
    user_id TEXT NOT NULL,                     -- 送評用戶 ID
    declared_value REAL,                       -- 申報價值 (USD)
    status TEXT NOT NULL,                      -- 'RECEIVED', 'SCANNED', 'AI_ANALYZED', 'HUMAN_REVIEW', 'APPROVED', 'PACKED', 'REJECTED_FAKE'
    ai_confidence REAL DEFAULT 0.0,            -- AI 綜合置信度
    assigned_grader TEXT,                      -- 分配的評級師 ID
    reviewed_by_manager TEXT,                  -- 審批的經理 ID
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 6.2 真偽鑑定明細表 (`card_authentications`)
```sql
CREATE TABLE IF NOT EXISTS card_authentications (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    mission_id TEXT NOT NULL,
    verdict TEXT NOT NULL,                     -- 'AUTHENTIC', 'COUNTERFEIT', 'ALTERED'
    physical_spec_pass INTEGER DEFAULT 1,      -- 尺寸克重公差 (1/0)
    core_layer_detected TEXT,                  -- 'BLACK_CORE', 'BLUE_CORE', 'WHITE_CORE'
    rosette_pattern_pass INTEGER DEFAULT 1,    -- 玫瑰印刷網點 (1/0)
    micro_text_pass INTEGER DEFAULT 1,         -- 微縮文字印製 (1/0)
    specimen_match_score REAL DEFAULT 0.0,     -- 官方標本相似度 (0~100%)
    audit_notes TEXT,                          -- 專家備註說明 (雙語)
    inspected_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(mission_id) REFERENCES grading_missions(id)
);
```

### 6.3 最終評級結果與卡磚數據表 (`grading_results`)
```sql
CREATE TABLE IF NOT EXISTS grading_results (
    cert_number TEXT PRIMARY KEY,              -- 證書號，如 'TGC-2026-88912'
    mission_id TEXT NOT NULL UNIQUE,
    final_grade REAL NOT NULL,                 -- 最終總分 (1.0~10.0)
    grade_name TEXT NOT NULL,                  -- 'Gem Mint', 'Mint', etc.
    subgrade_centering REAL NOT NULL,          -- 居中度小分
    subgrade_corners REAL NOT NULL,            -- 四角小分
    subgrade_edges REAL NOT NULL,              -- 邊緣小分
    subgrade_surface REAL NOT NULL,            -- 表面小分
    front_ratio_lr TEXT,                       -- 正面左右比例 '52/48'
    front_ratio_tb TEXT,                       -- 正面上Deprecated下比例 '51/49'
    back_ratio_lr TEXT,                        -- 背面左右比例 '58/42'
    back_ratio_tb TEXT,                        -- 背面上下比例 '60/40'
    override_reason TEXT,                      -- 觸發封頂說明 (折痕/水漬)
    ai_pre_grade REAL,                         -- AI 初評分
    human_adjust_reason TEXT,                  -- 人工調分理由標記
    graded_by TEXT NOT NULL,                   -- 簽署評級師 (Grader)
    approved_by TEXT NOT NULL,                 -- 終審經理 (Ops Manager)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(mission_id) REFERENCES grading_missions(id)
);
```
