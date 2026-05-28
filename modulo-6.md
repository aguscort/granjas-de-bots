**MÓDULO 6: ESTRATEGIAS DE DEFENSA Y PUNTOS DE RUPTURA**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, seréis capaces de diseñar un plan de defensa multicapa contra las granjas de dispositivos automatizados, basado en los hallazgos de los módulos anteriores, y de identificar los puntos de ruptura del ecosistema Gen para su neutralización.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza.*

Hemos llegado al último módulo del curso. En las sesiones anteriores hemos diseccionado el ecosistema Gen desde todos los ángulos: su arquitectura, su agente APK, sus protocolos de control, su gestión de identidad y red, y sus flujos de fraude. Ahora toca la parte más importante: **¿cómo lo paramos?**

Hoy no vamos a aprender nuevas técnicas del adversario, sino a construir nuestras defensas. Vamos a invertir la perspectiva. Durante todo el curso hemos sido forenses analizando un cadáver digital; ahora seremos ingenieros de defensa diseñando un castillo con sus murallas, fosos y torres de vigilancia.

Estructuraremos la defensa en cuatro capas, que se corresponden con las capas que hemos analizado del ecosistema Gen: defensa en el **dispositivo** (cliente), defensa en la **red** (servidor/plataforma), defensa a nivel de **cuenta** (modelos de comportamiento), y finalmente **neutralización y acción legal**. En cada capa, identificaremos los puntos de ruptura: esos eslabones débiles donde una pequeña intervención puede desmoronar toda la operación.

Al final del módulo, tendréis un framework mental que podréis aplicar en vuestras organizaciones para detectar, bloquear y desmantelar granjas de bots. ¿Listos para la batalla final?

---

**2. DETECCIÓN EN EL DISPOSITIVO (CLIENTE) (60 minutos)**

*El profesor muestra un esquema de las defensas on-device.*

Empecemos por la primera línea de defensa: el propio dispositivo. Si conseguimos que el teléfono sea un entorno hostil para el agente de GenFarmer, habremos cortado el problema de raíz.

**2.1 Escaneo de AccessibilityService (20 minutos)**

Como vimos en el Módulo 2, el talón de Aquiles del agente es su dependencia del `AccessibilityService`. Sin él, no puede automatizar toques sin root. Por tanto, la detección de servicios de accesibilidad maliciosos es nuestra prioridad número uno.

¿Cómo se implementa en la práctica? Vuestra app o vuestro SDK de seguridad puede ejecutar periódicamente:

```bash
adb shell settings get secure enabled_accessibility_services
```

Este comando devuelve una lista de los servicios activos, separados por dos puntos. Por ejemplo:
```
com.genfarmer.agent/.MyAccessibilityService:com.legit.app/.LegitService
```

Lo primero es analizar cada servicio. Preguntas que debe hacerse vuestro detector:
1. ¿El servicio declara `canPerformGestures="true"`? Si es así, tiene capacidad para simular toques. ¿Está justificado? Un lector de pantalla legítimo (TalkBack) lo necesita; una app de linterna, no.
2. ¿El nombre del paquete es conocido? Podéis mantener una blacklist de paquetes de automatización (GenFarmer, MacroDroid, AutoClicker, etc.). Pero cuidad con los falsos positivos: las herramientas de accesibilidad para personas con discapacidad deben estar en una whitelist.
3. ¿Cuál es el `description` declarado en el manifiesto? Si es genérico o sospechoso (“Servicio de ayuda para la app”), es una bandera roja.

**Implementación de un detector (código de ejemplo):**
```java
AccessibilityManager am = (AccessibilityManager) context.getSystemService(Context.ACCESSIBILITY_SERVICE);
List<AccessibilityServiceInfo> services = am.getInstalledAccessibilityServiceList();
for (AccessibilityServiceInfo info : services) {
    if ((info.flags & AccessibilityServiceInfo.FLAG_REQUEST_TOUCH_EXPLORATION_MODE) != 0
        || info.getCapabilities() != null && (info.getCapabilities() & AccessibilityServiceInfo.CAPABILITY_CAN_PERFORM_GESTURES) != 0) {
        // Analizar paquete, nombre, etc.
    }
}
```
Y monitorizad también cambios en tiempo real: cuando se active un nuevo servicio de accesibilidad, el sistema emite un broadcast que podéis capturar para lanzar un escaneo inmediato.

**Ejercicio práctico (10 minutos):**
Ejecutad el script detector que os proporciono en el emulador donde tenéis instalado el agente GenFarmer. Debe listar los servicios de accesibilidad y marcar el del agente como sospechoso. Luego, responded: ¿qué haríais si vuestro detector encuentra un servicio sospechoso? ¿Lo bloqueáis automáticamente o lo reportáis? Discutid el equilibrio entre seguridad y falsos positivos.

