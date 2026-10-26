# 🏎️ Lamborghini Robot Pro
Sistema de control robótico inalámbrico IoT basado en **ESP32**, diseñado sobre la carrocería a escala de un superdeportivo, con control de tracción independiente mediante driver **L298N Mini** y alimentación de **9V**.

---

## 📌 Características del Proyecto
* **Conectividad Wi-Fi (Modo AP):** El ESP32 genera su propia red local con IP fija (`192.168.4.1`) para conectarse directamente desde cualquier smartphone o tablet sin necesidad de internet.
* **Interfaz Web Táctil Avanzada:** Panel gráfico con joystick virtual interactivo optimizado mediante *Pointer Events*.
* **Seguridad y Telemetría:** Incorpora un sistema *Watchdog* por software (parada automática a los 700 ms sin señal) y un botón de frenado de emergencia.
* **Optimización para Motores DC (2V-6V):** Limitación de pulsos de giro a 200 ms para proteger los motores pequeños del coche de juguete frente a la batería de 9V.

---

## 🛠️ Componentes Hardware
* 1x Microcontrolador ESP32 NodeMCU
* 1x Driver de Motores L298N Mini / MX1508
* 2x Motores DC (2V - 6V) integrados en el chasis del auto a escala
* 1x Batería recargable de 9V
* Chasis de juguete tipo Lamborghini a escala

---

## 🔌 Diagrama de Conexiones (Pinout)

| Componente ESP32 | Pin / GPIO | Conexión en el Driver L298N Mini |
| :--- | :--- | :--- |
| **GPIO 16** | IN1 | Motor A (Avance) |
| **GPIO 17** | IN2 | Motor A (Retroceso) |
| **GPIO 18** | IN3 | Motor B (Giro Izquierda) |
| **GPIO 19** | IN4 | Motor B (Giro Derecha) |
| **GPIO 2** | LED_AZUL | LED indicador de estado integrado |

---

## 🚀 Guía de Instalación y Uso

1. Clona este repositorio o descarga el código fuente desde la carpeta `codigo/`.
2. Abre el archivo de código utilizando **Arduino IDE**.
3. Asegúrate de tener instalada la placa **ESP32** en tu gestor de tarjetas.
4. Conecta tu ESP32 por USB y sube el código.
5. En tu celular, busca la red Wi-Fi llamada **`Lamborghini Robot Pro`** e ingresa la contraseña: `123456789`.
6. Abre tu navegador web e ingresa a: `http://192.168.4.1` para controlar el vehículo.

---
<p align="center">Desarrollado para proyectos de ingeniería y robótica educativa.</p>
