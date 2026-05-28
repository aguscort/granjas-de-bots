**MÓDULO 1: VISIÓN GENERAL DEL ECOSISTEMA GEN Y SUS COMPONENTES**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, comprenderéis la arquitectura global del ecosistema Gen, cómo se interconectan sus cuatro productos principales y por qué esta plataforma representa un desafío de seguridad de primer orden.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza:*

Buenos días a todos. Arrancamos el curso de análisis forense del ecosistema GenFarmer. Antes de meter el bisturí, necesitamos ver al paciente entero. Este primer módulo es fundamental: si no entendéis cómo encajan las piezas, cuando lleguemos al análisis del agente APK o a las reglas iptables os vais a perder.

Hoy vamos a hacer tres cosas. Primero, os presentaré el ecosistema Gen como lo que es: una plataforma de automatización industrial con cuatro productos. Segundo, diseccionaremos cada uno de esos productos desde la óptica del investigador de seguridad. Y tercero, trazaremos las interconexiones entre ellos, porque ahí es donde aparecen las vulnerabilidades y los indicadores de compromiso que explotaremos en módulos posteriores.

Al final de la sesión, quiero que seáis capaces de dibujar el diagrama de arquitectura del ecosistema Gen en una servilleta y explicar qué hace cada componente. ¿Empezamos?

---

**2. EL ECOSISTEMA GEN COMO PLATAFORMA DE AUTOMATIZACIÓN INDUSTRIAL (45 minutos)**

*El profesor proyecta una diapositiva con los cuatro logos: GenLogin, GenFarmer, GenRouter y GenPlay.*

Mirad esta diapositiva. Cuatro productos. ¿Qué vende Gen? Vende automatización. Pero no automatización de andar por casa, sino automatización a escala industrial. Ellos lo llaman "ecosistema MMO" —Make Money Online— y "gestión de múltiples cuentas". Traducción para nosotros: suplantación de identidad digital a gran escala.

Vamos a entender el flujo completo. Imaginaos que queréis crear mil cuentas de Facebook que parezcan de usuarios reales. ¿Qué necesitáis?

Uno: un navegador que no os delate. Cada cuenta debe tener su propia huella digital: User-Agent, resolución de pantalla, WebGL, Canvas, timezone, idioma, fuentes instaladas... Si entráis con el mismo Chrome de siempre, Facebook os detecta en segundos. Para eso está **GenLogin**: es un navegador anti-detección que genera perfiles únicos e indistinguibles de un usuario real. Lo veremos en detalle en el Módulo 4, pero quedaos con esto: GenLogin es la fábrica de identidades.

Dos: un ejército de dispositivos que ejecuten las acciones. Las cuentas no se crean solas; alguien tiene que hacer clic, escribir, deslizar. Podríais usar emuladores, pero las plataformas importantes detectan emuladores. Así que necesitáis teléfonos reales. Cientos, miles. Y un software que los controle todos a la vez. Eso es **GenFarmer**: el panel de control desde el que un solo operador maneja mil teléfonos como si fuera un videojuego de estrategia en tiempo real.

Tres: una red que no levante sospechas. Si mil teléfonos salen a internet desde la misma IP, estáis muertos. Necesitáis proxies residenciales rotativos, y necesitáis que cada dispositivo tenga su propia IP, como si estuviera en una casa distinta. Para eso está **GenRouter**: el router personalizado que gestiona las IPs de salida de toda la granja.

Y cuatro: ¿qué pasa si no queréis montar vuestra propia granja? ¿Si solo queréis alquilar capacidad? Para eso está **GenPlay**: la plataforma en la nube que os alquila dispositivos reales alojados en racks de terceros. Básicamente, una granja como servicio.

*El profesor hace una pausa.*

Ved el flujo: **GenLogin** crea la identidad. **GenFarmer** la ejecuta en un dispositivo físico. **GenRouter** la enmascara en la red. Y **GenPlay** externaliza todo el proceso. Cuatro piezas que encajan como un puzle. Si entendéis esto, habéis entendido el 50% del ecosistema.

---

**3. BOX PHONE FARM: LA CAPA FÍSICA (45 minutos)**

*El profesor muestra imágenes del hardware Box Phone Farm obtenidas del sitio oficial.*

Pasamos a la capa física. Esto es el Box Phone Farm. No es una caja de zapatos con teléfonos amontonados. Es un rack metálico diseñado específicamente para alojar, alimentar, refrigerar y conectar teléfonos. Vamos a analizarlo como forenses.

**Especificaciones técnicas relevantes:**

