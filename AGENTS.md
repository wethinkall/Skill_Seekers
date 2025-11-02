# AGENTS.md - 錯誤分析與解決方案

此文件記錄本系統在執行 Skill 生成任務時所遇到的錯誤、原因分析與解決方案。

---

## 📋 Skill 打包 (Packaging) 錯誤總結

### ❌ 錯誤 1: YAML Frontmatter 缺失

**症狀：**
```
Upload Error: SKILL.md must start with YAML frontmatter (---)
```

**根本原因：**
- 在生成 SKILL.md 時，未在文件開頭添加 YAML frontmatter
- Skill 上傳系統要求每個 SKILL.md 必須以 `---` 開始
- YAML frontmatter 是 Skill 元數據的必需部分

**錯誤代碼示例：**
```markdown
# Kendo UI for Angular Skill    ❌ 直接從標題開始

## 🎯 When to Use This Skill
...
```

**正確格式：**
```markdown
---
name: kendo-ui-for-angular
description: Professional UI component library for building enterprise Angular applications...
---

# Kendo UI for Angular Skill    ✅ 標題在 frontmatter 之後

## 🎯 When to Use This Skill
...
```

**解決方案：**
1. 確保生成 SKILL.md 時在文件最開頭添加 YAML frontmatter
2. Frontmatter 結構必須是：
   - 第 1 行: `---`
   - 第 2 行: `name: <skill-name>`
   - 第 3 行: `description: <description>`
   - 第 4 行: `---`
   - 第 5 行開始: 正常 Markdown 內容

**涉及文件：**
- `output/kendo-angular-ui/SKILL.md` (原始錯誤)
- `output/mermaid/SKILL.md` (已修正)
- `output/mcp-redmine/SKILL.md` (已正確)

---

### ❌ 錯誤 2: Skill 名稱格式不符

**症狀：**
```
Upload Error: Skill name in SKILL.md can only contain lowercase letters, numbers, and hyphens.
Try 'kendo-ui-for-angular' instead of 'Kendo UI for Angular'
```

**根本原因：**
- Skill 命名系統有嚴格的格式要求
- 不允許：大寫字母、空格、特殊符號 (除了連字符)
- 只允許：小寫字母、數字、連字符 (`a-z`, `0-9`, `-`)

**錯誤名稱示例：**
```yaml
---
name: Kendo UI for Angular              ❌ 包含大寫和空格
name: "Kendo UI for Angular"            ❌ 引號無法幫助，仍是錯誤格式
name: kendo_ui_for_angular              ❌ 下劃線不被允許
name: kendoUIforAngular                 ❌ 包含大寫字母
---
```

**正確名稱格式：**
```yaml
---
name: kendo-ui-for-angular              ✅ 全小寫 + 連字符
name: mermaid-diagrams                  ✅ 全小寫 + 連字符
name: mcp-redmine                       ✅ 全小寫 + 連字符
name: react                             ✅ 簡單小寫
name: vue-3-framework                   ✅ 小寫 + 數字 + 連字符
---
```

**解決方案：**
1. Skill 名稱必須全部使用小寫字母
2. 單詞之間用連字符 (`-`) 分隔，不用空格或下劃線
3. 驗證方式：名稱應匹配正規表達式 `^[a-z0-9-]+$`

**涉及文件：**
- `output/kendo-angular-ui/SKILL.md` (原始錯誤: "Kendo UI for Angular" → 修正為 "kendo-ui-for-angular")

---

### ⚠️ 錯誤 3: Web Scraper 未能完整抓取 (Partial Crawl)

**症狀：**
```
Mermaid 文檔爬取結果：僅獲得 1 頁
預期：50+ 頁的完整文檔
```

**根本原因：**
- Mermaid 文檔使用 **VitePress** (基於 Vue + Vite 的靜態站點生成器)
- VitePress 大量使用 JavaScript 動態渲染和 SPA (Single Page Application) 路由
- 標準 HTTP 爬蟲 (requests + BeautifulSoup) 無法執行 JavaScript
- 爬蟲只能獲取初始 HTML，而 VitePress 的完整內容在運行時才被渲染

