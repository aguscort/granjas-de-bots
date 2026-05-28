**MÓDULO 3: PROTOCOLOS DE CONTROL Y AUTOMATIZACIÓN**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, seréis capaces de diseccionar los mecanismos de comunicación entre el panel de control y los dispositivos, comprender los distintos métodos de inyección de entrada táctil que emplea GenFarmer, y evaluar la detectabilidad de cada uno de ellos.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza.*

Bienvenidos al Módulo 3. Hasta ahora hemos visto el ecosistema Gen desde el mapa general y hemos abierto en canal al agente APK. Hoy vamos a meternos en el sistema nervioso de la granja: los **protocolos de comunicación y los mecanismos de automatización de entrada**.

Imaginad el agente APK como un soldado. En el Módulo 2 analizamos su uniforme, su armamento y su documento de identidad. Hoy vamos a estudiar cómo recibe las órdenes del general y cómo dispara su arma. Porque, creedme, no todos los soldados disparan igual, y cada método de disparo deja un tipo de casquillo diferente.

Nos centraremos en dos grandes áreas. Primero, el canal de comunicación: ¿cómo habla el panel de GenFarmer con el agente en el teléfono? ¿Qué protocolos usa? ¿Qué datos se intercambian? Y segundo, los métodos de automatización táctil: ¿cómo convierte el agente una orden de "pulsa aquí" en un toque real sobre la pantalla? Veremos que hay múltiples formas de hacerlo, cada una con ventajas tácticas y, para nosotros los defensores, diferentes niveles de detectabilidad.

Al final del módulo, sabréis identificar en un volcado de tráfico qué está haciendo una granja en tiempo real, y podréis decir con certeza si un toque proviene de un humano o de una máquina solo analizando sus parámetros. Vamos allá.

---

**2. COMUNICACIÓN PANEL-DISPOSITIVO (75 minutos)**

*El profesor proyecta un diagrama con tres escenarios: comunicación directa LAN, comunicación a través de la nube, y comunicación híbrida.*

**2.1 El modelo de comunicación (15 minutos)**

GenFarmer no es simplemente un script que envía comandos ADB. Es una arquitectura cliente-servidor bastante sofisticada. Existen dos modos principales de comunicación, y a menudo se combinan.

**Modo A: Comunicación local directa (LAN/USB)**

Este es el modo más simple, usado en granjas pequeñas o cuando el operador quiere mínima latencia. El panel de escritorio (Windows/macOS) está en la misma red local que los teléfonos o conectado directamente por USB. La comunicación sigue esta ruta:

1. El panel establece una conexión ADB con cada dispositivo.
2. A través de `adb forward`, crea un túnel TCP. Por ejemplo:
   ```bash
   adb forward tcp:5555 tcp:8080
   ```
   Esto redirige el puerto 5555 del PC al puerto 8080 del dispositivo.
3. El agente en el teléfono tiene un servidor socket escuchando en el puerto 8080. El panel le envía comandos en formato JSON o en un protocolo binario propietario a través de ese túnel.
4. La respuesta viaja de vuelta por el mismo túnel.

**Ventajas para el atacante:** baja latencia, no depende de internet, difícil de interceptar si no estás en la misma LAN.
**Desventajas para el defensor:** si no tienes acceso físico a la granja, no ves este tráfico. Pero si puedes inspeccionar el dispositivo (por ejemplo, con un volcado de logs), verás las conexiones TCP locales.

**Modo B: Comunicación a través de la nube (Gen Cloud)**

Para granjas distribuidas geográficamente o cuando el operador quiere control remoto total, se utiliza el backend en la nube de Gen. La arquitectura es:

1. El agente en el dispositivo se conecta al servidor cloud de Gen mediante WebSocket o MQTT sobre TLS (puerto 443). Mantiene una conexión persistente.
2. El panel de escritorio también se conecta al mismo servidor cloud, autenticándose con las credenciales de la cuenta Gen.
3. Cuando el operador pulsa un botón en el panel, el comando se envía al servidor cloud, que lo enruta al dispositivo correspondiente a través de la conexión WebSocket abierta.
4. La telemetría del dispositivo (capturas de pantalla, estado de la batería, logs) fluye de vuelta al panel por la misma vía.

