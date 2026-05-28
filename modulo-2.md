**MÓDULO 2: ANÁLISIS FORENSE DEL AGENTE DE DISPOSITIVO (APK DE GENFARMER)**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar la sesión, seréis capaces de obtener una muestra del agente GenFarmer, realizar una ingeniería inversa completa (estática y dinámica) y extraer un conjunto de indicadores de compromiso (IoCs) utilizables en vuestros entornos defensivos.

---

**1. INTRODUCCIÓN AL MÓDULO (10 minutos)**

*El profesor comienza.*

Bienvenidos al Módulo 2. En la sesión anterior vimos el ecosistema Gen desde arriba, como quien mira un mapa. Hoy vamos a coger una lupa y a examinar una pieza concreta: el **agente de GenFarmer**, la APK que se instala en cada uno de los teléfonos de la granja.

¿Por qué es tan importante esta APK? Porque es el eslabón más débil de la cadena. GenLogin está en el PC del operador, GenRouter en un armario de red; son difíciles de alcanzar. Pero el agente se despliega en cientos o miles de dispositivos. Si nosotros, como defensores, somos capaces de detectar esta APK, habremos encontrado un indicador de compromiso de altísima fidelidad. Si la APK está presente, ese dispositivo forma parte de una granja. Punto.

Hoy vamos a hacer exactamente lo que haríais en un laboratorio forense: obtener la muestra, diseccionarla en estático, ejecutarla en un entorno controlado, observar su comportamiento y, finalmente, extraer las firmas que nos permitan detectarla en el mundo real.

Estructura de la sesión: primero, cómo obtener la muestra de forma legal y segura. Segundo, análisis estático: descompilación, lectura del manifiesto, búsqueda de cadenas y análisis del código Java. Tercero, análisis dinámico en sandbox: Frida, interceptación de tráfico y monitorización de eventos. Y cuarto, redacción de un informe de IoCs. Al final, tendréis una metodología completa que podréis aplicar a cualquier APK maliciosa.

---

**2. OBTENCIÓN DE LA MUESTRA (30 minutos)**

*El profesor proyecta una diapositiva con el título "Adquisición de la evidencia".*

Antes de analizar, necesitamos la muestra. ¿Cómo conseguimos el archivo APK del agente GenFarmer? Aquí entramos en terreno delicado. No vamos a piratear nada. Vamos a utilizar fuentes legítimas para un investigador de seguridad.

**Fuentes legítimas:**

1. **Adquisición directa del dispositivo:** si durante una investigación autorizada (por ejemplo, un empleado que ha cedido su dispositivo para un análisis o un terminal incautado con orden judicial) encontráis la APK instalada, podéis extraerla con ADB:
   ```bash
   adb shell pm path com.genfarmer.agent
   adb pull /data/app/com.genfarmer.agent-xxx/base.apk
   ```
   Este método es el más fiable porque obtenéis la versión exacta que se está utilizando en producción.

2. **Repositorios de malware y sandboxes públicas:** plataformas como VirusTotal, Hybrid Analysis o MalwareBazaar a menudo albergan muestras de este tipo de herramientas. Buscad por nombres de paquete conocidos (`com.genfarmer.agent`, `com.gfarmer`, `com.gen.agent`) o por hashes de versiones anteriores. La ventaja es que podéis acceder a múltiples variantes y estudiar la evolución del software.

3. **Compra del software con fines de prueba:** el propio ecosistema Gen vende sus productos abiertamente. Nada impide que, como investigadores, adquiráis una licencia de GenFarmer para fines de análisis en un entorno completamente aislado. Es éticamente cuestionable financiar a estos actores, pero en algunos marcos legales es la única forma de obtener la muestra sin vulnerar la ley.

**Precauciones imprescindibles:**

*El profesor subraya este apartado.*

Jamás, repito, jamás instaléis esta APK en un dispositivo que contenga cuentas personales o datos sensibles. Vamos a trabajar en un emulador o en un teléfono de laboratorio, con una red completamente aislada. Todo el tráfico será interceptado con mitmproxy. No queremos que el agente "llame a casa" y delate nuestra investigación, ni queremos que se apodere de ninguna cuenta real.

