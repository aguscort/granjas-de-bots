**MÓDULO 5: FLUJOS DE TRABAJO TÍPICOS Y SU DETECCIÓN**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, seréis capaces de reconstruir las campañas de automatización más comunes (registro masivo de cuentas, fraude publicitario CPA y amplificación en redes sociales) a partir de evidencias forenses dispersas, y diseñar reglas de detección específicas para cada una.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza.*

Bienvenidos al Módulo 5. Hasta ahora hemos analizado piezas sueltas: el agente APK, los protocolos de control, la gestión de identidad y la red. Pero el delincuente no utiliza estas piezas de forma aislada; las combina en **campañas** con un objetivo final claro: dinero o influencia.

Hoy vamos a ponernos en la piel de un analista de fraude que recibe una alerta: “Se ha detectado un pico de registros sospechosos en nuestra plataforma”. Vuestro trabajo será reconstruir qué ha pasado, paso a paso, y diseñar defensas para que no vuelva a ocurrir. Para ello, estudiaremos tres de los flujos de trabajo más rentables para las granjas: el **registro masivo de cuentas** (account farming), el **fraude publicitario CPA** (coste por acción) y la **amplificación en redes sociales**.

En cada caso aplicaremos la misma metodología: primero, desgranamos el flujo operativo completo, segundo, identificamos los artefactos forenses que deja, y tercero, construimos reglas de detección robustas. A lo largo del módulo trabajaréis con volcados de tráfico, registros de eventos y capturas de pantalla reales (simuladas) para que desarrolléis el instinto de investigador.

¿Empezamos?

---

**2. REGISTRO MASIVO DE CUENTAS (ACCOUNT FARMING) (60 minutos)**

*El profesor proyecta un diagrama de secuencia: “Creación de 100 cuentas de Gmail en 10 minutos”.*

**2.1 El flujo operativo paso a paso (20 minutos)**

El registro masivo de cuentas es la base de casi todas las demás operaciones fraudulentas. Sin cuentas, no hay fraude. GenFarmer automatiza este proceso de forma industrial.

Imaginad que una granja quiere crear 500 cuentas de Instagram para luego vender seguidores. ¿Cómo lo hace? Vamos a seguir el rastro de una sola cuenta, pero multiplicado por 500 dispositivos.

**Paso 1 – Preparación del dispositivo:**
El operador, desde el panel de GenFarmer, ordena a un lote de teléfonos que se preparen. Esto implica:
- Restablecer el **Advertising ID (AAID)** para que Instagram vea un dispositivo nuevo.
- Asignar un **proxy residencial fresco**, geolocalizado en el país objetivo (por ejemplo, España).
- Limpiar las cookies y los datos de la app de Instagram (si ya estaba instalada) con `pm clear com.instagram.android`.

**Paso 2 – Apertura de la app y navegación al registro:**
GenFarmer abre Instagram mediante un comando `am start`. El agente, usando el `AccessibilityService`, espera a que aparezca la pantalla de inicio de sesión y localiza el texto “Crear cuenta nueva”. La detección se hace por `findAccessibilityNodeInfosByText("Crear cuenta nueva")`. Una vez localizado, ejecuta un `dispatchGesture` sobre sus coordenadas.

**Paso 3 – Relleno del formulario:**
Aquí es donde entra la **IA generativa** (que veremos en detalle en el Módulo 6). El agente no escribe “Usuario1234”; eso sería muy detectable. En su lugar, el panel de GenFarmer envía una petición a un LLM local (por ejemplo, Dolphin) con un prompt como: “Genera un nombre de usuario creíble para una chica española de 22 años, combinando nombre y aficiones”. El LLM devuelve algo como “laura_photography_22”. El agente pega ese texto en el campo de nombre de usuario mediante `AccessibilityService` (no usa `adb shell input text` porque, como vimos, deja rastro).

De igual forma, el LLM genera un nombre completo, una contraseña compleja y, si es necesario, una biografía corta. Todo ello se inyecta campo por campo.

**Paso 4 – Verificación por SMS:**
Instagram solicita un número de teléfono. Aquí la granja puede actuar de dos maneras:
- **SIM física:** la placa SIM conectada al PC recibe el SMS, un script extrae el código y lo envía al agente, que lo pega en el campo correspondiente.
- **Servicio de SMS (TextVerified/SMSPool):** el panel de GenFarmer hace una petición API al servicio, obtiene un número temporal, lo introduce en Instagram, y luego consulta periódicamente el servicio hasta recibir el código.

