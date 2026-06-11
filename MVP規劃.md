# Claw Draw MVP 規劃

## 目標

建立一個只給自己使用的本地繪畫基礎練習系統：

- Mac mini 跑本地 web service。
- iPad Air + Apple Pencil 透過 Safari 開啟畫布練習。
- 系統記錄每一筆 stroke，而不是只保存最後截圖。
- 第一版先用幾何規則做基礎檢查，不把 AI 當正式評分者。
- 之後再讓龍蝦 Telegram bot 只作為入口、提醒、統計查詢。

MVP 的核心不是做完整 DrawABox，而是先做一個可每天使用、可累積資料、可自動檢查最基礎問題的練習環境。

## 設計原則

1. iPad 使用流程要最短

   使用者打開網址就能畫，不需要安裝 app、不需要登入、不需要先調設定。

2. 第一版不做自由繪畫評論

   系統只檢查可觀察、可幾何計算的項目，例如線條偏離、重描、閉合度、抖動、完成數量。

3. 不直接複製 DrawABox 教材

   可以參考基礎教學結構，但練習文字、rubric、介面說明都自行重寫。

4. AI 放在後續版本

   MVP 不依賴 vision model。等 stroke 資料與練習記錄累積後，再加入 AI 輔助檢查。

5. 核心分析邏輯用 Rust

   幾何分析、rubric 判斷、資料結構放在 Rust core crate，讓型別與測試約束住邏輯。

## 技術架構

建議 repo 結構：

```text
claw_draw/
  crates/
    claw_draw_core/      # Rust: stroke geometry, metrics, rubric checks
    claw_draw_server/    # Rust axum: local API, static web hosting, SQLite
  web/                   # TypeScript + Canvas UI
  data/
    claw_draw.sqlite3    # 本地資料庫，加入 .gitignore
  docs/
    OER_SOURCES.md       # 開放教材來源與授權
```

建議技術：

- Frontend: TypeScript + HTML Canvas + Pointer Events
- Backend: Rust + axum
- Core analysis: Rust crate
- DB: SQLite
- Local deploy: launchd
- iPad access: `http://<mac-mini-ip>:8765/draw`

## MVP 使用流程

第一版可以先不接 Telegram：

1. Mac mini 啟動 `claw_draw_server`。
2. iPad Safari 開啟 `http://<mac-mini-ip>:8765/draw`。
3. 選擇練習：直線、圓、橢圓。
4. Apple Pencil 在畫布上畫。
5. 按 Submit。
6. 系統保存 stroke 與分析結果。
7. 頁面顯示簡短回饋。

第二階段再讓 `aka_no_claw` 加很薄的入口：

```text
/draw web
```

龍蝦只回本地網址，不承擔畫布與分析邏輯。

## MVP 練習模組

### 1. 兩點直線

目的：

- 練習一筆完成。
- 練習手眼協調與落點控制。
- 建立不要重描的習慣。

畫布行為：

- 系統顯示多組起點與終點。
- 使用者用 Apple Pencil 從起點畫到終點。
- 每條線原則上只畫一筆。

可計算指標：

- 起點距離目標點的誤差。
- 終點距離目標點的誤差。
- 線條中途偏離理想直線的程度。
- stroke 是否中途停頓過多。
- 同一目標是否被多筆重描。

回饋格式：

```text
完成度：12 / 12 條
落點：可接受
線條穩定度：中
重描：偏多
下一次目標：寧可畫歪，也不要補描。
```

### 2. 圓形

目的：

- 練習閉合形狀。
- 練習連續運筆。
- 練習大小控制。

畫布行為：

- 系統顯示不同大小的淡色參考圓或空白練習格。
- 使用者畫圓。
- 第一版不要求完美，只要求連續、閉合、少重描。

可計算指標：

- 是否閉合。
- 圓形長寬比例。
- stroke 平滑度。
- 是否重描。
- 是否填滿要求數量。

回饋格式：

```text
完成度：8 / 8 個
閉合度：良好
形狀穩定度：中
重描：少
下一次目標：保持同一速度畫完整個圓。
```

### 3. 橢圓

目的：

- 練習透視形體會用到的橢圓。
- 練習長短軸與角度控制。
- 為後續圓柱練習做準備。

