
# Curso del Ecosistema GenFarmer

**Duración estimada:** 24 horas lectivas (6 módulos de 4 horas)  
**Prerrequisitos:** Conocimientos de seguridad en Android, ADB, redes TCP/IP, análisis de APK, y fundamentos de proxies y servicios de accesibilidad.



### [Módulo 1: Visión General del Ecosistema Gen y sus Componentes](modulo-1.md)

**Objetivo:** Comprender la arquitectura global, los flujos de datos entre los componentes y el modelo de negocio que sostiene el ecosistema.

1. **El ecosistema Gen como plataforma de automatización industrial**
   - Descripción de los cuatro productos principales: GenLogin, GenFarmer, GenRouter, Box Phone Farm y GenPlay.
   - Cómo se interconectan para crear una cadena de suplantación completa: desde la creación de perfiles de navegador (GenLogin) hasta la ejecución en dispositivos físicos (GenFarmer), pasando por la gestión de red (GenRouter) y la externalización como servicio (GenPlay).
   - Público objetivo declarado: MMO (Make Money Online), marketing digital, gestión de múltiples cuentas. Implicaciones de este posicionamiento para la atribución de intenciones maliciosas.

2. **Box Phone Farm: la capa física**
   - Especificaciones del hardware: capacidad de 10 a más de 40 dispositivos, ventilación controlada por software, hubs USB integrados.
   - Implicaciones forenses: el rack no deja huella lógica en el dispositivo, pero la presencia de un mismo `Build.FINGERPRINT` en todos los terminales conectados al mismo controlador es un indicio de granja física.
   - Conectividad: USB vs. LAN. Cómo la opción LAN a través de GenRouter permite el control remoto de dispositivos sin que el controlador esté físicamente cerca.

3. **GenRouter: la joya de la red**
   - Basado en OpenWrt con módulos personalizados. Funciones: creación de VLANs por campaña, asignación de proxies a nivel de subred, priorización de tráfico.
   - Cómo se sincroniza con el backend en la nube: intercambio de información de topología, licencias y configuración de proxies mediante una API REST.
   - Posibilidad de inspeccionar el tráfico del router para extraer IoCs: dominios de callback, certificados TLS, direcciones IP del backend.

4. **GenPlay: la granja como servicio**
   - Plataforma en la nube que alquila acceso a dispositivos reales alojados en racks de terceros.
   - Riesgos de seguridad para el "cliente" y para la plataforma: exposición de cuentas, falta de control sobre el hardware.
   - Desde la óptica defensiva, GenPlay concentra las IPs de los dispositivos en centros de datos o racks geolocalizados, lo que facilita la creación de blocklists.



### [Módulo 2: Análisis Forense del Agente de Dispositivo (APK de GenFarmer)](modulo-2.md)

**Objetivo:** Aprender a extraer, descompilar y analizar el comportamiento del agente de automatización que se instala en cada teléfono, identificando sus capacidades y sus indicadores de compromiso.

1. **Obtención de la muestra**
   - Fuentes legítimas para el investigador: repositorios de malware, capturas de red en operaciones autorizadas, o adquisición del software con fines de prueba en un entorno aislado.
   - Precauciones: ejecución exclusiva en dispositivos sin cuentas reales, en red aislada y con monitorización de tráfico (mitmproxy).

2. **Ingeniería inversa estática**
   - Descompilación con `apktool` y análisis del manifiesto (`AndroidManifest.xml`):
     - Permisos solicitados: `BIND_ACCESSIBILITY_SERVICE`, `SYSTEM_ALERT_WINDOW`, `INTERNET`, `ACCESS_NETWORK_STATE`, `FOREGROUND_SERVICE`, posiblemente `WRITE_SECURE_SETTINGS`.
     - Declaración del `AccessibilityService`: configuraciones (`android:accessibilityEventTypes`, `android:canPerformGestures`). La presencia de `canPerformGestures="true"` es un fuerte indicio de automatización táctil.
   - Búsqueda de strings en el código (`grep`): URLs del backend, endpoints API, nombres de paquetes de aplicaciones objetivo, comandos ADB embebidos.
   - Análisis de clases Java (con `jadx` o `Bytecode Viewer`): identificación de los métodos de inyección de gestos (`dispatchGesture`, `InputManager.injectInputEvent`), gestión de proxies (`VpnService`, `ProxySelector`), y comunicación con el panel (HTTP, WebSocket).

