<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Calculadora UTI Completa</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: #f4f4f4;
      margin: 0;
      padding: 20px;
      color: #333;
    }

    h1, h2 {
      text-align: center;
      color: #0a74da;
      margin-top: 30px;
    }

    .section {
      background: white;
      border-radius: 10px;
      padding: 20px;
      margin-top: 20px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      max-width: 700px;
      margin-left: auto;
      margin-right: auto;
    }

    label {
      display: block;
      margin-top: 10px;
      font-weight: 500;
    }

    input, select, button {
      width: 100%;
      padding: 10px;
      margin-top: 5px;
      margin-bottom: 15px;
      border-radius: 5px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }

    button {
      background-color: #0a74da;
      color: white;
      border: none;
      cursor: pointer;
    }

    button:hover {
      background-color: #065bb7;
    }

    .resultado, pre {
      background: #eef4fb;
      border-left: 5px solid #0a74da;
      padding: 15px;
      border-radius: 5px;
      margin-top: 10px;
      white-space: pre-wrap;
    }

    .alerta {
      color: red;
      font-weight: bold;
    }

    .referencia {
      font-style: italic;
      color: #555;
      margin-top: 5px;
    }

    #reiniciar, #imprimir {
      margin-top: 30px;
      background-color: #28a745;
    }

    #reiniciar:hover {
      background-color: #218838;
    }

    #imprimir {
      background-color: #6c757d;
    }

    #imprimir:hover {
      background-color: #5a6268;
    }

    @media (max-width: 700px) {
      body {
        padding: 10px;
      }
    }
  </style>
