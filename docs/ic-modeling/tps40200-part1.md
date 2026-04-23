# [SIMPLIS 建模實戰 01] 拆解 TPS40200：從 Datasheet 到模型架構

在 IC 建模的世界裡，我們最怕的就是一頭栽進電路圖裡盲目連線。一個優秀的 Behavior Model，必須建立在對 IC 方塊圖 (Block Diagram) 的透徹理解之上。

在這個系列中，我們將以 TI 的經典降壓控制器 **TPS40200** 為例，一步步帶你用 SIMPLIS 打造出能精準模擬其行為的數位分身。

---

## 🔍 為什麼選 TPS40200？

TPS40200 是一款非常適合新手入門的 Voltage-Mode Buck Controller。它只有 8 根腳位，卻五臟俱全，包含了所有電源控制器最核心的機制。

更重要的是，它採用**電壓模式控制**，這表示我們需要為它設計外部的 **Type III 補償網路**（如果你還不知道怎麼設計，可以參考我們之前準備好的 [👉 Type III 參數計算機](../tool/type-iii-calculator.md)！）。
![image.png](https://cdn.jsdelivr.net/gh/joecanhelp/joe-can-help-images@main/images/20260405165511904.png)


---

## 🧩 腳位解析與建模任務拆解 (Top-Down Approach)

要建立 Behavior Model，我們的第一步是「拆解」。我們不需要畫出 IC 內部的每一顆電晶體，而是要把 8 根腳位歸納成 **4 個核心積木 (Sub-circuits)**：

| 腳位名稱               | 功能描述          | 對應的 SIMPLIS 建模積木 (Sub-circuit)                                   |
| :----------------- | :------------ | :--------------------------------------------------------------- |
| **VDD, GND**       | 電源輸入與地        | **電源管理模塊**：建立 UVLO (欠壓鎖定) 邏輯，確保電壓足夠才啟動 IC。                       |
| **RC**             | 振盪器頻率設定       | **時脈與 Ramp 產生器**：透過充放電行為產生 Clock 訊號與 PWM 比較用的 Ramp 波形。           |
| **FB, COMP**       | 誤差放大器反相輸入與輸出  | **誤差放大器 (Error Amplifier)**：建立具有高增益、受限於頻寬的理想運算放大器，並處理 VREF 基準電壓。 |
| **SS, ISNS, GATE** | 軟啟動、電流偵測、閘極驅動 | **PWM 與保護邏輯**：整合 PWM 比較器、過流保護 (OCP) 邏輯，以及最終推動 MOSFET 的 Driver。   |

![image.png](https://cdn.jsdelivr.net/gh/joecanhelp/joe-can-help-images@main/images/20260405171721955.png)

---

## 🚀 接下來的挑戰

透過上面的表格，我們已經把一個複雜的 IC 拆成了 4 個獨立的小任務。這種 **Top-Down (由上而下)** 的思維，是建構複雜 SIMPLIS 模型的心法。

在下一篇文章中，我們將從電路的「心跳」開始，教你如何利用 SIMPLIS 內建元件，打造出能精準響應外部 `RC` 腳位參數的**參數化振盪器 (Oscillator)**！

## 📚 參考資料 
* [Texas Instruments TPS40200 官方網頁與 Datasheet](https://www.ti.com/product/TPS40200)