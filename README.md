# 舊版圖片名稱適配 — Legacy Art Mods Compat (Plus)

為新版 **Degrees of Lewdity** 提供舊版美化資源兼容。

本模組會在新版 Renderer 請求圖片資源時，根據目前使用的 Modern path 動態生成可能存在的 Legacy Candidates；若原始新版資源無法命中，則依序嘗試對應的舊版圖片路徑。

> **兼容方案來源**
>
> 本版本的 **Request-time ImageLoader fallback** 核心延續自原 **Legacy-Art-Mods-Compat**。
>
> 新舊版本間的圖片路徑差異與命名轉換規則，主要基於 **DoL-Genesis-LTS 內建 Genesis Compat** 所整理的 Legacy → Modern 規則。
>
> 本模組將相關規則轉換並重新整理為適合 Request-time fallback 使用的 **Modern → Legacy Candidate Generator**，以規則式候選生成取代早期大型 `SPECIAL_RENAMES / PATTERN_RENAMES` 映射表。

---

## 功能

### 舊版資源自動回退

當 Renderer 請求圖片時，本模組會根據目前的 Modern path 判斷是否存在已知的舊版命名形式。

只有原始 Modern 資源無法命中時，才會依序嘗試 Legacy Candidates：

```text
Renderer 請求 Modern path
        │
        ▼
是否存在兼容候選？
        │
        ├── NO ─────► 不介入 ImageLoader
        │
        └── YES
             │
             ▼
      Before-native 規則
             │
             ├── HIT ─────► 使用特殊兼容資源
             │
             └── MISS / 無規則
                      │
                      ▼
              嘗試原 Modern path
                      │
                      ├── HIT ─────► 使用原圖
                      │
                      └── MISS
                           │
                           ▼
                After-native 特殊候選
                           │
                           ├── HIT ─────► 使用兼容資源
                           │
                           └── MISS / 無規則
                                    │
                                    ▼
                         Legacy Candidates
                                    │
                                    ├── HIT ─────► 使用舊圖
                                    │
                                    └── 全部 MISS
                                             │
                                             ▼
                                         交回原流程
```

### 特殊兼容規則

除了普通的 Modern → Legacy fallback 外，本模組亦保留少量需要特殊處理的兼容規則。

這些規則不一定遵循「Modern MISS 後才回退」的普通流程，而是依照各自需求決定優先級。

目前包括：

- **牛轉化耳朵 `spotted-*` 舊美化兼容**
  - 舊美化資源可能使用空格命名
  - 需要在官方原圖之前優先嘗試，避免直接命中原版資源

- **AU 面擴 `blush / tear(s)` 多版本命名兼容**
  - 提供不同歷史命名之間的雙向候選
  - 在原始圖片無法命中時再進行 fallback

特殊規則與官方歷史命名 Candidate Generator 相互獨立，方便後續繼續擴充，而不影響普通兼容流程。

---

## 與 Genesis Compat 的關係

本模組可以作為 **DoL-Genesis-LTS 內建 Genesis Compat 的補充兼容方案**。

Genesis Compat 主要在資源載入階段處理已知的舊版 Mod；本模組則保留原 Legacy-Art-Mods-Compat 的 Request-time fallback 方式，在實際圖片請求階段提供另一層兼容。

因此，本模組特別適合補充處理：

- 未被既有兼容機制涵蓋的舊版美化資源
- 新舊圖片命名混用的美化包
- 部分第三方美化使用的特殊歷史命名
- 需要特殊資源優先級的兼容情況

兩者並非互相替代，而是可以同時使用。

若 Genesis Compat 已經成功處理相關資源，正常情況下 Renderer 會直接取得可用圖片；本模組的普通 fallback 不需要介入。

---

## Plus 版本新增

相較 Legacy-Art-Mods-Compat，Plus 版本主要增加／調整：

- **Legacy Candidate Generator**
- 使用規則式候選生成取代大型硬編碼映射表
- 支援一個 Modern path 對應多個 Legacy Candidates
- 支援新舊資源混搭
- 支援可獨立設定優先級的特殊兼容規則
- AU 面擴多版本 `blush / tear(s)` 雙向候選
- 牛轉化耳朵耳舊美化資源優先兼容
- 一般兼容規則下 Modern 資源優先

---

## Debug

Debug 模式：

```js
setup.LegacyArtCompatDebugMode = "all";         <=預設
setup.LegacyArtCompatDebugMode = "fail-only";
setup.LegacyArtCompatDebugMode = "hit-only";
setup.LegacyArtCompatDebugMode = "off";
```

查看累計摘要：

```js
LegacyArtCompatDebugSummary();
```

查看某條路徑會生成哪些 Legacy Candidates：

```js
LegacyArtCompatGenerateCandidates(
    "img/clothes/handheld/cloud purse/right-hold-cover-light.png"
);
```

查看候選及規則來源：

```js
LegacyArtCompatGenerateCandidateDetails(
    "img/clothes/legs/plainthighhighs/back-acc.png"
);
```

查看某條路徑會命中哪些特殊兼容規則：

```js
LegacyArtCompatGenerateSpecialCandidateDetails(
    "img/transformations/cow/ears/spotted-black.png"
);
```

清除 Debug 累計：

```js
LegacyArtCompatClearDebugHistory();
```