**2.2 Verificación de integridad con Play Integrity API (20 minutos)**

*El profesor pasa a la siguiente diapositiva.*

El agente de GenFarmer funciona mejor con root, y los emuladores requieren módulos Magisk para ocultar su naturaleza. Aquí entra en juego la **Play Integrity API** (sucesora de SafetyNet). Esta API permite a vuestra app preguntar a Google: “¿Este dispositivo es íntegro?”. La respuesta incluye:

- **Integridad del dispositivo:** si el bootloader está desbloqueado, si el sistema ha sido modificado, si hay root.
- **Integridad de la app:** si vuestra app ha sido manipulada o se ejecuta en un entorno no reconocido.
- **Licensing:** si el usuario realmente instaló vuestra app desde Google Play.

Para el defensor, la integridad del dispositivo es el campo clave. Un dispositivo de granja con root devolverá un veredicto como `MEETS_BASIC_INTEGRITY` pero no `MEETS_DEVICE_INTEGRITY`. Un emulador directamente no superará ninguna verificación.

**Implementación básica:**
```java
IntegrityManager integrityManager = IntegrityManagerFactory.create(context);
Task<IntegrityTokenResponse> task = integrityManager.requestIntegrityToken(
    new IntegrityTokenRequest.builder().setNonce(randomNonce).build()
);
task.addOnSuccessListener(response -> {
    String verdict = response.token(); // Enviar a vuestro servidor para validación
});
```

**Precaución:** No toméis decisiones de bloqueo solo en el cliente. El servidor de Google valida el token, no la app. Implementad siempre la verificación en vuestro backend.

**Ejercicio práctico (15 minutos):**
He desplegado una app de prueba en el emulador rooteado y en un dispositivo físico limpio. Usando Play Integrity API, llamad a la API y comparad los veredictos. Documentad la diferencia. Luego, proponed una política de bloqueo: ¿bloquearíais a los usuarios con `MEETS_BASIC_INTEGRITY` pero sin `MEETS_DEVICE_INTEGRITY`? ¿Por qué? ¿Qué implicaciones tendría para usuarios con ROMs personalizadas legítimas?

**2.3 Análisis de procesos e interfaces de red (15 minutos)**

Dos chequeos más ligeros pero útiles:

- **Procesos sospechosos en segundo plano:** Podéis leer `/proc/self/maps` o listar procesos con `ActivityManager.getRunningAppProcesses()`. Buscad procesos con nombres como `genfarmer`, `gfarmer`, `autoclick`, `macro`. Esto es frágil (los nombres cambian), pero combinado con otros factores suma puntos de riesgo.
- **Interfaces de red no estándar:** Como vimos en el Módulo 4, `VPNService` crea una interfaz `tun0`. Podéis listar interfaces con `NetworkInterface.getNetworkInterfaces()`. Si encontráis `tun0` y no hay una app de VPN legítima en ejecución, es un fuerte indicio de túnel de proxy.

**Ejercicio práctico (5 minutos):**
Desde el emulador con el agente activo, ejecutad el script de detección de interfaces de red. ¿Aparece `tun0`? ¿Podéis identificar qué proceso la creó? Discutid cómo automatizar esta comprobación.

*Pausa de 15 minutos.*

---

**3. DETECCIÓN EN LA RED (SERVIDOR/PLATAFORMA) (45 minutos)**

*El profesor retoma.*

Subimos una capa. Ya no estamos en el dispositivo, sino en vuestros servidores: la red social, la tienda de aplicaciones, la plataforma publicitaria. Aquí tenéis una visión global del tráfico entrante, y podéis aplicar reglas a gran escala.

**3.1 Reputación de IP y detección de proxies (15 minutos)**

El punto más obvio, pero no por ello menos importante. Debéis mantener una base de datos actualizada de IPs de proxies residenciales, centros de datos y VPNs. Servicios como IPQualityScore, MaxMind o Spur.us proporcionan feeds de IPs con etiquetas de riesgo.

¿Cómo usarlos en la práctica? En tiempo real, vuestro sistema de enrutamiento o firewall debe consultar la reputación de la IP entrante. Si la IP está etiquetada como "residential proxy" o "hosting", se aplica una política restrictiva: CAPTCHA, limitación de tasa, o directamente bloqueo.

Pero cuidad con el bloqueo ciego: los proxies residenciales son, por definición, IPs de hogares reales. Bloquearlas puede afectar a usuarios legítimos cuyo tráfico pasa por un proxy por razones laborales. La clave es **correlacionar**: una IP de proxy residencial que intenta crear 50 cuentas en una hora es fraudulenta; una que solo hace una consulta, puede que no.

