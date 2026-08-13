# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本資料夾是 **`ipa-skill` 的散布包與交付物範例**，不是散開的原始碼專案。此技能把問卷調查資料轉換成有科學依據的改善優先序，方法論為 **IPA（重要度—表現度矩陣）＋ Kano 品質屬性模型 ＋ 柏拉圖 80/20 分析**。

## 檔案性質

- **`ipa-skill.skill`** — 打包好的技能檔（在 Claude 設定 → Capabilities 上傳安裝）。**技能的實際原始碼在這個 bundle 內**（`SKILL.md`、`references/plan-doc-outline.md`、`scripts/build_dashboard.py`、`scripts/dashboard-src/`、`scripts/docx-helpers/`），要改程式需先解包 `.skill`。
- **`滿意度分析互動儀表板_進階版.html`** — 交付物範例二：單一 HTML 互動儀表板（下述）。**本資料夾只保留進階版，基礎版已於 2026-08-13 依使用者要求刪除**（skill 本身仍可依需求產出基礎版格式，只是這裡不再放範例檔）。
- **`產品服務滿意度問卷分析計畫書_v2.docx`** — 交付物範例一：Word 計畫書。
- **`ipa-skill-eval-review.html`** — 技能評測／審閱報告。

## 兩種交付物

1. **計畫書（Word）** — 含方法論與執行計畫（IPA 四象限、Kano 五屬性對照、IPA×Kano 整合優先序、柏拉圖、時程里程碑、名詞白話對照）。下載格式為相容 Word 的 HTML 內嵌 `.doc`，非原生 `.docx`。
2. **互動式儀表板（單一 HTML）** — 七個分頁：資料匯入 / IPA 矩陣 / Kano 分析 / 整合優先序 / 柏拉圖 / 補充意見 / 匯出報告。**全部運算在瀏覽器本機執行，資料不上傳**。自動判斷三種輸入格式：IPA 寬表格、Kano 專用寬表格、長表格；判斷失敗時提供欄位對應介面。可重複上傳新批資料重算（適合定期追蹤）。

## 資料匯入分頁：清空資料與手動標註資料類型（僅「進階版」，2026-08-13）

`滿意度分析互動儀表板_進階版.html` 的「① 資料匯入」分頁，「已加入的資料來源」卡片新增：

- **🗑 清空資料**（卡片標題旁）：`clearAllSources()`，重置 `STATE.sources`／`rows`／`items`／`analyzed`／`hasIPAData`／`hasKanoData`／AI 摘要，並清掉 `localStorage` 的自動存檔（`ipaKanoAdvProject`），避免清空後重整頁面又跳出「還原上次資料」。點擊會先跳原生 `confirm()` 二次確認。
- **資料類型**欄（每個來源一列）：`<select data-type-src>`，可手動標註該來源「自動（不覆寫，預設）／僅計入 IPA／僅計入 Kano／IPA＋Kano 皆計入」，存在 `src.typeTag`。**這是有實際效果的過濾，不只是顯示標籤**：`finalizeMergedAnalysis()` 合併資料時，標「僅計入 IPA」會清空該來源每列的 `kanoFunc`/`kanoDys`，標「僅計入 Kano」會清空 `importance`/`performance`，藉此排除該來源對另一種分析的貢獻。用途：自動判斷為「長表格」的來源常常兩種欄位都對應到了，但使用者可能只想讓某一批資料計入其中一種分析。`typeTag` 會隨 `projectPayload()`／`restoreProjectPayload()` 存進與讀出專案存檔（.json），匯出報告的來源清單（`reportSourceSummary()`）也會顯示這一欄。
- 已用 Chrome 自動化端對端驗證：載入示範資料→標「僅計入 IPA」→套用分析後 `hasKanoData` 正確變 `false`／`hasIPAData` 維持 `true`；清空按鈕正確歸零所有狀態並清掉自動存檔。

## Kano／Better-Worse 係數象限圖：座標軸範圍可調整（僅「進階版」，2026-08-13）

比照「③ IPA 矩陣」既有的「座標軸範圍設定」UI 與資料結構（`ipaAxisRange`），「④ Kano 分析」的 Better-Worse 係數象限圖新增同款控制項：`let bwAxisRange = { xMin:0, xMax:1, yMin:0, yMax:1 }`，橫軸＝Better(SI)、縱軸＝Worse(DI)，輸入框 `step="0.05"`（IPA 是 `0.5`，因為 SI/DI 數值範圍是 0～1 的小數，量表分數是 0～10）。「套用」／「重設為 0～1」按鈕邏輯與 IPA 一致，UI 元件 id 前綴 `bw`（`bwXMin`/`bwXMax`/`bwYMin`/`bwYMax`/`btnApplyBWAxisRange`/`btnResetBWAxisRange`/`bwAxisMsg`）。`drawKanoBWChart()` 與匯出報告用的離屏渲染 `captureBWChartImage()` 都改讀 `bwAxisRange`（原本兩處都寫死 `min:0,max:1`）。

