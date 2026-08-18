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

## IPA 矩陣：橫軸與縱軸對調（僅「進階版」，2026-08-13）

應使用者要求，「③ IPA 矩陣」的橫軸／縱軸對調：**橫軸＝表現度、縱軸＝重要度**（原本橫軸是重要度、縱軸是表現度）。改動範圍很廣，牽涉的地方都要一起改，之後再調整這個圖表務必檢查以下每一處：

- `assignQuadrants(items, cx, cy)`：`cx`（橫軸門檻）現在對應表現度、`cy`（縱軸門檻）對應重要度——**Q1～Q4 的語意定義完全沒變**（Q1＝兩者皆高、Q2＝重要度高表現度低、Q3＝兩者皆低、Q4＝重要度低表現度高），只是判斷式裡 `highPerf`/`highImp` 各自比較哪個門檻值要對調，其餘引用 `QUADRANT_LABEL`／`PRIORITY_ACTION`／洞察文字等下游程式碼完全不受影響（都是依 Q1-Q4 語意標籤運作，跟畫面上哪個軸是哪個變數無關）。
- `currentCenter()`：`mean` 模式回傳值改成 `{x:STATE.grandPerf, y:STATE.grandImp}`。
- `drawIPAChart()` 的 `quadrantBgPlugin`：四象限背景色塊裡，Q1（綠，兩者皆高）與 Q3（灰，兩者皆低）在畫面上的位置（右上／左下）**不受橫縱軸對調影響**（因為兩個條件同時成立/不成立時，跟哪個變數在哪個軸無關）；但 Q2（紅）與 Q4（紫）因為兩個條件不同時成立，畫面位置會左右互換（Q2 從右下變左上、Q4 從左上變右下）——這是最容易漏改、改錯了色塊會跟實際資料點對不起來的地方。
- `drawIPAChart()`／`captureIPAChartImage()`（匯出報告用離屏渲染）：資料點 `{x:i.perfMean, y:i.impMean}`、座標軸標題文字對調。
- 圖表 tooltip：改成直接讀 `d.ref.impMean`/`d.ref.perfMean`（不再用 `d.x`/`d.y`），這樣不管以後橫縱軸再怎麼調整，文字說明都一定正確，不用每次跟著改。
- UI 文字：分頁說明（橫軸＝...縱軸＝...）、自訂十字線兩個輸入框的 `title` 屬性、「目前十字線」顯示文字、「座標軸範圍設定」的橫軸／縱軸標籤、匯出報告裡的「十字線位置」文字，共 6 處都要對調。
- **沒有改的部分**：`ipaAxisRange` 這個 state 物件本身的欄位名稱（`xMin`/`xMax`/`yMin`/`yMax`）維持不變，只是現在 `xMin`/`xMax` 代表的是表現度範圍、`yMin`/`yMax` 代表重要度範圍——因為預設值兩者都是 0～10，數值不用改，只有畫面上的標籤文字要跟著改，避免不必要的變數改名徒增風險。CSV 匯出（`IPA象限一覽表.csv`）、題項象限一覽表、報告裡的資料表格全部不受影響，因為那些都是欄位獨立的表格（重要度均分／表現度均分各自一欄），不是畫在 x/y 座標上的圖表。
- 已用 Chrome 自動化驗證：座標軸標題正確對調、已知為 Q1（兩者皆高）與 Q2（重要度高表現度低）的題項在圖表資料點座標與象限分類都正確對應。

## IPA 矩陣：象限編號重新編排（僅「進階版」，2026-08-13）

緊接著上面「橫軸與縱軸對調」之後，使用者再要求把「第X象限」這組**顯示數字**重新編排（跟畫面上的視覺位置無關，純粹是編號規則）：以新的「第一象限」為起點，順時針編號。對照表：

| 內部鍵值（不變） | 描述性名稱（不變） | 舊編號 | 新編號 |
|---|---|---|---|
| Q1 | 優勢保持區 | 第一象限 | **第二象限** |
| Q2 | 集中改善區 | 第二象限 | **第一象限** |
| Q3 | 次要改善區 | 第三象限 | **第四象限** |
| Q4 | 過度供給區 | 第四象限 | **第三象限** |