**Ventajas para el atacante:** control desde cualquier parte del mundo, escalabilidad, no requiere que el PC de control esté encendido 24/7.
**Desventajas para el defensor:** todo el tráfico pasa por un punto central. Si podemos monitorizar ese punto (los dominios e IPs del backend), tenemos visibilidad de toda la operación.

**Modo C: Híbrido**

En muchas granjas, se usa comunicación local para comandos en tiempo real (por baja latencia) y la nube para sincronización, actualizaciones de scripts y telemetría.

**Ejercicio práctico (10 minutos):**
Os entrego un diagrama de red simplificado de una supuesta granja. Identificad qué modo de comunicación está utilizando. Justificad vuestra respuesta basándoos en la presencia de ciertos puertos abiertos y conexiones salientes.

**2.2 Protocolos y formatos de mensaje (30 minutos)**

*El profesor abre una captura de tráfico simulada en Wireshark.*

Vamos a examinar el tráfico real. En nuestras investigaciones, hemos identificado patrones. Aunque no puedo mostraros tráfico de una granja activa por razones legales, he preparado una recreación fiel basada en nuestras observaciones.

**WebSocket sobre TLS:**
Si inspeccionáis una conexión WebSocket (una vez descifrada con mitmproxy), veréis un flujo constante de mensajes. Los mensajes del panel al dispositivo suelen tener una estructura JSON como esta:
```json
{
  "task_id": "abc-123",
  "type": "touch",
  "params": {
    "x": 540,
    "y": 960,
    "duration": 150,
    "pressure": 1.0
  },
  "device_id": "samsung_s8_42"
}
```
Fijaos en los campos: `task_id` para trazabilidad, `type` que indica la acción (touch, swipe, text, wait, app_open), y `params` con los detalles.

Los mensajes del dispositivo al panel suelen ser de telemetría:
```json
{
  "device_id": "samsung_s8_42",
  "status": "idle",
  "battery": 87,
  "temperature": 38.2,
  "current_app": "com.instagram.android",
  "screenshot": "base64_encoded_image..."
}
```
La captura de pantalla se envía comprimida en JPEG o WebP, codificada en base64. Esto permite al operador ver en tiempo real lo que muestra cada teléfono en la cuadrícula del panel.

**MQTT:**
Algunas versiones utilizan MQTT, un protocolo de mensajería ligero. Veríais tópicos como `genfarmer/devices/samsung_s8_42/command` y `genfarmer/devices/samsung_s8_42/telemetry`. Es eficiente para muchos dispositivos porque usa un modelo publish-subscribe.

**Latidos y keep-alive:**
Tanto en WebSocket como en MQTT, el agente envía un ping periódico (cada 30-60 segundos) para mantener la conexión viva. Este patrón es una firma de tráfico muy útil: si veis un flujo constante de paquetes pequeños cada 30 segundos hacia un mismo dominio, tenéis una conexión persistente de un agente.

**Ejercicio práctico (20 minutos):**
En el archivo PCAP que os he entregado (simulado), filtrad por `tcp.port == 443` y buscad tráfico WebSocket. Identificad:
- La frecuencia de los latidos.
- Un mensaje de comando y un mensaje de telemetría.
- La estructura del JSON.
- ¿Podríais escribir una regla de Suricata para detectar este tráfico? Probad a esbozarla.

*Pausa de 15 minutos.*

**2.3 Análisis de telemetría y comandos (30 minutos)**

*El profesor retoma.*

La telemetría no es solo para comodidad del operador. Para nosotros, es un tesoro. Cada mensaje de telemetría nos dice qué apps está ejecutando la granja, el estado del dispositivo, y a veces hasta qué campaña está activa.

Además, los comandos revelan las tácticas. ¿Qué tipo de acciones se repiten? En una campaña de fraude de instalación de apps, veréis una secuencia predecible:
1. Comando: `app_install` (paquete: `com.appobjetivo`).
2. Comando: `app_open` (paquete: `com.appobjetivo`).
3. Comando: `touch` en coordenadas del botón "Aceptar" del tutorial.
4. Comando: `wait` (3000 ms).
5. Comando: `touch` en coordenadas del botón "Comenzar".
6. Comando: `app_close`.