**Paso 5 – Confirmación y “calentamiento”:**
La cuenta está creada. Pero si se abandona inmediatamente, Instagram la marcará como inactiva y la borrará. GenFarmer ejecuta un script de “calentamiento” (warming up): la cuenta sigue a 5-10 cuentas populares, da like a 3 publicaciones, y sube una foto de perfil generada por IA (esto ya es más avanzado). Todo ello simula actividad humana inicial.

**Paso 6 – Repetición a escala:**
El mismo flujo se ejecuta en paralelo en 50, 100 o 500 dispositivos, con ligeras variaciones de tiempos y datos para evitar patrones idénticos.

**2.2 Artefactos forenses que deja (20 minutos)**

*El profesor muestra una tabla con los rastros.*

Como investigadores, ¿qué podemos encontrar al analizar este ataque?

- **En el tráfico de red:** picos masivos de peticiones a los endpoints de registro de Instagram (`i.instagram.com/api/v1/accounts/create/`) desde IPs de proxies residenciales. Si monitorizáis los dominios de los proveedores de proxies, veréis un aumento repentino en las conexiones hacia ellos.
- **En los registros del dispositivo:** si tenéis acceso a un teléfono de la granja, veréis:
  - Secuencias de llamadas al `AccessibilityService` con eventos `TYPE_WINDOW_CONTENT_CHANGED` sobre la app de Instagram.
  - Múltiples restablecimientos del AAID en un corto periodo (logs de GMS).
  - La presencia de apps de verificación de SMS o scripts de automatización.
- **En los servidores de la plataforma atacada (Instagram en este caso):**
  - Cuentas creadas con direcciones IP rotativas pero con `Build.FINGERPRINT` idéntico (si los teléfonos son del mismo modelo y no se falsifica). Aunque GenFarmer puede modificar algunos campos, el modelo exacto se repite.
  - Tiempos de registro anormalmente cortos (un humano tarda 2-3 minutos en crear una cuenta; un bot puede hacerlo en 20 segundos).
  - Nombres de usuario con baja entropía pero alta coherencia: combinaciones de palabras comunes con números o guiones bajos, generados por un modelo de lenguaje.

**Ejercicio práctico (20 minutos):**
Os entrego un archivo de log simulado que contiene 200 eventos de registro en una red social. Incluye timestamp, IP, User-Agent, AAID, tiempo de completado y nombre de usuario. Identificad los registros que probablemente provienen de una granja. Agrupadlos por IP, por AAID repetido (si lo hay) y por patrones de nombre de usuario. Explicad vuestro criterio.

*Pausa de 15 minutos.*

---

**3. FRAUDE PUBLICITARIO CPA (COSTE POR ACCIÓN) (75 minutos)**

*El profesor retoma tras la pausa.*

Pasamos al flujo más lucrativo: el fraude CPA. “CPA” significa Coste Por Acción. Los anunciantes pagan a los afiliados por cada acción: instalar una app, registrarse, hacer una compra. Las granjas simulan estas acciones y cobran comisiones fraudulentas.

**3.1 El flujo operativo del fraude de instalación de apps (25 minutos)**

Vamos a seguir el rastro de una instalación fraudulenta de una app de juegos, típico objetivo de las granjas.

**Fase 0 – Configuración inicial:**
- El dispositivo se resetea a un estado limpio: se restablece el AAID, se borran los datos de Google Play Store y se asigna un proxy residencial del país objetivo.
- El operador, desde el panel, carga una campaña: “Instalar SuperGame, completar tutorial, alcanzar nivel 3”.

**Fase 1 – Apertura de la tienda y búsqueda:**
GenFarmer abre Google Play Store y escribe en el buscador “SuperGame” (usando `AccessibilityService` para pegar el texto). Navega hasta el resultado correcto y pulsa “Instalar”.

**Fase 2 – Descarga e instalación:**
El agente espera a que la descarga termine, monitorizando el texto del botón (que cambia de “Instalar” a “Cancelar” y luego a “Abrir”). Para ello, el `AccessibilityService` está suscrito a eventos de cambio de texto en los nodos de la UI.

**Fase 3 – Apertura y tutorial:**
El agente pulsa “Abrir”. La app se inicia y muestra un tutorial. El agente, utilizando macros pregrabadas o detección de imágenes/colores, pulsa “Aceptar”, “Siguiente”, “Empezar”, etc., hasta completar el tutorial. Las coordenadas de estos botones suelen ser fijas, por lo que el agente usa `dispatchGesture` con una pequeña aleatorización (±10 píxeles).

