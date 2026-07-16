# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本資料夾是 **`ipa-skill` 的散布包與交付物範例**，不是散開的原始碼專案。此技能把問卷調查資料轉換成有科學依據的改善優先序，方法論為 **IPA（重要度—表現度矩陣）＋ Kano 品質屬性模型 ＋ 柏拉圖 80/20 分析**。

## 檔案性質

- **`ipa-skill.skill`** — 打包好的技能檔（在 Claude 設定 → Capabilities 上傳安裝）。**技能的實際原始碼在這個 bundle 內**（`SKILL.md`、`references/plan-doc-outline.md`、`scripts/build_dashboard.py`、`scripts/dashboard-src/`、`scripts/docx-helpers/`），要改程式需先解包 `.skill`。
- **`滿意度分析互動儀表板.html`** — 交付物範例二：單一 HTML 互動儀表板（下述）。
- **`產品服務滿意度問卷分析計畫書_v2.docx`** — 交付物範例一：Word 計畫書。
- **`ipa-skill-eval-review.html`** — 技能評測／審閱報告。

## 兩種交付物

1. **計畫書（Word）** — 含方法論與執行計畫（IPA 四象限、Kano 五屬性對照、IPA×Kano 整合優先序、柏拉圖、時程里程碑、名詞白話對照）。下載格式為相容 Word 的 HTML 內嵌 `.doc`，非原生 `.docx`。
2. **互動式儀表板（單一 HTML）** — 七個分頁：資料匯入 / IPA 矩陣 / Kano 分析 / 整合優先序 / 柏拉圖 / 補充意見 / 匯出報告。**全部運算在瀏覽器本機執行，資料不上傳**。自動判斷三種輸入格式：IPA 寬表格、Kano 專用寬表格、長表格；判斷失敗時提供欄位對應介面。可重複上傳新批資料重算（適合定期追蹤）。

## 開發／建置

一般用法是透過已安裝的技能對話產生交付物。若要客製化儀表板，須在解包後的 skill 內編輯 `scripts/dashboard-src/`，再合併成單一 HTML：
```bash
python scripts/build_dashboard.py scripts/dashboard-src 輸出檔名.html
```

## 已知限制

- Kano 分析需問卷同時具「正向題」與「反向題」；只有重要度／表現度的問卷只能做 IPA（會自動偵測並提示）。
- Google 試算表連結需設為「知道連結者皆可查看」，且偶因瀏覽器跨網域限制被擋，建議改下載 CSV 再上傳。

### 桌面版 exe（dashboard-exe/）

`dashboard-exe/IPAKanoDashboard.exe` 是可攜式單檔桌面版（做法比照 `icap-generator/icap/`）：`launcher.py` 把「滿意度分析互動儀表板_進階版.html」＋ `manual.html`（頁面右上角「操作手冊」按鈕、頁尾連結都指向它，**兩個檔案都要打包，漏了 manual.html 按鈕會 404**）打包進 exe，執行時於 `127.0.0.1:8775` 起本機伺服器並開預設瀏覽器（**固定 8775 埠**——工作區埠號分配：8765 ai-course-hub、8766 video-editor、8767 fruit-ninja-cam、8771 icap exe、8772 sbir exe、8773 ai-video-studio、8774 ai-video-studio 桌面版 exe、**8775 本專案 exe**）。圖表函式庫（Chart.js/PapaParse/xlsx）與 Google 試算表匯入走 CDN／線上連結，**需要網路連線**；問卷資料本身仍全部在瀏覽器本機運算，不上傳。**修改 html 後 exe 不會自動更新，需重建**（PowerShell、絕對路徑）：

```powershell
$proj = "C:\Users\mark_\AI Test\IPA_Kano"
cd $proj
python -m PyInstaller --onefile --console --name IPAKanoDashboard `
  --distpath "$proj\dashboard-exe" --workpath "$env:TEMP\pyi-build-ipakano" --specpath "$env:TEMP" `
  --add-data "$proj\滿意度分析互動儀表板_進階版.html;." --add-data "$proj\manual.html;." `
  launcher.py
```

exe 未簽章，首次執行會遇 SmartScreen 警告。測試 exe 時注意：PyInstaller onefile 會有父子兩個程序，`taskkill //IM IPAKanoDashboard.exe //F` 才殺得乾淨；本機沙箱下 PowerShell `Start-Process` 啟動 exe 會 Access denied，改用 Git-Bash 背景執行；重建前若舊 exe 仍在執行會因檔案鎖定 `PermissionError: [WinError 5]` 建置失敗，需先 taskkill 再重跑。
