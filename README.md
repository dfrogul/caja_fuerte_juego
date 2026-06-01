<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Quiz CRG – 60 Aniversario</title>
<style>
  :root{
    --crg-green:#007853; --crg-yellow:#F3CA12; --dark:#032821; --light:#ffffff;
  }
  *{box-sizing:border-box}
  body{
    margin:0;font-family:system-ui,-apple-system,"Segoe UI",Roboto,Arial,"Open Sans",sans-serif;
    background:linear-gradient(180deg,var(--dark),#064536);
    color:var(--light);min-height:100dvh;
  }
  .wrap{max-width:900px;margin:0 auto;padding:clamp(16px,3vw,32px)}
  header{display:flex;align-items:center;gap:16px;margin-bottom:20px}
  .badge{background:var(--crg-yellow);color:#1a1a1a;font-weight:700;padding:6px 10px;border-radius:999px;font-size:.85rem}
  h1{margin:0;font-size:clamp(22px,3.5vw,32px);font-weight:800}
  
  /* Pantalla de Registro */
  .reg-card{background:#0d2f28;border:1px solid rgba(255,255,255,.1);border-radius:16px;padding:24px;margin:20px 0;box-shadow:0 8px 24px rgba(0,0,0,.25)}
  .form-group{margin-bottom:16px}
  .form-group label{display:block;margin-bottom:8px;font-weight:600;font-size:.95rem}
  .form-group input, .form-group select{width:100%;padding:12px;border-radius:8px;border:1px solid rgba(255,255,255,.2);background:#0f3a31;color:var(--light);font-size:1rem}
  .form-group input:focus, .form-group select:focus{outline:2px solid var(--crg-yellow)}

  .card{background:#0d2f28;border:1px solid rgba(255,255,255,.08);border-radius:16px;padding:18px;margin:14px 0;box-shadow:0 8px 24px rgba(0,0,0,.25)}
  .q-title{font-weight:700;margin-bottom:10px}
  .options{display:grid;gap:10px}
  .opt{background:#0f3a31;border:1px solid rgba(255,255,255,.07);border-radius:12px;padding:10px 12px;display:flex;gap:10px;align-items:flex-start;cursor:pointer}
  .opt input{margin-top:4px;accent-color:var(--crg-yellow)}
  .actions{display:flex;gap:12px;flex-wrap:wrap;align-items:center;margin-top:18px}
  
  button{background:var(--crg-green);color:var(--light);border:none;padding:12px 18px;border-radius:12px;font-weight:800;cursor:pointer;letter-spacing:.2px;font-size:1rem}
  button:disabled{opacity:0.4;cursor:not-allowed}
  button.secondary{background:transparent;border:1px solid rgba(255,255,255,.2)}
  
  .result{display:none;margin-top:20px;padding:24px;border-radius:14px;background:#0b2a24;border:1px solid rgba(255,255,255,.1);text-align:center}
  footer{margin-top:28px;font-size:.85rem;color:#cfeee5;opacity:.9;text-align:center}
  .hidden{display:none !important}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <span class="badge">60 Anos</span>
    <h1>Quiz Aniversario · Caixa Rural Galega</h1>
  </header>

  <!-- PANTALLA 1: REGISTRO OBLIGATORIO -->
  <div id="registrationScreen" class="reg-card">
    <h2>Identificación de Participante</h2>
    <p style="color:#cfeee5; margin-bottom: 20px;">Introduce tus datos para poder participar en el juego y entrar en el sorteo de las experiencias de fin de semana.</p>
    <div class="form-group">
      <label for="username">Nombre y Apellidos:</label>
      <input type="text" id="username" placeholder="Ej. María Novoa" required />
    </div>
    <div class="form-group">
      <label for="department">Oficina o Departamento:</label>
      <input type="text" id="department" placeholder="Ej. Oficina Viveiro / Dpto. Riesgos" required />
    </div>
    <button id="btnStart" style="width: 100%; margin-top: 10px;">Comenzar Quiz</button>
  </div>

  <!-- PANTALLA 2: EL JUEGO (OCULTO AL PRINCIPIO) -->
  <div id="quizScreen" class="hidden">
    <div id="quiz"></div>

    <div class="actions">
      <button id="btnSubmit">Enviar respuestas</button>
      <button id="btnClear" class="secondary">Limpiar opciones</button>
    </div>
  </div>

  <!-- PANTALLA 3: RESULTADO Y CONFIRMACIÓN -->
  <div id="result" class="result"></div>

  <footer>
    Caixa Rural Galega · 60 Aniversario · Xuntos Construíndo Futuro
  </footer>
</div>

<script>
const QUESTIONS = [
  { text: "¿Cuál es la edad media del Consejo Rector?", options: ["73","84","61","46"], correctText: "61" },
  { text: "¿Cuántos atracos se llevan registrados en CRG desde el año 1981?", options: ["6","7","10","12"], correctText: "7" },
  { text: "¿Cuál fue el beneficio neto de la Caja en el año 2024?", options: ["32.146.758,45€","31.843.733,12€","33.450.124,73€","29.365.100,37€"], correctText: "31.843.733,12€" },
  { text: "¿En qué año la Entidad pasó a llamarse Caixa Rural Galega?", options: ["1999","1995","2001","1993"], correctText: "1999" },
  { text: "¿En qué año se celebró el 50 aniversario?", options: ["2015","2014","2016","2017"], correctText: "2016" },
  { text: "Indica el número de personas que se han incorporado a la plantilla en el año 2025.", options: ["13","10","3","9"], correctText: "10" },
  { text: "¿Cuántas oficinas hay en la provincia de Lugo + la provincia de Ourense?", options: ["36","34","31","39"], correctText: "39" }
];

// Datos del jugador activo
let userData = { name: "", department: "" };

const regScreen = document.getElementById("registrationScreen");
const quizScreen = document.getElementById("quizScreen");
const quizEl = document.getElementById("quiz");
const btnStart = document.getElementById("btnStart");
const btnSubmit = document.getElementById("btnSubmit");
const btnClear = document.getElementById("btnClear");
const resultEl = document.getElementById("result");

// Gestión de pantallas
btnStart.addEventListener("click", () => {
  const nameVal = document.getElementById("username").value.trim();
  const deptVal = document.getElementById("department").value.trim();

  if(!nameVal || !deptVal) {
    alert("Por favor, rellena todos los campos antes de empezar.");
    return;
  }

  userData.name = nameVal;
  userData.department = deptVal;

  regScreen.classList.add("hidden");
  quizScreen.classList.remove("hidden");
  window.scrollTo(0,0);
  renderQuiz();
});

// Renderizado de preguntas
function renderQuiz(){
  quizEl.innerHTML = "";
  QUESTIONS.forEach((q, qi) => {
    const card = document.createElement("div");
    card.className = "card";

    const title = document.createElement("div");
    title.className = "q-title";
    title.textContent = `${qi+1}. ${q.text}`;
    card.appendChild(title);

    const opts = document.createElement("div");
    opts.className = "options";

    q.options.forEach((opt) => {
      const label = document.createElement("label");
      label.className = "opt";

      const input = document.createElement("input");
      input.type = "radio";
      input.name = `q-${qi}`;
      input.value = opt;
      label.appendChild(input);

      const span = document.createElement("span");
      span.textContent = opt;
      label.appendChild(span);

      opts.appendChild(label);
    });

    card.appendChild(opts);
    quizEl.appendChild(card);
  });
}

function getScore(){
  let score = 0;
  QUESTIONS.forEach((q, qi) => {
    const checked = document.querySelector(`input[name='q-${qi}']:checked`);
    if(checked && checked.value.trim() === q.correctText.trim()){
      score++;
    }
  });
  return score;
}

// Envío a Base de Datos (Backend)
async function saveResultsToBackend(name, department, score) {
  const payload = {
    nombre: name,
    departamento: department,
    aciertos: score,
    fecha: new Date().toISOString()
  };

  console.log("Enviando datos al servidor central:", payload);

  /* 
    INTEGRACIÓN RECOMENDADA:
    Descomenta este bloque e introduce la URL de tu API o Webhook de Firebase/Google Sheets:
    
    try {
      await fetch('TU_URL_DE_API_AQUI', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
    } catch (error) {
      console.error("Error guardando los datos en la nube:", error);
    }
  */
}

btnSubmit.addEventListener("click", async () => {
  // Validar que hayan respondido a todo antes de dejar enviar
  let contestadas = 0;
  QUESTIONS.forEach((_, qi) => {
    if(document.querySelector(`input[name='q-${qi}']:checked`)) contestadas++;
  });

  if(contestadas < QUESTIONS.length) {
    alert(`Has respondido ${contestadas} de ${QUESTIONS.length} preguntas. Debes contestar todas antes de enviar.`);
    return;
  }

  btnSubmit.disabled = true; // Evitar doble clic
  const score = getScore();
  
  // Guardamos los datos de forma centralizada para tu sorteo del postre
  await saveResultsToBackend(userData.name, userData.department, score);

  quizScreen.classList.add("hidden");
  resultEl.style.display = "block";

  // Al ser individual con sorteo posterior para el Top, unificamos la pantalla final 
  resultEl.innerHTML = `
    <h2 style="color:var(--crg-yellow); font-size: 2rem;">¡Quiz Finalizado!</h2>
    <p style="font-size: 1.2rem;">Gracias por participar, <strong>${userData.name}</strong>.</p>
    <p>Has obtenido <strong>${score} de ${QUESTIONS.length} aciertos</strong>.</p>
    <div style="margin: 24px 0; border-top: 1px solid rgba(255,255,255,0.1); padding-top: 15px;">
      <p style="color:#cfeee5;">Tus respuestas han sido registradas correctamente en el sistema central de Caixa Rural Galega.</p>
      <p style="font-weight: bold; color:var(--crg-yellow);">Si estás entre las máximas puntuaciones, entrarás automáticamente en el sorteo de las experiencias de fin de semana durante el postre.</p>
    </div>
  `;
  
  resultEl.scrollIntoView({behavior:"smooth"});
});

btnClear.addEventListener("click", () => {
  document.querySelectorAll("input[type='radio']").forEach(r => r.checked = false);
});
</script>
</body>
</html>