</head>
<body>

  <h1>Calculadora UTI + Gasometria</h1>

  <div class="section">
    <h2>Analisador de Gasometria</h2>
    <label>pH:</label><input type="number" step="0.01" id="ph">
    <label>PaCO₂ (mmHg):</label><input type="number" id="paco2_gas">
    <label>PaO₂ (mmHg):</label><input type="number" id="pao2_gas">
    <label>HCO₃⁻ (mEq/L):</label><input type="number" id="hco3">
    <label>BE:</label><input type="number" id="be">
    <label>SatO₂ (%):</label><input type="number" id="sato2">
    <button onclick="interpretarGasometria()">Interpretar</button>
    <pre id="resultado_gasometria"></pre>
  </div>

  <div class="section">
    <h2>1. Volume Corrente Ideal</h2>
    <label>Sexo:</label><select id="sexo"><option value="M">Masculino</option><option value="F">Feminino</option></select>
    <label>Altura (cm):</label><input type="number" id="altura">
    <button onclick="calcularVC()">Calcular</button>
    <div class="resultado" id="resultadoVC"></div>
    <div class="referencia">Referência: Pinsp até 40.</div>
  </div>

  <div class="section">
    <h2>2. Relação PaO2/FiO2</h2>
    <label>PaO2:</label><input type="number" id="pao2_fio">
    <label>FiO2 (%):</label><input type="number" id="fio2">
    <button onclick="calcularPaO2FiO2()">Calcular</button>
    <div class="resultado" id="resultadoPaO2FiO2"></div>
  </div>

  <div class="section">
    <h2>3. Avaliação da PEEP</h2>
    <label>PAM:</label><input type="number" id="pam">
    <button onclick="avaliarPEEP()">Avaliar</button>
    <div class="resultado" id="resultadoPEEP"></div>
  </div>

  <div class="section">
    <h2>4. Avaliação de FiO2</h2>
    <label>PaO₂:</label><input type="number" id="pao2_avalia">
    <button onclick="avaliarFiO2()">Avaliar</button>
    <div class="resultado" id="resultadoFiO2"></div>
  </div>

  <div class="section">
    <h2>5. Avaliação de PaCO₂</h2>
    <label>PaCO₂:</label><input type="number" id="paco2_avalia">
    <button onclick="avaliarPaCO2()">Avaliar</button>
    <div class="resultado" id="resultadoPaCO2"></div>
  </div>

  <div class="section">
    <h2>6. Mecânica da Ventilação</h2>
    <label>FR do paciente:</label><input type="number" id="fr_paciente">
    <label>FR do ventilador:</label><input type="number" id="fr_vm">
    <label>Volume Corrente (ml):</label><input type="number" id="vc_mec">
    <label>Pplatô:</label><input type="number" id="pplato">
    <label>PEEP:</label><input type="number" id="peep_mec">
    <label>Ppico:</label><input type="number" id="ppico">
    <label>Fluxo (L/min):</label><input type="number" id="fluxo">
    <button onclick="calcularMecanica()">Calcular</button>
    <div class="resultado" id="resultadoCest"></div>
    <div class="resultado" id="resultadoRVA"></div>
    <div class="resultado" id="resultadoDP"></div>
  </div>

  <div class="section">
    <h2>7. Teste de Respiração Espontânea (TRE)</h2>
    <label>FR:</label><input type="number" id="fr_tre">
    <label>Volume Corrente (ml):</label><input type="number" id="vc_tre">
    <button onclick="calcularTRE()">Calcular</button>
    <div class="resultado" id="resultadoTRE"></div>
  </div>

  <div class="section" style="text-align: center;">
    <button id="reiniciar" onclick="location.reload()">Reiniciar Dados</button>
    <button id="imprimir" onclick="window.print()">Imprimir Página</button>
  </div>

  <script>
    // Função utilitária para truncar em duas casas decimais sem arredondar
    function truncarDuasCasas(numero) {
      return Math.floor(numero * 100) / 100;
    }

    // Gasometria
    function interpretarGasometria() {
      const ph = parseFloat(document.getElementById("ph").value);
      const paco2 = parseFloat(document.getElementById("paco2_gas").value);
      const pao2 = parseFloat(document.getElementById("pao2_gas").value);
      const hco3 = parseFloat(document.getElementById("hco3").value);
      const be = parseFloat(document.getElementById("be").value);
      const sato2 = parseFloat(document.getElementById("sato2").value);

      let status_ph = ph < 7.35 ? "Acidemia" : ph > 7.45 ? "Alcalemia" : "pH normal";
      let status_paco2 = paco2 > 45 ? "acidose respiratória" : paco2 < 35 ? "alcalose respiratória" : "PaCO₂ normal";
      let status_hco3 = hco3 < 22 ? "acidose metabólica" : hco3 > 26 ? "alcalose metabólica" : "HCO₃⁻ normal";
      let status_pao2 = pao2 < 80 ? "com hipoxemia" : pao2 > 100 ? "com hiperoxemia" : "com oxigenação normal";

      let interpretacao = "";
      if (status_ph === "Acidemia") {
        if (status_paco2.includes("acidose") && status_hco3.includes("acidose")) interpretacao = `${status_ph} por acidose mista ${status_pao2}.`;
        else if (status_paco2.includes("acidose")) interpretacao = `${status_ph} por acidose respiratória ${status_pao2}.`;
        else if (status_hco3.includes("acidose")) interpretacao = `${status_ph} por acidose metabólica ${status_pao2}.`;
        else interpretacao = `${status_ph} com causa indefinida ${status_pao2}.`;
      } else if (status_ph === "Alcalemia") {
        if (status_paco2.includes("alcalose") && status_hco3.includes("alcalose")) interpretacao = `${status_ph} por alcalose mista ${status_pao2}.`;
        else if (status_paco2.includes("alcalose")) interpretacao = `${status_ph} por alcalose respiratória ${status_pao2}.`;
        else if (status_hco3.includes("alcalose")) interpretacao = `${status_ph} por alcalose metabólica ${status_pao2}.`;
        else interpretacao = `${status_ph} com causa indeterminada ${status_pao2}.`;
      } else {
        interpretacao = `${status_paco2} e ${status_hco3} (${status_ph}) ${status_pao2}.`;
      }

      document.getElementById("resultado_gasometria").textContent =
        `====== Resultado da Gasometria ======\n` +
        `pH: ${ph}\nPaCO₂: ${paco2} mmHg\nPaO₂: ${pao2} mmHg\nHCO₃⁻: ${hco3} mEq/L\nBE: ${be}\nSatO₂: ${sato2}%\n\n` +
        `Resultado: ${interpretacao}`;
    }

    // Volume Corrente Ideal
    function calcularVC() {
      const sexo = document.getElementById('sexo').value;
      const altura = parseFloat(document.getElementById('altura').value);
      const pesoIdeal = sexo === 'M' ? 50 + 0.91 * (altura - 152.4) : 45.5 + 0.91 * (altura - 152.4);
      const vc6 = (pesoIdeal * 6).toFixed(1);
      const vc8 = (pesoIdeal * 8).toFixed(1);
      document.getElementById('resultadoVC').innerHTML = `Peso ideal: ${pesoIdeal.toFixed(1)} kg<br>VC ideal: ${vc6} - ${vc8} ml`;
    }

    let pao2fio2Global = null;

    // PaO2/FiO2
    function calcularPaO2FiO2() {
      const pao2 = parseFloat(document.getElementById('pao2_fio').value);
      const fio2 = parseFloat(document.getElementById('fio2').value);
      const relacao = (pao2 / (fio2 / 100)).toFixed(2);
      pao2fio2Global = relacao;
      document.getElementById('resultadoPaO2FiO2').innerHTML = `<strong>${relacao}</strong>`;
    }

    // Avaliação PEEP
    function avaliarPEEP() {
      const pam = parseFloat(document.getElementById('pam').value);
      const r = document.getElementById('resultadoPEEP');
      if (!pao2fio2Global) { r.innerHTML = '<span class="alerta">Calcule PaO₂/FiO₂ primeiro</span>'; return; }
      if (pam <= 65) r.innerHTML = 'PAM ≤ 65: Não aumentar PEEP.';
      else if (pao2fio2Global >= 300) r.innerHTML = 'PaO₂/FiO₂ ≥ 300: Não precisa aumentar PEEP.';
      else if (pao2fio2Global >= 200) r.innerHTML = 'PaO₂/FiO₂ entre 200-300: Aumentar PEEP é opcional, cuidado com PAM.';
      else r.innerHTML = '<span class="alerta">PaO₂/FiO₂ < 200: Aumentar PEEP,cuidado PAM.</span>';
    }

    // Avaliação FiO2
    function avaliarFiO2() {
      const p = parseFloat(document.getElementById('pao2_avalia').value);
      const r = document.getElementById('resultadoFiO2');
      if (p < 60) r.innerHTML = 'PaO₂ < 60: Não aumentar FiO₂, obaervar SatO2.';
      else if (p < 80) r.innerHTML = '<span class="alerta">PaO₂ < 80: Deve aumentar FiO₂, observar SatO2.</span>';
      else if (p > 100) r.innerHTML = '<span class="alerta">PaO₂ > 100: Deve diminuir FiO₂, observar SatO2.</span>';
      else r.innerHTML = 'FiO₂ adequada, observar SatO2.';
    }

    // Avaliação PaCO2
    function avaliarPaCO2() {
      const p = parseFloat(document.getElementById('paco2_avalia').value);
      const r = document.getElementById('resultadoPaCO2');
      if (p > 45) r.innerHTML = '<span class="alerta">PaCO₂ > 45: Aumentar VC e FR, observar vc ideal e DP.</span>';
      else if (p < 35) r.innerHTML = '<span class="alerta">PaCO₂ < 35: Diminuir VC e FR, observar vc ideal e DP.</span>';
      else r.innerHTML = 'PaCO₂ adequada.';
    }

    // Mecânica da Ventilação com RVA atualizado
    function calcularMecanica() {
      const fr1 = parseFloat(document.getElementById('fr_paciente').value);
      const fr2 = parseFloat(document.getElementById('fr_vm').value);
      const vc = parseFloat(document.getElementById('vc_mec').value);
      const pplato = parseFloat(document.getElementById('pplato').value);
      const peep = parseFloat(document.getElementById('peep_mec').value);
      const ppico = parseFloat(document.getElementById('ppico').value);
      const fluxo = parseFloat(document.getElementById('fluxo').value);
      
      if (fr1 !== fr2) {
        document.getElementById('resultadoCest').innerText = 'FR ≠ VM: não calcular.';
        return;
      }

      // Cest
      const cest = (vc / (pplato - peep)).toFixed(3);
      document.getElementById('resultadoCest').innerHTML = `Cest: <span class="${cest < 40 ? 'alerta' : ''}">${cest}</span> ml/cmH₂O`;

      // RVA com truncamento
      const fluxoConvertido = fluxo / 60;
      const fluxoTruncado = truncarDuasCasas(fluxoConvertido);
      const deltaP = ppico - pplato;
      const rva = (deltaP / fluxoTruncado).toFixed(3);
      document.getElementById('resultadoRVA').innerHTML =
        `Fluxo/60: ${fluxoTruncado.toFixed(3)} L/s<br>` +
        `Ppico - Pplatô: ${deltaP} cmH₂O<br>` +
        `RVA: ${rva} cmH₂O/L/s`;

      // Drive Pressure
      const dp = (pplato - peep).toFixed(3);
      document.getElementById('resultadoDP').innerHTML = `Drive Pressure: <span class=\"${dp > 15 ? 'alerta' : ''}\">${dp}</span> cmH₂O`;
    }

    // TRE
    function calcularTRE() {
      const fr = parseFloat(document.getElementById('fr_tre').value);
      const vc = parseFloat(document.getElementById('vc_tre').value);
      const tobin = (fr / (vc / 1000)).toFixed(3);
      document.getElementById('resultadoTRE').innerHTML = tobin > 105
        ? `<span class="alerta">Tobin: ${tobin} - Não pode realizar desmame.</span>`
        : `Tobin: ${tobin} - Pode realizar desmame.`;
    }
  </script>

</body>
</html>
