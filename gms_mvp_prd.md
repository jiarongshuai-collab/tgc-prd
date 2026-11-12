# TGC GMS（評級實驗室管理系統）MVP 版本產品需求文件 (PRD)

> **所屬體系**：TGC (The Good Cherry / 櫻桃未來) 智能質檢評級流水線  
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
| **機台故障調度與掃描重測 (H05/H06)** | 標記機台異常並轉移至備用掃描儀 | 審批調度任務與重測工單；**嚴禁手動翻拍評級** | 機台校準管理、鎖定異常機台與全量卡牌強制重掃 (H06) |
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
【第一道防線：物理規格】 ──(尺寸出廠公差 ±0.20mm, 厚度 0.28~0.32mm, 重量公差 ±0.10g)──> 短縮 ≥0.30mm 警報
      │
      ▼
【第二道防線：夾層黑芯】 ──(透光反應 + 截面光學檢測：BLACK_CORE 黑芯層)────────> 白芯/透光不均判定假卡 (COUNTERFEIT)
      │
      ▼
【第三道防線：印刷網點】 ──(高倍微距比對膠印 Rosette Pattern 玫瑰網點)───────────> 噴墨/數碼打印顆粒判定假卡 (COUNTERFEIT)
      │
      ▼
【第四道防線：微縮文字】 ──(高精度檢測版權字輪廓與特殊光學油墨)──────────────────> 塗改/拼裝/重背判定改卡 (ALTERED)
```

* **判定為假卡、改卡與不可評級卡牌的分流處置機制（嚴格對齊 Heidi 異常規範 E03 / E04 / A10 / E05 / E12 / E13 與三大支柱標準）**：
  1. **假卡/偽造卡 (`COUNTERFEIT` / E03) 處置**：
     * GMS 立即終止後續評級打分，狀態標記為 `REJECTED_FAKE`。
     * 出具《未評級，真偽存疑（Did Not Grade, Questionable Authenticity）》標籤與防偽攔截記錄（標註微觀對比不合格圖例與專家簽名存證）。
     * **不進行封裝 (Not slabbed)**。
     * **收費與退款規則**：**收取全額評級費，不予退款**（E03 規定），不添加至用戶收藏夾（Portfolio）。卡片退回原包裝卡套，隨訂單其他正常卡牌一同回寄。
  2. **人為改裝/篡改卡 (`ALTERED` / E04 / A10) 處置**：
     * 卡牌確認為原廠正品基底，但遭受後天人為物理/化學改動（裁剪修邊 Trimming、補色描邊 Inking、壓平去折 Pressing、重噴光油 Resurfacing），歸類為 N-1、N-2 或 N-5。
     * 狀態標記為 `AUTHENTIC_ALTERED`，**系統凍結數值評分（No Grade 1-10，不給予 1~10 數字分）**。
     * **封裝處置**：**嚴格依照用戶在下單「服務」步驟（A10）設定的偏好 `altered_action_preference` 執行**：
       - 若偏好為 `SLAB_AA`（封裝專屬卡磚）：封裝為專屬認證卡磚，標籤專屬標註 **`Authentic Altered (AA)`**。
       - 若偏好為 `RETURN_RAW`（默認選項）：不予封裝，直接作為裸卡退回。
     * **收費規則**：**費用不予退還**（E04 規定），不添加至用戶收藏夾。
  3. **無法評級或嚴重翹曲卡牌 (E05 / E12 / E13) 處置**：
     * **E05（卡牌過厚、已預先封裝、未拆封或卡夾不合規）**：作為裸卡退回，不收取評級費。若全單無卡可評，發送退回通知單收取退貨運費與 SPP。
     * **E12（無法核實真偽）**：真偽鑑定結果不明確，不進行評級，隨訂單作為裸卡退回，不添加至收藏夾，收取全額評級費（同 E03）。
     * **E13（嚴重彎曲或翹曲的卡牌）**：評級師無法拍攝出足夠清晰的照片以進行評級，不進行評級，**TGC 絕不對卡牌進行壓平或按壓處理**，隨訂單作為裸卡退回，不收取評級費（同 E05）。

---

### 4.3 第三支柱：品相定級與量化打分（Grading Standards & Rules）

#### 4.3.1 四大維度權重加權公式
$$\text{Grade}_{\text{Raw}} = 0.35 \times \text{Centering} + 0.25 \times \text{Surface} + 0.20 \times \text{Corners} + 0.20 \times \text{Edges}$$

#### 4.3.2 居中度（Centering）比例容差表
* **正面 (Front Ratio)**：
  * Gem Mint 10: 允許在 $50/50 \sim 55/45$ 範圍內。
  * Mint+ 9.5: 允許在 $56/44 \sim 58/42$ 範圍內。
  * Mint 9: 允許在 $59/41 \sim 60/40$ 範圍內。
  * NM-MT 8: 允許在 $61/39 \sim 65/35$ 範圍內。
  * EX 6 及以下: 偏心大於 $75/25$ 觸發 `OC` (Off-Center) 修飾符。
* **背面 (Back Ratio)**：
  * Gem Mint 10: 允許在 $50/50 \sim 75/25$ 範圍內。
  * Mint 9: 允許在 $\le 80/20$ 範圍內。

#### 4.3.3 硬性約束與封頂規則（Hard Override Constraints）
為保證公信力，打分設有一票封頂機制：
1. **折痕封頂 (Crease Cap)**：卡面若檢測到微小表面折痕（Minor Crease $< 1\text{ inch}$），總分**強制封頂最高 5.0 (EX)**；貫穿性深折痕或多處折痕**強制封頂 4.0 (VG-EX)**。
2. **正面壓痕封頂 (Front Dent Override)**：卡片正面肉眼可見凹痕/壓痕，強制封頂 **7.0 (Near Mint)**。
3. **缺肉/孔洞封頂 (Paper Loss / Pinholes)**：強制封頂 **1.5 分**。
4. **單項最差拖累規則 (Worst-Subgrade Penalty)**：最終總分不得高於「四項子分中最低分 $+ 1.0$ 分」（例如 Corners 得 7.0 分，總分最高不得超過 8.0 分）。

---

## 5. 核心工位詳細規格與異常兜底

### 5.1 定制相機機檢工位規格（硬件聯調專項）
* **5.1.1 硬件通信架構**：
  * GMS 前端透過本地代理服務（WebSocket / Local HTTP `127.0.0.1:8989`）與定制相機主板驅動通信。
  * 實時反饋狀態：頂部狀態欄綠燈展示 `Hardware Online (Active 120 FPS / 4K UHD)`，畫布頂部顯示通道狀態 `微距雙通道 · 4K RGB + 3D光度立體 (Dual Optical Channels · 4K RGB + 3D Stereo)`。
* **5.1.2 採圖流水線規格（雙面四圖 Dual-Side 4-Image 質檢體系）**：
  * 每張卡牌進機檢驗時，光學感測陣列與四向分時頻閃光源協同作業，嚴格輸出 **雙面共 4 張 4K 級無反光圖像**：
    1. **正面 4K 彩色掃描圖 (Front Color 4.0K RGB 48-bit)**：用於幾何居中率（Centering 50/50 比例計算）、印刷網點（Rosette pattern）、邊緣白邊（Edge/Corner Whitening）與原畫色彩飽和度核驗。
    2. **正面 3D 表面光度立體浮雕圖 (Front Surface 4.0K Photometric Stereo Gray)**：消除原畫顏色與閃卡反光干擾，以 50% 中性灰（#808080）為基準重建微觀三維凹凸拓撲，用於微米級表面劃痕（Hairline Scratches）、壓痕、指紋、燙金壓印深度及印刷凸凹線精確測量。
    3. **背面 4K 彩色掃描圖 (Back Color 4.0K RGB 48-bit)**：用於背面幾何居中、背面四角磨損與紙纖維白邊檢測。系統自動依據卡牌語言與版本（日版 vs 國際版）自動匹配正品真實背面（如日版 2001-Present 經典六色能量球宇宙星空背面 vs 國際版藍邊深渦背面）。
    4. **背面 3D 表面光度立體浮雕圖 (Back Surface 4.0K Photometric Stereo Gray)**：用於背面指甲掐痕、壓痕凹陷、摩擦微劃痕等物理瑕疵的立體分析。
* **5.1.3 GMS 工作台視圖模式與雙通道操作規格**：
  1. **雙聯並排比對 (Dual Split View - 1:1 機台實機界面還原)**：默認視圖模式，左側展示 `CH-A · 4K 高清彩色`，右側展示 `CH-B · 3D 光度立體浮雕`。評級師可在單一畫布上同步觀察色彩瑕疵與物理微觀凹凸，兩側畫面保持 100% 幾何比例同步縮放（Zoom）與平移。
  2. **彩色原圖模式 (Color Scan Mode)**：單幅大圖集中檢視卡面印刷精度、色彩漸變及外邊框幾何比例。
  3. **3D表面浮雕模式 (Surface Relief Mode)**：單幅大圖高對比呈現表面微觀凹凸，表面劃痕與折痕以亮金琥珀色標記框（Surface Highlight）高亮鎖定。
  4. **正/反面切換聯動 (Front/Back Switching)**：點擊「正面 / 背面」時，雙聯兩側同步切換對應面的彩色與浮雕圖，瑕疵層（Defect Layer）依據面別動態切換（正面攻擊欄/閃卡框瑕疵 vs 背面角落掉白/表面磨損瑕疵）。
* **5.1.4 機台故障調度與掃描校準異常規範（嚴格對齊 Heidi 規範 H05 / H06）**：
  * **H05 掃描儀故障與嚴禁翻拍打分**：若某台評級相機掃描儀故障停機或傳動中斷，操作員將卡牌轉移至另一台已校準的備用掃描儀繼續評級。**系統恪守 H05 紅線原則：絕不根據人工拍攝/手機翻拍的照片進行評級或真偽鑑定**，杜絕非標準光源與非校準視角的翻拍數據進入 AI 引擎。
  * **H06 掃描儀每日校準防線**：每日例行校準檢查若未通過，系統自動鎖定該機台，禁止接收新評級任務。自上一次合格校準以來該機台所掃描的所有卡牌任務，一律強制打回重掃，確保微觀 4K 與 3D 浮雕數據的高保真度與法律憑證效力。

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
  * **合流條件**：當訂單內全部單卡均達到 `PACKED` (標準封磚)、`PACKED_ALTERED` (AA 認證封磚)、`RETURN_RAW` (改卡偏好裸卡/無法評級裸卡退回) 或 `REJECTED_FAKE` (假卡標記退回) 終態時，GMS 觸發 `GMS_ORDER_COMPLETED` Webhook，將整包數據推給 OMS，推進至 `SLABING` / `READY_FOR_REVEAL`，正式進入在線查分與回寄發貨工序。

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
    status TEXT NOT NULL,                      -- 'RECEIVED', 'SCANNED', 'AI_ANALYZED', 'HUMAN_REVIEW', 'APPROVED', 'PACKED', 'PACKED_ALTERED', 'RETURN_RAW', 'REJECTED_FAKE'
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
    verdict TEXT NOT NULL,                     -- 'AUTHENTIC', 'COUNTERFEIT', 'ALTERED', 'QUESTIONABLE'
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
    final_grade REAL,                          -- 最終總分 (1.0~10.0；若為改卡 Authentic Altered 則為 NULL 或 0)
    grade_name TEXT NOT NULL,                  -- 'Gem Mint 10', 'Mint 9', 'Authentic Altered (AA)' 等
    subgrade_centering REAL,                   -- 居中度小分 (AA 磚可為 NULL 或實測參考)
    subgrade_corners REAL,                     -- 四角小分
    subgrade_edges REAL,                       -- 邊緣小分
    subgrade_surface REAL,                     -- 表面小分
    front_ratio_lr TEXT,                       -- 正面左右比例 '52/48'
    front_ratio_tb TEXT,                       -- 正面上下比例 '51/49'
    back_ratio_lr TEXT,                        -- 背面左右比例 '58/42'
    back_ratio_tb TEXT,                        -- 背面上下比例 '60/40'
    qualifiers TEXT,                           -- 修飾符，如 'OC', 'MK', 'None'
    override_reason TEXT,                      -- 觸發封頂說明 (折痕/水漬/AA標註)
    ai_pre_grade REAL,                         -- AI 初評分
    human_adjust_reason TEXT,                  -- 人工調分理由標記
    graded_by TEXT NOT NULL,                   -- 簽署評級師 (Grader)
    approved_by TEXT NOT NULL,                 -- 終審經理 (Ops Manager)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(mission_id) REFERENCES grading_missions(id)
);
```
);
```