**Ejercicio práctico (10 minutos):**
Os he dejado en la carpeta compartida tres ficheros APK. Son muestras reales de agentes GenFarmer de distintas versiones, obtenidas de repositorios públicos. Descargadlas en vuestra máquina virtual de análisis. Aseguraos de que la máquina no tiene conexión a internet antes de manipularlas. Calculad el hash SHA256 de cada una y anotadlo. Nos servirá para referenciarlas durante el resto del módulo.

---

**3. ANÁLISIS ESTÁTICO (90 minutos - con pausa de 15 minutos intercalada)**

*El profesor muestra una diapositiva con las herramientas: apktool, jadx, grep, strings.*

Vamos a abrir la APK sin ejecutarla. Esto se llama análisis estático. Usaremos tres herramientas: `apktool` para descompilar los recursos y el manifiesto, `jadx` para ver el código Java, y los comandos `strings` y `grep` para búsquedas rápidas.

**Paso 1: Descompilación con apktool (10 minutos)**
Ejecutad en vuestra terminal:
```bash
apktool d genfarmer_agent_v1.apk -o output_dir
```
Esto os generará una carpeta con el contenido descomprimido y, lo más importante, el `AndroidManifest.xml` en formato legible. Abridlo ahora.

**Paso 2: Análisis del manifiesto (30 minutos)**

*El profesor va guiando la lectura.*

Empecemos por los permisos. ¿Qué permisos solicita la APK? Buscad las etiquetas `<uses-permission>`. Deberíais encontrar al menos:

- `android.permission.BIND_ACCESSIBILITY_SERVICE`: esto es la clave. Obliga al usuario a activar el servicio de accesibilidad del agente en los ajustes. Sin este permiso, el agente no puede simular toques ni leer la pantalla de otras apps. Cualquier APK que pida esto y no sea una app de accesibilidad legítima (como TalkBack) debe disparar todas las alarmas.
- `android.permission.SYSTEM_ALERT_WINDOW`: permite dibujar sobre otras aplicaciones. Se usa para superponer interfaces falsas (overlays) o para mantener el servicio en primer plano.
- `android.permission.INTERNET`, `ACCESS_NETWORK_STATE`: comunicación con el panel de control.
- `FOREGROUND_SERVICE`: para ejecutarse en segundo plano sin ser destruido por el sistema.
- Posiblemente `WRITE_SECURE_SETTINGS` si el agente modifica configuraciones del sistema.

Ahora buscad la declaración del `AccessibilityService`. Estará dentro de `<application>`, en una etiqueta `<service>` con un `intent-filter` para `android.accessibilityservice.AccessibilityService`. Mirad los metadatos. Encontrareis un fichero XML (por ejemplo, `accessibility_service_config.xml`) donde se especifica:

- `android:accessibilityEventTypes`: qué eventos escucha (normalmente `typeAllMask` o eventos de ventana y notificaciones).
- `android:canPerformGestures="true"`: **este es el indicador forense más potente**. Significa que el servicio puede ejecutar `dispatchGesture()`, es decir, simular toques y deslizamientos. Un servicio de accesibilidad legítimo rara vez necesita esto, salvo asistentes de voz muy concretos.
- `android:packageNames`: si está restringido a ciertos paquetes (por ejemplo, `com.facebook.katana`), es una prueba de que el agente está dirigido a apps específicas.

**Ejercicio práctico (15 minutos):**
De la muestra que estáis analizando, extraed todos los permisos solicitados y la configuración completa del `AccessibilityService`. Responded por escrito: ¿podría esta app confundirse con una herramienta de accesibilidad legítima? ¿Por qué?

*Pausa de 15 minutos.*

**Paso 3: Búsqueda de cadenas y endpoints (30 minutos)**

*El profesor retoma la clase.*

Volvemos. Ahora vamos a buscar texto dentro del código y los recursos. Las APK maliciosas a menudo contienen URLs, nombres de servidores o tokens que nos delatan. Usad el comando `strings` sobre el archivo `classes.dex` o sobre los ficheros de la carpeta `res`:

```bash
strings classes.dex | grep -i "http"
strings classes.dex | grep -i "api"
strings classes.dex | grep -E "[a-zA-Z0-9.-]+\.(com|io|net)"
```

¿Qué podéis encontrar?:
- URLs del backend: `api.genfarmer.com`, `sync.genlogin.io`.
- Endpoints de la API: `/v1/device/register`, `/task/status`, `/proxy/assign`.
- Nombres de paquetes de aplicaciones objetivo: `com.instagram.android`, `com.facebook.katana`.
- Credenciales embebidas (en versiones muy descuidadas): tokens de API o claves de cifrado.