**Fase 4 – Interacción post-instalación:**
Algunas campañas requieren que el usuario juegue hasta el nivel 3. GenFarmer puede cargar un script que automatiza las primeras partidas (por ejemplo, tocando repetidamente en zonas de la pantalla). En juegos más complejos, puede usarse un bot de inyección (como vimos con OSMBB) adaptado a ese juego.

**Fase 5 – Limpieza y repetición:**
Tras completar la acción, la app se desinstala (`pm uninstall`), se restablece de nuevo el AAID, y el dispositivo está listo para la siguiente campaña.

**3.2 Artefactos forenses (20 minutos)**

*El profesor muestra capturas de logs de red y eventos de dispositivo.*

- **En el tráfico de red:** veréis una secuencia muy marcada:
  1. Conexión a Google Play para descargar la app.
  2. Tras la instalación, conexiones a los servidores de atribución (Adjust, AppsFlyer, Branch) que envían el evento de instalación. Estos eventos llevan el AAID y la IP del dispositivo. Si el AAID es nuevo y la IP es de un pool de proxies, es sospechoso.
  3. A continuación, tráfico propio de la app (gameplay) durante un tiempo fijo y luego silencio (desinstalación).
- **En los logs del dispositivo:** 
  - Registros de `PackageManager` instalando y desinstalando apps en rápida sucesión.
  - Restablecimientos frecuentes del AAID.
  - Uso intensivo de `AccessibilityService` sobre `com.android.vending` (Play Store) y sobre la app objetivo.
- **En los servidores de atribución:** si sois el anunciante o el proveedor de atribución, podéis detectar:
  - **Click injection:** un clic fraudulento se registra justo antes de la instalación orgánica. Si veis un clic en un enlace de afiliado y, menos de un segundo después, la instalación, es click injection.
  - **Granjas de dispositivos:** múltiples instalaciones desde IPs de un mismo rango de ASN de un proveedor de proxies.
  - **Tiempos de interacción:** un usuario real tarda varios minutos en completar un tutorial; un bot lo hace en segundos.

**Ejercicio práctico (30 minutos):**
Os entrego un conjunto de datos de atribución de una campaña real (anonimizada). Incluye: timestamp de clic, timestamp de instalación, IP del clic, IP de instalación, AAID, tiempo hasta completar el tutorial, y si se realizó una compra in-app. Vuestro objetivo es detectar las instalaciones fraudulentas. Aplicad lo aprendido: buscad AAIDs que aparecen varias veces, IPs repetidas o de data centers, tiempos de tutorial anormalmente bajos, y correlacionad clics con instalaciones muy rápidas. Escribid vuestras conclusiones en un breve informe.

---

**4. AMPLIFICACIÓN EN REDES SOCIALES (60 minutos)**

*El profesor cambia a un caso de manipulación de tendencias.*

Ahora abordamos un flujo menos “transaccional” pero muy peligroso: la manipulación de la opinión pública mediante cuentas falsas.

**4.1 El flujo operativo: una campaña de amplificación (25 minutos)**

Supongamos que alguien paga para que un hashtag sea tendencia. La granja recibe una orden: “100 cuentas deben tuitear #MiHashtag en la próxima hora, con comentarios variados, desde IPs de España”.

**Fase 1 – Selección de cuentas y dispositivos:**
El operador selecciona en GenFarmer un grupo de 100 cuentas ya creadas y envejecidas (calentadas). Cada una está vinculada a un dispositivo con su propio proxy.

**Fase 2 – Generación de contenido con LLM:**
El panel envía los parámetros de la campaña al LLM local (por ejemplo, Dolphin 70B). El prompt es algo así: “Eres un usuario español de Twitter. Tienes que publicar un tuit con el hashtag #MiHashtag. El tono debe ser entusiasta y parecer natural. Añade una errata leve. No uses exactamente las mismas palabras en cada tuit.”

El LLM genera 100 variaciones:
- “¡Qué pasada! No puedo creer lo que acabo de ver #MiHashtag 😱”
- “Esto es increible, no me lo esperaba. #MiHashtag”
- “Pues a mí me ha encantado, la verdad #MiHashtag”

**Fase 3 – Inyección y publicación:**
GenFarmer, de forma coordinada, abre la app de Twitter en los 100 dispositivos. El agente pega el texto generado (vía portapapeles, usando `AccessibilityService`) y pulsa “Twittear”. Para evitar sincronización perfecta, introduce un retardo aleatorio de entre 0 y 30 segundos.