畫布行為：

- 系統給出水平、斜 30 度、斜 60 度的橢圓練習格。
- 使用者畫橢圓。
- 第一版只看閉合、比例、角度，不判斷高階透視。

可計算指標：

- 是否閉合。
- 長短軸比例。
- 主軸角度是否接近目標。
- stroke 平滑度。
- 是否重描。

回饋格式：

```text
完成度：9 / 9 個
閉合度：可接受
角度控制：右上斜向較不穩
重描：中
下一次目標：先 ghost 兩次，再一筆畫完。
```

## 非 MVP 範圍

第一版不做：

- AI vision 評分。
- 盒子透視正式通關判定。
- 課程影片。
- 社群 critique。
- 付費或公開多人帳號系統。
- Procreate / GoodNotes 檔案匯入。
- Telegram 圖片上傳評分。

這些功能可以等基礎 stroke recording 與練習資料穩定後再規劃。

## 資料模型草案

MVP 只需要保存本地練習歷史。

```text
practice_sessions
- id
- exercise_type
- started_at
- submitted_at
- canvas_width
- canvas_height
- result_summary_json

strokes
- id
- session_id
- stroke_index
- tool
- color
- width
- started_at
- ended_at

stroke_points
- id
- stroke_id
- point_index
- x
- y
- pressure
- timestamp_ms

exercise_targets
- id
- session_id
- target_index
- target_type
- target_json
```

`result_summary_json` 第一版可保存：

- completion_count
- expected_count
- start_error_avg
- end_error_avg
- path_deviation_avg
- closure_error_avg
- wobble_score
- redraw_score
- feedback_text

## Rubric 原則

MVP rubric 必須很窄，每個練習只檢查少數項目。

兩點直線只檢查：

- 是否完成指定數量。
- 是否接近起點與終點。
- 是否偏離理想直線過多。
- 是否重描。

圓形只檢查：

- 是否完成指定數量。
- 是否閉合。
- 是否嚴重變形。
- 是否重描。

橢圓只檢查：

- 是否完成指定數量。
- 是否閉合。
- 主軸角度是否大致符合。
- 是否重描。

MVP 不輸出分數，只輸出狀態與下一次目標。

## 開放教材來源

MVP 可參考下列 OER 的課程順序與術語，但不直接搬原文或圖片：

- Drawing is Seeing, Open Oregon Pressbooks, CC BY-NC-SA 4.0  
  https://openoregon.pressbooks.pub/drawing/

- Drawing Basics: Introduction to Observational Drawing, College of Lemoore OER, CC BY-SA 4.0  
  https://lemoorecollege.edu/oer/documents/2024-drawing-basics-art-005a-oer-textbook.pdf

- Foundation Drawing for Art 1100, University of Nebraska Pressbooks, CC BY 4.0  
  https://pressbooks.nebraska.edu/1100foundations/

授權策略：

- MVP 文案全部自行重寫。
- 若後續引用教材觀念或術語，保留 attribution。
- 不使用教材圖片、學生作品、頁面截圖作為產品素材。

## 驗收標準

MVP 完成時應該能做到：

1. Mac mini 可啟動本地 server。
2. iPad Safari 可以開啟畫布頁。
3. Apple Pencil stroke 能被完整記錄。
4. 可以完成兩點直線、圓形、橢圓三種練習。
5. Submit 後資料寫入 SQLite。
6. 頁面顯示一段簡短、非 AI 的幾何分析回饋。
7. 重新整理後仍能看到最近練習紀錄。
8. 不需要外部付費 API。

## 建議實作順序

1. 建立 Rust workspace 與 `claw_draw_core`。
2. 實作 stroke / point / exercise target 型別。
3. 實作兩點直線的分析函式與 unit tests。
4. 建立 TypeScript Canvas 頁，先只記錄 stroke。
5. 建立 Rust axum server，提供靜態頁與 submit API。
6. 加 SQLite schema 與 session 保存。
7. 加圓形與橢圓分析。
8. 加簡單 history 頁。
9. 寫 launchd 安裝腳本。
10. 再回頭規劃 `aka_no_claw` 的 `/draw web` 入口。
