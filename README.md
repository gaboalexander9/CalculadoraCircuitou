<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculadora de Circuito RL</title>
    <style>
        body { background: white; font-family: Arial, sans-serif; margin: 0; padding: 20px; color: #333; line-height: 1.7; }
        h1 { text-align: center; color: #007BFF; font-size: 2.5em; margin: 20px 0; }
        .subtitle { text-align: center; font-size: 1.2em; color: #555; margin-bottom: 40px; }
        .container { max-width: 1000px; margin: 0 auto; }
        .input-group { margin: 25px 0; display: flex; align-items: center; flex-wrap: wrap; gap: 20px; }
        label { font-weight: bold; width: 300px; font-size: 1.1em; }
        input { padding: 12px; width: 220px; border: 2px solid #007BFF; border-radius: 8px; font-size: 1.1em; }
        .preview { padding: 18px; background: #f0f8ff; border-radius: 12px; min-height: 80px; flex: 1; font-size: 1.05em; box-shadow: 0 2px 8px rgba(0,123,255,0.1); }
        button { background: #007BFF; color: white; padding: 16px 50px; border: none; border-radius: 10px; cursor: pointer; font-size: 1.4em; display: block; margin: 40px auto; box-shadow: 0 4px 15px rgba(0,123,255,0.3); }
        button:hover { background: #0056b3; transform: translateY(-2px); }
        #results { margin-top: 60px; display: none; background: #f8fbff; padding: 30px; border-radius: 15px; box-shadow: 0 5px 20px rgba(0,0,0,0.1); }
        .graph { width: 100%; height: 400px; margin: 40px 0; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .formula-box { background: #e3f2fd; padding: 20px; border-left: 6px solid #007BFF; margin: 20px 0; border-radius: 0 10px 10px 0; font-size: 1.1em; }
        .pasos { background: #fff8e1; padding: 25px; border-radius: 12px; margin: 40px 0; border: 2px solid #ffca28; }
        table { width: 100%; border-collapse: collapse; margin: 30px 0; font-size: 1em; }
        th, td { border: 1px solid #90caf9; padding: 12px; text-align: center; }
        th { background: #e3f2fd; font-weight: bold; }
        .banda { display: inline-block; width: 30px; height: 50px; margin: 0 3px; border-radius: 4px; }
        @media (max-width: 768px) { .input-group { flex-direction: column; align-items: stretch; } .graph { height: 300px; } }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
<div class="container">
    <h1>Calculadora de Circuito RL</h1>
    <div class="subtitle">Simulador educativo del decaimiento de corriente en un circuito RL serie<br>
        (Ecuación diferencial de primer orden homogénea)</div>

    <div class="input-group">
        <label>Resistencia R (ohm o kΩ):</label>
        <input type="number" id="R" step="any" placeholder="Ej: 1399" oninput="updateResistor()">
        <div class="preview" id="resistorColors">Ingresa un valor mayor que 0</div>
    </div>

    <div class="input-group">
        <label>Inductancia L (henry, mH o µH):</label>
        <input type="number" id="L" step="any" placeholder="Ej: 49 para 49 mH" oninput="updateInductor()">
        <div class="preview" id="inductorInfo">Ingresa un valor mayor que 0</div>
    </div>

    <div class="input-group">
        <label>Corriente inicial I₀ (A):</label>
        <input type="number" id="I0" step="any" required>
    </div>

    <div class="input-group">
        <label>Tiempo máximo t_max (s):</label>
        <input type="number" id="tmax" step="any" required>
    </div>

    <div class="input-group">
        <label>Paso de tiempo Δt (s):</label>
        <input type="number" id="dt" step="any" required>
    </div>

    <center><button onclick="calcular()">Calcular</button></center>
</div>

<div id="results" class="container">
    <h2 style="color:#007BFF; text-align:center;">Resultados del Circuito RL Serie</h2>
    
    <div id="summary" style="font-size:1.3em; text-align:center; background:#e3f2fd; padding:20px; border-radius:12px;"></div>
    
    <div class="formula-box">
        <strong>Constante de tiempo:</strong> τ = L/R = <span id="tauValue"></span> s<br>
        <strong>Función de corriente:</strong> i(t) = <span id="I0Value"></span> × e<sup>-t/τ</sup> A
    </div>

    <div style="display:flex; flex-wrap:wrap; gap:20px;">
        <div class="graph"><canvas id="graficaI"></canvas>
            <div class="formula-box">i(t) = I₀ × e<sup>-t/τ</sup> → Corriente decrece exponencialmente</div></div>
        <div class="graph"><canvas id="graficaVR"></canvas>
            <div class="formula-box">v_R(t) = R × i(t) → Voltaje en resistencia</div></div>
        <div class="graph"><canvas id="graficaVL"></canvas>
            <div class="formula-box">v_L(t) = -R × i(t) → Voltaje en inductancia (opone el cambio)</div></div>
    </div>

    <h3 style="color:#007BFF;">Tabla de valores</h3>
    <table id="tabla">
        <thead><tr><th>t (s)</th><th>i(t) (A)</th><th>v_R(t) (V)</th><th>v_L(t) (V)</th></tr></thead>
        <tbody></tbody>
    </table>

    <div class="pasos">
        <h3 style="color:#d81b60; margin-top:0;">Desarrollo matemático paso a paso (Memoria de cálculo)</h3>
        <ol>
            <li><strong>Ecuación del circuito (Ley de Kirchhoff):</strong><br>
                v_R + v_L = 0 → R×i + L×di/dt = 0</li>
            <li><strong>Ecuación diferencial:</strong><br>
                L×di/dt + R×i = 0 → di/dt = -(R/L)×i</li>
            <li><strong>Solución general:</strong><br>
                i(t) = K × e<sup>-(R/L)×t</sup></li>
            <li><strong>Condición inicial:</strong> en t = 0 → i(0) = I₀<br>
                I₀ = K × e<sup>0</sup> → K = I₀</li>
            <li><strong>Solución particular:</strong><br>
                <strong>i(t) = I₀ × e<sup>-t/(L/R)</sup> = I₀ × e<sup>-t/τ</sup></strong></li>
            <li><strong>Constante de tiempo:</strong><br>
                τ = L/R = <span id="tauCalc"></span> s</li>
            <li><strong>Voltajes instantáneos:</strong><br>
                v_R(t) = R×i(t) = <span id="vR0"></span> × e<sup>-t/τ</sup> V<br>
                v_L(t) = L×di/dt = -R×i(t) = -<span id="vL0"></span> × e<sup>-t/τ</sup> V</li>
        </ol>
        <p><strong>Conclusión:</strong> La corriente decae exponencialmente con constante de tiempo τ = L/R. En t = 5τ, i(t) ≈ 0 (corriente prácticamente extinguida).</p>
    </div>
</div>

<script>
// Formato chileno: coma y 3 decimales
function formato(num) {
    return num.toFixed(3).replace(".", ",");
}

// Colores reales para resistencias
const colorCodes = {
    0: "#000000", 1: "#8B4513", 2: "#FF0000", 3: "#FFA500", 4: "#FFFF00",
    5: "#008000", 6: "#0000FF", 7: "#EE82EE", 8: "#808080", 9: "#FFFFFF"
};

function updateResistor() {
    let input = document.getElementById("R").value;
    let val = parseFloat(input);
    if (isNaN(val) || val <= 0) { 
        document.getElementById("resistorColors").innerHTML = "Ingresa un valor mayor que 0"; 
        return; 
    }
    if (input.toLowerCase().includes("k")) val *= 1000;

    let valor = val;
    let exp = 0;
    while (valor >= 100) { valor /= 10; exp++; }
    valor = Math.round(valor * 100) / 100;

    let d1 = Math.floor(valor / 10);
    let d2 = Math.floor(valor % 10);
    let mult = exp;

    document.getElementById("resistorColors").innerHTML = `
        <strong>Resistencia = ${formato(val)} Ω</strong><br>
        Bandas: 
        <span class="banda" style="background:${colorCodes[d1]}"></span>
        <span class="banda" style="background:${colorCodes[d2]}"></span>
        <span class="banda" style="background:${colorCodes[mult]}"></span>
        <span class="banda" style="background:gold"></span>
        → ${d1}${d2} × 10<sup>${mult}</sup> Ω ±5%<br>
        <small>Ejemplo visual de resistencia de 4 bandas</small>`;
}

function updateInductor() {
    let val = parseFloat(document.getElementById("L").value);
    if (isNaN(val) || val <= 0) { 
        document.getElementById("inductorInfo").innerHTML = "Ingresa un valor mayor que 0"; 
        return; 
    }
    
    let Lh = val;
    if (document.getElementById("L").value.toLowerCase().includes("m")) Lh /= 1000;
    if (document.getElementById("L").value.includes("µ")) Lh /= 1e6;

    let vueltas = Math.round(300 * Math.pow(Lh, 0.4));
    let diametro = Lh < 0.01 ? "pequeño (5-10 mm)" : Lh < 0.1 ? "mediano (10-30 mm)" : "grande (mayor que 30 mm)";

    document.getElementById("inductorInfo").innerHTML = `
        <strong>Bobina = ${formato(Lh)} H</strong><br>
        Estimación didáctica:<br>
        • Aproximadamente ${vueltas} vueltas de alambre<br>
        • Diámetro aproximado: ${diametro}<br>
        <small>Valores típicos en bobinas reales</small>`;
}

function calcular() {
    let R = parseFloat(document.getElementById("R").value) || 0;
    if (document.getElementById("R").value.toLowerCase().includes("k")) R *= 1000;

    let L = parseFloat(document.getElementById("L").value) || 0;
    if (document.getElementById("L").value.toLowerCase().includes("m")) L /= 1000;
    if (document.getElementById("L").value.includes("µ")) L /= 1e6;

    const I0 = parseFloat(document.getElementById("I0").value);
    const tmax = parseFloat(document.getElementById("tmax").value);
    const dt = parseFloat(document.getElementById("dt").value);

    if (R <= 0 || L <= 0 || !I0 || !tmax || !dt) { 
        alert("Complete todos los campos obligatorios con valores positivos"); 
        return; 
    }

    const tau = L / R;
    const VR0 = I0 * R;

    // Resumen
    document.getElementById("summary").innerHTML = `
        <strong>R = ${formato(R)} Ω  L = ${formato(L)} H  I₀ = ${formato(I0)} A  τ = ${formato(tau)} s</strong>`;

    document.getElementById("tauValue").innerText = formato(tau);
    document.getElementById("I0Value").innerText = formato(I0);
    document.getElementById("tauCalc").innerText = formato(tau);
    document.getElementById("vR0").innerText = formato(VR0);
    document.getElementById("vL0").innerText = formato(VR0);

    // Datos para gráficas y tabla
    const tiempos = [], corrientes = [], vR = [], vL = [];
    for (let t = 0; t <= tmax; t += dt) {
        tiempos.push(t.toFixed(3));
        const i = I0 * Math.exp(-t / tau);
        corrientes.push(i);
        vR.push(i * R);
        vL.push(-i * R);
    }

    // Tabla
    let tbody = "";
    tiempos.forEach((t, i) => {
        tbody += `<tr><td>${t}</td><td>${formato(corrientes[i])}</td><td>${formato(vR[i])}</td><td>${formato(vL[i])}</td></tr>`;
    });
    document.querySelector("#tabla tbody").innerHTML = tbody;

    // Gráficas grandes
    crearGrafica("graficaI", tiempos, corrientes, "Corriente i(t) [A]", "#007BFF");
    crearGrafica("graficaVR", tiempos, vR, "Voltaje en resistencia v_R(t) [V]", "#D32F2F");
    crearGrafica("graficaVL", tiempos, vL, "Voltaje en inductancia v_L(t) [V]", "#D32F2F");

    document.getElementById("results").style.display = "block";
    window.scrollTo(0, document.getElementById("results").offsetTop);
}

function crearGrafica(id, labels, data, titulo, color) {
    new Chart(document.getElementById(id), {
        type: 'line',
        data: { 
            labels: labels, 
            datasets: [{ 
                label: titulo, 
                data: data.map(d => Number(d.toFixed(3))), 
                borderColor: color, 
                backgroundColor: color + '33',
                fill: true,
                tension: 0.4,
                pointRadius: 3
            }] 
        },
        options: { 
            responsive: true,
            maintainAspectRatio: false,
            scales: { 
                x: { title: { display: true, text: 'Tiempo (s)', font: { size: 14 }}},
                y: { title: { display: true, text: titulo.includes('Corriente') ? 'Corriente (A)' : 'Voltaje (V)', font: { size: 14 }}}
            },
            plugins: { legend: { labels: { font: { size: 14 }}} }
        }
    });
}
</script>
</body>
</html>
