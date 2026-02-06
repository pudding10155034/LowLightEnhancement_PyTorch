# 新手快速導覽：LowLightEnhancement_PyTorch

這份指南給第一次接觸這個專案的人，幫你在最短時間搞懂「專案在做什麼、怎麼跑、下一步學什麼」。

---

## 1) 這個專案在做什麼？

這是一個「低光影像增強」的整合型專案：
- 同時包含 **傳統演算法**（LIME、Retinex、頻域濾波）
- 也包含 **深度學習模型**（RetinexNet、DRBN、EnlightenGAN、Zero-DCE）
- 支援完整流程：**訓練 → 推論 → 客觀評估（PSNR/SSIM/BRISQUE/PI）→ 視覺化比較**

---

## 2) 專案資料夾要怎麼看？

先記住 5 個最重要的目錄：

- `scripts/`：入口。你平常要執行的訓練/推論/評估都在這裡。
- `networks/`：模型架構定義。
- `losses/`：訓練用的損失函數。
- `utils/`：資料集讀取、曲線套用、後處理等工具函式。
- `traditional/`：非深度學習方法（可當 baseline）。

簡單心法：
> 「先看 scripts，遇到看不懂再追到 networks / losses / utils。」

---

## 3) 一個模型從訓練到評估的路徑

以 Zero-DCE 為例：

1. `python -m scripts.train_zero_dce` 進行訓練（會在 `checkpoints/ZeroDCE/` 存模型）
2. `python -m scripts.infer_zero_dce` 針對驗證圖推論（輸出到 `results/ZeroDCE/`）
3. `python -m scripts.evaluate_all_models` 計算與 GT 的客觀指標
4. `python -m scripts.visualize_comparison` 產生多模型對照圖

同理可替換成其他模型腳本（`train_drbn`、`infer_retinexnet` 等）。

---

## 4) 新手最常踩的坑

- **一定要在專案根目錄執行**，並使用 `python -m scripts.xxx`。
- `data/Raw/low`, `data/Raw/high`, `data/Raw/low_val`, `data/Raw/high_val` 沒放好，訓練/評估會直接失敗。
- 推論前若 `checkpoints/<Model>/` 沒有對應 `.pth`，模型無法載入。
- `evaluate_all_models` 需要各模型先有輸出圖，否則會出現缺圖或跳過。

---

## 5) 建議學習順序（很實用）

### 第一步：先跑傳統方法建立直覺
先執行：
- `python -m scripts.infer_lime`
- `python -m scripts.infer_retinex_traditional`
- `python -m scripts.infer_freqfilter`

目的：快速看懂「亮度提升、對比、偏色、雜訊」之間的 trade-off。

### 第二步：看 Zero-DCE（最容易理解）
- 模型本體小、推論邏輯清楚（先預測曲線參數 A，再套曲線）
- 能理解「不是直接輸出圖片，而是輸出增強函數參數」的概念

### 第三步：看 DRBN / RetinexNet（中階）
- 開始接觸較複雜的損失組合與訓練設計
- 理解為何要混合 L1 / SSIM / 色彩一致性等目標

### 第四步：看 EnlightenGAN（進階）
- 有 Generator + Discriminator + 多種損失，訓練策略較複雜
- 適合理解 GAN 在影像增強上的實務折衷

---

## 6) 想繼續深入，下一步重點是什麼？

- 先固定一個模型（建議 Zero-DCE），只改一個超參數做 ablation（例如 learning rate 或 loss 權重）
- 觀察客觀分數（PSNR/SSIM/BRISQUE/PI）與主觀視覺是否一致
- 嘗試把後處理（gamma/gain）獨立成可配置參數，量化它對結果的影響
- 最後再做跨模型比較，整理每個模型最適用的場景（偏暗程度、雜訊情況、色偏程度）

---

## 7) 30 分鐘上手清單

1. 安裝套件 `pip install -r requirements.txt`
2. 放好 `data/` 與（如要推論）`checkpoints/`
3. 跑一個推論腳本確認環境
4. 跑 `evaluate_all_models` 看數字
5. 跑 `visualize_comparison` 看對照圖
6. 選 Zero-DCE 作為第一個閱讀模型，照著 `scripts -> networks -> losses -> utils` 追程式

完成以上流程，你就已經超過「只會執行」的階段，開始進入「能解釋和修改」的階段。