**3.2 Fingerprinting de TLS (JA3/JA4) y HTTP/2 (15 minutos)**

*El profesor muestra una captura de Wireshark con JA3 fingerprints.*

Cada cliente TLS tiene una "huella" única en la negociación inicial (JA3 fingerprint). Un navegador Chrome en Windows produce un JA3 diferente al de un script Python con la librería `requests`. GenFarmer, al ejecutarse en Android pero con un agente que interfiere en la pila de red (túneles VPN), puede tener un JA3 ligeramente anómalo.

Podéis generar una base de datos de JA3 legítimos (por ejemplo, Chrome en Android, WebView, OkHttp) y marcar como sospechosos aquellos que se desvíen. Esto requiere un esfuerzo inicial de recolección, pero es muy efectivo.

**Implementación:** Herramientas como Zeek (antes Bro) pueden extraer JA3 de vuestro tráfico de red. Luego, un modelo simple compara el JA3 observado con los de vuestra whitelist.

**Ejercicio práctico (15 minutos):**
Os entrego un PCAP con tráfico de varias conexiones TLS, algunas legítimas (Chrome, Firefox) y otras generadas por un script de automatización (simulación de GenFarmer). Con Zeek o con `tshark`, extraed los JA3 y clasificad las conexiones como legítimas o sospechosas. Discutid los resultados.

**3.3 Análisis de comportamiento de red (15 minutos)**

Más allá de IP y JA3, el comportamiento de red es muy revelador. Una granja deja patrones que podéis detectar con reglas de correlación en vuestro SIEM:

- **Ráfagas de registros o instalaciones desde un mismo rango de IPs** (aunque sean IPs distintas, pueden pertenecer al mismo pool).
- **Secuencias de eventos a velocidad sobrehumana:** una instalación de app seguida de apertura, tutorial completado en 10 segundos, desinstalación a los 5 minutos.
- **Tráfico de telemetría periódico:** si veis peticiones a un mismo dominio cada 30 segundos desde cientos de dispositivos, es una conexión persistente de agentes.

**Ejercicio práctico (10 minutos):**
Con los logs de acceso a una API que os proporciono, identificad sesiones con patrones de velocidad anómala (más de X peticiones en Y segundos) y relaciones entre IPs (mismo rango /24). Escribid una regla de detección en pseudocódigo.

---

**4. DETECCIÓN A NIVEL DE CUENTA (MODELOS DE COMPORTAMIENTO) (45 minutos)**

*El profesor cambia a la capa de cuenta.*

Subimos la última capa técnica: la cuenta de usuario. Incluso si un dispositivo supera todas las defensas anteriores, el comportamiento de la cuenta a lo largo del tiempo puede delatarla.

**4.1 Modelado de actividad circadiana (15 minutos)**

Los humanos dormimos. Los bots, si no se programan pausas, no. Un modelo simple puede calcular las horas de inactividad de una cuenta. Si una cuenta publica 24/7, 365 días al año, es un bot.

Más sutil: las granjas programan pausas, pero suelen ser fijas (por ejemplo, 6 horas de inactividad de 02:00 a 08:00). Un humano tiene variabilidad: un día duerme 7 horas, otro 5, otro se despierta a las 03:00 para ir al baño y mira el móvil. La entropía de los patrones de sueño es más alta en humanos.

Implementad un modelo que calcule la desviación estándar de las horas de inicio y fin de actividad diaria. Si es muy baja (siempre se acuesta a las 23:00 y se despierta a las 07:00 exactas), es sospechoso.

**4.2 Análisis de gestos táctiles (15 minutos)**

Recordad del Módulo 3: los toques de los bots, incluso con `dispatchGesture`, son demasiado perfectos. Podéis registrar las coordenadas, duraciones y presiones de los eventos táctiles en vuestra app (si estáis en una posición privilegiada, como una app bancaria o un juego). Luego, entrenar un clasificador (SVM o red neuronal simple) para distinguir humano de bot.

Características útiles:
- Velocidad media y máxima del trazo.
- Aceleración (derivada de la velocidad).
- Curvatura de la trayectoria (desviación de una línea recta).
- Variación de la presión (si el dispositivo lo soporta).
- Tiempo entre toque y toque.

Un bot que toca siempre con la misma duración (150 ms) y en línea recta tendrá una puntuación de bot alta.

**4.3 Detección de texto generado por LLM (15 minutos)**

*El profesor enlaza con el Módulo 6 sobre IA (que veremos en profundidad en un curso aparte, pero damos una introducción aquí).*

