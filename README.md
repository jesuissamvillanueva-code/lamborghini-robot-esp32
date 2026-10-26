#include <WiFi.h>
#include <WebServer.h>

// =====================================================
//      LAMBORGHINI ROBOT - EDICIÓN DE LUJO ESP32
// =====================================================

// CONFIGURACIÓN Wi-Fi (MODO AP)
const char* ssid = "Lamborghini Robot Pro";
const char* password = "123456789";

// IP FIJA PARA ACCESO DIRECTO
IPAddress local_IP(192, 168, 4, 1);
IPAddress gateway(192, 168, 4, 1);
IPAddress subnet(255, 255, 255, 0);

// SERVIDOR WEB EN PUERTO 80
WebServer server(80);

// SEGURIDAD Y TEMPORIZACIÓN
unsigned long ultimaOrdenMovimiento = 0;
const unsigned long TIEMPO_MAXIMO_SIN_ORDEN = 700; // Watchdog de seguridad (ms)

// CONTROL DE GIROS (Pulsos de 200 ms para motores DC de 2V-6V)
const unsigned long TIEMPO_GIRO = 200;
unsigned long finPulsoGiro = 0;
bool giroTemporizado = false;

// PINES DE CONTROL DE MOTORES (Driver L298N Mini / MX1508)
const int IN1 = 16; // Motor A - Adelante
const int IN2 = 17; // Motor A - Atrás
const int IN3 = 18; // Motor B - Izquierda
const int IN4 = 19; // Motor B - Derecha

// INDICADOR LED DE ESTADO
const int LED_AZUL = 2;


// =====================================================
//               INTERFAZ HTML / CSS / JS
// =====================================================

