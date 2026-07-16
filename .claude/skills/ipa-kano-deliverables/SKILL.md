---
name: ipa-kano-deliverables
description: 在 IPA_Kano 專案產出或更新 IPA×Kano 滿意度分析交付物。當使用者在此資料夾要求更新滿意度分析儀表板、計畫書，或客製化 IPA/Kano 儀表板原始碼時使用。方法論為 IPA（重要度—表現度矩陣）＋ Kano 品質屬性 ＋ 柏拉圖 80/20。實際分析與文件產出委由已安裝的 ipa-skill；本 skill 補充本資料夾的檔案位置與儀表板建置指令。
---

# IPA_Kano 交付物產出／更新

本資料夾是 `ipa-skill` 的散布包與交付物範例。分析與計畫書／儀表板的產出邏輯由**已安裝的 ipa-skill** 負責（提到滿意度問卷分析、IPA、Kano 等會自動觸發）；本 skill 只補充此專案的在地資訊。

## 兩種交付物

- **計畫書（Word）**：`產品服務滿意度問卷分析計畫書_v2.docx` 為範例；含 IPA 四象限、Kano 五屬性、IPA×Kano 整合優先序、柏拉圖、時程。輸出為相容 Word 的 HTML 內嵌 `.doc`。
- **互動式儀表板（單一 HTML）**：`滿意度分析互動儀表板.html` 為範例；七分頁（資料匯入／IPA／Kano／整合優先序／柏拉圖／補充意見／匯出），全部本機運算、不上傳。自動判斷 IPA 寬表格／Kano 寬表格／長表格。

## 客製化儀表板原始碼

儀表板原始碼在 `ipa-skill.skill` 內（需先解包），位於 `scripts/dashboard-src/`。修改後重新合併成單一 HTML：
```bash
python scripts/build_dashboard.py scripts/dashboard-src 輸出檔名.html
```

## 注意

- Kano 分析需問卷同時具「正向題」與「反向題」；只有重要度／表現度者只能做 IPA。
- Google 試算表連結需設「知道連結者皆可查看」，偶因跨網域限制被擋時改下載 CSV 上傳。