**只改 `QUADRANT_LABEL` 常數與另外三處直接寫死中文編號的地方，不動內部鍵值**：`Q1`/`Q2`/`Q3`/`Q4` 這組英文鍵、對應的 `--q1`~`--q4` 顏色變數、`QUADRANT_URGENCY`／`PRIORITY_ACTION`／洞察文字等所有依鍵值運作的下游程式碼完全不用改，因為它們都是靠 `QUADRANT_LABEL[item.quadrant]` 這種方式間接取得顯示文字，改常數就全部自動生效。**額外手動修的 3 處**（沒有透過 `QUADRANT_LABEL` 常數、直接寫死中文字串）：③ IPA 矩陣分頁圖表下方的圖例、「改善優先序（IPA 象限＋落差排序）」卡片裡的說明文字（原本寫「優先處理第二象限...其次為第三象限」現在對應到的實際語意不變但編號要改成「優先處理第一象限...其次為第四象限」）、匯出報告裡的象限統計文字。已用 Chrome 驗證 `QUADRANT_LABEL` 常數與畫面圖例文字皆正確對調。

**顯示順序也改成依數字排序（同一天內緊接著的第二次調整）**：圖例與匯出報告的象限統計文字，原本是照內部鍵值順序 Q1→Q2→Q3→Q4 排列（重新編號後畫面上就變成「第二／第一／第四／第三」這種不照順序的排法），改成直接按數字順序寫死成「第一象限(Q2)／第二象限(Q1)／第三象限(Q4)／第四象限(Q3)」——這兩處都只有 4 個固定項目，用寫死順序最簡單可靠，沒有另外寫排序函式。若之後象限編號規則又要調整，記得這兩處的項目順序要跟著手動重排，不會自動跟著 `QUADRANT_LABEL` 改變而重新排序。已用 Chrome 驗證圖例確實依「第一／第二／第三／第四」順序顯示。

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

## 資料匯入：長表格欄位對應自動猜測修正（2026-08-18）

「①資料匯入」在系統無法自動判斷寬表格格式時，會退回「欄位對應（長表格模式）」讓使用者手動指定每個欄位對應到原始檔案的哪一欄，並用 `guessMapping()` 先幫使用者猜測一次預設值。原本的猜測邏輯有 bug：`item`（題項名稱）欄位的關鍵字清單 `["題項","項目","服務項目","功能","構面","題目"]` 把最精準的「題目」放在最後，而「功能」這個字很generic，如果檔案裡剛好有 Kano 正向題欄位叫「功能性問題」，`功能` 這個關鍵字會先比對到那一欄（因為是包含比對、且排在「題目」之前），導致「題項名稱」誤配到 Kano 正向題欄位，而不是真正放題目文字的那一欄。

修法（`guessMapping()`，滿意度分析互動儀表板_進階版.html 855 行附近）：改成兩階段比對——先找**完全相符**的欄名（例如欄名剛好就是「題目」），找不到才退回原本的「包含比對」；`item` 關鍵字順序也調整為 `["題目","題項","服務項目","構面","項目","功能"]`，把最精準的字放最前面、最容易誤判的generic字（「項目」「功能」）放最後當保底。這個 `guessMapping()` 是 IPA／Kano 共用同一份邏輯（欄位對應表單同時列出 importance/performance 與 kanoFunc/kanoDys），改一次兩邊都生效，不需要分別處理。已用 Node 腳本模擬「題目＋功能性問題＋非功能性問題」同時出現的欄位組合驗證修正後 `item` 正確對應到「題目」欄。

**同一天內接續發現的真正根因，範圍更大**：上面的 `guessMapping()` 修正只是治標——使用者實際上傳的 `IPA.csv`（問卷平台常見匯出格式，每個題項各佔兩欄，欄名分別是「XX題目（重要度）」「XX題目（滿意度）」，例如「需求對接與規劃（重要度）」「需求對接與規劃（滿意度）」）根本沒被判斷成「IPA 寬表格」，而是整批落到「長表格模式」手動欄位對應——因為 `detectWideFormat(headers)` 原本只認「重要度欄與表現度欄的欄名完全一模一樣、且剛好重複出現兩次」這一種寬表格寫法（`groups[key].length === 2`），對「同一題目、欄名各自加了不同尾綴」這種業界更常見的寫法完全沒有偵測能力，於是每次都判斷失敗、退回長表格模式，使用者才會一直卡在「欄位對應」畫面手動選欄位。

