<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora ENGEL CC100 Resistores</title>
    <style>
        :root {
            --primary-color: #0056b3;
            --bg-color: #f4f7f6;
            --card-bg: #ffffff;
            --text-color: #333333;
            --border-color: #cccccc;
            --alert-color: #d9534f;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            line-height: 1.6;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
            background: var(--card-bg);
            padding: 25px;
            border-radius: 8px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        }

        h1, h2 {
            color: var(--primary-color);
            border-bottom: 2px solid var(--primary-color);
            padding-bottom: 10px;
            margin-bottom: 20px;
        }

        .section {
            margin-bottom: 40px;
        }

        .row {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            margin-bottom: 15px;
        }

        .col {
            flex: 1;
            min-width: 200px;
        }

        label {
            display: block;
            font-weight: 600;
            margin-bottom: 5px;
            font-size: 0.9em;
        }

        input[type="number"], select {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            font-size: 1em;
            box-sizing: border-box;
        }

        input[type="range"] {
            width: 100%;
            margin-top: 10px;
        }

        .result-box {
            background-color: #e9ecef;
            padding: 20px;
            border-radius: 8px;
            text-align: center;
            font-size: 1.5em;
            font-weight: bold;
            transition: all 0.3s ease;
            margin-top: 20px;
        }

        .result-details {
            font-size: 0.6em;
            font-weight: normal;
            display: block;
            margin-top: 5px;
            color: #555;
        }

        .volt-relation {
            font-size: 0.85em;
            color: var(--primary-color);
            font-weight: bold;
            margin-top: 5px;
            display: block;
            min-height: 20px;
        }

        /* Animación de sobrecarga / quemado */
        @keyframes burnEffect {
            0% { background-color: #ffcccc; box-shadow: 0 0 10px red; color: #900; }
            50% { background-color: var(--alert-color); box-shadow: 0 0 25px red; color: white; transform: scale(1.02); }
            100% { background-color: #ffcccc; box-shadow: 0 0 10px red; color: #900; }
        }

        .overload {
            animation: burnEffect 0.5s infinite alternate;
            border: 2px solid red;
        }

        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 12px 20px;
            font-size: 1em;
            border-radius: 4px;
            cursor: pointer;
            width: 100%;
            transition: background 0.3s;
        }

        button:hover {
            background-color: #004494;
        }

        .button-secondary {
            background-color: #6c757d;
            margin-top: 10px;
        }

        .button-secondary:hover {
            background-color: #5a6268;
        }

        /* Ajustes para móviles */
        @media (max-width: 600px) {
            .row { flex-direction: column; }
        }
    </style>
</head>
<body>

<div class="container">
    <div class="section">
        <h1>CALCULADORA DE EVALUACIÓN EN VARIACIONES DE POTENCIA</h1>
        <p>Evalúa el impacto de las variaciones de tensión en la resistencia de moldes en máquinas de inyección.</p>
        
        <div class="row">
            <div class="col">
                <label for="moldeBaseVolt">Modelo / Tensión Nominal (V)</label>
                <select id="moldeBaseVolt" onchange="actualizarTensionBase()">
                    <option value="24">24V (Control / Baja tensión)</option>
                    <option value="110">110V</option>
                    <option value="230" selected>230V (Industrial Monofásico)</option>
                    <option value="400">400V (Industrial Trifásico)</option>
                </select>
            </div>
            <div class="col">
                <label for="moldeMaxPower">Límite de Potencia (W) antes de quemarse</label>
                <input type="number" id="moldeMaxPower" value="2000" step="100" oninput="calcularMolde()">
            </div>
        </div>

        <div class="row">
            <div class="col">
                <label for="moldeTension">Tensión de Operación Real: <span id="tensionDisplay">230</span> V</label>
                <input type="range" id="moldeTension" min="0" max="500" value="230" step="1" oninput="calcularMolde()">
            </div>
            <div class="col">
                <label for="moldeResistencia">Resistencia del Molde (Ω)</label>
                <input type="number" id="moldeResistencia" value="30" step="0.1" min="0.1" oninput="calcularMolde()">
            </div>
        </div>

        <div id="moldeResultBox" class="result-box">
            Potencia: <span id="moldePotenciaOut">0</span> W
            <span class="result-details" id="moldeStatusOut">Estado: Operación Normal</span>
        </div>
    </div>

    <div class="section">
        <h2>Calculadora de Ley de Ohm y Potencia</h2>
        <p>Selecciona el tipo de sistema e ingresa al menos <strong>dos valores</strong> para calcular el resto.</p>

        <div class="row">
            <div class="col">
                <label for="ohmSistema">Tipo de Sistema</label>
                <select id="ohmSistema" onchange="recalcularRelacion()">
                    <option value="monofasico" selected>Monofásico</option>
                    <option value="trifasico">Trifásico (Equilibrado)</option>
                </select>
            </div>
        </div>
        
        <div class="row">
            <div class="col">
                <label for="ohmVolt">Tensión (V) en Volts <span style="font-weight: normal; font-size: 0.8em;">(Línea-Línea si es Trifásico)</span></label>
                <input type="number" id="ohmVolt" placeholder="Ej: 400" oninput="recalcularRelacion()">
                <span id="voltRelationDisplay" class="volt-relation"></span>
            </div>
            <div class="col">
                <label for="ohmCurr">Corriente (I) en Amperios</label>
                <input type="number" id="ohmCurr" placeholder="Ej: 10">
            </div>
        </div>

        <div class="row">
            <div class="col">
                <label for="ohmRes">Resistencia Equivalente (R) en Ohmios</label>
                <input type="number" id="ohmRes" placeholder="Ej: 23">
            </div>
            <div class="col">
                <label for="ohmPow">Potencia Activa (P) en Watts</label>
                <input type="number" id="ohmPow" placeholder="Ej: 2300">
            </div>
        </div>

        <div class="row">
            <div class="col">
                <label for="ohmFP">Factor de Potencia (FP) para cálculo en VA</label>
                <input type="number" id="ohmFP" value="1.0" step="0.01" min="0.1" max="1.0">
            </div>
            <div class="col">
                <label for="ohmVA">Potencia Aparente (S) en VA</label>
                <input type="number" id="ohmVA" readonly style="background-color: #eee;">
            </div>
        </div>

        <button onclick="calcularOhm()">Calcular Valores Faltantes</button>
        <button class="button-secondary" onclick="limpiarOhm()">Limpiar Campos</button>
    </div>
</div>

<script>
    // --- LÓGICA PARA LA CALCULADORA DE MOLDE ---
    function actualizarTensionBase() {
        const baseVolt = document.getElementById('moldeBaseVolt').value;
        const tensionSlider = document.getElementById('moldeTension');
        
        tensionSlider.value = baseVolt;
        tensionSlider.max = Math.ceil(baseVolt * 1.5); 
        calcularMolde();
    }

    function calcularMolde() {
        const v = parseFloat(document.getElementById('moldeTension').value);
        const r = parseFloat(document.getElementById('moldeResistencia').value);
        const pMax = parseFloat(document.getElementById('moldeMaxPower').value);
        
        document.getElementById('tensionDisplay').innerText = v;

        if (isNaN(v) || isNaN(r) || r <= 0) return;

        const p = Math.pow(v, 2) / r;
        
        const resultBox = document.getElementById('moldeResultBox');
        const statusOut = document.getElementById('moldeStatusOut');
        
        document.getElementById('moldePotenciaOut').innerText = p.toFixed(2);

        if (p > pMax) {
            resultBox.classList.add('overload');
            statusOut.innerText = "¡ADVERTENCIA: Límite superado! Riesgo de quemar la resistencia.";
        } else {
            resultBox.classList.remove('overload');
            statusOut.innerText = "Estado: Operación Normal (Dentro de los límites)";
        }
    }

    window.onload = function() {
        calcularMolde();
    };

    // --- LÓGICA PARA LA CALCULADORA MÚLTIPLE (1F / 3F) ---
    function recalcularRelacion() {
        const v = parseFloat(document.getElementById('ohmVolt').value);
        const sys = document.getElementById('ohmSistema').value;
        const display = document.getElementById('voltRelationDisplay');

        if (isNaN(v)) {
            display.innerText = "";
            return;
        }

        if (sys === 'trifasico') {
            const v_ln = v / Math.sqrt(3);
            display.innerText = `Equivalencia L-N (Fase-Neutro): ${v_ln.toFixed(2)} V`;
        } else {
            display.innerText = `Sistema Monofásico: V = V L-N`;
        }
    }

    function calcularOhm() {
        let v = parseFloat(document.getElementById('ohmVolt').value);
        let i = parseFloat(document.getElementById('ohmCurr').value);
        let r = parseFloat(document.getElementById('ohmRes').value);
        let p = parseFloat(document.getElementById('ohmPow').value);
        let fp = parseFloat(document.getElementById('ohmFP').value);
        let sys = document.getElementById('ohmSistema').value;

        if (isNaN(fp) || fp <= 0 || fp > 1) { fp = 1; document.getElementById('ohmFP').value = 1; }

        let count = 0;
        if (!isNaN(v)) count++;
        if (!isNaN(i)) count++;
        if (!isNaN(r)) count++;
        if (!isNaN(p)) count++;

        if (count < 2) {
            alert("Por favor, ingresa al menos dos valores para calcular.");
            return;
        }

        // Factor de sistema
        let k = (sys === 'trifasico') ? Math.sqrt(3) : 1;
        let pMult = (sys === 'trifasico') ? 3 : 1; 

        // Despejes matemáticos cruzados
        if (!isNaN(v) && !isNaN(i)) {
            p = k * v * i * fp;
            r = (v / (sys === 'trifasico' ? Math.sqrt(3) : 1)) / i;
        } else if (!isNaN(v) && !isNaN(r)) {
            i = (v / (sys === 'trifasico' ? Math.sqrt(3) : 1)) / r;
            p = k * v * i * fp;
        } else if (!isNaN(v) && !isNaN(p)) {
            i = p / (k * v * fp);
            r = (v / (sys === 'trifasico' ? Math.sqrt(3) : 1)) / i;
        } else if (!isNaN(i) && !isNaN(r)) {
            let v_ln = i * r;
            v = (sys === 'trifasico') ? v_ln * Math.sqrt(3) : v_ln;
            p = k * v * i * fp;
        } else if (!isNaN(i) && !isNaN(p)) {
            v = p / (k * i * fp);
            r = (v / (sys === 'trifasico' ? Math.sqrt(3) : 1)) / i;
        } else if (!isNaN(r) && !isNaN(p)) {
            i = Math.sqrt(p / (pMult * r * fp));
            let v_ln = i * r;
            v = (sys === 'trifasico') ? v_ln * Math.sqrt(3) : v_ln;
        }

        // Asignación de resultados a los campos
        document.getElementById('ohmVolt').value = v.toFixed(2);
        document.getElementById('ohmCurr').value = i.toFixed(2);
        document.getElementById('ohmRes').value = r.toFixed(2);
        document.getElementById('ohmPow').value = p.toFixed(2);
        
        // Cálculo de VA
        let s = p / fp;
        document.getElementById('ohmVA').value = s.toFixed(2);

        // Actualizar la relación de tensión debajo del input
        recalcularRelacion();
    }

    function limpiarOhm() {
        document.getElementById('ohmVolt').value = '';
        document.getElementById('ohmCurr').value = '';
        document.getElementById('ohmRes').value = '';
        document.getElementById('ohmPow').value = '';
        document.getElementById('ohmVA').value = '';
        document.getElementById('ohmFP').value = '1.0';
        document.getElementById('voltRelationDisplay').innerText = '';
    }
</script>

</body>
</html>
