<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Calculadora 5G NR Avançada</title>
  <style>
    body {
        font-family: Arial, sans-serif;
        background-color: #E8F1FF; /* azul claro */
        margin: 0;
        padding: 20px;
    }

    .container {
        max-width: 600px;
        margin: auto;
        background: #FFFFFF;
        border-radius: 10px;
        padding: 20px;
        box-shadow: 0 0 15px rgba(26, 115, 232, 0.2);
    }

    h2 {
        text-align: center;
        color: #0F3266; /* azul escuro */
    }

    label {
        font-weight: bold;
        color: #0F3266;
    }

    input, select {
        width: 100%;
        padding: 10px;
        margin: 8px 0 15px;
        border: 1px solid #1A73E8;
        border-radius: 6px;
        background: #FFFFFF;
    }

    input:focus, select:focus {
        border-color: #0F3266;
        outline: none;
        box-shadow: 0 0 6px rgba(26, 115, 232, 0.4);
    }

    button {
        width: 100%;
        background-color: #1A73E8; /* azul principal */
        color: white;
        padding: 12px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-size: 16px;
        font-weight: bold;
    }

    button:hover {
        background-color: #0F3266; /* azul mais escuro */
    }

    .resultado {
        margin-top: 20px;
        padding: 15px;
        background: #E8F1FF;
        border-left: 4px solid #1A73E8;
        border-radius: 6px;
        color: #0F3266;
        font-weight: bold;
        text-align: center;
    }