修法（`detectWideFormat()`，滿意度分析互動儀表板_進階版.html 743 行附近，新增 `IPA_IMP_SUFFIX`／`IPA_PERF_SUFFIX`／`stripHeaderSuffix()`／`classifyIPAHeaderSuffix()`）：比照既有 `detectKanoWideFormat()`/`classifyKanoHeader()` 的「尾綴分類」手法——先保留原本「欄名完全重複兩次」的偵測（相容舊格式），沒抓到重複的欄位再進第二輪：把每個欄名去掉結尾的「（重要度）」「(重要度)」「重要度」「重要性」（歸類為 imp）或「（滿意度）」「(滿意度)」「滿意度」「表現度」「滿意」「表現」（歸類為 perf），取得「核心題目文字」；再用核心文字比對 imp／perf 兩邊是否有相同題目，相同的才配成一對。`normHeader()` 已經把全形（）轉半形，所以尾綴比對只需處理半形括號即可涵蓋兩種寫法。已用 Node 腳本直接讀取使用者實際的 `C:\Users\mark_\Downloads\IPA.csv`（UTF-8 with BOM，20 欄 IPA 題項 ＋ 4 欄人口統計欄位）驗證：10 個題項全部正確配對、題項名稱等於原始題目文字（例如「需求對接與規劃」），人口統計欄位（回應ID／性別／年齡區間／教育程度）不受影響地被忽略；另用寫死測資驗證舊版「欄名完全重複兩次」寫法仍相容。

## Kano 寬表格偵測：問卷平台完整句子式欄名的支援（2026-08-18）

同一天內接續前面 IPA 寬表格偵測的修正，使用者接著回報 Kano 也有一樣的問題（上傳 `enterprise_training_kano_grouped_survey.csv` 抓不到題項名稱）。實測發現 `detectKanoWideFormat()` 對這份真實檔案完全偵測失敗，根因有三層：

1. **人口統計欄位混進判斷**：原本的分類（`classifyKanoHeader`）對「檔案裡所有欄位」一視同仁地跑，這份檔案前面多了「回應ID／性別／年齡區間／教育程度」4 欄，這些欄名沒有負向詞、被誤判成「正向題」，直接拖亂正負向題數量的統計。
2. **完整句子式欄名讓尾綴比對完全失效**：原本 `classifyKanoHeader` 是拿**整個欄名**去比對結尾是否為「好」「不佳」之類的詞。但這份檔案的欄名是問卷平台常見的完整句子，例如「如果需求對接與規劃良好，您的感受是？」「如果需求對接與規劃不佳，您的感受是？」——結尾固定是「，您的感受是？」，真正的正/反向措辭「良好」「不佳」被包在句子中間，尾綴比對完全抓不到，兩邊也因此配不成對。
3. **少數措辭沒被涵蓋**：「如果課程效益與CP值高／低」用「高」「低」表達正反向，而「高」「低」都不在 `NEG_WORDS`（沒/不/無/未）也不在原本的尾綴清單裡，落到預設分支被誤判成「正向」，讓正負向題數對不起來（`positives.length !== negatives.length` 直接整批放棄偵測）。

修法（`滿意度分析互動儀表板_進階版.html`，`detectKanoWideFormat()` 及其相依函式，約 808-903 行附近）：
- **新增候選欄位篩選**：`sampleLooksLikeKanoAnswers()` 改成逐欄單獨檢查（而非把所有欄位的樣本混在一起檢查），只有作答內容本身看起來像 Kano 五點量表的欄位才會進入後續判斷，人口統計欄位這類欄位天生就會被排除，不需要看欄名。
- **新增 `stripCommonWrapper()`**：對所有候選欄位的欄名，計算彼此共同的最長前綴與最長後綴（例如「如果」與「，您的感受是？」），拿掉這段共同的「包裝句模板」文字，不需要預先寫死模板內容本身，換一套問卷平台的用語也能適用。
- **`POS_SUFFIX`／`NEG_SUFFIX` 補上「高」「低」「良好」「穩定」「不穩定」等常見措辭**，讓拆掉包裝句之後的核心文字能正確配對。
- **配對邏輯改成三輪、由嚴謹到保底**：① 拆完尾綴後兩邊核心文字完全相同直接配對（多數題目）；② 核心文字裡插入了否定詞而非尾綴時（例如「未提供ＸＸ」對「提供ＸＸ」），移除該否定字後再比對一次；③ 前兩輪仍配不到的，退回原本「核心文字最長共同前綴」的模糊比對保底，避免遇到沒預期到的措辭就整批漏掉不偵測。
- **附帶修正 `normalizeKanoAnswer()`**：這份真實檔案的「普通」等級作答文字是「沒有感覺」，但原本清單只有「沒感覺」（缺一個「有」字），完全比對不到會導致該筆作答被當成無效值排除、悄悄拉低分析準確度，已補上「沒有感覺」「沒什麼感覺」。

