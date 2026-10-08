# SWP08：色卡反應挑戰

已完成可部署的靜態網頁遊戲與實際訓練的圖片分類模型。**GitHub Pages 尚未發布，需要將網頁上傳至自己的 GitHub。**

## 最簡單的使用方式

1. 下載單檔版 `SWP08-color-game.html`，重新命名為 `index.html`。
2. 在新的 GitHub repository 上傳這一個檔案即可。樣式、遊戲程式、模型架構、分類標籤與模型權重都已內嵌，不需要 vendor 或模型資料夾。
3. 到 **Settings → Pages**，Source 選 **Deploy from a branch**，Branch 選 **main**、資料夾選 **/(root)**，按 **Save**。
4. 等部署完成，透過 Pages 的 **Visit site** 開啟 HTTPS 網址。
5. 開啟攝影機並允許權限，測試紅、藍色卡，再按「開始挑戰」。

TensorFlow.js 與 Teachable Machine 辨識套件會從 CDN 載入，因此仍需要網路。攝影機需透過安全連線使用，請以 GitHub Pages 的 HTTPS 網址或 localhost 開啟。部署教學：https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site

本機測試可在解壓縮的資料夾執行 `python3 -m http.server 8000`，再以 Chrome 開啟 `http://localhost:8000`。

## 作業要求對照

| 作業要求 | 實作與狀態 |
| --- | --- |
| 使用 Teachable Machine，至少訓練兩類，說明操作 | 已使用 Google 官方 `@teachablemachine/image` 套件訓練 Red、Blue、Neutral 三類。遊戲頁面附操作說明。訓練方式為官方 API，並非已完成 Teachable Machine 網站上的訓練操作。 |
| 顯示攝影機／麥克風、辨識結果與信心分數 | 頁面顯示攝影機的中央方形影像、目前類別、最高信心，以及三類各自的機率進度條。 |
| 辨識結果觸發畫面變化 | 正確且穩定的辨識觸發加分、文字回饋、目標色塊變色與成功動畫。 |
| 部署 GitHub Pages | 純靜態單檔版已準備完成；實際發布仍需執行上方步驟。 |

## 遊戲操作

- 準備紅、藍兩張色卡。也可用另一支手機開啟此網頁，在「開啟測試色卡」中顯示紅／藍色塊。
- 本機開啟攝影機後，將色卡靠近鏡頭，填滿顯示的中央方形畫面，避免反光。
- 按「開始挑戰」後，目標會顯示紅色或藍色。
- 模型辨識到目標顏色，信心至少 **85%** 並持續 **0.5 秒**，獲得 **1 分**。
- 每次得分後目標會切換成另一個顏色；不能一直拿著同一張色卡連續計分。
- 每局 **30 秒**。可提前結束或再玩一次。最高分只存於自己的瀏覽器。
- 按「關閉攝影機」會停止鏡頭；切換到其他分頁、離開頁面也會停止攝影機並結束本局。
- 攝影機影像在本機瀏覽器進行辨識，本網頁不會將影像上傳至伺服器。

## 訓練方法與測試

訓練使用 Google 官方 Teachable Machine 圖片套件 **0.8.5**，透過 `createTeachable()`、`addExample()`、`train()` 與 `save()` 完成。Teachable Machine 的網站訓練頁面在本次環境中無回應，因此採官方套件 API 路線；沒有將未完成的網站操作當作完成的訓練。

模型以官方 MobileNet V2、alpha 0.35、輸入大小 224×224 作為特徵擷取器，使用 Teachable Machine 的分類頭訓練。遊戲實際呼叫 `tmImage.loadFromFiles()` 與 `model.predict()`，由分類機率控制遊戲。