También buscad en los recursos (`res/values/strings.xml`) cadenas que se muestran al usuario: mensajes como "Activar servicio de accesibilidad" o "Conectando con el servidor...". Esto os dará pistas sobre cómo el agente convence a la víctima (si se instala mediante ingeniería social) para que conceda permisos.

**Paso 4: Análisis del código Java con jadx (45 minutos)**

*El profesor abre jadx con una muestra y comparte pantalla.*

Ahora viene lo más profundo: leer el código. Abrid jadx y cargad la APK. Navegad por los paquetes. Normalmente el código no está ofuscado (estos productos no se esconden; se venden abiertamente), así que los nombres de clases y métodos son bastante descriptivos.

Buscad las siguientes clases y métodos:

- **Clase de inyección de gestos:** buscad `dispatchGesture`, `GestureDescription`, `AccessibilityService`. Estas son las funciones que simulan los toques. Veréis cómo se construyen rutas de dedo (`Path`) y se despachan.
- **Clase de gestión de red:** buscad `VpnService`, `ProxySelector`, `OkHttpClient`, `retrofit`. Estas clases establecen el túnel VPN o configuran el proxy para la app objetivo. Observad si se utiliza `iptables` (requiere root) o se crea una interfaz `tun0`.
- **Clase de comunicación con el panel:** buscad `WebSocket`, `MQTT`, `retrofit`, `OkHttp`. Esta es la conexión con el servidor de mando y control. Identificad la URL base y los parámetros de autenticación (normalmente un token de dispositivo).

**Ejercicio práctico (20 minutos):**
En la clase que gestiona la inyección de gestos, localizad el método exacto que ejecuta un toque en coordenadas `(x, y)`. Escribid un breve resumen de cómo lo hace: ¿usa `dispatchGesture`, `input tap` o `InputManager.injectInputEvent`? ¿Cómo aleatoriza las coordenadas para simular un humano? ¿Qué rango de retardo introduce entre toque y toque?

**Paso 5: Primer borrador de IoCs estáticos (10 minutos)**
Con lo que habéis visto, ya podéis empezar a listar indicadores de compromiso:

- Nombre(s) de paquete (`com.genfarmer.agent`, etc.).
- Hash SHA256 del archivo APK.
- Permisos sospechosos combinados (`BIND_ACCESSIBILITY_SERVICE` + `SYSTEM_ALERT_WINDOW`).
- Configuración del `AccessibilityService` con `canPerformGestures=true`.
- URLs de backend hardcodeadas.
- Firmas digitales (certificado de la APK).

Guardad esta lista; la ampliaremos con el análisis dinámico.

---

**4. ANÁLISIS DINÁMICO EN SANDBOX (60 minutos)**

*El profesor muestra el entorno de laboratorio: un emulador Android sin conexión a internet salvo la red interna del laboratorio, con mitmproxy configurado.*

Una imagen vale más que mil líneas de código. Vamos a ejecutar la APK en un entorno controlado y observar qué hace realmente.

**Configuración del entorno:**
1. Emulador de Android Studio con una imagen x86_64, API 30, sin Google Play (para evitar SafetyNet).
2. Red interna del laboratorio: `192.168.100.0/24`, con salida a internet solo a través de un proxy transparente (mitmproxy) para inspeccionar el tráfico.
3. Frida-server instalado en el emulador para instrumentación dinámica.
4. ADB conectado.

**Paso 1: Instalación y primeras interacciones (15 minutos)**
Instalamos la APK:
```bash
adb install genfarmer_agent_v1.apk
```
Al abrir la app, probablemente solicitará activar el servicio de accesibilidad. En nuestro laboratorio, lo activamos manualmente para ver cómo se comporta. Observad los logs:
```bash
adb logcat | grep -i genfarmer
```
Anotad cualquier conexión saliente, errores o mensajes de registro.

**Paso 2: Instrumentación con Frida (30 minutos)**

*El profesor comparte pantalla con una consola de Frida.*