</style>
</head>
<body>
  <div class="container">
    <h2>Calculadora 5G NR (Throughput & Link Budget)</h2>

    <label for="tipo">Escolha o tipo de cálculo:</label>
    <select id="tipo">
      <option value="throughput">Throughput</option>
      <option value="linkbudget">Link Budget</option>
    </select>

    <div id="throughput-form">
      <label>Modo de rede:</label>
      <select id="modo">
        <option value="FDD">FDD</option>
        <option value="TDD">TDD</option>
      </select>

      <label>Número de portadoras (CA):</label>
      <input id="numCA" type="number" min="1" value="1">

      <label>Camadas MIMO:</label>
      <input id="mimo" type="number" min="1" value="1">

      <label>Largura de banda (MHz):</label>
      <input id="bw" type="number" min="0">

      <label>Sub-portadora (SCS) (kHz):</label>
      <select id="scs">
        <option value="15">15</option>
        <option value="30">30</option>
        <option value="60">60</option>
        <option value="120">120</option>
      </select>

      <label>Modulação (bits por símbolo):</label>
      <select id="mod">
        <option value="2">QPSK (2)</option>
        <option value="4">16QAM (4)</option>
        <option value="6">64QAM (6)</option>
        <option value="8">256QAM (8)</option>
        <option value="10">1024QAM (10)</option>
      </select>

      <label>Code rate (eficiência):</label>
      <input id="coderate" type="number" step="0.01" min="0" max="1" value="0.5">

      <label>Overhead (OH) (fração 0–1):</label>
      <input id="oh" type="number" step="0.01" min="0" max="1" value="0.2">
    </div>

    <div id="link-form" style="display:none;">
      <label>Potência de transmissão (dBm):</label>
      <input id="ptx" type="number" value="30">

      <label>Ganho TX (dBi):</label>
      <input id="gtx" type="number" value="0">

      <label>Perda TX (cabos, dB):</label>
      <input id="ltx" type="number" value="0">

      <label>Ganho RX (dBi):</label>
      <input id="grx" type="number" value="0">

      <label>Perda RX (cabos, dB):</label>
      <input id="lrx" type="number" value="0">

      <label>Distância Tx-Rx (m):</label>
      <input id="dist" type="number" value="100">

      <label>Frequência (GHz):</label>
      <input id="freq" type="number" step="0.1" value="3.5">

      <label>Noise figure (dB):</label>
      <input id="nf" type="number" value="5">

      <label>SINR alvo (dB):</label>
      <input id="sinr" type="number" value="0">

      <label>Largura de banda para ruído (MHz):</label>
      <input id="bw_noise" type="number" value="20">
    </div>

    <button onclick="calcular()">Calcular</button>

    <div class="resultado" id="resultado"></div>
  </div>

  <script>
    const tipoSelect = document.getElementById('tipo');
    const throughputForm = document.getElementById('throughput-form');
    const linkForm = document.getElementById('link-form');
    const resultadoDiv = document.getElementById('resultado');

    tipoSelect.addEventListener('change', () => {
      if (tipoSelect.value === 'throughput') {
        throughputForm.style.display = 'block';
        linkForm.style.display = 'none';
      } else {
        throughputForm.style.display = 'none';
        linkForm.style.display = 'block';
      }
      resultadoDiv.innerHTML = '';
    });

    function calcular() {
      resultadoDiv.innerHTML = '';
      if (tipoSelect.value === 'throughput') {
        calcularThroughput();
      } else {
        calcularLinkBudget();
      }
    }

    function calcularThroughput() {
      const modo = document.getElementById('modo').value;
      const numCA = parseFloat(document.getElementById('numCA').value);
      const mimo = parseFloat(document.getElementById('mimo').value);
      const bw = parseFloat(document.getElementById('bw').value) * 1e6; // converter para Hz
      const scs = parseFloat(document.getElementById('scs').value) * 1e3; // Hz
      const mod = parseFloat(document.getElementById('mod').value);
      const coderate = parseFloat(document.getElementById('coderate').value);
      const oh = parseFloat(document.getElementById('oh').value);

      // Estimativa simplificada:
      // calcula símbolos por segundo: assume que numa portadora, o número de sub-portadoras * símbolos por segundo
      // Essa parte é simplificada; modelo real é mais complexo (RB, slots, numerologia).
      const simbolosPorSeg = bw / scs; // simplificação grosseira
      const bitsPorSeg = simbolosPorSeg * mod * coderate * mimo * (1 - oh) * numCA;

      const throughputMbps = bitsPorSeg / (1e6);

      resultadoDiv.innerHTML = `Throughput estimado: <b>${throughputMbps.toFixed(2)} Mbps</b>`;
    }

    function calcularLinkBudget() {
      const ptx = parseFloat(document.getElementById('ptx').value);
      const gtx = parseFloat(document.getElementById('gtx').value);
      const ltx = parseFloat(document.getElementById('ltx').value);
      const grx = parseFloat(document.getElementById('grx').value);
      const lrx = parseFloat(document.getElementById('lrx').value);
      const dist = parseFloat(document.getElementById('dist').value);
      const freq = parseFloat(document.getElementById('freq').value);
      const nf = parseFloat(document.getElementById('nf').value);
      const sinr = parseFloat(document.getElementById('sinr').value);
      const bw_noise = parseFloat(document.getElementById('bw_noise').value) * 1e6;

      // Calcular path loss pelo modelo FSPL (simplificado)
      // FSPL (dB) = 20 log10(d) + 20 log10(f) + 32.44, d em km, f em MHz
      const d_km = dist / 1000;
      const f_MHz = freq * 1e3;
      const fspl = 20 * Math.log10(d_km) + 20 * Math.log10(f_MHz) + 32.44;

      // Potência recebida:
      const prx = ptx + gtx - ltx + grx - lrx - fspl;

      // Ruído térmico: N = kT B => em dBm = -174 dBm/Hz + 10 log10(B)
      const noiseFloor = -174 + 10 * Math.log10(bw_noise);

      // Sensibilidade:
      const sensibilidade = noiseFloor + nf + sinr;

      const margin = prx - sensibilidade;

      resultadoDiv.innerHTML = `
        <b>Link Budget:</b><br>
        Potência recebida (pRx): ${prx.toFixed(2)} dBm<br>
        Ruído térmico: ${noiseFloor.toFixed(2)} dBm<br>
        Sensibilidade: ${sensibilidade.toFixed(2)} dBm<br>
        Margem de Link (pRx − Sens): ${margin.toFixed(2)} dB<br>
        ${(margin >= 0) ? '<span style="color:green">Link OK (Pass)</span>' : '<span style="color:red">Link Fail</span>'}
      `;
    }
  </script>
</body>
</html>