**Fase 4 – Interacciones secundarias:**
Para dar más credibilidad, 20 de esas cuentas dan like a los tuits de otras 10, y 5 retuitean. Incluso pueden comentar entre sí, simulando un debate. Esto también se coordina desde GenFarmer.

**4.2 Artefactos forenses (20 minutos)**

*El profesor muestra una línea temporal de eventos en Twitter.*

- **En la plataforma (Twitter/X):**
  - Un pico repentino de tuits con un mismo hashtag, provenientes de cuentas creadas recientemente o con baja actividad previa.
  - Correlación temporal: los tuits se publican en una ventana muy estrecha.
  - Similitud semántica: aunque los textos son distintos, un análisis de embeddings (vectorización) muestra que están muy próximos entre sí (todos hablan de lo mismo con variaciones mínimas). La perplejidad es baja y uniforme.
  - Las cuentas comparten metadatos: mismo modelo de dispositivo (Build.MODEL), mismo operador (si usan SIMs del mismo lote), o IPs que rotan pero pertenecen al mismo pool.
- **En el tráfico de la granja (si podemos interceptarlo):**
  - Veremos comandos del panel a los dispositivos con `type: "tweet"` y el texto generado.
  - Las peticiones al endpoint de creación de tweet de Twitter (`/1.1/statuses/update.json`) llegarán desde múltiples IPs de proxies residenciales, pero con el mismo agente de automatización (misma firma TLS, mismo patrón de latencia).
- **En el dispositivo:**
  - El `AccessibilityService` registra eventos sobre `com.twitter.android` con secuencias de pegado de texto y pulsaciones de botones.
  - El agente puede almacenar caché de los textos generados en una base de datos local (por ejemplo, en `/data/data/com.genfarmer.agent/databases/tasks.db`). Durante un análisis forense, podéis volcar esa base de datos y ver los prompts y respuestas.

**Ejercicio práctico (15 minutos):**
Os proporciono un dataset de 500 tuits con sus metadatos (timestamp, ID de usuario, texto, IP, modelo de dispositivo). Debéis identificar el clúster de tuits generados por una granja. Usad análisis de frecuencias de hashtags, similitud de textos (con TF-IDF o simplemente visual), y agrupad por IP y modelo de dispositivo. Presentad vuestra metodología y resultados.

---

**5. CIERRE DEL MÓDULO (15 minutos)**

*El profesor muestra una tabla resumen de los tres flujos y sus indicadores de detección.*

Hoy hemos visto la granja en acción, no como piezas sueltas, sino como una fábrica de fraudes con cadenas de montaje muy definidas. ¿Cuáles son las lecciones más importantes?

1. **Todos los flujos comparten patrones:** restablecimiento del AAID, uso de proxies rotativos, automatización vía AccessibilityService, y tiempos de ejecución anormalmente cortos. Si construís defensas que monitoricen esos cuatro pilares, cubriréis la mayoría de los ataques.
2. **La IA generativa es un arma de doble filo:** hace que los textos sean más creíbles, pero también introduce artefactos detectables (perplejidad uniforme, embeddings agrupados). En el Módulo 6 profundizaremos en esto.
3. **La detección debe ser en tiempo real:** para el fraude publicitario, cada minuto que pasa sin detectar una instalación fraudulenta es dinero perdido. Necesitáis pipelines de streaming que procesen eventos de atribución y apliquen modelos de ML.
4. **La colaboración es clave:** compartid IoCs entre plataformas. Si una red social detecta una granja, sus IPs y AAIDs deberían bloquearse también en tiendas de aplicaciones y redes publicitarias. El ecosistema Gen se combate con un ecosistema de defensa.

**Tarea para el Módulo 6:**
Diseñad una regla de detección (en formato Sigma o pseudocódigo) para vuestra plataforma (elegid una de las tres: red social, tienda de aplicaciones, o red publicitaria) que detecte uno de los flujos vistos hoy. Debéis incluir las fuentes de datos, los umbrales y la lógica de correlación. Además, investigad sobre los modelos de lenguaje sin censura (Dolphin, Llama 3 sin alineamiento); en el próximo módulo los diseccionaremos y aprenderemos a identificar sus huellas textuales.

**Ronda de preguntas (5 minutos).**

¿Alguna duda? Perfecto. Nos vemos en el Módulo 6, donde pondremos la guinda al curso: la inteligencia artificial en manos del adversario y cómo combatirla.

---