已用 Node 腳本直接讀取使用者實際的 `C:\Users\mark_\Downloads\enterprise_training_kano_grouped_survey.csv`（100 筆作答、10 個題項＋4 個人口統計欄位）驗證：10 個題項全部正確配對，題項名稱正確還原成可讀的題目文字（例如「需求對接與規劃」「課程效益與CP值」），人口統計欄位正確被排除在外；另外用短句式舊格式欄名（如「服務態度好」／「服務態度不好」，沒有「如果...您的感受是？」包裝句）驗證向下相容、仍正確配對。

## Kano Better-Worse 象限圖：座標軸改為 0～10 顯示，與 IPA 矩陣一致（2026-08-18）

延續前面「座標軸範圍可調整」的功能（見上面同名段落），使用者接著要求 Kano 的 Better-Worse 係數圖橫縱軸也改成跟 IPA 矩陣一樣的 0～10 制。先跟使用者確認過改法：**只改圖表的顯示刻度，SI/DI 係數本身的計算方式不變，仍是 0～1 的比例**（圖上刻度＝係數×10），不是把 Kano 分析方法論本身改成 10 分制——因為 Better-Worse 係數在方法論上就是「(某類票數)／(有效樣本數)」的比例，天生就是 0～1，跟 IPA 的重要度/表現度是使用者直接填答的 0～10 量表分數，兩者性質不同，只是讓兩張圖「看起來」的座標軸範圍一致，方便使用者閱讀/報告呈現。

具體改動（`滿意度分析互動儀表板_進階版.html`）：
- `bwAxisRange` 預設值與「重設」按鈕都從 `{0,1,0,1}` 改成 `{0,10,0,10}`，座標軸範圍輸入框 `step` 從 `0.05` 改成 `0.5`（跟 IPA 的 `ipaXMin`/`ipaXMax` 等輸入框一致）。
- `drawKanoBWChart()`：畫出的資料點座標改成 `{x:i.si*10, y:i.di*10}`（原本是 `{x:i.si, y:i.di}`），`ref:i` 仍保留指向原始資料物件。M/O/A/I 四象限的分界十字線 `bwZonePlugin` 裡的 `getPixelForValue(0.5)` 兩處都改成 `getPixelForValue(5)`（分界線的「相對位置」完全沒變，只是換算成新刻度下的座標）。
- **tooltip 改讀 `d.ref.si`/`d.ref.di`（原始 0～1 係數）而不是 `d.x`/`d.y`（畫面上乘以10後的座標）**——比照 CLAUDE.md 前面「IPA 矩陣橫縱軸對調」那次記錄的教訓：tooltip 直接讀原始資料物件的欄位，不管圖表座標軸刻度以後怎麼調整，文字說明都一定顯示正確的原始係數，不用每次跟著改。
- `captureBWChartImage()`（匯出報告用離屏渲染）比照同步 `×10`。
- 分頁說明文字（「以 0.5 為分界」→「以 5（原始係數 0.5）為分界」）與 12 字元的座標軸標題不變（`Better 係數`／`Worse 係數`），因為文字沒有寫死刻度數字。
- **刻意沒有改的部分**：資料表格（各題項 Kano 屬性統計表、CSV 匯出、整合優先序的 Better/Worse 排行榜）裡顯示的「Better (SI)」「Worse (DI)」數值全部維持原始 0～1 兩位小數（`fmt2(i.si)`），只有**圖表**改成 0～10 顯示——使用者只要求圖表座標軸，數據表格的原始係數不該被混淆成別的單位。

已用 Playwright 對本機預覽伺服器端對端驗證：載入示範資料套用分析後，`bwAxisRange` 預設為 `{0,10,0,10}`、圖表 `scales.x/y` 的 `min`/`max` 皆為 `0`/`10`、資料點座標正確等於原始 SI/DI×10（例如 `si=0.6129` 對應圖上 `x=6.129`）、tooltip callback 輸出正確顯示原始係數（`Better 0.61／Worse 1.00`，不是 6.1/10.0）、重設按鈕文字與輸入框 step 皆已同步更新。

## Kano Better/Worse 數值顯示：全面改為 ×10，不只圖表（同一天內接續調整，2026-08-18）