3. **Comportamiento dinámico en sandbox**
   - Instalación en emulador instrumentado con Frida o Xposed para interceptar llamadas a APIs sensibles.
   - Monitorización de la creación de la VPN local: inspección de la tabla de enrutamiento (`ip route show table all`) y reglas `iptables` cuando el agente activa el túnel. El agente redirige solo el tráfico de apps seleccionadas mediante `uid_owner` si tiene root, o encapsula tráfico completo si usa `VpnService`.
   - Registro de eventos de accesibilidad: observar qué aplicaciones vigila el agente (p.ej., `com.facebook.katana`, `com.instagram.android`) y cómo reacciona a eventos de ventana o notificaciones.

4. **Indicadores de compromiso (IoCs)**
   - Nombres de paquete: si el agente usa un nombre de paquete fijo (p.ej., `com.genfarmer.agent`), es detectable directamente. Las versiones más recientes pueden ofuscar el nombre o permitir personalizarlo.
   - Hashes de archivos y firmas digitales.
   - Permisos de accesibilidad no declarados: cualquier `AccessibilityService` cuyo `description` en el manifiesto no tenga una justificación de accesibilidad real es sospechoso. Esto es detectable con escaneos de ADB: `adb shell settings get secure enabled_accessibility_services`.



### [Módulo 3: Protocolos de Control y Automatización](modulo-3.md)

**Objetivo:** Diseccionar los mecanismos que utiliza GenFarmer para enviar comandos desde el panel a los dispositivos y ejecutarlos, evaluando su detectabilidad.

1. **Comunicación panel-dispositivo**
   - Modelo: el panel de escritorio (Windows/macOS) se comunica con el backend en la nube, que retransmite las órdenes al agente en el teléfono vía WebSocket o MQTT sobre TLS.
   - Alternativa sin nube (local): si el controlador y los dispositivos están en la misma LAN, la comunicación puede ser directa a través de sockets TCP tunelizados sobre ADB (`adb forward`).
   - Análisis de tráfico: captura con `tcpdump` en el dispositivo (si tiene root) o en el router GenRouter. Búsqueda de paquetes periódicos de telemetría (latidos, estado del dispositivo) y de comandos etiquetados (tipo de acción, coordenadas, parámetros).

2. **Métodos de automatización de la entrada táctil (ordenados por detectabilidad)**
   - **`AccessibilityService.dispatchGesture()`**: El método preferido cuando no hay root. El agente crea un `GestureDescription` con trazos de movimiento y lo despacha. El sistema genera un `MotionEvent` genuino con fuente `SOURCE_TOUCHSCREEN`. La detección desde la app objetivo es difícil, pero posible analizando la coherencia de los gestos (velocidad, aceleración, número de trazos). A nivel de sistema, se puede auditar qué servicios de accesibilidad están ejecutando gestos mediante `dumpsys accessibility`.
   - **`adb shell input`**: GenFarmer solo lo utiliza para tareas de mantenimiento (instalación de APKs) o en versiones antiguas. Su uso deja rastro en los logs (`logcat -s input`). Muchas apps monitorizan la propiedad `sys.inputmethod.active` o la presencia de `input` en la lista de procesos.
   - **`InputManager.injectInputEvent()` (root)**: Llamada directa a la API interna de Android. No pasa por el framework de accesibilidad, lo que la hace invisible para `dumpsys accessibility`. Sin embargo, requiere acceso root, que deja huellas como la presencia de `su` o particiones modificadas. Se puede detectar mediante escaneos de integridad del sistema (RootBeer, SafetyNet, Play Integrity).
   - **Emulación HID externa**: El agente de GenFarmer no emplea esta técnica directamente, pero los operadores más sofisticados la combinan con el ecosistema. Un Arduino/Teensy conectado al puerto USB-C del teléfono emula un teclado o pantalla táctil. Los eventos HID son indistinguibles de un periférico real. La defensa se basa en detectar la presencia de un dispositivo USB que se identifica como HID de pantalla táctil cuando el teléfono no tiene ese periférico conectado a su pantalla real (análisis del descriptor USB en `/sys/bus/usb/devices/`).