Así concluye el texto para el profesor del Módulo 5. ¿Deseas que continúe con el Módulo 6?Aquí tienes el texto para el profesor del Módulo 5, siguiendo la línea de profundidad técnica y enfoque práctico de los módulos anteriores.

---

**MÓDULO 5: FLUJOS DE TRABAJO TÍPICOS Y SU DETECCIÓN**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, seréis capaces de reconstruir las campañas de automatización más comunes (registro masivo de cuentas, fraude publicitario CPA y amplificación en redes sociales) a partir de evidencias forenses dispersas, y diseñar reglas de detección específicas para cada una.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza.*

Bienvenidos al Módulo 5. Hasta ahora hemos analizado piezas sueltas: el agente APK, los protocolos de control, la gestión de identidad y la red. Pero el delincuente no utiliza estas piezas de forma aislada; las combina en **campañas** con un objetivo final claro: dinero o influencia.

Hoy vamos a ponernos en la piel de un analista de fraude que recibe una alerta: “Se ha detectado un pico de registros sospechosos en nuestra plataforma”. Vuestro trabajo será reconstruir qué ha pasado, paso a paso, y diseñar defensas para que no vuelva a ocurrir. Para ello, estudiaremos tres de los flujos de trabajo más rentables para las granjas: el **registro masivo de cuentas** (account farming), el **fraude publicitario CPA** (coste por acción) y la **amplificación en redes sociales**.

En cada caso aplicaremos la misma metodología: primero, desgranamos el flujo operativo completo, segundo, identificamos los artefactos forenses que deja, y tercero, construimos reglas de detección robustas. A lo largo del módulo trabajaréis con volcados de tráfico, registros de eventos y capturas de pantalla reales (simuladas) para que desarrolléis el instinto de investigador.

¿Empezamos?

---

**2. REGISTRO MASIVO DE CUENTAS (ACCOUNT FARMING) (60 minutos)**

*El profesor proyecta un diagrama de secuencia: “Creación de 100 cuentas de Gmail en 10 minutos”.*

**2.1 El flujo operativo paso a paso (20 minutos)**

El registro masivo de cuentas es la base de casi todas las demás operaciones fraudulentas. Sin cuentas, no hay fraude. GenFarmer automatiza este proceso de forma industrial.

Imaginad que una granja quiere crear 500 cuentas de Instagram para luego vender seguidores. ¿Cómo lo hace? Vamos a seguir el rastro de una sola cuenta, pero multiplicado por 500 dispositivos.

**Paso 1 – Preparación del dispositivo:**
El operador, desde el panel de GenFarmer, ordena a un lote de teléfonos que se preparen. Esto implica:
- Restablecer el **Advertising ID (AAID)** para que Instagram vea un dispositivo nuevo.
- Asignar un **proxy residencial fresco**, geolocalizado en el país objetivo (por ejemplo, España).
- Limpiar las cookies y los datos de la app de Instagram (si ya estaba instalada) con `pm clear com.instagram.android`.

**Paso 2 – Apertura de la app y navegación al registro:**
GenFarmer abre Instagram mediante un comando `am start`. El agente, usando el `AccessibilityService`, espera a que aparezca la pantalla de inicio de sesión y localiza el texto “Crear cuenta nueva”. La detección se hace por `findAccessibilityNodeInfosByText("Crear cuenta nueva")`. Una vez localizado, ejecuta un `dispatchGesture` sobre sus coordenadas.

**Paso 3 – Relleno del formulario:**
Aquí es donde entra la **IA generativa** (que veremos en detalle en el Módulo 6). El agente no escribe “Usuario1234”; eso sería muy detectable. En su lugar, el panel de GenFarmer envía una petición a un LLM local (por ejemplo, Dolphin) con un prompt como: “Genera un nombre de usuario creíble para una chica española de 22 años, combinando nombre y aficiones”. El LLM devuelve algo como “laura_photography_22”. El agente pega ese texto en el campo de nombre de usuario mediante `AccessibilityService` (no usa `adb shell input text` porque, como vimos, deja rastro).

De igual forma, el LLM genera un nombre completo, una contraseña compleja y, si es necesario, una biografía corta. Todo ello se inyecta campo por campo.

**Paso 4 – Verificación por SMS:**
Instagram solicita un número de teléfono. Aquí la granja puede actuar de dos maneras:
- **SIM física:** la placa SIM conectada al PC recibe el SMS, un script extrae el código y lo envía al agente, que lo pega en el campo correspondiente.
- **Servicio de SMS (TextVerified/SMSPool):** el panel de GenFarmer hace una petición API al servicio, obtiene un número temporal, lo introduce en Instagram, y luego consulta periódicamente el servicio hasta recibir el código.

