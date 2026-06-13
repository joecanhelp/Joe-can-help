# [SIMPLIS 建模實戰 04] 控制中樞：受限頻寬的誤差放大器 (E/A) 與 VREF

在電源控制器的世界裡，**誤差放大器 (Error Amplifier, E/A)** 是決定系統動態響應的「大腦」。

許多初學者在建模時，會直接放一個理想的 OP-AMP 或是增益極大的電壓受控電壓源 (VCVS)。但在真實的 IC 開發中，EA 的**直流增益 (DC Gain)** 與 **增益頻寬積 (GBW)** 是有限的。如果建模時忽略了這些非理想特性，你的模擬結果可能會過於樂觀，導致實際電路在補償網路的高頻處發生震盪。

---

## 📍 基準電壓 (VREF) 的設定

TPS40200 的基準電壓非常單純，為 $700\text{mV}$。
在 SIMPLIS 中，我們通常會在子電路內部建立一個理想電壓源 `VREF`。

> **💡 資深工程師的提醒：**
> 在進行 **Super hot lot**（極限參數測試）的模擬時，我們會將 VREF 設定為變數（如 `{VREF_val}`），以便同時驗證在 $686\text{mV}$ 或 $714\text{mV}$ ($\pm 2\%$) 下，系統的輸出精度是否符合規範。

---

## 🛠️ 誤差放大器建模：兩種路徑的對決

在 SIMPLIS 中，建立 EA 模型路徑通常有兩種：

### 方法 A：使用內建 `Operational Amplifier (Level 2)`
這是您平常用最順手的方法，優點是快、簡潔。
*   **優點**：設定簡單，直接輸入 `Open Loop Gain`、`GBW` 與 `Output Limits`。
*   **缺點**：它是個「黑盒子」，當你需要模擬某些特殊的非線性行為（如特定的輸入偏置電流或複雜的鉗位邏輯）時，調整空間較小。

### 方法 B：VCVS + RC 濾波器 (White Box Approach)
這是我在開發控制 IC 行為模型時更推薦的做法。我們利用一個電壓受控電壓源 (VCVS) 產生增益，再搭配一組 RC 電路來人為製造出**極點 (Pole)**。

*   **優點**：結構完全透明。你可以精確控制每一級的轉折頻率，甚至能在中間插入二極體來模擬更真實的 **Output Clamping (輸出鉗位)** 行為。
*   **計算公式**：
    若設定 $R = 1\text{M}\Omega$，則 $C$ 的數值由預期的頻寬決定：
    $$f_{pole} = \frac{GBW}{A_{OL}}$$
    $$C = \frac{1}{2\pi \cdot R \cdot f_{pole}}$$

---

## 📉 互動實驗：EA 的頻寬對相位的影響

下方的互動圖表展示了 EA 的 **Open Loop 響應**。
試著調整 **GBW**，觀察在較高頻率（如 $100\text{kHz}$ 到 $1\text{MHz}$）時，相位裕度 (Phase Margin) 是如何因為受限頻寬而開始崩塌的。

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  .ea-container { background: var(--md-default-bg-color); padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 20px; }
  .input-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; margin-bottom: 20px; background: var(--md-code-bg-color); padding: 15px; border-radius: 6px; }
  .slider-group label { display: flex; justify-content: space-between; font-weight: bold; margin-bottom: 8px; }
  .slider-group input[type="range"] { width: 100%; cursor: pointer; }
</style>

<div class="ea-container">
  <div class="input-grid">
    <div class="slider-group">
      <label><span>DC Gain (dB):</span> <span id="valGain" style="color: #2196F3;">80 dB</span></label>
      <input type="range" id="gain" min="40" max="120" step="10" value="80" oninput="updateBode()">
    </div>
    <div class="slider-group">
      <label><span>GBW (MHz):</span> <span id="valGbw" style="color: #4CAF50;">5 MHz</span></label>
      <input type="range" id="gbw" min="0.1" max="20" step="0.1" value="5" oninput="updateBode()">
    </div>
  </div>
  <canvas id="bodeChart" width="400" height="250"></canvas>