Esta secuencia se repite idéntica en cientos de dispositivos con ligeras variaciones de tiempo. Es una coreografía robótica. Si en vuestra plataforma detectáis que 500 cuentas completan el tutorial de una app en una ventana de 5 minutos con tiempos de reacción casi idénticos, no es una coincidencia: es una granja.

---

**3. MÉTODOS DE AUTOMATIZACIÓN DE LA ENTRADA TÁCTIL (90 minutos)**

*El profesor muestra una diapositiva con los cuatro métodos ordenados de mayor a menor detectabilidad.*

Esta es la parte más técnica del módulo. Vamos a desmontar cómo un comando de "toque en (540, 960)" se convierte en un evento táctil que la app objetivo interpreta como un dedo humano. GenFarmer emplea varios métodos, y los elige en función del nivel de acceso (sin root vs. con root) y de la paranoia del operador. Los analizaremos en orden de detectabilidad, del más torpe al más sigiloso.

**3.1 ADB shell input (15 minutos)**

*El profesor escribe en la pizarra:*
```bash
adb shell input tap 540 960
adb shell input swipe 300 1000 300 500
adb shell input text "Hola mundo"
```

Este es el método más burdo. GenFarmer lo utiliza solo para tareas de mantenimiento: instalar APKs, reiniciar el dispositivo, o conceder permisos iniciales. ¿Por qué no para automatización fina? Porque es muy fácil de detectar.

Cuando ejecutáis `input tap`, el sistema Android crea un `MotionEvent` inyectado a través del shell. Este evento es visible en el log del sistema (`logcat -s input`). Cualquier app que monitorice los logs del sistema (y las apps bancarias y de redes sociales lo hacen) puede ver que se está usando `input`.

Además, muchas apps comprueban la propiedad `sys.inputmethod.active` o buscan el proceso `input` en `/proc/self/cmdline`. Si detectan que hay un shell ejecutando `input`, bloquean la interacción.

**Detectabilidad:** Muy alta. Deja rastros en logs, en la línea de comandos y en las propiedades del sistema.

**Ejercicio práctico (5 minutos):**
En vuestro emulador, ejecutad `adb shell input tap 500 500` mientras monitorizáis `adb logcat | grep -i input`. Observad los mensajes de registro. Esta es la huella que un defensor puede capturar.

**3.2 AccessibilityService.dispatchGesture() (30 minutos)**

*El profesor abre una diapositiva con código Java.*

Este es el método estrella cuando el dispositivo no tiene root. Es el caballo de batalla de GenFarmer y de la mayoría de herramientas de automatización modernas.

Recordad del Módulo 2 que el agente se registra como `AccessibilityService` con `canPerformGestures="true"`. Cuando el panel le ordena un toque, el agente hace algo así:

```java
AccessibilityService accessibilityService = this;
GestureDescription.Builder gestureBuilder = new GestureDescription.Builder();

Path path = new Path();
path.moveTo(x, y);  // Coordenadas del toque

gestureBuilder.addStroke(new GestureDescription.StrokeDescription(path, 0, duration));
accessibilityService.dispatchGesture(gestureBuilder.build(), null, null);
```

Esto genera un `MotionEvent` con fuente `SOURCE_TOUCHSCREEN`, como si un dedo real hubiera tocado la pantalla. La app objetivo no puede distinguirlo de un toque humano a nivel de evento, porque el sistema lo considera legítimo.

**¿Cómo podemos detectarlo entonces?**

No es imposible. Hay varias aproximaciones forenses:

1. **Auditoría de servicios de accesibilidad**: el comando `adb shell dumpsys accessibility` lista todos los servicios activos y las operaciones que han realizado. Podemos ver cuántos gestos ha despachado cada servicio y hacia qué paquetes. Si un servicio con un nombre sospechoso ha despachado miles de gestos en una hora hacia `com.instagram.android`, no es un usuario con discapacidad; es un bot.

2. **Análisis estadístico de los trazos**: aunque el evento parece humano, la forma en que se generan los trazos es demasiado perfecta. Un toque humano tiene microtemblores (jitter) de unos pocos píxeles. El `Path` de un script suele ser una línea recta perfecta o una curva bezier predefinida. La desviación estándar de las coordenadas de muestreo de un toque humano es mayor que la de un bot. Esto requiere modelos de ML entrenados, pero es factible.