**Paso 5 – Confirmación y “calentamiento”:**
La cuenta está creada. Pero si se abandona inmediatamente, Instagram la marcará como inactiva y la borrará. GenFarmer ejecuta un script de “calentamiento” (warming up): la cuenta sigue a 5-10 cuentas populares, da like a 3 publicaciones, y sube una foto de perfil generada por IA (esto ya es más avanzado). Todo ello simula actividad humana inicial.

**Paso 6 – Repetición a escala:**
El mismo flujo se ejecuta en paralelo en 50, 100 o 500 dispositivos, con ligeras variaciones de tiempos y datos para evitar patrones idénticos.

**2.2 Artefactos forenses que deja (20 minutos)**

*El profesor muestra una tabla con los rastros.*

Como investigadores, ¿qué podemos encontrar al analizar este ataque?

- **En el tráfico de red:** picos masivos de peticiones a los endpoints de registro de Instagram (`i.instagram.com/api/v1/accounts/create/`) desde IPs de proxies residenciales. Si monitorizáis los dominios de los proveedores de proxies, veréis un aumento repentino en las conexiones hacia ellos.
- **En los registros del dispositivo:** si tenéis acceso a un teléfono de la granja, veréis:
  - Secuencias de llamadas al `AccessibilityService` con eventos `TYPE_WINDOW_CONTENT_CHANGED` sobre la app de Instagram.
  - Múltiples restablecimientos del AAID en un corto periodo (logs de GMS).
  - La presencia de apps de verificación de SMS o scripts de automatización.
- **En los servidores de la plataforma atacada (Instagram en este caso):**
  - Cuentas creadas con direcciones IP rotativas pero con `Build.FINGERPRINT` idéntico (si los teléfonos son del mismo modelo y no se falsifica). Aunque GenFarmer puede modificar algunos campos, el modelo exacto se repite.
  - Tiempos de registro anormalmente cortos (un humano tarda 2-3 minutos en crear una cuenta; un bot puede hacerlo en 20 segundos).
  - Nombres de usuario con baja entropía pero alta coherencia: combinaciones de palabras comunes con números o guiones bajos, generados por un modelo de lenguaje.

**Ejercicio práctico (20 minutos):**
Os entrego un archivo de log simulado que contiene 200 eventos de registro en una red social. Incluye timestamp, IP, User-Agent, AAID, tiempo de completado y nombre de usuario. Identificad los registros que probablemente provienen de una granja. Agrupadlos por IP, por AAID repetido (si lo hay) y por patrones de nombre de usuario. Explicad vuestro criterio.

*Pausa de 15 minutos.*

---

**3. FRAUDE PUBLICITARIO CPA (COSTE POR ACCIÓN) (75 minutos)**

*El profesor retoma tras la pausa.*

Pasamos al flujo más lucrativo: el fraude CPA. “CPA” significa Coste Por Acción. Los anunciantes pagan a los afiliados por cada acción: instalar una app, registrarse, hacer una compra. Las granjas simulan estas acciones y cobran comisiones fraudulentas.

**3.1 El flujo operativo del fraude de instalación de apps (25 minutos)**

Vamos a seguir el rastro de una instalación fraudulenta de una app de juegos, típico objetivo de las granjas.

**Fase 0 – Configuración inicial:**
- El dispositivo se resetea a un estado limpio: se restablece el AAID, se borran los datos de Google Play Store y se asigna un proxy residencial del país objetivo.
- El operador, desde el panel, carga una campaña: “Instalar SuperGame, completar tutorial, alcanzar nivel 3”.

**Fase 1 – Apertura de la tienda y búsqueda:**
GenFarmer abre Google Play Store y escribe en el buscador “SuperGame” (usando `AccessibilityService` para pegar el texto). Navega hasta el resultado correcto y pulsa “Instalar”.

**Fase 2 – Descarga e instalación:**
El agente espera a que la descarga termine, monitorizando el texto del botón (que cambia de “Instalar” a “Cancelar” y luego a “Abrir”). Para ello, el `AccessibilityService` está suscrito a eventos de cambio de texto en los nodos de la UI.

**Fase 3 – Apertura y tutorial:**
El agente pulsa “Abrir”. La app se inicia y muestra un tutorial. El agente, utilizando macros pregrabadas o detección de imágenes/colores, pulsa “Aceptar”, “Siguiente”, “Empezar”, etc., hasta completar el tutorial. Las coordenadas de estos botones suelen ser fijas, por lo que el agente usa `dispatchGesture` con una pequeña aleatorización (±10 píxeles).