</div>

<script>
let bodeChart;
function updateBode() {
    const gainDb = parseFloat(document.getElementById('gain').value);
    const gbwMhz = parseFloat(document.getElementById('gbw').value);
    const A0 = Math.pow(10, gainDb / 20);
    const GBW = gbwMhz * 1e6;
    const fp = GBW / A0;

    document.getElementById('valGain').innerText = gainDb + ' dB';
    document.getElementById('valGbw').innerText = gbwMhz + ' MHz';

    const freq = [];
    const mag = [];
    const phase = [];

    // 生成對數座標的頻率點
    for (let i = 1; i <= 7; i += 0.1) {
        let f = Math.pow(10, i);
        freq.push(f);
        
        // 增益計算 (1st Order Pole)
        let ratio = f / fp;
        let gain = gainDb - 20 * Math.log10(Math.sqrt(1 + ratio * ratio));
        mag.push(gain);
        
        // 相位計算
        let p = -Math.atan(ratio) * (180 / Math.PI);
        phase.push(p);
    }

    const ctx = document.getElementById('bodeChart').getContext('2d');
    if (bodeChart) { bodeChart.destroy(); }
    
    bodeChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: freq,
            datasets: [
                { 
                    label: 'Gain (dB)', 
                    data: mag, 
                    borderColor: '#2196F3', 
                    yAxisID: 'y', 
                    pointRadius: 0, 
                    borderWidth: 2,
                    tension: 0.1
                },
                { 
                    label: 'Phase (Deg)', 
                    data: phase, 
                    borderColor: '#F44336', 
                    yAxisID: 'y1', 
                    pointRadius: 0, 
                    borderWidth: 2,
                    tension: 0.1
                }
            ]
        },
        options: {
            responsive: true,
            interaction: { mode: 'index', intersect: false },
            scales: {
                x: { 
                    type: 'logarithmic', 
                    title: { display: true, text: 'Frequency (Hz)' },
                    ticks: {
                        callback: function(value) {
                            return value.toExponential();
                        }
                    }
                },
                y: { 
                    title: { display: true, text: 'Magnitude (dB)' }, 
                    min: -20, 
                    max: 120 
                },
                y1: { 
                    position: 'right', 
                    title: { display: true, text: 'Phase (Degree)' }, 
                    min: -90, 
                    max: 0,
                    grid: { drawOnChartArea: false }
                }
            }
        }
    });
}
updateBode();
</script>

---

## 💻 SIMPLIS 實作：VCVS + RC 結構

如果你想嘗試「白盒子」建模，可以按照以下步驟：

1.  **第一級 (Gain Stage)**：放置一個 VCVS (E1)，增益設定為 `{A0}`。
2.  **第二級 (Pole Stage)**：在 E1 的輸出端接一個電阻 $R_1 = 1\text{M}\Omega$ 與電容 $C_1 = \{C_{val}\}$ 到地。
    *   在 F11 中定義：`.var C_val = {1 / (2 * 3.14159 * 1meg * (GBW / A0))}`
3.  **第三級 (Buffer & Clamp)**：使用一個理想 OP-AMP 作為 Unity Gain Buffer，並在其輸出端串接二極體或使用 SIMPLIS 內建的 `Limiter` 元件，將輸出範圍限制在 $0.5\text{V} \sim V_{DD}-1.5\text{V}$ 之間。

> **為什麼要限制輸出？**
> 誤差放大器的輸出 (`COMP`) 如果衝到天際，PWM 就會一直維持 $100\%$ Duty，這在軟啟動或是過流保護時會造成嚴重的系統問題。

---

## 📥 接下來...

我們已經搞定了心跳 (Oscillator) 與大腦 (EA)。下一篇，我們將進入最精彩的部分：**Soft-Start (軟啟動)** 與 **OCP (過流保護)**。這兩個模塊將展示如何利用邏輯門與電流源，模擬出 IC 在異常狀態下的行為邏輯！