**刻意保留、沒有比照的部分**：IPA 矩陣另有「十字線位置」（總平均／量表中點／自訂）可調整分象限的十字線位置；Kano 的 M/O/A/I 四象限分界線**仍固定在 0.5／0.5**、不隨座標軸範圍調整，因為 0.5 是 Kano Better-Worse 方法論裡有意義的診斷門檻（不是像 IPA 重要度/表現度那樣「總平均」也是合理的十字線位置），使用者只要求「橫軸縱軸可調整」，沒有要求連分界線都能移動。若某次調整後的軸範圍不包含 0.5（例如橫軸設 0.6～1），分界線與其色塊會顯示在可視範圍邊界外／整片同色，屬預期行為，不是 bug。已用 Chrome 自動化驗證：套用自訂範圍、擋下不合法輸入（起始≥結束）、重設皆正確同步到圖表與輸入框。

## 匯出文件水印（僅「進階版」，2026-08-13）

「⑧ 匯出報告」下載的 Word 報告（.doc）、列印／另存 PDF、以及畫面上的「報告預覽」，都會疊加使用者提供的「馬克老師AI」品牌浮水印（角色插畫＋文字），透明度刻意壓低到不影響文字閱讀。

- **來源圖檔**：使用者提供的 `C:\Users\mark_\Downloads\ChatGPT Image 2026年8月10日 下午05_42_45.png`（1536×1024，已含 alpha 透明背景）。已用 Pillow 依實際內容裁去透明邊界（`getbbox()`）、縮小到寬 480px、RGB 量化到 48 色降低檔案體積（113KB），另存一份到本資料夾 `watermark-source.png` 供之後重新調整用；原始檔留在 Downloads 沒有搬進專案。
- **內嵌方式**：處理過的 PNG 轉成 base64，寫成 `滿意度分析互動儀表板_進階版.html` 裡的 `const WATERMARK_DATA_URI`（單一長字串常數，位於 `reportSourceSummary()` 之前）——**這個常數體積很大（base64 約 15 萬字元），改動水印圖片時直接重新產生整個常數替換掉，不要嘗試手動編輯 base64 內容**。改圖步驟：Pillow 重新裁切/縮放/量化 → base64 編碼 → 用腳本（不要用 Edit 工具，avoid 把巨大字串整包載入對話上下文）直接對 html 檔案做文字取代。
- **畫面預覽／列印／PDF**：`watermarkOverlayHtml()` 回傳 `<div class="doc-watermark"><img></div>`，插入 `initExportTab()` 的 `#exportPreview` 與 `printPdfReport()` 的 `#printReportRoot`。CSS：`.doc-watermark{position:absolute;...}`（screen／預覽情境，相對 `.report-doc`——已補上 `position:relative`）；`@media print{ #printReportRoot .doc-watermark{position:fixed;} }`（讓水印在 Chrome 列印/另存 PDF 時每一頁都重複出現，不只出現一次）。圖片本身 `opacity:.08`、`pointer-events:none`，且用 `.report-doc .doc-watermark img` 蓋掉 `.report-doc img` 原本的 `margin`/`border-radius`/`background:#fff` 樣式（那組樣式是給報告內文圖表截圖用的，水印不能套用）。
- **Word (.doc) 匯出**：Word 的 HTML 渲染引擎對 `position:fixed/absolute` 支援不穩定，改用最相容的做法——直接在 `body{background-image:url(...)}` 設定水印，`background-position:center 400pt`、`background-size:45% auto`、不重複。**已知限制**：這種寫法在 Word 裡通常只會在文件背景出現一次（不像瀏覽器列印那樣每頁重複），這是 Word HTML 相容格式的固有限制，不是 bug；如果之後需要「Word 也要每頁都有水印」，要改用 Word 原生的頁首/頁尾水印機制，無法只靠 HTML/CSS 達成。
- 已用 Chrome 自動化驗證：預覽區塊正確插入水印圖片、`opacity` 電腦運算後確實是 `0.08`、`pointer-events:none`、容器 `position:relative`／水印 `position:absolute`，畫面渲染時機正確（水印不會擋住點擊）；`window.print()` 實際觸發列印對話框未逐一實測（該操作會跳出系統對話框，不適合自動化測試），CSS 規則已用語法檢查與程式碼審視確認正確。

