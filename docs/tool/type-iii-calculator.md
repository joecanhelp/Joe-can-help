# Type III Calculator
![image.png](https://cdn.jsdelivr.net/gh/joecanhelp/joe-can-help-images@main/images/20260403144255373.png)


<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
  .calculator-container {
    background: var(--md-default-bg-color);
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    margin-bottom: 20px;
  }
  
  .layout-flex {
    display: flex;
    gap: 30px;
    margin-bottom: 20px;
  }
  .column {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 12px;
    background: var(--md-code-bg-color);
    padding: 15px;
    border-radius: 8px;
  }
  .column h4 {
    margin-top: 0;
    margin-bottom: 10px;
    color: var(--md-typeset-color);
    border-bottom: 1px solid var(--md-default-fg-color--lighter);
    padding-bottom: 5px;
  }
  
  .freq-settings {
    display: flex;
    gap: 15px;
    margin-bottom: 20px;
    padding: 15px;
    background: var(--md-code-bg-color);
    border-radius: 6px;
  }
  
  .input-group {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .input-group label { font-weight: bold; width: 80px; }
  .input-group input { 
    flex: 1; padding: 6px; 
    border: 1px solid var(--md-default-fg-color--lighter); 
    border-radius: 4px; background: var(--md-default-bg-color); color: var(--md-default-fg-color);
  }

  .btn-copy {
    background-color: var(--md-primary-fg-color);
    color: var(--md-primary-bg-color);
    border: none;
    padding: 10px;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
    margin-top: 8px;
    transition: opacity 0.2s;
  }
  .btn-copy:hover { opacity: 0.8; }
  
  .results-grid {
    display: grid;
    grid-template-columns: auto auto 1fr;
    align-items: center;
    row-gap: 15px;
    column-gap: 25px;
    background: var(--md-code-bg-color);
    padding: 20px;
    border-radius: 6px;
    margin-bottom: 20px;
  }
  
  .results-grid.show-g2 {
    grid-template-columns: auto auto 1fr 1fr; 
  }
  
  .results-grid > div {
    border-bottom: 1px solid var(--md-default-fg-color--lightest);
    padding-bottom: 15px;
  }
  .results-grid > div.no-border { border-bottom: none; padding-bottom: 0; }
  
  .grid-header {
    font-weight: bold;
    color: var(--md-typeset-color);
    border-bottom: 2px solid var(--md-default-fg-color--lighter) !important;
    padding-bottom: 10px !important;
  }
  
  .g2-col { display: none; }
  .results-grid.show-g2 .g2-col { display: block; }

  .math-fraction {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    vertical-align: middle;
    font-family: serif;
    font-size: 1.1em;
  }
  .math-fraction .num { padding: 0 5px; }
  .math-fraction .den {
    border-top: 1px solid var(--md-default-fg-color);
    padding: 2px 5px 0;
  }
  .math-vars i { font-style: italic; }

  .val-g1 { color: #2196F3; font-weight: bold; font-size: 1.1em; }
  .val-g2 { color: #4CAF50; font-weight: bold; font-size: 1.1em; }
</style>

<div class="calculator-container">
  <h3>1. 參數設定</h3>
  <div class="layout-flex">
    <div class="column">
      <h4>組別一 (基準)</h4>
      <div class="input-group"><label>R1 (kΩ)</label><input type="number" id="R1" value="20" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>R2 (kΩ)</label><input type="number" id="R2" value="20" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>R3 (kΩ)</label><input type="number" id="R3" value="1" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>C1 (pF)</label><input type="number" id="C1" value="2200" step="10" oninput="updatePlot()"></div>
      <div class="input-group"><label>C2 (pF)</label><input type="number" id="C2" value="220" step="10" oninput="updatePlot()"></div>
      <div class="input-group"><label>C3 (pF)</label><input type="number" id="C3" value="47" step="1" oninput="updatePlot()"></div>
    </div>
    <div class="column">
      <h4>組別二 (比較)</h4>
      <div class="input-group"><label>R1 (kΩ)</label><input type="number" id="R1_2" placeholder="-" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>R2 (kΩ)</label><input type="number" id="R2_2" placeholder="-" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>R3 (kΩ)</label><input type="number" id="R3_2" placeholder="-" step="0.1" oninput="updatePlot()"></div>
      <div class="input-group"><label>C1 (pF)</label><input type="number" id="C1_2" placeholder="-" step="10" oninput="updatePlot()"></div>
      <div class="input-group"><label>C2 (pF)</label><input type="number" id="C2_2" placeholder="-" step="10" oninput="updatePlot()"></div>
      <div class="input-group"><label>C3 (pF)</label><input type="number" id="C3_2" placeholder="-" step="1" oninput="updatePlot()"></div>
      <button class="btn-copy" onclick="copyToGroup2()">⬇️ 代入第一組</button>
    </div>
  </div>

  <h3>2. 頻率範圍設定</h3>
  <div class="freq-settings">
    <div class="input-group"><label>最低頻率 (Hz)</label><input type="number" id="fMin" value="100" oninput="updatePlot()"></div>
    <div class="input-group"><label>最高頻率 (Hz)</label><input type="number" id="fMax" value="10000000" oninput="updatePlot()"></div>
  </div>

  <h3>3. 計算結果</h3>
  <div class="results-grid math-vars" id="resultsGrid">
    <div class="grid-header">參數</div>
    <div class="grid-header">公式</div>
    <div class="grid-header" style="color: #2196F3;">基準值 (G1)</div>
    <div class="grid-header g2-col" style="color: #4CAF50;">對照值 (G2)</div>

    <div><strong>100Hz Gain</strong></div>
    <div style="text-align: center;">-</div>
    <div class="val-g1" id="gain100">0 dB</div>
    <div class="val-g2 g2-col" id="gain100_2">0 dB</div>

    <div><strong>Zero 1 (<i>f<sub>z1</sub></i>)</strong></div>
    <div><span class="math-fraction"><span class="num">1</span><span class="den">2π <i>R<sub>2</sub> C<sub>1</sub></i></span></span></div>
    <div class="val-g1" id="fz1">0</div>
    <div class="val-g2 g2-col" id="fz1_2">0</div>

    <div><strong>Zero 2 (<i>f<sub>z2</sub></i>)</strong></div>
    <div><span class="math-fraction"><span class="num">1</span><span class="den">2π (<i>R<sub>1</sub>+R<sub>3</sub></i>) <i>C<sub>2</sub></i></span></span></div>
    <div class="val-g1" id="fz2">0</div>
    <div class="val-g2 g2-col" id="fz2_2">0</div>

    <div><strong>Pole 1 (<i>f<sub>p1</sub></i>)</strong></div>
    <div><span class="math-fraction"><span class="num">1</span><span class="den">2π <i>R<sub>3</sub> C<sub>2</sub></i></span></span></div>
    <div class="val-g1" id="fp1">0</div>
    <div class="val-g2 g2-col" id="fp1_2">0</div>

    <div class="no-border"><strong>Pole 2 (<i>f<sub>p2</sub></i>)</strong></div>
    <div class="no-border"><span class="math-fraction"><span class="num">1</span><span class="den">2π <i>R<sub>2</sub></i> [ <i>C<sub>1</sub> C<sub>3</sub></i> / (<i>C<sub>1</sub>+C<sub>3</sub></i>) ]</span></span></div>
    <div class="val-g1 no-border" id="fp2">0</div>
    <div class="val-g2 g2-col no-border" id="fp2_2">0</div>
  </div>

  <h3>4. Bode Plot</h3>
  <canvas id="bodeChart" width="400" height="200"></canvas>
</div>

<script>
let bodeChart;

function formatTextFrequency(value) {
    if (value >= 1e6) return (value / 1e6).toFixed(2) + ' MHz';
    if (value >= 1e3) return (value / 1e3).toFixed(2) + ' kHz';
    return value.toFixed(2) + ' Hz';
}

function copyToGroup2() {
    const ids = ['R1', 'R2', 'R3', 'C1', 'C2', 'C3'];
    ids.forEach(id => {
        document.getElementById(id + '_2').value = document.getElementById(id).value;
    });
    updatePlot();
}

function updatePlot() {
    const R1 = parseFloat(document.getElementById('R1').value) * 1e3;
    const R2 = parseFloat(document.getElementById('R2').value) * 1e3;
    const R3 = parseFloat(document.getElementById('R3').value) * 1e3;
    const C1 = parseFloat(document.getElementById('C1').value) * 1e-12;
    const C2 = parseFloat(document.getElementById('C2').value) * 1e-12;
    const C3 = parseFloat(document.getElementById('C3').value) * 1e-12;

    const R1_2_raw = document.getElementById('R1_2').value;
    const R2_2_raw = document.getElementById('R2_2').value;
    const R3_2_raw = document.getElementById('R3_2').value;
    const C1_2_raw = document.getElementById('C1_2').value;
    const C2_2_raw = document.getElementById('C2_2').value;
    const C3_2_raw = document.getElementById('C3_2').value;

    const isGroup2Active = R1_2_raw !== '' && R2_2_raw !== '' && R3_2_raw !== '' && 
                           C1_2_raw !== '' && C2_2_raw !== '' && C3_2_raw !== '';

    const fMin = parseFloat(document.getElementById('fMin').value) || 100;
    const fMax = parseFloat(document.getElementById('fMax').value) || 10000000;

    const fz1 = 1 / (2 * Math.PI * R2 * C1);
    const fz2 = 1 / (2 * Math.PI * (R1 + R3) * C2);
    const fp1 = 1 / (2 * Math.PI * R3 * C2);
    const fp2 = 1 / (2 * Math.PI * R2 * ((C1 * C3) / (C1 + C3)));

    document.getElementById('fz1').innerText = formatTextFrequency(fz1);
    document.getElementById('fz2').innerText = formatTextFrequency(fz2);
    document.getElementById('fp1').innerText = formatTextFrequency(fp1);
    document.getElementById('fp2').innerText = formatTextFrequency(fp2);

    const resultsGrid = document.getElementById('resultsGrid');
    
    // 🌟 將變數宣告在 if 區塊外部，確保全域都能存取到
    let fz1_2, fz2_2, fp1_2, fp2_2;

    if (isGroup2Active) {
        const R1_2 = parseFloat(R1_2_raw) * 1e3, R2_2 = parseFloat(R2_2_raw) * 1e3, R3_2 = parseFloat(R3_2_raw) * 1e3;
        const C1_2 = parseFloat(C1_2_raw) * 1e-12, C2_2 = parseFloat(C2_2_raw) * 1e-12, C3_2 = parseFloat(C3_2_raw) * 1e-12;

        fz1_2 = 1 / (2 * Math.PI * R2_2 * C1_2);
        fz2_2 = 1 / (2 * Math.PI * (R1_2 + R3_2) * C2_2);
        fp1_2 = 1 / (2 * Math.PI * R3_2 * C2_2);
        fp2_2 = 1 / (2 * Math.PI * R2_2 * ((C1_2 * C3_2) / (C1_2 + C3_2)));

        document.getElementById('fz1_2').innerText = formatTextFrequency(fz1_2);
        document.getElementById('fz2_2').innerText = formatTextFrequency(fz2_2);
        document.getElementById('fp1_2').innerText = formatTextFrequency(fp1_2);
        document.getElementById('fp2_2').innerText = formatTextFrequency(fp2_2);
        
        resultsGrid.classList.add('show-g2');
    } else {
        resultsGrid.classList.remove('show-g2');
    }

    const frequencies = [];
    const gains = [], phases = [];
    const gains2 = [], phases2 = [];
    
    const logMin = Math.log10(fMin);
    const logMax = Math.log10(fMax);
    const points = 300; 
    const step = (logMax - logMin) / points;

    for (let i = logMin; i <= logMax; i += step) { 
        frequencies.push(Math.pow(10, i)); 
    }

    frequencies.forEach(f => {
        const numGain = Math.sqrt(1 + Math.pow(f/fz1, 2)) * Math.sqrt(1 + Math.pow(f/fz2, 2));
        const denGain = 2 * Math.PI * f * R1 * (C1 + C3) * Math.sqrt(1 + Math.pow(f/fp1, 2)) * Math.sqrt(1 + Math.pow(f/fp2, 2));
        const gain = 20 * Math.log10(numGain / denGain);
        gains.push(gain);

        const phase = 90 + Math.atan(f/fz1)*(180/Math.PI) + Math.atan(f/fz2)*(180/Math.PI) 
                         - Math.atan(f/fp1)*(180/Math.PI) - Math.atan(f/fp2)*(180/Math.PI);
        phases.push(phase);

        if (Math.abs(f - 100) < (100 * Math.pow(10, step) - 100)) {
            document.getElementById('gain100').innerText = gain.toFixed(2) + ' dB';
        }

        if (isGroup2Active) {
            const R1_2 = parseFloat(R1_2_raw) * 1e3, C1_2 = parseFloat(C1_2_raw) * 1e-12, C3_2 = parseFloat(C3_2_raw) * 1e-12;
            const numGain2 = Math.sqrt(1 + Math.pow(f/fz1_2, 2)) * Math.sqrt(1 + Math.pow(f/fz2_2, 2));
            const denGain2 = 2 * Math.PI * f * R1_2 * (C1_2 + C3_2) * Math.sqrt(1 + Math.pow(f/fp1_2, 2)) * Math.sqrt(1 + Math.pow(f/fp2_2, 2));
            const gain2 = 20 * Math.log10(numGain2 / denGain2);
            gains2.push(gain2);

            const phase2 = 90 + Math.atan(f/fz1_2)*(180/Math.PI) + Math.atan(f/fz2_2)*(180/Math.PI) 
                              - Math.atan(f/fp1_2)*(180/Math.PI) - Math.atan(f/fp2_2)*(180/Math.PI);
            phases2.push(phase2);

            if (Math.abs(f - 100) < (100 * Math.pow(10, step) - 100)) {
                document.getElementById('gain100_2').innerText = gain2.toFixed(2) + ' dB';
            }
        }
    });

    const ctx = document.getElementById('bodeChart').getContext('2d');
    if (bodeChart) { bodeChart.destroy(); }
    
    const chartDatasets = [
        { label: 'Gain (G1)', data: gains, borderColor: '#2196F3', yAxisID: 'y', tension: 0.1, pointRadius: 0, borderWidth: 2 },
        { label: 'Phase (G1)', data: phases, borderColor: '#FF9800', yAxisID: 'y1', tension: 0.1, pointRadius: 0, borderWidth: 2 }
    ];

    if (isGroup2Active) {
        chartDatasets.push(
            { label: 'Gain (G2)', data: gains2, borderColor: '#4CAF50', yAxisID: 'y', tension: 0.1, pointRadius: 0, borderWidth: 2, borderDash: [5, 5] },
            { label: 'Phase (G2)', data: phases2, borderColor: '#E91E63', yAxisID: 'y1', tension: 0.1, pointRadius: 0, borderWidth: 2, borderDash: [5, 5] }
        );
    }
    
    bodeChart = new Chart(ctx, {
        type: 'line',
        data: { labels: frequencies, datasets: chartDatasets },
        options: {
            responsive: true,
            interaction: { mode: 'index', intersect: false },
            scales: {
                x: { 
                    type: 'logarithmic', title: { display: true, text: 'Frequency' },
                    grid: { 
                        color: function(context) {
                            if (context.tick && context.tick.value) {
                                let log10 = Math.log10(context.tick.value);
                                if (Math.abs(Math.round(log10) - log10) < 1e-5) return 'rgba(128, 128, 128, 0.4)';
                            }
                            return 'rgba(128, 128, 128, 0.1)';
                        }
                    },
                    ticks: { 
                        autoSkip: false, maxTicksLimit: 999,
                        callback: function(value) {
                            let log10 = Math.log10(value);
                            if (Math.abs(Math.round(log10) - log10) < 1e-5) {
                                let val = Math.round(value);
                                if (val >= 1e6) return (val / 1e6) + ' MHz';
                                if (val >= 1e3) return (val / 1e3) + ' kHz';
                                return val + ' Hz';
                            }
                            return ''; 
                        }
                    }
                },
                y: { type: 'linear', display: true, position: 'left', title: { display: true, text: 'Gain (dB)' }, grid: { color: 'rgba(128, 128, 128, 0.3)' } },
                y1: { type: 'linear', display: true, position: 'right', title: { display: true, text: 'Phase (deg)' }, grid: { drawOnChartArea: false } }
            }
        }
    });
}

updatePlot();
</script>