const char html_page[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#080b10">
<title>LAMBORGHINI ROBOT | PRO CONTROL</title>
<style>
:root{
  --cyan:#00f0ff;
  --blue:#0077ff;
  --dark-bg:#05070b;
  --panel-bg:linear-gradient(145deg, #101722, #070a0f);
  --accent-red:#ff2a5f;
  --neon-glow:0 0 25px rgba(0, 240, 255, 0.35);
}

*{box-sizing:border-box;-webkit-tap-highlight-color:transparent;}

html{
  min-height:100%;
  background:var(--dark-bg);
}

body{
  margin:0;
  min-height:100vh;
  padding:20px 12px;
  font-family:system-ui,-apple-system,sans-serif;
  color:#f0f6fc;
  text-align:center;
  background:radial-gradient(circle at 50% 0%, #122238 0%, #05070b 70%);
}

body:before{
  content:"";
  position:fixed;
  inset:0;
  pointer-events:none;
  opacity:.12;
  background-image:
    linear-gradient(rgba(0,240,255,.2) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0,240,255,.2) 1px, transparent 1px);
  background-size:40px 40px;
}

header, main, footer{position:relative;z-index:2;}

.brand-badge{
  width:64px;
  height:64px;
  margin:0 auto 12px;
  display:grid;
  place-items:center;
  border:2px solid rgba(0,240,255,0.6);
  border-radius:18px;
  background:linear-gradient(135deg, #1a2a40, #05070b);
  box-shadow:var(--neon-glow);
  color:var(--cyan);
  font-size:28px;
  font-weight:900;
  letter-spacing:1px;
}

h1{
  margin:0;
  font-size:clamp(24px, 7vw, 38px);
  font-weight:900;
  letter-spacing:.12em;
  color:#ffffff;
  text-shadow: 0 2px 10px rgba(0,0,0,0.8);
}

h1 span{color:var(--cyan); text-shadow:0 0 15px rgba(0,240,255,0.6);}

.subtitle{
  margin-top:6px;
  font-size:9px;
  font-weight:800;
  letter-spacing:.3em;
  color:#7eaac7;
  text-transform:uppercase;
}

.status-chip{
  display:inline-flex;
  align-items:center;
  gap:8px;
  margin-top:14px;
  padding:7px 16px;
  border:1px solid rgba(0,240,255,0.3);
  border-radius:30px;
  background:rgba(0,119,255,0.1);
  font-size:10px;
  font-weight:800;
  letter-spacing:.15em;
  color:#b3e5fc;
  backdrop-filter:blur(5px);
}

.dot{
  width:8px;
  height:8px;
  border-radius:50%;
  background:#00ff88;
  box-shadow:0 0 10px #00ff88;
}

.panel{
  width:min(100%, 420px);
  margin:20px auto 0;
  padding:20px 16px;
  border:1px solid rgba(0,240,255,0.25);
  border-radius:28px;
  background:var(--panel-bg);
  box-shadow:0 20px 50px rgba(0,0,0,0.7), inset 0 1px 0 rgba(255,255,255,0.1);
}

.panel-header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:14px;
  padding:0 4px;
}

.panel-title{
  margin:0;
  font-size:12px;
  font-weight:900;
  letter-spacing:.2em;
  color:#a5d6ff;
  text-align:left;
}

.live-telemetry{
  font-size:9px;
  font-weight:700;
  padding:3px 8px;
  background:rgba(255,42,95,0.15);
  border:1px solid rgba(255,42,95,0.4);
  border-radius:6px;
  color:var(--accent-red);
}

.status-box{
  min-height:42px;
  display:flex;
  align-items:center;
  justify-content:center;
  gap:10px;
  margin-bottom:18px;
  padding:8px 12px;
  border:1px solid rgba(0,240,255,0.2);
  border-radius:12px;
  background:#020408;
  color:var(--cyan);
  font-size:11px;
  font-weight:900;
  letter-spacing:.1em;
}

.status-indicator{
  width:8px;
  height:8px;
  border-radius:50%;
  background:#00ff88;
  box-shadow:0 0 10px #00ff88;
  transition:all 0.3s ease;
}

.joystick-container{
  display:flex;
  justify-content:center;
  padding:5px 0 15px;
}

.joystick-pad{
  position:relative;
  width:min(70vw, 250px);
  height:min(70vw, 250px);
  border-radius:50%;
  border:2px solid rgba(0,240,255,0.5);
  background:radial-gradient(circle, #152238 0%, #080f1a 70%, #030508 100%);
  box-shadow: 
    0 0 0 10px rgba(0,119,255,0.05),
    0 0 30px rgba(0,240,255,0.15),
    inset 0 0 25px rgba(0,0,0,0.8);
  touch-action:none;
  user-select:none;
}

.joystick-pad:before, .joystick-pad:after{
  content:"";
  position:absolute;
  background:rgba(0,240,255,0.15);
  pointer-events:none;
}
.joystick-pad:before{width:2px; height:80%; left:50%; top:10%; transform:translateX(-50%);}
.joystick-pad:after{height:2px; width:80%; top:50%; left:10%; transform:translateY(-50%);}

.dir-indicator{
  position:absolute;
  z-index:2;
  color:rgba(0,240,255,0.5);
  font-size:16px;
  font-weight:900;
  pointer-events:none;
}
.d-up{top:12px; left:50%; transform:translateX(-50%);}
.d-down{bottom:12px; left:50%; transform:translateX(-50%);}
.d-left{left:14px; top:50%; transform:translateY(-50%);}
.d-right{right:14px; top:50%; transform:translateY(-50%);}

.stick-knob{
  position:absolute;
  z-index:5;
  left:50%;
  top:50%;
  width:36%;
  height:36%;
  transform:translate(-50%, -50%);
  border-radius:50%;
  border:2px solid #80d8ff;
  background:radial-gradient(circle at 35% 35%, #b9f6ca, #00b0ff 40%, #01579b 75%, #002171);
  box-shadow: 
    0 0 0 8px rgba(0,240,255,0.15),
    0 10px 25px rgba(0,0,0,0.8);
  pointer-events:none;
  transition: box-shadow 0.2s;
}

.stick-knob.active{
  box-shadow: 0 0 30px var(--cyan), 0 10px 25px rgba(0,0,0,0.9);
}

.btn-emergency{
  display:block;
  width:100%;
  margin-top:10px;
  padding:14px;
  border:1px solid rgba(255,42,95,0.8);
  border-radius:14px;
  background:linear-gradient(135deg, #a81c3e, #420815);
  color:white;
  font-size:12px;
  font-weight:900;
  letter-spacing:.2em;
  cursor:pointer;
  box-shadow:0 5px 20px rgba(255,42,95,0.3);
  touch-action:manipulation;
  transition: transform 0.1s;
}

.btn-emergency:active{transform:scale(0.97);}

.specs-footer{
  margin-top:18px;
  font-size:10px;
  color:#6b8299;
  line-height:1.6;
}

footer{
  margin:25px auto 10px;
  color:#546e7a;
  font-size:10px;
  letter-spacing:.05em;
}

.author{color:#90caf9; font-weight:700;}
</style>
</head>
<body>

<header>
  <div class="brand-badge">L</div>
  <h1>LAMBORGHINI <span>PRO</span></h1>
  <div class="subtitle">Control Inalámbrico ESP32 • Motores DC 9V</div>
  <div class="status-chip">
    <i class="dot"></i> SISTEMA ACTIVO EN LÍNEA
  </div>
</header>

<main class="panel">
  <div class="panel-header">
    <span class="panel-title">CONTROL TÁCTIL VECTORIAL</span>
    <span class="live-telemetry">BATERÍA 9V OK</span>
  </div>

  <div class="status-box" role="status">
    <span class="status-indicator" id="indicator"></span>
    <span id="status">SISTEMA EN ESPERA</span>
  </div>

  <div class="joystick-container">
    <div class="joystick-pad" id="joystick">
      <span class="dir-indicator d-up">▲</span>
      <span class="dir-indicator d-down">▼</span>
      <span class="dir-indicator d-left">◀</span>
      <span class="dir-indicator d-right">▶</span>
      <div class="stick-knob" id="stick"></div>
    </div>
  </div>

  <button class="btn-emergency" id="stopButton">
    ■ &nbsp; FRENADO DE EMERGENCIA
  </button>

  <div class="specs-footer">
    Motor A (Tracción): Adelante / Atrás<br>
    Motor B (Dirección): Izquierda / Derecha (Pulsos 200ms)
  </div>
</main>

<footer>
  SISTEMA ROBÓTICO INTEGRADO<br>
  <span class="author">Desarrollado para Presentación de Alto Rendimiento © 2026</span>
</footer>

<script>
(function(){
  const pad = document.getElementById('joystick');
  const stick = document.getElementById('stick');
  const statusTxt = document.getElementById('status');
  const indicator = document.getElementById('indicator');
  const stopButton = document.getElementById('stopButton');

  let pointerId = null;
  let currentCmd = 'stop';
  let lastSent = 0;
  let heartbeat = null;
  let requestChain = Promise.resolve();

  const labels = {
    forward: 'MOTOR A: ADELANTE',
    backward: 'MOTOR A: ATRÁS',
    left: 'MOTOR B: GIRO IZQUIERDA',
    right: 'MOTOR B: GIRO DERECHA',
    stop: 'MOTORES DETENIDOS'
  };

  function sendCommand(cmd, force) {
    const now = Date.now();
    if (!force && cmd === currentCmd && now - lastSent < 160) return;
    
    currentCmd = cmd;
    lastSent = now;

    statusTxt.textContent = labels[cmd];
    indicator.style.background = cmd === 'stop' ? '#00ff88' : '#00f0ff';
    indicator.style.boxShadow = cmd === 'stop' ? '0 0 10px #00ff88' : '0 0 12px #00f0ff';

    requestChain = requestChain
      .catch(function(){})
      .then(function(){
        return fetch('/' + cmd, { method: 'GET', cache: 'no-store' });
      })
      .then(function(res){
        if(!res.ok) throw new Error();
      })
      .catch(function(){
        statusTxt.textContent = 'ERROR DE CONEXIÓN CON EL ROBOT';
        indicator.style.background = '#ff2a5f';
      });
  }

  function centerStick() {
    stick.style.left = '50%';
    stick.style.top = '50%';
    stick.classList.remove('active');
  }

  function stopHeartbeat() {
    if (heartbeat !== null) {
      clearInterval(heartbeat);
      heartbeat = null;
    }
  }

  function fullStop() {
    stopHeartbeat();
    pointerId = null;
    centerStick();
    sendCommand('stop', true);
  }

  function handleMove(e) {
    const rect = pad.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;

    let dx = e.clientX - cx;
    let dy = e.clientY - cy;

    const maxRadius = rect.width * 0.32;
    const dist = Math.hypot(dx, dy);

    if (dist > maxRadius) {
      dx = (dx / dist) * maxRadius;
      dy = (dy / dist) * maxRadius;
    }

    stick.style.left = 'calc(50% + ' + dx + 'px)';
    stick.style.top = 'calc(50% + ' + dy + 'px)';
    stick.classList.add('active');

    if (dist < maxRadius * 0.2) {
      sendCommand('stop', true);
      return;
    }

    const angle = Math.atan2(dy, dx) * 180 / Math.PI;
    let cmd;

    if (angle >= -45 && angle < 45) {
      cmd = 'right';
    } else if (angle >= 45 && angle < 135) {
      cmd = 'backward';
    } else if (angle >= -135 && angle < -45) {
      cmd = 'forward';
    } else {
      cmd = 'left';
    }

    sendCommand(cmd, false);
  }

  pad.addEventListener('pointerdown', function(e){
    e.preventDefault();
    if (pointerId !== null) fullStop();
    pointerId = e.pointerId;
    try { pad.setPointerCapture(pointerId); } catch(_) {}
    handleMove(e);
    
    stopHeartbeat();
    heartbeat = setInterval(function(){
      if (pointerId !== null && currentCmd !== 'stop') {
        sendCommand(currentCmd, true);
      }
    }, 250);
  });

  pad.addEventListener('pointermove', function(e){
    if (e.pointerId === pointerId) {
      e.preventDefault();
      handleMove(e);
    }
  });

  pad.addEventListener('pointerup', function(e){ if (e.pointerId === pointerId) fullStop(); });
  pad.addEventListener('pointercancel', function(e){ if (e.pointerId === pointerId) fullStop(); });
  stopButton.addEventListener('click', fullStop);

  window.addEventListener('pagehide', function(){
    stopHeartbeat();
    fetch('/stop', { method: 'GET', keepalive: true }).catch(function(){});
  });
})();
</script>

</body>
</html>
)rawliteral";

// =====================================================
//               CONTROL DE MOTORES DC
// =====================================================

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  giroTemporizado = false;
}

void moveForward() {
  stopMotors();
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
}

void moveBackward() {
  stopMotors();
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);
}

void turnLeft() {
  stopMotors();
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  giroTemporizado = true;
  finPulsoGiro = millis() + TIEMPO_GIRO;
}

void turnRight() {
  stopMotors();
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  giroTemporizado = true;
  finPulsoGiro = millis() + TIEMPO_GIRO;
}

void handleRoot() {
  server.send_P(200, "text/html; charset=utf-8", html_page);
}

// =====================================================
//                     CONFIGURACIÓN
// =====================================================

void setup() {
  Serial.begin(115200);
  ultimaOrdenMovimiento = millis();

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
  pinMode(LED_AZUL, OUTPUT);

  stopMotors();
  digitalWrite(LED_AZUL, LOW);

  WiFi.mode(WIFI_AP);
  WiFi.softAPConfig(local_IP, gateway, subnet);
  WiFi.softAP(ssid, password);

  digitalWrite(LED_AZUL, HIGH);

  server.on("/", HTTP_GET, handleRoot);
  
  server.on("/forward", HTTP_GET, [](){
    ultimaOrdenMovimiento = millis();
    moveForward();
    server.send(200, "text/plain", "OK");
  });

  server.on("/backward", HTTP_GET, [](){
    ultimaOrdenMovimiento = millis();
    moveBackward();
    server.send(200, "text/plain", "OK");
  });

  server.on("/left", HTTP_GET, [](){
    ultimaOrdenMovimiento = millis();
    turnLeft();
    server.send(200, "text/plain", "OK");
  });

  server.on("/right", HTTP_GET, [](){
    ultimaOrdenMovimiento = millis();
    turnRight();
    server.send(200, "text/plain", "OK");
  });

  server.on("/stop", HTTP_GET, [](){
    ultimaOrdenMovimiento = millis();
    stopMotors();
    server.send(200, "text/plain", "OK");
  });

  server.begin();
  Serial.println("¡Lamborghini Robot Pro Iniciado!");
  Serial.println("Conéctate a la red Wi-Fi: Lamborghini Robot Pro");
  Serial.println("Abre en tu navegador: http://192.168.4.1");
}

// =====================================================
//                     BUCLE PRINCIPAL
// =====================================================

void loop() {
  server.handleClient();

  if (giroTemporizado && (long)(millis() - finPulsoGiro) >= 0) {
    stopMotors();
  }

  if (millis() - ultimaOrdenMovimiento > TIEMPO_MAXIMO_SIN_ORDEN) {
    stopMotors();
  }
}