## 開發／建置

一般用法是透過已安裝的技能對話產生交付物。若要客製化儀表板，須在解包後的 skill 內編輯 `scripts/dashboard-src/`，再合併成單一 HTML：
```bash
python scripts/build_dashboard.py scripts/dashboard-src 輸出檔名.html
```

## 頂部共用跑馬燈與序號授權（僅「進階版」，2026-08-13）

`滿意度分析互動儀表板_進階版.html` 加了兩個彼此獨立的模組（做法比照 `ai-image-prompt-studio/index.html`）：

- **跑馬燈**：`#marqueeBar` 內容抓自工作區既有的共用公告 Google Sheet（與 `Prompt`／`ai-prompt-generator`／`ai-image-prompt-studio`／`ai-video-studio` 系列共用同一個端點），`localStorage` key `ipaKanoMarquee`，每 20 分鐘背景重抓一次。改內容直接編輯共用 Sheet，不需重新部署。
- **序號授權**：`#licenseGate` 全螢幕遮罩，預設鎖定**整個工具**，通過驗證才顯示 `.hidden`；載入時對後端即時重驗（不只信任 localStorage 快取），背景每 20 分鐘重驗一次。`localStorage` key：`ipaKanoSerial`。12 個月使用期限。
  - `Code.gs`（本資料夾）——GAS 後端骨架，`VALID_AMOUNT = 12`，欄位「序號／開始日期／結束日期」。**這不是本資料夾在跑的檔案**，是貼到下面那個 Sheet 的「擴充功能 → Apps Script」編輯器裡部署成 Web App 的原始碼備份。
  - 綁定 Sheet（**IPA_Kano 專用新開，不與其他工具共用**）：<https://docs.google.com/spreadsheets/d/1VCBT78MeDm2nC1ftQGdBGrkA2lYsC6lqqjWxsYpfKqA/edit>，欄位「任務／優先順序／負責人／狀態／序號／開始日期／結束日期／交件／附註」，已有測試序號 `mark0131`（2026/8/13～2027/12/31）。
  - **已完成部署（2026-08-13）**。`滿意度分析互動儀表板_進階版.html` 的 `LICENSE_CHECK_URL` 已填入實際部署網址：`https://script.google.com/macros/s/AKfycbxXLZgonrHEOodBqiJbr5ysU_c7v8bTqVTKnmgz72QpdzDCyZrb0SE_hPJ7qaJleCqoLw/exec`。部署過程：直接複製貼上到 Apps Script 編輯器出現語法錯誤（`Invalid left-hand side in assignment`，本機 `node --check` 已確認程式碼本身無誤，屬編輯器貼上過程弄亂內容的已知踩坑），改用 `clasp login`（使用者自行完成 OAuth）→ `clasp clone` → 覆蓋 `Code.js` → `clasp push --force` 一次成功，跳過複製貼上；部署為 Web App（新增部署，因為這是全新腳本專案）仍由使用者手動完成。`doGet`（curl）／`doPost`（Node `fetch()`，測試序號 `mark0131` 與一組亂填序號）皆已驗證行為正確；瀏覽器端對端測試填入 `mark0131` 後遮罩正確解鎖顯示「✓ 剩餘 505 天可用」。
  - 這支後端只做序號驗證，不代理任何付費 API（本工具沒有內建服務代理模式）。
  - **剩餘天數持續顯示（2026-08-13 補上）**：原本「剩餘 N 天可用」只出現在驗證通過那一瞬間的 `#gateStatus`，遮罩一隱藏（`.hidden`）整段文字就跟著消失，使用者解鎖後完全看不到還剩幾天。改法：header 的 `.header-actions` 新增常駐徽章 `#licenseBadge`（🔑 剩餘 N 天，hover 顯示到期日），`unlock()` 時同步寫入、`lock()`（含每 20 分鐘背景重驗失敗時）隱藏；剩餘 ≤7 天時徽章變色（`.license-badge.warn`，沿用 `--warn-bg`/`--warn-fg`）。做法比照 `icap_s`／`sbir-gen-s` 那種「只鎖單一功能」模式裡本來就有的常駐 `#licenseStatus` 徽章，但那邊是徽章跟被鎖功能長在一起、不會被隱藏；本專案是「鎖整個工具」模式，遮罩本身會消失，所以徽章要另外搬到頁面 chrome（header）才會持續可見。已用 Chrome 自動化驗證解鎖後徽章正確顯示「🔑 剩餘 505 天」與到期日 tooltip。
  - **修改 `滿意度分析互動儀表板_進階版.html` 後，`dashboard-exe/IPAKanoDashboard.exe` 需依 CLAUDE.md 最上方的 PyInstaller 指令重建才會吃到最新版**（尚未重建——2026-08-13 兩次重建都異常卡死在近乎零 CPU 狀態，不像單純的 SAC 信譽查核慢，疑似當時機器有其他狀況造成資源排擠，已中止並清理殘留程序，待稍後重試）。