3. **Velocidad y aceleración del gesto**: un `dispatchGesture` típico usa una duración fija (por ejemplo, 150 ms) y una velocidad constante. Un humano varía la velocidad a lo largo del toque: acelera al inicio, decelera al final. Monitorizar la velocidad instantánea del `MotionEvent` revela patrones robóticos.

**Ejercicio práctico (15 minutos):**
Vais a simular un ataque y una defensa. En el emulador, activad un script (que os proporciono) que usa `dispatchGesture` para hacer 100 toques en la app de calculadora, con coordenadas ligeramente aleatorizadas. Simultáneamente, ejecutad un script defensor que captura los eventos táctiles y calcula la desviación estándar de las coordenadas y la duración de los toques. ¿Podéis distinguir los toques del script de los vuestros propios? Documentad las diferencias.

**3.3 InputManager.injectInputEvent() (con root) (25 minutos)**

*El profesor muestra una diapositiva con una advertencia: "Solo para investigación, no ejecutar en producción".*

Cuando el dispositivo tiene root, el agente de GenFarmer puede usar un método mucho más sigiloso: la API interna `InputManager.injectInputEvent()`. Esta API no está disponible para aplicaciones normales; requiere permisos de sistema (firmar la app con la clave de la plataforma o ejecutarse como root).

El código se ve así (mediante reflexión):
```java
IBinder b = ServiceManager.getService("input");
IInputManager inputManager = IInputManager.Stub.asInterface(b);

MotionEvent event = MotionEvent.obtain(downTime, eventTime, MotionEvent.ACTION_DOWN, x, y, 0);
inputManager.injectInputEvent(event, InputManager.INJECT_INPUT_EVENT_MODE_ASYNC);
```

La diferencia crucial con `dispatchGesture` es que este método **no pasa por el `AccessibilityService`**. El evento se inyecta directamente en la cola de entrada del sistema, sin interacción con el framework de accesibilidad.

**¿Qué implica esto para la detección?**

1. **No aparece en `dumpsys accessibility`**: no podemos auditar los gestos a través de la API de accesibilidad. Es invisible para las herramientas que monitorizan servicios de accesibilidad.
2. **No deja rastro en logs normales**: a diferencia de `input tap`, no hay mensajes en `logcat` a menos que se active el logging detallado del `InputManager`.
3. **Pero requiere root**: la presencia de root es en sí misma un indicador de compromiso. Herramientas como RootBeer, SafetyNet o la API Play Integrity detectan el acceso root. Si un dispositivo está rooteado, ya es sospechoso.

**Detección avanzada:**

Incluso con `injectInputEvent`, el evento generado sigue teniendo características de máquina. La fuente del evento será `SOURCE_TOUCHSCREEN` (como un humano), pero la aplicación de seguridad puede analizar el `InputDevice` asociado. Si el evento proviene de un dispositivo virtual en lugar de un `touchscreen` físico, es una pista.

Además, podemos monitorizar las llamadas a `injectInputEvent` mediante Frida en tiempo real, o mediante un módulo del kernel auditando las syscalls de entrada. Esto es más avanzado, pero factible en entornos de alta seguridad (como apps bancarias que cargan su propio módulo de seguridad).

**Ejercicio práctico (10 minutos):**
Os paso un script de Frida que engancha `InputManager.injectInputEvent`. En un emulador rooteado (que tenéis en el laboratorio), ejecutad una herramienta de automatización que use este método (yo os proporcionaré una de prueba) y observad las trazas. ¿Qué parámetros podéis extraer? ¿Cómo se comparan con los de `dispatchGesture`?

**3.4 Emulación HID externa: la frontera de la detección (20 minutos)**

*El profesor coloca un Arduino sobre la mesa y lo conecta a un teléfono del laboratorio.*

Llegamos al método más difícil de contrarrestar. Hemos hablado de software. Pero, ¿qué pasa si el atacante conecta un dispositivo de hardware que emula una pantalla táctil o un teclado?

Los microcontroladores como **Arduino Micro/Leonardo**, **Teensy** o **ESP32-S3** pueden programarse para identificarse como dispositivos USB HID. Si se programan como **HID Touchscreen**, el teléfono literalmente cree que hay una pantalla táctil externa conectada por USB, y los toques generados por el microcontrolador son indistinguibles de un dedo real a nivel del sistema operativo.