上一段「Kano Better-Worse 象限圖座標軸改為 0～10」原本刻意只改圖表座標，數據表格（④ Kano 分頁統計表、⑤ 整合優先序 Kano-only 版本、⑧ 匯出報告）維持顯示原始 0～1 係數，理由是「圖表座標軸調整跟數據表格單位是兩件事，不該混用」。但使用者接著明確要求「⑤ 整合優先序」（以及連帶 Kano 相關） 的 Better/Worse 數值也要跟圖表同步，也就是**整個工具裡任何顯示給使用者看的 Better/Worse 數字，一律都改成 ×10**，只有內部分類邏輯（M/O/A/I 判定、優先序排序比較）繼續用原始 0～1 比例，這樣才不會有些地方看到「0.61」、有些地方看到「6.1」的不一致。

實作：新增共用格式化函式 `fmtBW(n)`（`滿意度分析互動儀表板_進階版.html`，緊接在 `fmt2()` 之後，約 710 行附近）：`(n*10).toFixed(2)`，取代所有原本顯示 SI/DI 的 `fmt2(...)` 呼叫。改動涵蓋的位置：
- ④ Kano 分析分頁的「各題項 Kano 屬性票數分布」統計表與其 CSV 匯出（欄位標題同步加註「Better (SI×10)」「Worse (DI×10)」，避免使用者誤以為 SI/DI 本身的方法論定義變了）。
- Better-Worse 象限圖的 tooltip（原本讀 `d.ref.si`/`d.ref.di` 顯示原始係數，現在一樣讀 `d.ref` 但改用 `fmtBW()`，跟軸線刻度一致）。
- ⑤ 整合優先序分頁「Kano-only（無 IPA 資料）」版本的表格與 CSV 匯出，欄位標題同步加註「×10」。
- 題項詳情卡（點擊圖表/表格列展開的 `detail-panel`）裡的 Better／Worse 徽章。
- 自動洞察文字（「做好後最能提升滿意度的是...」那段）。
- ⑧ 匯出報告的 Kano 統計表與 Kano-only 排行榜表格。
- **沒有改的部分**：`kanoOnlyRank`／`integratedRank` 等排序比較邏輯繼續直接比較原始 `i.si`/`i.di`（`sort((a,b)=>b.si-a.si)` 這類），因為只是要找相對大小順序、乘不乘 10 結果都一樣，維持用原始值比較最單純、風險最低。

已用 Playwright 端對端驗證：④ Kano 表格列與表頭正確顯示「6.13」「10.00」與「Better (SI×10)」；圖表 tooltip 輸出 `Better 6.13／Worse 10.00`；暫時模擬「只有 Kano 沒有 IPA」情境重新渲染⑤整合優先序分頁，確認該分支表格與表頭同樣正確顯示 ×10 數值。

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

已啟用 GitHub Pages（`gh api repos/.../pages -X POST -f build_type=workflow`，`.github/workflows/deploy-pages.yml` 標準 Actions 部署，比照 `ai-image-prompt-studio`／`ai-music-prompt-studio` 的模式，不用 legacy branch-source）。線上網址：<https://m255525.github.io/IPA_Kano/>（根目錄），會自動轉址到 <https://m255525.github.io/IPA_Kano/滿意度分析互動儀表板_進階版.html>（儀表板本身）。

**`index.html` 的角色演變（2026-08-13 同一天內三次調整，記錄下來避免之後又繞回去）**：
1. 一開始做了彙整入口頁（連到儀表板／manual／skill 檔／範例 docx）。
2. 使用者接著表示只要儀表板網址可用、根目錄不要能開，於是**整個刪除** `index.html`，讓根目錄變成 404。
3. 使用者再表示希望根目錄能自動轉址到儀表板，於是**改回一個純轉址用的極簡 `index.html`**（`<meta http-equiv="refresh">` ＋ JS `location.replace()` 雙保險，GitHub Pages 是純靜態託管、沒有伺服器端轉址機制，只能用這種前端轉址做法），不再是彙整多個連結的入口頁。

**目前定案（之後預設維持這個狀態，除非使用者再改）**：根目錄 = 純轉址頁，唯一內容出口是儀表板本身；`README.md` 的「線上使用」連結指向根目錄（因為會自動轉過去）。

已用 Chrome 對正式線上網址（非本機測試伺服器）端對端驗證過序號解鎖／跑馬燈／剩餘天數徽章皆正確顯示（驗證發生在改成純轉址版之前，走的是舊版入口頁轉連過去，但儀表板本身的行為與網址不變，結論仍然有效；純轉址版本身只是 `location.replace`，未另外重新跑一次端對端測試）。

## 加入主畫面（PWA，2026-08-14 新增）