3. **Ejecución de macros y flujos de trabajo**
   - GenFarmer permite grabar macros o definir flujos visuales (clic en imagen, esperar, escribir). El panel almacena estos scripts en formato JSON/XML. El agente los interpreta secuencialmente.
   - Para la investigación, la captura de estos scripts en tránsito o en reposo (en el almacenamiento del dispositivo, si no está cifrado) revela la lógica de automatización: qué apps se manipulan, qué secuencias de clics se repiten y con qué frecuencia.



### [Módulo 4: Gestión de Identidad y Red](modulo-4.md)
**Objetivo:** Desvelar cómo el ecosistema Gen construye y mantiene identidades digitales falsas creíbles, y cómo se enmascara la red para evadir bloqueos.

1. **GenLogin y la creación de perfiles de navegador**
   - GenLogin no se ejecuta en los teléfonos, sino en el PC. Es un navegador anti-detección que genera huellas digitales únicas (User-Agent, WebGL, Canvas, WebRTC, timezone, idioma) para cada perfil.
   - Se integra con GenFarmer al exportar perfiles de cookies/sesiones que pueden inyectarse en las apps móviles mediante la restauración de backups o la manipulación de `WebView`.
   - Para el defensor, la correlación entre un perfil de GenLogin y un dispositivo de GenFarmer puede romper la compartimentación si se detecta un mismo token de sesión o un patrón de navegación común.

2. **Gestión de proxies en el dispositivo**
   - **Opción A (sin root, VPNService):** el agente crea una interfaz VPN local (`tun0`). Todo el tráfico de la app seleccionada se enruta a través de esta interfaz hacia un proxy SOCKS5 o HTTP remoto. La VPN local puede inspeccionarse con `ifconfig` y `ip route`. La IP pública vista por el servicio remoto será la del proxy, no la de la granja. La detección se basa en que una aplicación de seguridad puede notar la presencia de una VPN no declarada y analizar su destino.
   - **Opción B (con root, iptables):** el agente añade reglas `nat` y `filter` para redirigir el tráfico de una app específica (por UID) hacia un proxy transparente o un túnel SOCKS5. Esto es más sigiloso porque no se crea interfaz virtual visible. Sin embargo, un escaneo de reglas de iptables (`iptables -t nat -L -v`) revela las redirecciones. La presencia de reglas que apuntan a un puerto local (por ejemplo, 8123) controlado por el agente es una clara señal de manipulación.
   - **Rotación de IP:** el agente recibe comandos del panel para cambiar de proxy cada *N* minutos o al detectar un bloqueo. La monitorización de la frecuencia de cambios de IP desde una misma cuenta es una métrica defensiva clásica.

3. **Gestión del Advertising ID (AAID)**
   - Para el fraude de instalación de apps, GenFarmer restablece el AAID después de cada ciclo. El método depende del nivel de acceso:
     - Sin root: `adb shell pm clear com.google.android.gms` (requiere confirmación del usuario, detectable).
     - Con root: inyección de un AAID falso mediante módulos Xposed o manipulación de la base de datos `adid.db`.
   - La defensa consiste en monitorizar el ritmo de cambio de AAID: un dispositivo real no restablece su AAID decenas de veces al día.



### [Módulo 5: Flujos de Trabajo Típicos y su Detección](modulo-5.md)

**Objetivo:** Reconstruir las campañas de automatización más comunes (fraude publicitario, manipulación de redes sociales, farming de cuentas) a partir de los datos forenses recogidos.

1. **Registro masivo de cuentas (account farming)**
   - Secuencia típica: el dispositivo recibe orden de abrir Gmail/Instagram/Facebook → el agente rellena formularios con datos generados por un LLM (ver Módulo 6) → recibe SMS de verificación a través de una SIM física o servicio de SMS → completa el registro.
   - Artefactos: picos de tráfico hacia endpoints de registro, múltiples SMS desde el mismo remitente en un intervalo corto, cuentas con nombres de usuario con patrones (`user_12345`).

2. **Fraude publicitario CPA (Coste por Acción)**
   - GenFarmer se utiliza para instalar y abrir aplicaciones de anunciantes, completar tutoriales, realizar compras in-app simuladas.
   - El flujo completo: limpiar AAID → conectar proxy residencial → abrir Google Play → descargar e instalar app → abrir app → esperar X segundos → interactuar con botones predeterminados → cerrar app.
   - Detectores: atribución de instalación desde IPs de pools de proxies, tiempos de interacción anormalmente rápidos, patrones de clics en coordenadas fijas de los botones de tutorial.