| 類別名稱 | 用途 | 生成圖片總數 | 提供訓練 API 的數量 | 獨立測試數量 | 獨立測試正確數 |
| --- | --- | ---: | ---: | ---: | ---: |
| Red | 紅色色卡 | 80 | 60 | 20 | 20 |
| Blue | 藍色色卡 | 80 | 60 | 20 | 20 |
| Neutral | 灰／綠／黃色與非目標背景 | 80 | 60 | 20 | 19 |

資料是程式生成的色卡圖片，變化包含色塊大小、位置、色調、背景與模糊。隨機種子為 807。每類第 1～60 張交給訓練 API，第 61～80 張保留作獨立測試；官方訓練 API 另外從前 60 張中切出內部驗證資料。

訓練設定：50 epochs、batch size 16、learning rate 0.001、dense units 100。三類合計的獨立測試為 **59／60 正確，98.3%**。紅色測試卡機率約 **95.8%**、藍色測試卡約 **99.3%**、灰色卡的 Neutral 機率約 **99.99%**。

**這些結果只代表本次生成色卡資料，不代表真實攝影機環境也有 98.3% 的準確率。**模型未以人臉、手勢或多種真實背景照片訓練，請以大面積色卡進行遊戲。若要提升現場表現，可加入實際攝影機拍攝的色卡與沒有色卡的背景樣本，再訓練。

官方套件說明：https://github.com/googlecreativelab/teachablemachine-community/tree/master/libraries/image

## 如果老師要求用 Teachable Machine 網站介面訓練

附上完整訓練圖片，可以在自己的瀏覽器完成相同類別的網站訓練：

1. 開啟 https://teachablemachine.withgoogle.com/train/image ，使用標準圖片專案。
2. 建立並精確命名 **Red**、**Blue**、**Neutral** 三個類別。
3. 在每個類別點 **Upload**，上傳相對應 `training` 子資料夾中第 001～060 張圖片；第 061～080 張留作測試。
4. 按 **Train Model**，完成後以紅、藍測試卡檢查分類。
5. 按 **Export Model → TensorFlow.js → Download**，取得 `model.json`、`metadata.json`、`weights.bin`。保留訓練完成畫面的截圖。
6. 如需以自己的模型重新內嵌到單檔網頁，可使用這三個匯出檔案更新 HTML 中的 `embedded-model` JSON：model 存模型 JSON，metadata 存 metadata JSON，weights 為 weights.bin 的 Base64，weightName 與模型 manifest 的檔名保持一致。

目前提供的單檔網頁已含這次官方套件訓練完成的模型，直接部署即可執行。若課程明確要求網站訓練紀錄，請依上述流程另行完成並提交紀錄。

## 完整 ZIP 內容

- `index.html`：已內嵌模型的單檔網頁，部署只需要這個檔案。
- `model/`：實際匯出的模型 JSON、權重與標籤 metadata，供檢查與重用。
- `training/`：三個類別各 80 張生成圖片。
- `red-card.png`、`blue-card.png`、`neutral-card.png`：測試色卡，可由另一台裝置開啟給攝影機辨識。
- `model-validation.json`：逐 epoch 紀錄、獨立測試結果與測試卡機率。
- `work/`：網頁樣板、遊戲 JavaScript、訓練腳本及所需的官方模型與套件副本。訓練腳本使用 Node.js 與 `@napi-rs/canvas`，重跑前需安裝該套件。
- `README.md`：本說明。

## 已完成的檢查與限制

- JavaScript 語法、HTML ID 唯一性、單檔模型內容、模型權重長度已檢查。
- 已測試 85% 門檻、0.5 秒穩定時間、不穩定輸入重置、交替目標、避免重複得分、30 秒到期與重新開始。
- 實際模型已完成訓練並測試；匯出後再次透過官方 Teachable Machine loader 載入，分類標籤與紅色測試卡機率正確。
- 尚未完成你裝置上的攝影機與瀏覽器視覺驗收，也尚未發布 GitHub Pages。部署後請測試鏡頭授權、紅藍辨識、計分與關閉攝影機。