比照工作區 `expense-tracker-pwa` 的做法：`manifest.json`＋`icons/`（淺灰 `#F5F5F7` 背景、藍色 `#0071E3`「度」字圖示，對照 `--bg`／`--primary`）＋`service-worker.js`（network-first＋同源快取備援，跨網域的 Chart.js／PapaParse／xlsx CDN 一律略過不進快取，不需要每次改動升版 `CACHE_NAME`）。**manifest／SW／安裝按鈕都掛在 `滿意度分析互動儀表板_進階版.html`（實際內容頁），不是 `index.html`（純轉址 stub）**——`manifest.json` 的 `start_url` 也指向 `./滿意度分析互動儀表板_進階版.html`，從主畫面啟動會直接開到儀表板，不會經過轉址那一跳。安裝按鈕（`#installBtn`）放在 `.header-actions`（跟「操作手冊」「🌙 深色模式」同排、同 `.theme-toggle` 樣式），本工具沒有 `showToast`，安裝失敗走「暫時置換按鈕文字」的簡易 fallback。**`dashboard-exe/IPAKanoDashboard.exe` 桌面版不受影響**（桌面版本來就不透過瀏覽器安裝機制，manifest/SW 只在瀏覽器直接開啟網頁時生效）。已用 Playwright 實測 Chromium 觸發 `beforeinstallprompt`、SW 註冊成功（測試時用 devtools 對 `#licenseGate` 加 `.hidden` 繞過鎖定畫面）。


**iOS／iPadOS／macOS 相容性補強（2026-08-14 同日追加）**：Safari（含 iOS 上的 Chrome/Firefox，底層都是 WebKit）**永遠不會觸發 `beforeinstallprompt`**，原本的按鈕邏輯在這些瀏覽器上一律落入「瀏覽器不支援」這句話，其實是誤導——蘋果裝置本來就能加入主畫面，只是要透過分享選單手動操作，不像 Chrome/Edge 有自動彈窗。修法：安裝腳本新增 `isIOSDevice`（`/iPad|iPhone|iPod/` 或 `navigator.platform==='MacIntel' && maxTouchPoints>1`——後者是因為 iPadOS 13+ 預設偽裝成 Mac 桌面版 UA，要用觸控點數才分得出來是 iPad 還是真的 Mac）與 `isMacDesktop && isSafariEngine`（macOS 桌面版 Safari 17+ 是「檔案→加入 Dock」，跟手機的分享選單操作不同）兩種判斷，各自顯示對應的操作指引文字，不再顯示「不支援」；`isStandalone`（`matchMedia('(display-mode: standalone)')` 或 iOS 專有的 `navigator.standalone`）為真時直接隱藏按鈕（已經是安裝後開啟，不需要再顯示安裝按鈕）。`<head>` 同步補上 `apple-touch-icon`（180×180 專用尺寸，`icons/apple-touch-icon.png`，純色不透明背景）＋ `apple-mobile-web-app-capable`／`mobile-web-app-capable`（兩個都要，前者給 Safari、後者是 Chrome 主張的新標準，只寫一個 Chrome 會在主控台噴 deprecation warning）＋ `apple-mobile-web-app-status-bar-style`／`apple-mobile-web-app-title`。這五個判斷/訊息字串在全部 9 個已加裝 PWA 的專案裡是逐字複製的同一段邏輯，日後若要調整任一處的措辭或判斷式，建議九個一起改，避免各專案的安裝體驗不一致。

**回饋機制與快取踩坑修正（2026-08-14，使用者實測回報「加入主畫面沒有功能」才發現兩層問題）**：(1) 原本無 `showToast` 時用「暫時置換按鈕文字」當提示，在工具列裡太不明顯，使用者完全沒注意到訊息出現過——改成 `window.alert(fallbackMessage())`，`deferredPrompt.prompt()` 也包 try/catch。(2) 改完使用者仍回報沒反應，追查發現 `service-worker.js` 的 `fetch(event.request)` 沒有繞過瀏覽器 HTTP 快取——GitHub Pages 對回應下 `Cache-Control: max-age=600`，10 分鐘內「network-first」名不符實，可能吃到舊版內容重新存進 Cache Storage。改成 `fetch(event.request, {cache:'reload'})` 強制略過 HTTP 快取，`CACHE_NAME` 同步升版 v1→v2 清掉已污染的快取。這是跟 `expense-tracker-pwa` 那次「install 階段 `cache.addAll()` 忘記加 `{cache:'reload'}`」同一個 bug class 的 runtime 版本，細節見 [[pwa-install-rollout]]。

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