Frida nos permite engancharnos a procesos en tiempo real. Vamos a rastrear las llamadas a métodos sensibles. Inyectamos un script Frida para monitorizar `dispatchGesture`:
```javascript
Java.perform(function() {
    var Gesture = Java.use('android.accessibilityservice.GestureDescription');
    Gesture.StrokeDescription.$init.overload('android.graphics.Path', 'long', 'long').implementation = function(path, startTime, duration) {
        console.log('[+] dispatchGesture called with duration: ' + duration);
        return this.$init(path, startTime, duration);
    };
});
```
Lanzamos el script y, desde el panel de control de GenFarmer (simulado o real si tenemos una instancia aislada), enviamos una orden de toque. Veremos la salida en consola, lo que confirma que el agente está utilizando `dispatchGesture`.

También podemos rastrear la creación de la VPN:
```javascript
var VpnService = Java.use('android.net.VpnService');
VpnService.onRevoke.implementation = function() {
    console.log('[+] VPN revoked');
};
// También podemos interceptar el establecimiento de la interfaz tun0
```

**Ejercicio práctico (15 minutos):**
Ejecutad el script Frida de monitorización de gestos y confirmad en vuestro emulador que el agente efectivamente llama a `dispatchGesture` cuando recibe una orden. Capturad la salida y documentad la latencia entre orden y ejecución.

**Paso 3: Monitorización del tráfico de red (15 minutos)**
Con mitmproxy activo, solicitamos una tarea que implique acceder a una app objetivo (por ejemplo, una app de prueba que hemos creado). Observamos:
- Las peticiones DNS: ¿hacia dónde resuelve?
- Las conexiones TLS: ¿qué SNI se negocia? ¿Podemos ver el certificado del servidor de Gen?
- Si el tráfico va por un túnel VPN, a nivel de red veremos una conexión persistente hacia el servidor proxy. Si el agente usa `iptables`, el tráfico de la app objetivo saldrá directamente por la interfaz WiFi pero enrutado al proxy.

**Ejercicio práctico (10 minutos):**
Con el tráfico capturado en mitmproxy, identificad la petición de registro del dispositivo al panel de control. Extraed los parámetros que envía: ¿qué información del dispositivo se transmite (modelo, IMEI, Android ID, número de serie)? Esto es valioso para entender qué huellas digitales recopila el ecosistema.

**Paso 4: Ampliación de IoCs con datos dinámicos (10 minutos)**
Añadimos a nuestra lista de IoCs:
- Dominios y direcciones IP del servidor de mando y control.
- Patrones de tráfico: peticiones periódicas de telemetría, protocolos específicos (WebSocket, MQTT).
- Comportamiento del sistema: aparición de una interfaz de red `tun0` o reglas `iptables` añadidas tras la instalación de la APK.
- Eventos de accesibilidad: si se activa un `AccessibilityService` con `canPerformGestures=true` que escucha paquetes como `com.facebook.katana`, es una firma de altísima confianza.

---

**5. CIERRE DEL MÓDULO Y TAREA (15 minutos)**

*El profesor muestra una diapositiva con un resumen de los IoCs estáticos y dinámicos.*

Hemos diseccionado el agente GenFarmer. Ahora tenéis una metodología sólida. ¿Qué os lleváis de esta sesión?

1. El agente es la pieza más vulnerable del ecosistema. Se despliega masivamente, es detectable y proporciona IoCs de altísima fidelidad.
2. El análisis estático revela su propósito: permisos como `BIND_ACCESSIBILITY_SERVICE` y `canPerformGestures=true` son imposibles de ocultar si queréis automatizar toques.
3. El análisis dinámico confirma la teoría y proporciona datos de red y comportamiento en tiempo real.
4. Con esta información, podéis crear reglas YARA para escanear repositorios de APK, configurar SIEMs para detectar tráfico hacia los dominios del backend, o escribir políticas de detección en vuestras plataformas (Google Play Protect, MDM empresarial).

**Tarea para el Módulo 3:**
Os enviaré un volcado de tráfico de un GenRouter capturado durante una campaña real de fraude. Quiero que, antes de la próxima sesión, identifiquéis:
- Cuántos dispositivos distintos aparecen.
- Qué tipo de tareas están ejecutando (registro, instalación de apps, interacción en redes sociales).
- Los endpoints de la API del panel de control que se repiten.

Esto nos servirá de puente hacia el siguiente módulo, donde analizaremos los protocolos de control y la automatización a escala de flota.

**Ronda de preguntas final (5 minutos).**

¿Alguna duda? Perfecto. Nos vemos en el Módulo 3.