---

## 版本差異

### Plus V1.x

- 延續原版 **Request-time ImageLoader fallback**
- 擴充新版／舊版圖片路徑兼容範圍
- 補強部分舊版手持物圖片命名兼容
- 加入 **Legacy 服裝灰階資源優先載入**，生成舊版服裝候選路徑時優先嘗試 `_gray.png`，不存在時再嘗試普通 `.png`
- 加入部分第三方美化資源兼容
- 新增可擴展的 **通配符路徑兼容規則**
  - 支援 `*` 通配符及 `{n}` 數字占位符
  - 支援 **Fallback** 與 **Override** 兩種模式
  - 支援單一規則對應多個候選路徑
- 官方新舊版本路徑差異仍主要以明確的硬編碼映射處理
- 說明
    * V1.x 主要是在原版 Legacy-Art-Mods-Compat 的架構上持續擴充兼容範圍。
    * 其中新舊版本間的圖片名稱差異仍以明確映射為主，而另外加入的通配符規則則作為可擴展的特殊兼容機制，用於處理第三方美化及無法單純依靠固定映射處理的情況。

### ~~Plus V2.x~~（廢棄）

- ~~嘗試改用 **Load-time Normalization**~~
- ~~主要基於 **DoL-Genesis-LTS / Genesis Compat** 的資源轉換方案~~
- ~~引入 **Asset-level Legacy Detection**~~
- ~~可直接根據實際圖片資源判斷 Legacy 命名，不再完全依賴 Mod 的 `compat` 聲明~~
- ~~在圖片索引載入階段提前完成 Legacy → Modern 映射~~
- ~~增加衝突、防重複轉換及映射歷史等機制~~
- 廢棄說明
    * 此方案本身可以正常工作，但實際上與 **Genesis Compat** 的功能及實作方式高度重疊。
    * 若本模組同樣採用 Load-time Normalization，除了增加「無 `compat` 聲明自動識別」等少量功能外，整體功能與 Genesis Compat 高度重疊，實際上近似於另一套擴展版 Genesis Compat。
    * 對已使用 **DoL-Genesis-LTS** 的使用者而言，繼續使用原生 Genesis Compat 反而更加直接，也使本模組失去作為獨立兼容工具的定位。
    * 因此重新調整模組方向，廢棄 V2.x 方案，後續版本回歸 **Legacy-Art-Mods-Compat 原有的 Request-time fallback**，將本模組定位為 **Genesis Compat 的補充兼容層，而非另一套 Genesis Compat**。

### Plus V3.x

- 回歸並延續 **Request-time ImageLoader fallback**
- 參考 **Genesis Compat**將官方新舊版本的大型硬編碼映射重構為 **Modern → Legacy Candidate Generator**
- 支援一個 Modern path 動態生成多個 Legacy Candidates
- 保留少量 `EXACT` 映射處理無法可靠規則化的特殊路徑
- 保留 V1.x 的 **Legacy 服裝灰階資源優先載入**
- 將 V1.x 的通配符兼容機制重構為 **Extra / Third-party Compatibility Rules**
  - 改由 `match()` 判斷規則是否適用
  - 改由 `candidates()` 動態生成候選路徑
  - 使用 `before-native / after-native` 控制候選優先級
  - 可處理普通 fallback 以外的特殊兼容需求
- 保留並擴充第三方美化兼容
  - AU 面擴 `blush / tear(s)` 多版本命名
  - 牛耳 `spotted-*` 舊美化優先載入

簡單來說：

```text
Plus 1
硬編碼 Request-time fallback
        ↓
Plus 2 (廢棄)
Genesis Compat 式 Load-time Normalization
        ↓
Plus 3
規則化 Request-time Candidate Generator
+ 少量硬編碼例外
+ 獨立特殊兼容規則
```
---

## 致謝

特別感謝以下項目與作者對 DoL 圖片兼容工作的貢獻。

### Legacy-Art-Mods-Compat（舊版圖片名稱適配）

感謝原作者 **鎖鏈蝴蝶＠百度貼吧** 建立最初的 **Legacy-Art-Mods-Compat**，為本項目提供了最早的圖片兼容基礎。

**項目來源：**  
https://github.com/mirrormirroronwall/Legacy-Art-Mods-Compat

本 Plus 版本基於原項目繼續擴充，並延續其 **Request-time ImageLoader fallback** 的核心兼容方式。

### DoL-Genesis-LTS / Genesis Compat

感謝 **DoL-Genesis-LTS** 及其內建 **Genesis Compat** 的作者與貢獻者。

本模組使用 **Genesis Compat** 所整理的 Legacy → Modern 圖片路徑轉換規則，並將相關規則重新整理及轉換為適用於 Request-time fallback 的 **Modern → Legacy Candidate Generator**。

因此，Genesis Compat 提供的版本間圖片路徑差異與轉換規則，是目前 Plus V3 官方新舊資源兼容規則的主要來源。

**項目來源：**  
https://github.com/anlimi555s/DoL-Genesis-LTS

---

感謝所有曾參與 DoL 美化、圖片資源製作、ModLoader、生態工具及兼容工作的作者與貢獻者。

也希望這個兼容模組能讓更多舊版美化資源繼續在新版遊戲中正常使用。