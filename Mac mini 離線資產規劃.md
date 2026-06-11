# Mac mini 離線資產規劃

## 目的

Mac mini 不應被視為未來 App Store 版本的必要 runtime。

它在 Claw Draw 裡更適合扮演：

- 課程 authoring 機器
- 規則與閾值校準機器
- 小模型訓練機器
- 教學文案與回饋語句產生機器
- 高成本實驗環境

最終 App Store 版本應盡量把成果編譯或打包進 app，讓使用者不需要 Mac mini、不需要雲端、不需要付費 API。

## 可嵌入 App 的資產

### 1. 課程知識庫

這是最優先建立的資產。

內容：

- 課程模組樹
- 每題練習說明
- 每題預期時間
- 每題目標數量
- 常見錯誤類型
- 錯誤與補練的對應關係
- 升級條件
- 弱項補練規則

建議格式：

- authoring 階段使用 Markdown / YAML / JSON。
- app 內建階段轉成 SQLite 或壓縮 JSON。

價值：

- 它是產品的核心，不是單純資料。
- 即使分析公式被複製，課程順序與補練邏輯仍需要長期打磨。

### 2. Rubric 與閾值資料

Mac mini 用於分析練習紀錄，校準不同練習的合理門檻。

可校準項目：

- 直線起點誤差
- 直線終點誤差
- 路徑偏離量
- 圓形閉合誤差
- 橢圓主軸角度偏差
- 抖動指標
- 重描指標
- 完成數量

輸出資產：

```text
rubric_parameters
- exercise_type
- metric_name
- pass_threshold
- warning_threshold
- retry_threshold
- version
```

價值：

- 第一版可以先用人工設定。
- 後續用真實練習資料調整，避免回饋太嚴或太鬆。

### 3. 小型評估模型

若規則系統不足，可在 Mac mini 訓練小模型，再嵌入 app。

建議輸入：

- start_error_avg
- end_error_avg
- path_deviation_avg
- closure_error_avg
- wobble_score
- redraw_score
- angle_error_avg
- completion_ratio

建議輸出：

- pass
- borderline
- retry
- primary_issue

適合模型：

- logistic regression
- decision tree
- gradient boosting
- tiny MLP

部署方式：

- 轉成 Core ML。
- 或輸出成可解釋的 decision table。

原則：

- 小模型只處理 feature vector，不直接吃圖片。
- 模型應可被測試與解釋。
- 不應讓模型取代 rubric，只作為輔助分類。

### 4. 回饋語句庫

Mac mini 可以用本地 LLM 協助產生候選文案，但正式 app 內應使用審過的固定語句。

內容：

- 每種錯誤的簡短說明
- 不同嚴重度的回饋
- 下一次只修一件事的建議
- 鼓勵持續練習但不誇張的語句
- 中文、英文、日文版本

建議格式：

```text
feedback_templates
- issue_code
- severity
- locale
- title
- message
- next_action
```

價值：

- 避免 app 內即時生成文字造成不穩定。
- 保持回饋短、準、可預期。
- 有利於 App Store 版本離線運作。

### 5. Observation 對照特徵

這是第二版後期或第三版才應投入的資產。

Mac mini 可先實驗：

- 輪廓抽取
- 大形邊界框
- 主軸偵測
- 明暗大區塊分割
- 單張照片與手繪結果的對齊
- 多張照片或 Object Capture 的可行性

可嵌入 app 的成果：

- 輕量影像處理參數
- 可接受的拍攝流程
- 參考物件練習資料
- 不依賴雲端的簡化對照流程

原則：

- 只比較大形、比例、角度、位置。
- 不把它做成自由素描評分。

## 開發機限定資產

這些資產可以放在 Mac mini 上，但不一定要進 app。

### 1. 原始練習資料

內容：

- stroke 原始紀錄
- 版本化的分析結果
- 手動標記的錯誤類型
- 自評結果
- 重新分析用的歷史 session

用途：

- 校準 threshold。
- 測試新 rubric。
- 訓練小型分類模型。

### 2. 本地 LLM 產生資料

用途：

- 草擬課程文案。
- 草擬回饋語句。
- 將分析結果改寫成自然語言。
- 產生多語版本初稿。

限制：

- LLM 輸出需要人工審核後才可進 app。
- 不把未審核生成內容直接當教學內容。

### 3. 高成本視覺實驗

可在 Mac mini 研究：

- monocular depth
- photogrammetry
- Object Capture
- 輪廓對齊
- 影像分割

但這些不應成為 MVP 前置條件。

## 先不要做的方向

第一階段不建議投入：

- app 內嵌大型 LLM
- app 內嵌大型 vision-language model
- 以 RAG 作為課程核心
- 以 3D reconstruction 作為 MVP 主路徑
- 雲端 GPU 推論
- 自由素描 AI 評分

原因：

- 成本高。
- 體積大。
- 可控性差。
- 對第一版基礎練習的價值不如 stroke geometry 明確。

## 建議實作順序

1. 建立課程知識庫 schema。
2. 將 MVP 三個練習轉成結構化課程資料。
3. 建立 rubric parameter schema。
4. 先用人工 threshold 支援 MVP 分析。
5. 收集自己的練習 stroke data。
6. 用 Mac mini 批次重跑分析，調整 threshold。
7. 建立 feedback template library。
8. 視資料量再考慮小型分類模型。
9. 第二版後期再研究 observation 對照。

## App Store 版本原則

若未來上架 App Store，核心功能應符合：

- 可離線使用。
- 不依賴 Mac mini。
- 不依賴付費 API。
- 基礎分析全部裝置端完成。
- 進階功能依裝置能力開關。

產品價值不應建立在大型模型本身，而應建立在：

- 課程順序
- 練習摩擦低
- 回饋可靠
- 長期進度追蹤
- 弱項補練
- 經校準的 rubric