## GitHub 與線上部署

公開 repo：<https://github.com/M255525/IPA_Kano>（2026-08-13 建立並推送）。`README.md` 是給 GitHub repo 首頁看的說明文件，與 `CLAUDE.md`（給 Claude Code 的開發筆記）分工不同，兩者都要在功能變動時同步更新。

已啟用 GitHub Pages（`gh api repos/.../pages -X POST -f build_type=workflow`，`.github/workflows/deploy-pages.yml` 標準 Actions 部署，比照 `ai-image-prompt-studio`／`ai-music-prompt-studio` 的模式，不用 legacy branch-source）。線上網址（直接指向儀表板本身）：<https://m255525.github.io/IPA_Kano/滿意度分析互動儀表板_進階版.html>。

**沒有 `index.html`（2026-08-13 建立後隨即依使用者要求移除）**：原本試過做一個彙整入口頁（連到儀表板／manual／skill 檔／範例 docx），但使用者明確表示只要儀表板那個網址可用、根目錄 `https://m255525.github.io/IPA_Kano/` 不要能開——換句話說希望 Pages 根目錄沒有內容（訪問會 404），只有儀表板自己的網址是「正式對外的頁面」。**之後不要再自動加回 `index.html`**，除非使用者重新要求。`README.md` 的「線上使用」連結已改指向儀表板完整網址，不是根目錄。

已用 Chrome 對正式線上網址（非本機測試伺服器）端對端驗證過序號解鎖／跑馬燈／剩餘天數徽章皆正確顯示（驗證發生在移除 `index.html` 之前，走的是入口頁轉連過去，但儀表板本身的行為與網址不變，結論仍然有效）。

## 已知限制

- Kano 分析需問卷同時具「正向題」與「反向題」；只有重要度／表現度的問卷只能做 IPA（會自動偵測並提示）。
- Google 試算表連結需設為「知道連結者皆可查看」，且偶因瀏覽器跨網域限制被擋，建議改下載 CSV 再上傳。

### 桌面版 exe（dashboard-exe/）

`dashboard-exe/IPAKanoDashboard.exe` 是可攜式單檔桌面版（做法比照 `icap-generator/icap/`）：`launcher.py` 把「滿意度分析互動儀表板_進階版.html」＋ `manual.html`（頁面右上角「操作手冊」按鈕、頁尾連結都指向它，**兩個檔案都要打包，漏了 manual.html 按鈕會 404**）打包進 exe，執行時於 `127.0.0.1:8775` 起本機伺服器並開預設瀏覽器（**固定 8775 埠**——工作區埠號分配：8765 ai-course-hub、8766 video-editor、8767 fruit-ninja-cam、8771 icap exe、8772 sbir exe、8773 ai-video-studio、8774 ai-video-studio 桌面版 exe、**8775 本專案 exe**）。圖表函式庫（Chart.js/PapaParse/xlsx）與 Google 試算表匯入走 CDN／線上連結，**需要網路連線**；問卷資料本身仍全部在瀏覽器本機運算，不上傳。**修改 html 後 exe 不會自動更新，需重建**（PowerShell、絕對路徑）：

```powershell
$proj = "C:\Users\mark_\AI Test\資料儀表板\IPA_Kano"
cd $proj
python -m PyInstaller --onefile --console --name IPAKanoDashboard `
  --distpath "$proj\dashboard-exe" --workpath "$env:TEMP\pyi-build-ipakano" --specpath "$env:TEMP" `
  --add-data "$proj\滿意度分析互動儀表板_進階版.html;." --add-data "$proj\manual.html;." `
  launcher.py
```

exe 未簽章，首次執行會遇 SmartScreen 警告。測試 exe 時注意：PyInstaller onefile 會有父子兩個程序，`taskkill //IM IPAKanoDashboard.exe //F` 才殺得乾淨；本機沙箱下 PowerShell `Start-Process` 啟動 exe 會 Access denied，改用 Git-Bash 背景執行；重建前若舊 exe 仍在執行會因檔案鎖定 `PermissionError: [WinError 5]` 建置失敗，需先 taskkill 再重跑。
