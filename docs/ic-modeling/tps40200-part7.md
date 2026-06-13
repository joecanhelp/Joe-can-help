# [SIMPLIS 建模實戰 07] 終極試煉：模型驗證與系統級 Benchmark

恭喜你！我們已經從零開始，一步步完成了 TPS40200 的數位分身。現在，我們要將這顆 IC 放入一個真實的 Buck Converter 電路中，進行最終的「壓力測試」。

在這一章，我們不只要看波形，更要利用 SIMPLIS 最強大的兩項武器：**POP 分析** 與 **AC 迴路分析**，來證明這個模型的實戰價值。

---

## 🏗️ 搭建測試平台 (Testbench)

為了驗證模型，我們需要搭建一個標準的降壓轉換器環境：
- **功率級**：包含 P-MOSFET、電感 ($10\mu\text{H}$)、電容 ($100\mu\text{F}$) 以及續流二極體。
- **補償網路**：使用我們先前提供的 [Type III 計算機](../tool/type-iii-calculator.md) 算出的 R、C 數值，接在 IC 的 `FB` 與 `COMP` 腳位之間。
- **輸入電壓**：設定為 $12\text{V}$。

---

## ⚡ POP 分析：瞬間進入穩態

在一般的 SPICE 軟體中，你必須跑很長的 Transient 模擬，等待輸出電壓慢慢爬升到穩態（Steady State）。但在 SIMPLIS 中，我們有 **POP (Periodic Operating Point)**。

### 為什麼需要 POP？
POP 能透過數學迭代，直接找到電路在週期性切換下的「平衡點」。
- **設定重點**：在電路中放置一個 **POP Trigger** 元件，並將其連接到我們 IC 內部的 `CLK` 或 `GATE` 訊號。
- **效益**：原本需要跑 $10\text{ms}$ 的模擬，現在只需不到 1 秒就能直接看到穩態波形，極大地提升了開發效率。

---

## 📉 AC 分析：驗證迴路穩定性

這是檢驗 Behavior Model 擬真度最嚴苛的一關。我們要在 `FB` 迴路中注入一個微小的擾動訊號，跑出 **Bode Plot (波德圖)**。

### 驗證指標：
1.  **分壓比精度**：觀察輸出電壓是否精準穩定在 $V_{REF} \times (1 + R_{top}/R_{bot})$。
2.  **穿越頻率 (Crossover Frequency)**：實際跑出來的 $f_c$ 是否與我們在 Type III 計算機中預設的目標一致？
3.  **相位裕度 (Phase Margin)**：
    還記得我們在 Part 4 提到的 **EA 頻寬限制** 嗎？如果當時你用了理想 OP，這裡的相位裕度會顯得很完美；但用了我們建立的受限頻寬模型，你會看到在高頻處相位會更貼近真實 IC 的表現。

---

## 🏆 結語：建模的心法總結

回顧這七章的旅程，我們學習到的不只是如何畫圖，更是一套 **Top-Down (由上而下)** 的建模思維：

1.  **分解 (Decompose)**：從 Datasheet 的方塊圖拆解核心模塊。
2.  **參數化 (Parameterize)**：利用變數讓模型具備高度的彈性。
3.  **非線性處理 (Non-linear)**：透過邏輯門與受控源模擬保護與啟動行為。
4.  **驗證 (Validate)**：從單一模塊測試到最後的系統級 Benchmark。

擁有這套心法後，不論是 TPS40200 還是更複雜的多相位控制器 (Multi-phase Controller)，你都能在 SIMPLIS 中精準地打造出它們的數位分身。

---

## 📚 延伸閱讀與工具
- [👉 Type III 參數快速計算機](../tool/type-iii-calculator.md)
- [👉 SIMPLIS 官方文件：如何優化 POP 收斂](https://www.simplis.com)