- Capacidad: entre 10 y 40 dispositivos por bandeja, ampliable en vertical. He visto instalaciones con 20 bandejas. Haced cuentas: 20 bandejas x 40 teléfonos = 800 dispositivos en un solo bastidor.
- Ventilación: forzada, con control por software. Esto es importante: la velocidad de los ventiladores se ajusta remotamente desde el panel GenFarmer. ¿Por qué? Porque 800 teléfonos funcionando 24/7 generan muchísimo calor, y si no lo disipas, las baterías se degradan en semanas.
- Hubs USB 3.0 integrados: cada bandeja tiene un hub que conecta los teléfonos al PC de control. Esto permite la comunicación ADB sobre USB, que es más rápida y fiable que el WiFi.
- Switch de red integrado (en modelos avanzados): los teléfonos se conectan por Ethernet mediante adaptadores USB-Ethernet. Esto elimina la latencia del WiFi y permite que el GenRouter gestione el tráfico de forma centralizada.

**Implicaciones forenses:**

*El profesor subraya este punto con énfasis.*

Prestad atención, porque esto es importante para vuestro trabajo de campo. El rack en sí **no deja huella lógica en el dispositivo**. El teléfono no sabe que está en un rack; solo sabe que está conectado por USB y recibiendo carga. No hay un proceso, ni un archivo, ni una entrada de registro que diga "este teléfono está en un Box Phone Farm".

Entonces, ¿cómo detectamos que un teléfono forma parte de una granja física? Por correlación de evidencias. Si durante una investigación encontráis que 200 dispositivos comparten exactamente el mismo `Build.FINGERPRINT` (mismo modelo, misma versión de Android, mismos parches de seguridad), y además todos ejecutan el mismo `AccessibilityService` sospechoso, y además todos se conectan al mismo panel de control... no estáis ante 200 usuarios idénticos; estáis ante una granja.

Otro indicio: la conectividad USB permanente. Un teléfono normal no está conectado por USB a un ordenador las 24 horas del día. Podemos detectar esto monitorizando el estado del puerto USB (`/sys/class/power_supply/usb/online`) o los eventos de conexión ADB.

Y un tercer indicio: la temperatura de batería. Un teléfono en un rack, cargando continuamente y ejecutando automatización, tiene una temperatura de batería anormalmente alta y constante. Los teléfonos reales tienen picos de temperatura durante la carga y luego se enfrían. Monitorizar la temperatura de la batería a lo largo del tiempo es una técnica de detección muy eficaz.

**Ejercicio práctico (10 minutos):**

*El profesor reparte una hoja con datos simulados de 10 dispositivos.*

Os doy estos datos de 10 dispositivos. Mirad el `Build.FINGERPRINT`, la lista de `AccessibilityService` y el tiempo de conexión USB. Identificad cuáles son probablemente parte de una granja y justificad vuestra respuesta. Trabajad en parejas, luego lo ponemos en común.

---

**4. GENROUTER: LA JOYA DE LA RED (45 minutos)**

*Tras la pausa, el profesor retoma la sesión.*

Volvemos. Hemos visto los teléfonos. Ahora vamos a ver cómo se conectan al mundo. GenRouter es un router basado en OpenWrt con módulos personalizados. No es un router cualquiera: está diseñado específicamente para gestionar el tráfico de una granja de dispositivos.

**Funciones principales:**

1. **Creación de VLANs por campaña.** Imaginad que estáis ejecutando tres campañas distintas: una para crear cuentas de Instagram, otra para fraude de instalación de apps, y otra para amplificar mensajes en X. No queréis que el tráfico de una campaña contamine la otra. GenRouter os permite crear una VLAN para cada campaña, aislando el tráfico a nivel de red local.

2. **Asignación de proxies a nivel de subred.** Esta es la función estrella. Cada VLAN puede asociarse a un proxy distinto. Por ejemplo, la VLAN 10 (campaña Instagram) sale por un proxy residencial de EE.UU., la VLAN 20 (campaña X) sale por un proxy de Reino Unido. Esto se configura en el panel de GenRouter y se sincroniza automáticamente con el backend en la nube.

3. **Priorización de tráfico.** GenRouter implementa QoS (Quality of Service) para priorizar el tráfico de automatización sobre otras comunicaciones. Si un dispositivo está descargando una app y otro está enviando un comentario, el router da prioridad al comentario para reducir la latencia.

**Sincronización con el backend en la nube:**

GenRouter se comunica con el servidor central de Gen mediante una API REST. ¿Qué intercambia? Información de topología (qué dispositivos están conectados a qué puerto), licencias (cada router tiene una licencia vinculada a una cuenta de Gen), y configuración de proxies (listas de IPs, puertos, credenciales).

Para nosotros, los investigadores, esto es una mina de oro. Si podemos interceptar el tráfico del GenRouter —por ejemplo, durante una operación autorizada o mediante un volcado de tráfico obtenido legalmente—, podemos extraer:

- Los dominios de callback del backend. Ejemplos reales (simulados para este curso): `api.genfarmer.com`, `sync.genlogin.io`.
- Los certificados TLS que utiliza el router para autenticarse.
- Las IPs de los proxies residenciales que está utilizando la granja.
- El inventario de dispositivos conectados.

**Implicaciones defensivas:**

Si trabajáis en un equipo de seguridad de una plataforma, monitorizad el tráfico hacia estos dominios conocidos. Si veis un pico de tráfico desde una IP hacia `api.genfarmer.com`, tenéis una granja localizada. Y si esa IP es residencial, tenéis un operador de granja. Reportadlo a vuestro equipo de respuesta a incidentes.

**Ejercicio práctico (15 minutos):**

*El profesor entrega un archivo de captura de tráfico (PCAP) simulado.*

Este PCAP contiene tráfico entre un GenRouter y su backend. Analizadlo con Wireshark. Identificad: el dominio del backend, los endpoints de la API, y cualquier credencial o token que veáis en las peticiones HTTP. Luego lo discutimos.

---

**5. GENPLAY: LA GRANJA COMO SERVICIO (30 minutos)**

*El profesor cambia de diapositiva.*

Último producto del ecosistema: GenPlay. ¿Recordáis que al principio dije que podíais alquilar capacidad en lugar de montar vuestra propia granja? Pues esto es GenPlay.

GenPlay es una plataforma en la nube que ofrece acceso remoto a dispositivos reales alojados en racks. Vosotros, como "clientes", pagáis una suscripción mensual y accedéis a un panel web donde seleccionáis qué tipo de dispositivo queréis (modelo, versión de Android, ubicación) y durante cuánto tiempo. Luego podéis instalar apps, ejecutar automatización, etc.

**Implicaciones de seguridad para el "cliente" (y para nosotros):**

- **Exposición de cuentas:** Las credenciales que usáis en GenPlay pasan por sus servidores. Si GenPlay es comprometido, todas vuestras cuentas falsas quedan expuestas. Irónico, ¿verdad? Los estafadores confiando en otros potenciales estafadores.
- **Falta de control sobre el hardware:** No sabéis si el dispositivo que os alquilan tiene malware, keyloggers o si está siendo monitorizado por un tercero.
- **Concentración de IPs:** Desde la óptica defensiva, GenPlay es una bendición. Todos los dispositivos alquilados salen desde las mismas IPs (los racks donde están alojados). Si identificáis una IP de GenPlay, podéis bloquearla y de un plumazo neutralizáis decenas o cientos de cuentas fraudulentas.

**Cómo identificar tráfico de GenPlay:**

Buscad IPs que:
- Estén geolocalizadas en centros de datos o racks comerciales (no residenciales).
- Tengan una alta densidad de cuentas creadas recientemente.
- Presenten patrones de automatización idénticos (mismos tiempos de respuesta, mismas secuencias de clics).
- Se conecten a los mismos endpoints de API que GenFarmer y GenRouter.

---

**6. SÍNTESIS Y CIERRE DEL MÓDULO (30 minutos)**

*El profesor muestra el diagrama completo del ecosistema Gen.*

Cerramos el módulo. Mirad este diagrama. Quiero que os quedéis con tres ideas clave:

**Primera idea: El ecosistema Gen es una cadena de suplantación de identidad.**
- GenLogin crea la identidad (huella digital del navegador).
- GenFarmer la ejecuta (acciones en dispositivo real).
- GenRouter la enmascara (IP residencial).
- GenPlay la externaliza (servicio en la nube).
Cada eslabón es necesario. Si falla uno, la cadena se rompe.

**Segunda idea: Cada componente deja un rastro diferente.**
- El Box Phone Farm deja rastros de correlación (muchos dispositivos idénticos, conexión USB permanente, temperatura anómala).
- GenRouter deja rastros de red (dominios de callback, tráfico API, concentración de proxies).
- GenPlay deja rastros de IP (bloques de direcciones de centros de datos).

**Tercera idea: La defensa debe ser multicapa.**
No basta con bloquear IPs. Hay que detectar la APK agente (Módulo 2), analizar los patrones de automatización (Módulo 3), inspeccionar la gestión de identidad (Módulo 4), y correlacionar todo a nivel de plataforma (Módulo 6).

**Ronda de preguntas (15 minutos).**

*El profesor responde dudas y luego asigna la tarea para la próxima sesión:*

Para el Módulo 2, quiero que leáis la documentación sobre `AccessibilityService` y `GestureDescription` de Android. Vamos a descompilar el agente de GenFarmer, y necesito que entendáis cómo funcionan estos servicios a nivel de API. Nos vemos la próxima semana.