**技術分析：**
```
VitePress 架構:
┌─────────────────────────────────────┐
│ HTTP 請求                           │
│ ├─ 接收 index.html (基礎框架)        │
│ ├─ 內容為空 (動態渲染)               │
│ └─ JavaScript bundle                │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ 瀏覽器/JS Runtime                   │
│ ├─ 執行 Vue 組件                    │
│ ├─ 設置路由                         │
│ └─ 渲染實際內容                     │
└─────────────────────────────────────┘
```

**解決方案：**
1. **對於 JavaScript 重型網站：** 使用 Selenium、Puppeteer 或 Playwright 而非標準爬蟲
2. **替代方案 (已採用)：** 手動撰寫高品質文檔
   - 基於官方 API 文件編寫
   - 包含所有常用功能
   - 添加實際代碼示例
   - 提供最佳實踐

**涉及文件：**
- `configs/mermaid.json` (爬蟲配置)
- `output/mermaid/SKILL.md` (手動編寫的替代方案)

---

## 🔍 Skill 生成流程中的關鍵檢查點

生成 Skill 時應檢查以下項目：

### 1️⃣ YAML Frontmatter 驗證
```
☑️ 第一行是否為 `---`
☑️ 第二行是否為 `name: <lowercase-name>`
☑️ 第三行是否為 `description: <string>`
☑️ 第四行是否為 `---`
☑️ name 是否只包含小寫字母、數字、連字符
```

### 2️⃣ Skill 名稱驗證
```
☑️ 正規表達式: ^[a-z0-9-]+$
☑️ 無大寫字母
☑️ 無空格 (用連字符代替)
☑️ 無特殊符號 (除連字符外)
☑️ 無下劃線
```

### 3️⃣ 目錄結構驗證
```
output/{skill-name}/
├── SKILL.md                 ✅ 必需 (含 frontmatter)
├── references/
│   ├── index.md            ✅ 推薦
│   └── *.md                ✅ 分類文檔
├── scripts/                ✅ 可選 (用於用戶腳本)
└── assets/                 ✅ 可選 (用於圖片/資源)
```

### 4️⃣ 內容質量檢查
```
☑️ 代碼範例有正確的語言標記
☑️ 表格格式正確
☑️ 連結有效
☑️ 無拼寫錯誤
```

---

## 📊 錯誤分布與修復狀態

| 技能 | 錯誤類型 | 狀態 | 修復方法 |
|------|--------|------|--------|
| MCP-Redmine | 無 | ✅ 成功 | N/A |
| Kendo UI Angular | Frontmatter 缺失 + 名稱格式 | ✅ 已修復 | 手動添加 frontmatter + 改名為 kendo-ui-for-angular |
| Mermaid | 爬蟲不完整 | ✅ 已修復 | 手動編寫完整文檔 |

---

## 🎓 經驗教訓

### 1. Frontmatter 是強制要求
- 必須在文件最開頭
- 不能遺漏或延遲
- 格式必須精確

### 2. 名稱驗證應自動化
- 建議在打包前驗證名稱格式
- 可用正規表達式檢查：`^[a-z0-9-]+$`
- 提前捕捉錯誤，避免上傳失敗

### 3. JavaScript 重型網站需特殊處理
- 檢測 VitePress/Next.js/Nuxt 等框架
- 使用無頭瀏覽器 (Puppeteer/Playwright)
- 或採用手動文檔編寫方式

### 4. 測試流程應包括上傳驗證
- 不僅檢查本地文件結構
- 應針對上傳系統的驗證規則進行檢查
- 建立驗證清單

---

## 📝 建議改進

### 短期改進 (立即可實施)
1. 在 `package_skill.py` 中添加 YAML frontmatter 驗證
2. 在 `package_skill.py` 中添加名稱格式驗證 (正規表達式)
3. 生成打包前驗證報告

### 長期改進 (未來優化)
1. 偵測文檔網站使用的框架 (VitePress/Next.js/等)
2. 自動選擇適當的爬蟲策略
3. 集成 Puppeteer/Playwright 支持動態渲染網站
4. 建立自動化測試，模擬上傳驗證流程

---

**上次更新：** 2025-10-30
**文件負責人：** Claude Code
**相關文件：** CLAUDE.md (應參考本文件了解錯誤詳情)