Como vimos en el Módulo 5, los textos generados por Dolphin o Llama 3 tienen características estadísticas medibles: perplejidad uniforme, baja burstiness, ciertos artefactos de fine-tuning.

Para la defensa, podéis integrar herramientas como GPTZero o modelos open-source como RoBERTa fine-tuned para detección de texto sintético. Pero recordad: el adversario puede postprocesar el texto para añadir errores y variabilidad. La carrera es constante.

**Ejercicio práctico (15 minutos):**
Os entrego un dataset de 200 textos, 100 humanos y 100 generados por Dolphin 2.9 (simulados). Usando una herramienta de detección online o un script simple de entropía, clasificad los textos y calculad la precisión y recall. Discutid las limitaciones.

---

**5. NEUTRALIZACIÓN Y ACCIÓN LEGAL (30 minutos)**

*El profesor proyecta una diapositiva con logos de organismos internacionales.*

No basta con detectar. Debemos desmantelar. La defensa técnica debe combinarse con acciones legales y colaboración industrial.

**5.1 Estrangulamiento de servicios auxiliares (10 minutos)**

Recordad que las granjas dependen de servicios externos: proxies residenciales, servicios de SMS, GenLogin, GenFarmer (como software). Si cortamos esos suministros, la granja se asfixia.

- **Proveedores de proxies:** trabajad con vuestros equipos legales para enviar cartas de cese y desistimiento a los proveedores que se anuncian abiertamente como herramientas para fraude. Muchos están en jurisdicciones donde el fraude publicitario es delito.
- **Servicios de SMS:** monitorizad los números de teléfono utilizados para verificaciones. Si un mismo número verifica 100 cuentas en una hora, bloqueadlo y compartid el IOC.
- **Plataformas de pago:** si la granja monetiza mediante CPA, follow the money. Identificad a los afiliados fraudulentos y cerrad sus cuentas en la red de afiliados.

**5.2 Colaboración con fuerzas de seguridad (10 minutos)**

Los casos de granjas a gran escala pueden constituir delitos de fraude informático, blanqueo de capitales y asociación ilícita. Documentad cuidadosamente las evidencias forenses (como las que habéis aprendido a recoger en este curso) y presentadlas a las unidades de cibercrimen. Operaciones como el desmantelamiento de ProxySmart demuestran que la acción policial coordinada funciona.

**5.3 Intercambio de IoCs y estándares abiertos (10 minutos)**

La defensa colectiva es más fuerte. Compartid IoCs con otras plataformas a través de grupos como APWG (Anti-Phishing Working Group) o M3AAWG (Messaging, Malware and Mobile Anti-Abuse Working Group). Adoptad estándares como **IAB Tech Lab Invalid Traffic Detection and Filtration** para asegurar que toda la industria habla el mismo idioma.

---

**6. CIERRE DEL CURSO Y TAREA FINAL (30 minutos)**

*El profesor muestra una diapositiva con los seis módulos y las competencias adquiridas.*

Hemos llegado al final del curso. Hace seis módulos, empezamos viendo el ecosistema Gen desde fuera. Ahora sois capaces de:

- Dibujar su arquitectura completa.
- Realizar un análisis forense del agente APK y extraer IoCs.
- Entender y detectar los diferentes métodos de inyección táctil.
- Desvelar cómo enmascaran la identidad y la red.
- Reconstruir campañas de fraude a partir de datos dispersos.
- Y, finalmente, diseñar defensas multicapa y puntos de ruptura.

**Lo que no hemos cubierto (y deberíais explorar por vuestra cuenta):**
- Defensa contra HID externo (Arduino/Teensy) a nivel de kernel.
- Análisis de radiofrecuencia para detectar granjas de SIMs.
- Técnicas avanzadas de detección de deepfakes en fotos de perfil y videos generados.

**Tarea final (para certificación):**
Os entrego un escenario completo: un volcado de tráfico de red, un conjunto de logs de dispositivos (simulados), y un dataset de eventos de una plataforma social. Vuestro objetivo es determinar si hay una granja operando, cuántos dispositivos la componen, qué método de automatización usan, qué campaña de fraude ejecutan, y proponer un plan de defensa concreto para la plataforma. Debéis entregar un informe detallado con vuestras conclusiones y las evidencias que las soportan.

Tenéis dos semanas para entregarlo.

**Ronda final de preguntas y despedida (15 minutos).**

Quiero agradeceros vuestro esfuerzo y participación. La lucha contra las granjas de bots es una carrera armamentística, pero con el conocimiento que habéis adquirido, estáis en la primera línea de defensa. Mucha suerte y nos vemos en futuras sesiones avanzadas.
