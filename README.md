<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Reino & Eventos Medievales</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    }

    body {
      background-color: #121214;
      color: #e1e1e6;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      padding: 15px;
    }

    .container {
      width: 100%;
      max-width: 480px;
      background: #1e1e22;
      border: 1px solid #323238;
      border-radius: 12px;
      padding: 20px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.5);
    }

    h1 {
      font-size: 1.4rem;
      text-align: center;
      margin-bottom: 15px;
      color: #e5b869;
    }

    /* Panel de Estado */
    .status-bar {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
      background: #29292e;
      padding: 12px;
      border-radius: 8px;
      margin-bottom: 20px;
      font-size: 0.9rem;
    }

    .stat {
      display: flex;
      justify-content: space-between;
      border-bottom: 1px solid #3c3c42;
      padding-bottom: 4px;
    }

    .stat span {
      font-weight: bold;
      color: #208838;
    }

    /* Tarjeta de Evento */
    .event-card {
      background: #202024;
      border: 1px solid #323238;
      border-radius: 8px;
      padding: 16px;
      margin-bottom: 20px;
    }

    .event-title {
      font-size: 1.2rem;
      color: #e5b869;
      margin-bottom: 8px;
    }

    .event-text {
      font-size: 0.95rem;
      line-height: 1.4;
      color: #c4c4cc;
      margin-bottom: 15px;
    }

    .options-list {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    button {
      background: #323238;
      color: #fff;
      border: 1px solid #4d4d57;
      padding: 12px;
      border-radius: 6px;
      font-size: 0.95rem;
      cursor: pointer;
      transition: background 0.2s, transform 0.1s;
      text-align: left;
    }

    button:active {
      transform: scale(0.98);
      background: #4d4d57;
    }

    .btn-next {
      width: 100%;
      background: #e5b869;
      color: #121214;
      font-weight: bold;
      text-align: center;
      margin-top: 10px;
      border: none;
    }

    .btn-next:active {
      background: #c79e53;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>👑 Gestión del Reino</h1>

  <!-- Panel de Recursos -->
  <div class="status-bar">
    <div class="stat">Oro: <span id="val-oro">100</span></div>
    <div class="stat">Prestigio: <span id="val-prestigio" style="color:#2b7fff;">10</span></div>
    <div class="stat">Riesgo Herejía: <span id="val-herejia" style="color:#ff5555;">0%</span></div>
    <div class="stat">Lealtad Feudos: <span id="val-lealtad" style="color:#e5b869;">80</span></div>
  </div>

  <!-- Contenedor del Evento -->
  <div class="event-card" id="event-container">
    <div class="event-title" id="event-title">Cargando...</div>
    <div class="event-text" id="event-text"></div>
    <div class="options-list" id="event-options"></div>
  </div>

  <button class="btn-next" id="btn-next" style="display:none;" onclick="siguienteTurno()">Siguiente Evento</button>
</div>

<script>
  // 1. ESTADO DEL JUEGO Y PERSONAJE
  const personaje = {
    rasgos: ["borracho"], // Puedes cambiar esto a "poseido" para probar los pesos
    viajando: false
  };

  const estado = {
    oro: 100,
    prestigio: 10,
    herejiaRiesgo: 0,
    feudos: [
      { id: 1, tipo: "ducado", lealtad: 80 },
      { id: 2, tipo: "marquesado", lealtad: 80 }
    ]
  };

  function ganarPrestigio(cant) {
    estado.prestigio += cant;
  }

  // 2. LISTA DE EVENTOS (Tu código)
  const eventos = [
    {
      id: "esp_5",
      titulo: "Embriaguez en la Corte",
      texto: "Te has emborrachado durante un banquete frente a los nobles.",
      peso: c => c.rasgos.includes("borracho") ? 2 : 0.05,
      opciones: [
        {
          txt: "Disculparse (-5 lealtad)",
          efecto: () => {
            estado.feudos.forEach(f => {
              if (f.tipo !== "reino") f.lealtad = Math.max(0, f.lealtad - 5);
            });
          }
        },
        { txt: "Ignorar", efecto: () => {} }
      ]
    },
    {
      id: "esp_6",
      titulo: "Posesión Demoníaca",
      texto: "Susurros oscuros atormentan tus noches. La iglesia te vigila de cerca.",
      peso: c => c.rasgos.includes("poseido") ? 2 : 0.05,
      opciones: [
        {
          txt: "Exorcismo (-40 Oro, -10 Risk Herejía)",
          efecto: () => {
            estado.oro -= 40;
            estado.herejiaRiesgo = Math.max(0, estado.herejiaRiesgo - 10);
          }
        },
        {
          txt: "Aceptar la voz (+15 Risk Herejía)",
          efecto: () => {
            estado.herejiaRiesgo += 15;
          }
        }
      ]
    },
    {
      id: "esp_7",
      titulo: "Invitación al Torneo",
      texto: "Un reino vecino celebra un torneo en honor a la corona.",
      peso: c => !c.viajando ? 0.7 : 0,
      requiere: c => !c.viajando,
      opciones: [
        {
          txt: "Asistir (-30 Oro, +10 Prestigio)",
          efecto: () => {
            estado.oro -= 30;
            ganarPrestigio(10);
          }
        },
        { txt: "Rechazar", efecto: () => {} }
      ]
    }
  ];

  // 3. MOTOR DE EVENTOS
  function obtenerEventoAleatorio() {
    // Filtrar eventos con el método "requiere" si existe
    const permitidos = eventos.filter(e => !e.requiere || e.requiere(personaje));

    // Calcular suma total de pesos
    const pesoTotal = permitidos.reduce((acc, e) => acc + e.peso(personaje), 0);
    let rand = Math.random() * pesoTotal;

    for (let ev of permitidos) {
      const p = ev.peso(personaje);
      if (rand < p) return ev;
      rand -= p;
    }
    return permitidos[0];
  }

  // 4. INTERFAZ GRÁFICA
  function actualizarUI() {
    document.getElementById("val-oro").textContent = estado.oro;
    document.getElementById("val-prestigio").textContent = estado.prestigio;
    document.getElementById("val-herejia").textContent = estado.herejiaRiesgo + "%";
    
    // Promedio de lealtad de los feudos
    const lealtadProm = Math.round(estado.feudos.reduce((a, b) => a + b.lealtad, 0) / estado.feudos.length);
    document.getElementById("val-lealtad").textContent = lealtadProm;
  }

  function mostrarEvento(ev) {
    document.getElementById("event-title").textContent = ev.titulo;
    document.getElementById("event-text").textContent = ev.texto;
    
    const containerOpciones = document.getElementById("event-options");
    containerOpciones.innerHTML = "";

    ev.opciones.forEach(opcion => {
      const btn = document.createElement("button");
      btn.textContent = opcion.txt;
      btn.onclick = () => {
        opcion.efecto();
        actualizarUI();
        
        // Deshabilitar botones de opción y mostrar botón de siguiente turno
        containerOpciones.querySelectorAll("button").forEach(b => b.disabled = true);
        document.getElementById("btn-next").style.display = "block";
      };
      containerOpciones.appendChild(btn);
    });

    document.getElementById("btn-next").style.display = "none";
  }

  function siguienteTurno() {
    const ev = obtenerEventoAleatorio();
    mostrarEvento(ev);
  }

  // Inicio del juego
  actualizarUI();
  siguienteTurno();
</script>
</body>
</html>