**Fase 4 – Interacción post-instalación:**
Algunas campañas requieren que el usuario juegue hasta el nivel 3. GenFarmer puede cargar un script que automatiza las primeras partidas (por ejemplo, tocando repetidamente en zonas de la pantalla). En juegos más complejos, puede usarse un bot de inyección (como vimos con OSMBB) adaptado a ese juego.

**Fase 5 – Limpieza y repetición:**
Tras completar la acción, la app se desinstala (`pm uninstall`), se restablece de nuevo el AAID, y el dispositivo está listo para la siguiente campaña.

**3.2 Artefactos forenses (20 minutos)**

*El profesor muestra capturas de logs de red y eventos de dispositivo.*

- **En el tráfico de red:** veréis una secuencia muy marcada:
  1. Conexión a Google Play para descargar la app.
  2. Tras la instalación, conexiones a los servidores de atribución (Adjust, AppsFlyer, Branch) que envían el evento de instalación. Estos eventos llevan el AAID y la IP del dispositivo. Si el AAID es nuevo y la IP es de un pool de proxies, es sospechoso.
  3. A continuación, tráfico propio de la app (gameplay) durante un tiempo fijo y luego silencio (desinstalación).
- **En los logs del dispositivo:** 
  - Registros de `PackageManager` instalando y desinstalando apps en rápida sucesión.
  - Restablecimientos frecuentes del AAID.
  - Uso intensivo de `AccessibilityService` sobre `com.android.vending` (Play Store) y sobre la app objetivo.
- **En los servidores de atribución:** si sois el anunciante o el proveedor de atribución, podéis detectar:
  - **Click injection:** un clic fraudulento se registra justo antes de la instalación orgánica. Si veis un clic en un enlace de afiliado y, menos de un segundo después, la instalación, es click injection.
  - **Granjas de dispositivos:** múltiples instalaciones desde IPs de un mismo rango de ASN de un proveedor de proxies.
  - **Tiempos de interacción:** un usuario real tarda varios minutos en completar un tutorial; un bot lo hace en segundos.

**Ejercicio práctico (30 minutos):**
Os entrego un conjunto de datos de atribución de una campaña real (anonimizada). Incluye: timestamp de clic, timestamp de instalación, IP del clic, IP de instalación, AAID, tiempo hasta completar el tutorial, y si se realizó una compra in-app. Vuestro objetivo es detectar las instalaciones fraudulentas. Aplicad lo aprendido: buscad AAIDs que aparecen varias veces, IPs repetidas o de data centers, tiempos de tutorial anormalmente bajos, y correlacionad clics con instalaciones muy rápidas. Escribid vuestras conclusiones en un breve informe.

---

**4. AMPLIFICACIÓN EN REDES SOCIALES (60 minutos)**

*El profesor cambia a un caso de manipulación de tendencias.*

Ahora abordamos un flujo menos “transaccional” pero muy peligroso: la manipulación de la opinión pública mediante cuentas falsas.

**4.1 El flujo operativo: una campaña de amplificación (25 minutos)**

Supongamos que alguien paga para que un hashtag sea tendencia. La granja recibe una orden: “100 cuentas deben tuitear #MiHashtag en la próxima hora, con comentarios variados, desde IPs de España”.

**Fase 1 – Selección de cuentas y dispositivos:**
El operador selecciona en GenFarmer un grupo de 100 cuentas ya creadas y envejecidas (calentadas). Cada una está vinculada a un dispositivo con su propio proxy.

**Fase 2 – Generación de contenido con LLM:**
El panel envía los parámetros de la campaña al LLM local (por ejemplo, Dolphin 70B). El prompt es algo así: “Eres un usuario español de Twitter. Tienes que publicar un tuit con el hashtag #MiHashtag. El tono debe ser entusiasta y parecer natural. Añade una errata leve. No uses exactamente las mismas palabras en cada tuit.”

El LLM genera 100 variaciones:
- “¡Qué pasada! No puedo creer lo que acabo de ver #MiHashtag 😱”
- “Esto es increible, no me lo esperaba. #MiHashtag”
- “Pues a mí me ha encantado, la verdad #MiHashtag”

**Fase 3 – Inyección y publicación:**
GenFarmer, de forma coordinada, abre la app de Twitter en los 100 dispositivos. El agente pega el texto generado (vía portapapeles, usando `AccessibilityService`) y pulsa “Twittear”. Para evitar sincronización perfecta, introduce un retardo aleatorio de entre 0 y 30 segundos.