3. **Amplificación en redes sociales**
   - Cuentas falsas gestionadas por GenFarmer reciben instrucciones para seguir, dar me gusta, comentar o compartir publicaciones. Los textos de los comentarios son generados por un LLM local (Dolphin) e inyectados vía portapapeles (`adb shell input text` o `AccessibilityService`). GenFarmer puede coordinar campañas enteras: 100 cuentas comentan en un hilo, 50 le dan like, 20 retuitean, todo sincronizado.
   - Detección: correlación temporal entre cuentas con IPs geográficamente dispersas pero que comparten la misma secuencia de acciones; texto con perplejidad de LLM.



### [Módulo 6: Estrategias de Defensa y Puntos de Ruptura](modulo-6.md)

**Objetivo:** Diseñar un plan de defensa multicapa basado en los hallazgos de los módulos anteriores.

1. **Detección en el dispositivo (cliente)**
   - Escaneo de `AccessibilityService`: buscar servicios no legítimos con capacidad de realizar gestos.
   - Verificación de integridad con Play Integrity API: detectar root, bootloader desbloqueado, emuladores, o módulos de automatización que requieren permisos elevados.
   - Análisis de procesos en segundo plano: buscar procesos con nombres sospechosos (`genfarmer`, `gfarmer`, `agent`).
   - Monitorización de la tabla de enrutamiento y reglas iptables: detección de túneles locales o redirecciones no estándar.

2. **Detección en la red (servidor/plataforma)**
   - Reputación de IP: bloquear IPs de centros de datos y pools de proxies residenciales conocidos. Usar listas actualizadas de proveedores de proxies.
   - Análisis de comportamiento de red: un dispositivo que completa un flujo de registro en 60 segundos con cambios de IP sospechosos es un bot.
   - Fingerprinting de TLS y HTTP/2: los clientes de Android gestionados por GenFarmer pueden mostrar huellas de TLS ligeramente diferentes si el agente interfiere en la pila de red (p.ej., a través de un proxy local). El JA3 fingerprint puede delatar la manipulación.

3. **Detección a nivel de cuenta**
   - Modelos de ML entrenados con la actividad de cuentas legítimas vs. cuentas de granja. Variables: tiempo entre acciones, variabilidad de coordenadas de toque, patrones de actividad diaria.
   - Honeypots: desplegar cuentas señuelo con contenido absurdo; si una cuenta interactúa, es un bot.

4. **Neutralización y acción legal**
   - Cese y desistimiento a proveedores de servicios que alimentan el ecosistema (proxies, SMS, SIM farms).
   - Colaboración con fuerzas de seguridad para desmantelar los centros de mando y control (backend en la nube de GenFarmer, GenPlay). Análisis de los dominios y certificados para identificar a los operadores.
   - Intercambio de IoCs entre plataformas para construir una base de datos unificada de indicadores de compromiso del ecosistema Gen.

---

### Evaluación Final

Para completar el curso, los alumnos deberán:
- Realizar el análisis forense completo de una muestra del agente GenFarmer en un sandbox, documentando todos los IoCs encontrados.
- Proponer una estrategia de detección para una plataforma de redes sociales que quiera identificar cuentas gestionadas por este ecosistema, detallando qué señales utilizaría y cómo las combinaría.
- Analizar un volcado de tráfico de red proveniente de un GenRouter (simulado) para identificar patrones de automatización y extraer los endpoints de mando y control.



*Este curso tiene fines exclusivamente educativos y defensivos. Toda manipulación de software y hardware debe realizarse en entornos controlados y sobre dispositivos de los que se es propietario, cumpliendo la legislación vigente.*De acuerdo. A continuación presento un curso estructurado en seis módulos que analiza en profundidad el ecosistema GenFarmer desde la óptica exclusiva de la investigación en ciberseguridad, el análisis forense y el diseño de contramedidas. El objetivo es capacitar al profesional para identificar, desmontar y neutralizar las capacidades técnicas de esta plataforma, nunca para operarla.

