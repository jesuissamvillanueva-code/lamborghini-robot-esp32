# 🏎️ Repositorio Técnico: Lamborghini Robot Pro (IoT / ESP32)

Sistema ciberfísico autónomo orientado al control cinemático inalámbrico mediante una topología de red de área local inalámbrica en modalidad de Punto de Acceso (*Access Point - AP*). El núcleo de procesamiento está sustentado por un microcontrolador **ESP32**, el cual hospeda un servidor HTTP embebido capaz de servir una interfaz gráfica de usuario (*GUI*) hipermedia optimizada con eventos de puntero asíncronos y diseño responsivo (*Glassmorphism* / Neón Tecnológico).

---

## 📑 Tabla de Contenidos
1. [Resumen Ejecutivo y Arquitectura del Sistema](#-resumen-ejecutivo-y-arquitectura-del-sistema)
2. [Especificaciones Técnicas del Hardware y Topología de Circuitos](#-especificaciones-técnicas-del-hardware-y-topología-de-circuitos)
3. [Protocolos de Comunicación y Gestión de Concurrencia](#-protocolos-de-comunicación-y-gestión-de-concurrencia)
4. [Mecanismos de Seguridad Activa (Fail-Safe)](#-mecanismos-de-seguridad-activa-fail-safe)
5. [Guía Metodológica de Despliegue e Implementación](#-guía-metodológica-de-despliegue-e-implementación)

---

## 📌 Resumen Ejecutivo y Arquitectura del Sistema

El proyecto **Lamborghini Robot Pro** encarna un paradigma de ingeniería embebida e IoT aplicado sobre un modelo a escala. La arquitectura de control segrega las funciones de tracción longitudinal y orientación direccional, garantizando una respuesta de baja latencia y alta fidelidad en entornos de experimentación robótica móvil.

---

## 🛠️ Especificaciones Técnicas del Hardware y Topología de Circuitos

El subsistema electromecánico ha sido dimensionado para operar bajo un diferencial de potencial de **9V** mediante una fuente de alimentación recargable, regulando la dinámica de actuación sobre motores de corriente continua (*DC*) con rangos de operación nominales de $2V - 6V$.

### Matriz de Asignación de Pines (Pinout Topology)

| Periférico de Control | Pin / GPIO (ESP32) | Interfaz de Potencia (L298N Mini / MX1508) | Parámetro Funcional |
| :--- | :--- | :--- | :--- |
| **GPIO 16** | `IN1` | Puente H - Motor A (Tracción) | Polarización directa de avance |
| **GPIO 17** | `IN2` | Puente H - Motor A (Tracción) | Polarización inversa de retroceso |
| **GPIO 18** | `IN3` | Puente H - Motor B (Dirección) | Vectorización de giro sinistroverso |
| **GPIO 19** | `IN4` | Puente H - Motor B (Dirección) | Vectorización de giro dextroverso |
| **GPIO 2** | `LED_AZUL` | LED SMD Integrado | Testigo de estatus operacional y enlace |

---

## 🔄 Protocolos de Comunicación y Gestión de Concurrencia

1. **Establecimiento de Enlace Inalámbrico Estático:**
   * El microcontrolador inicializa la interfaz inalámbrica bajo el protocolo `WIFI_AP`, configurando una dirección IP estática (`192.168.4.1`), puerta de enlace predeterminada y máscara de subred para mitigar la fluctuación de paquetes (*jitter*).
2. **Encapsulamiento y Entrega de Recursos en Memoria Flash:**
   * La interfaz interactiva es compilada directamente en la memoria de programa mediante la directiva `PROGMEM`, optimizando el consumo de la memoria de acceso aleatorio dinámica (*RAM*) del SoC.
3. **Control Asíncrono de Peticiones RESTful:**
   * El cliente web interpreta los vectores direccionales del usuario a través de un joystick táctil flotante, emitiendo solicitudes HTTP asíncronas (`fetch`) reguladas por una cadena de promesas (*request chaining*) que evita la saturación del búfer del servidor web embebido.

---

## 🛡️ Mecanismos de Seguridad Activa (Fail-Safe)

* **Vigilante de Enlace (Watchdog Timer Software):** Se ha implementado una directiva de monitoreo temporal continua (`TIEMPO_MAXIMO_SIN_ORDEN = 700 ms`). En caso de pérdida de paquetes o interrupción abrupta de la señal de control, el microcontrolador ejecuta de manera síncrona la interrupción de emergencia `stopMotors()`.
* **Optimización Térmica y Dinámica para Motores $2V - 6V$:** Dado el delta de voltaje entre la alimentación de la batería ($9V$) y la tolerancia nominal de los actuadores del modelo a escala, los giros direccionales se encuentran acotados mediante un algoritmo de temporización estricta (`TIEMPO_GIRO = 200 ms`), previniendo la saturación térmica y la fatiga mecánica del bobinado.

---

## 🚀 Guía Metodológica de Despliegue e Implementación

Para replicar y desplegar esta infraestructura en su entorno de desarrollo, proceda según las siguientes directrices operativas:

1. **Clonación o Inicialización del Repositorio:**
   * Asegúrese de estructurar los directorios locales manteniendo la separación entre el código fuente (`/firmware`) y la documentación analítica (`README.md`).
2. **Configuración del Entorno de Desarrollo (IDE):**
   * Abra el entorno de compilación **Arduino IDE** con soporte para arquitecturas **ESP32** habilitado en el gestor de tarjetas.
   * Verifique la correcta inclusión de las librerías nativas del framework: `<WiFi.h>` y `<WebServer.h>`.
3. **Flujo de Compilación y Destello (Flashing):**
   * Conecte el módulo ESP32 a través del puerto serial físico, seleccione el puerto COM correspondiente y ejecute la compilación y transferencia de binarios.
4. **Validación de Campo:**
   * Conecte un dispositivo terminal a la red Wi-Fi de difusión denominada **`Lamborghini Robot Pro`** (Credencial de acceso: `123456789`).
   * Inserte la URI de circunvalación local `http://192.168.4.1` en el navegador web para iniciar la supervisión telemétrica y el pilotaje del sistema ciberfísico.

---
<p align="center">Desarrollado bajo estándares avanzados de ingeniería ciberfísica e IoT.</p>