**Fase 4 – Interacciones secundarias:**
Para dar más credibilidad, 20 de esas cuentas dan like a los tuits de otras 10, y 5 retuitean. Incluso pueden comentar entre sí, simulando un debate. Esto también se coordina desde GenFarmer.

**4.2 Artefactos forenses (20 minutos)**

*El profesor muestra una línea temporal de eventos en Twitter.*

- **En la plataforma (Twitter/X):**
  - Un pico repentino de tuits con un mismo hashtag, provenientes de cuentas creadas recientemente o con baja actividad previa.
  - Correlación temporal: los tuits se publican en una ventana muy estrecha.
  - Similitud semántica: aunque los textos son distintos, un análisis de embeddings (vectorización) muestra que están muy próximos entre sí (todos hablan de lo mismo con variaciones mínimas). La perplejidad es baja y uniforme.
  - Las cuentas comparten metadatos: mismo modelo de dispositivo (Build.MODEL), mismo operador (si usan SIMs del mismo lote), o IPs que rotan pero pertenecen al mismo pool.
- **En el tráfico de la granja (si podemos interceptarlo):**
  - Veremos comandos del panel a los dispositivos con `type: "tweet"` y el texto generado.
  - Las peticiones al endpoint de creación de tweet de Twitter (`/1.1/statuses/update.json`) llegarán desde múltiples IPs de proxies residenciales, pero con el mismo agente de automatización (misma firma TLS, mismo patrón de latencia).
- **En el dispositivo:**
  - El `AccessibilityService` registra eventos sobre `com.twitter.android` con secuencias de pegado de texto y pulsaciones de botones.
  - El agente puede almacenar caché de los textos generados en una base de datos local (por ejemplo, en `/data/data/com.genfarmer.agent/databases/tasks.db`). Durante un análisis forense, podéis volcar esa base de datos y ver los prompts y respuestas.

**Ejercicio práctico (15 minutos):**
Os proporciono un dataset de 500 tuits con sus metadatos (timestamp, ID de usuario, texto, IP, modelo de dispositivo). Debéis identificar el clúster de tuits generados por una granja. Usad análisis de frecuencias de hashtags, similitud de textos (con TF-IDF o simplemente visual), y agrupad por IP y modelo de dispositivo. Presentad vuestra metodología y resultados.

---

**5. CIERRE DEL MÓDULO (15 minutos)**

*El profesor muestra una tabla resumen de los tres flujos y sus indicadores de detección.*

Hoy hemos visto la granja en acción, no como piezas sueltas, sino como una fábrica de fraudes con cadenas de montaje muy definidas. ¿Cuáles son las lecciones más importantes?

1. **Todos los flujos comparten patrones:** restablecimiento del AAID, uso de proxies rotativos, automatización vía AccessibilityService, y tiempos de ejecución anormalmente cortos. Si construís defensas que monitoricen esos cuatro pilares, cubriréis la mayoría de los ataques.
2. **La IA generativa es un arma de doble filo:** hace que los textos sean más creíbles, pero también introduce artefactos detectables (perplejidad uniforme, embeddings agrupados). En el Módulo 6 profundizaremos en esto.
3. **La detección debe ser en tiempo real:** para el fraude publicitario, cada minuto que pasa sin detectar una instalación fraudulenta es dinero perdido. Necesitáis pipelines de streaming que procesen eventos de atribución y apliquen modelos de ML.
4. **La colaboración es clave:** compartid IoCs entre plataformas. Si una red social detecta una granja, sus IPs y AAIDs deberían bloquearse también en tiendas de aplicaciones y redes publicitarias. El ecosistema Gen se combate con un ecosistema de defensa.

**Tarea para el Módulo 6:**
Diseñad una regla de detección (en formato Sigma o pseudocódigo) para vuestra plataforma (elegid una de las tres: red social, tienda de aplicaciones, o red publicitaria) que detecte uno de los flujos vistos hoy. Debéis incluir las fuentes de datos, los umbrales y la lógica de correlación. Además, investigad sobre los modelos de lenguaje sin censura (Dolphin, Llama 3 sin alineamiento); en el próximo módulo los diseccionaremos y aprenderemos a identificar sus huellas textuales.

**Ronda de preguntas (5 minutos).**

¿Alguna duda? Perfecto. Nos vemos en el Módulo 6, donde pondremos la guinda al curso: la inteligencia artificial en manos del adversario y cómo combatirla.