La librería `libaoa_hid`, por ejemplo, permite a un Arduino emular una pantalla táctil a través del protocolo AOA 2.0. El descriptor USB del dispositivo especifica que es un digitizador. Android, al verlo, lo acepta como fuente de entrada legítima.

**¿Por qué es tan difícil de detectar?**

- No necesita root ni ADB.
- No ejecuta código en el dispositivo.
- Los eventos generados pasan por el driver de entrada del kernel como si vinieran de un periférico real.

**¿Hay alguna esperanza para el defensor?**

Sí, pero es difícil. Podemos detectarlo por:

1. **Inspección de dispositivos USB conectados**: si un teléfono tiene un dispositivo USB conectado que se identifica como `Usage Page: 0x0D (Digitizer)` y `Usage: 0x04 (Touch Screen)`, pero el teléfono es un Samsung Galaxy que no tiene pantalla táctil externa, hay una discrepancia. El comando `lsusb` (si el dispositivo está rooteado) o la lectura de `/sys/bus/usb/devices/` revelan los descriptores USB. Un servicio de seguridad podría monitorizar la conexión de periféricos HID no esperados.

2. **Análisis de la velocidad de muestreo**: una pantalla táctil real envía eventos a una frecuencia de 60-120 Hz. Un microcontrolador puede enviar eventos a frecuencias irregulares o demasiado perfectas. Monitorizar la tasa de interrupciones del driver táctil puede revelar anomalías.

3. **Correlación con otras señales**: si un dispositivo genera miles de toques precisos pero no tiene actividad de sensores (el acelerómetro no detecta movimiento, el giroscopio está inmóvil), es sospechoso. Un humano mueve el teléfono ligeramente al tocar.

**Demostración práctica (10 minutos):**
Voy a conectar este Arduino programado como HID Touchscreen al teléfono de laboratorio. Veréis cómo, sin tocar la pantalla, se abren aplicaciones y se pul san botones. Luego ejecutaré `lsusb` en una shell root para mostrar el descriptor. Esto es lo que deberíais buscar en una investigación forense de hardware.

---

**4. SÍNTESIS Y CIERRE DEL MÓDULO (20 minutos)**

*El profesor muestra un cuadro comparativo de los cuatro métodos.*

Llegamos al final. Resumamos lo que hemos aprendido:

| Método | Requisitos | Detectabilidad | Huella forense |
|--------|------------|----------------|----------------|
| `adb shell input` | ADB habilitado | Muy alta | Logs del sistema, procesos `input` |
| `dispatchGesture` | `AccessibilityService` | Media | `dumpsys accessibility`, estadísticas de trazos |
| `injectInputEvent` | Root | Baja (pero requiere root) | Requiere Frida o módulo kernel para detectar |
| HID externo | Hardware externo | Muy baja | Descriptores USB, correlación de sensores |

Vuestra misión como defensores es implementar una estrategia de detección por capas. No podéis confiar en una sola técnica. Si bloqueáis `adb shell input`, el atacante usará `dispatchGesture`. Si bloqueáis `AccessibilityService`, usarán root. Si bloqueáis root, usarán HID externo. La defensa debe ser proporcional y escalonada.

**Preguntas para reflexionar antes de la tarea:**

1. ¿Qué método creéis que utiliza la mayoría de granjas actuales? ¿Por qué?
2. ¿Cuál es el punto débil común de todos los métodos de software?
3. ¿Cómo integraríais la detección de estos métodos en vuestra plataforma actual?

**Tarea para el Módulo 4:**

Os entregaré un dataset de eventos táctiles (coordenadas, duraciones, presiones) de 10.000 toques, mezclando humanos y bots generados por diferentes métodos. Vuestro trabajo es construir un clasificador que distinga entre toque humano y toque automatizado, con métricas de precisión y recall. Traed los resultados a la próxima sesión.

Además, leed la sección sobre `VpnService` y `ProxySelector` de la documentación de Android. En el Módulo 4 analizaremos cómo GenFarmer enmascara la red.

**Ronda de preguntas (5 minutos).**

¿Alguna duda? Perfecto. Nos vemos la semana que viene.
