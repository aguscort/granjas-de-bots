# Hardware: Dispositivos y Estructura Física

# 1. Introducción: el ecosistema de la automatización a escala

Las granjas de bots han evolucionado desde simples scripts en servidores hacia sofisticadas infraestructuras híbridas que combinan miles de dispositivos móviles reales, redes proxy residenciales rotativas, emuladores de Android y modelos de lenguaje de última generación. El **Bad Bot Report 2024** de Imperva reveló que casi la mitad del tráfico de internet ya no es humano, y que los bots maliciosos representan casi un tercio del total. Detrás de ese tráfico se oculta un entramado técnico meticulosamente diseñado para imitar el comportamiento humano, evadir sistemas antifraude y operar a una escala que desborda los mecanismos de defensa tradicionales.

Este artículo disecciona la infraestructura de las granjas modernas, desde los dispositivos físicos hasta la integración de modelos de lenguaje, siguiendo un enfoque de ingeniería inversa aplicada a la ciberdefensa.

# 2. Capa física: el ejército de silicio

## 2.1 Modelos de dispositivos y racks especializados

A diferencia de los bots puramente basados en servidor, las granjas actuales recurren masivamente a teléfonos reales para heredar las huellas digitales de hardware, sensores y radios de comunicación que las plataformas (Google, Facebook, X) consideran como señales de legitimidad. Los modelos elegidos responden a un equilibrio entre coste, disponibilidad y requisitos técnicos mínimos.

- **Samsung Galaxy S8, S8+ y S10** aparecen de forma recurrente porque ofrecen un rendimiento aceptable, soporte para las versiones de Android requeridas por las apps objetivo y un precio de segunda mano muy competitivo cuando se adquieren en lotes.
- **Google Pixel 2** es citado como dispositivo de entrada idóneo para granjas de monetización pasiva. Cumple sobradamente los requisitos de línea base (Android 7.0+, 16 GB de almacenamiento, 2 GB de RAM) y su naturaleza “stock Android” simplifica la automatización.
- **Xiaomi** completa el abanico en operaciones que buscan la máxima escala; sus modelos de gama media y baja son populares en granjas localizadas en Asia.

Estos teléfonos no se amontonan sobre una mesa, sino que se montan en **placas o racks metálicos que agrupan 20, 40 o hasta más dispositivos por unidad**. Soluciones comerciales como **Box Phone Farm** (parte del ecosistema GenFarmer) ofrecen bandejas modulares con ventilación forzada controlable remotamente, hubs USB integrados y conmutadores de carga. Estas bandejas permiten apilar varias unidades verticalmente y conectarlas en cascada a un solo equipo de control mediante conexiones LAN o USB.

En las estanterías de granjas que operan en países asiáticos, además, se han documentado sistemas de **aislamiento térmico** y refrigeración por aire acondicionado dedicado para disipar el calor generado por miles de dispositivos funcionando 24/7, un detalle que subraya la dimensión industrial de estas operaciones.

## 2.2 Conectividad, alimentación y tarjetas SIM

Mantener encendidos y conectados miles de terminales exige una infraestructura eléctrica robusta: **hubs de carga masiva con múltiples puertos USB de alta potencia** y, en instalaciones más profesionales, fuentes de alimentación industriales que convierten la corriente de forma centralizada.

La conectividad de red es uno de los pilares más sensibles. Se documentan dos configuraciones típicas:

1. **Conexión directa por cable Ethernet mediante adaptadores USB-Ethernet o hubs con puerto RJ45**. Así se evita la latencia y la inestabilidad del WiFi, y se facilita la gestión centralizada del tráfico. El componente **GenRouter**, por ejemplo, se anuncia como un router optimizado para granjas, capaz de manejar el flujo de datos de cientos de dispositivos y de integrarse con software de control como GenLogin y GenFarmer.
2. **WiFi con múltiples puntos de acceso y VLANs segmentadas** para separar el tráfico por país o por campaña, a menudo combinado con proxies.

En cuanto a la verificación de identidad, ciertas plataformas (especialmente Facebook) exigen números de teléfono reales para validar cuentas. Aquí entran en juego las **placas con ranuras para tarjetas SIM físicas**, que permiten insertar SIMs de operadores locales o internacionales y programar la recepción de SMS de verificación. Estas placas se conectan mediante módems GSM integrados o vía hubs SIM conectados por USB al PC de control. Servicios como **TextVerified** o **SMSPool** externalizan esta función, proporcionando números temporales a bajo coste, pero los operadores de granjas a gran escala a menudo integran sus propias baterías de SIMs para reducir costes recurrentes y aumentar la fiabilidad.

---

# 3. Capa de control: orquestación centralizada

### 3.1 Plataformas de gestión de flotas

El verdadero cerebro de una granja es el software que permite manejar cientos o miles de dispositivos desde una única consola. **GenFarmer** es el ejemplo paradigmático encontrado en las fuentes: un ecosistema que se presenta abiertamente como solución para “MMO” (Make Money Online) y automatización de marketing. Su arquitectura típica consta de:

- **Servicio en la nube (Backend)**: Gestiona las licencias, almacena scripts, coordina tareas programadas y recibe telemetría de los dispositivos.
- **Cliente de escritorio (Controlador)**: Instalado en un PC con Windows o macOS, se conecta al servicio en la nube y a los dispositivos locales (vía ADB sobre USB o TCP/IP). Desde aquí se definen los flujos de trabajo, se replican comandos y se visualiza en tiempo real la pantalla de cualquier terminal.
- **Agentes en los dispositivos**: Pequeñas aplicaciones o binarios que se instalan en cada teléfono (a menudo con permisos de accesibilidad) y que ejecutan las órdenes de automatización: abrir apps, deslizar, tocar coordenadas, introducir texto, leer SMS, etc.

Otro enfoque es el **control puramente remoto mediante paneles web** (Cloud Control), donde los dispositivos físicos están alojados en centros de datos o en racks alquilados a terceros (“phone farm as a service”). Los operadores ni siquiera tocan físicamente los teléfonos; alquilan capacidad de automatización por horas o por campañas.

### 3.2 Técnicas de automatización y emuladores

Para tareas que no requieren un sensor de huella dactilar ni una cámara real, los atacantes recurren a **emuladores de Android** ejecutados en servidores. Esto abarata drásticamente el coste, ya que un solo servidor puede albergar docenas de instancias virtuales. Emuladores como **BlueStacks, LDPlayer, Nox o Memu** permiten clonar configuraciones, cambiar identificadores de dispositivo (IMEI, Android ID, MAC) y, mediante scripts de automatización, simular la actividad de múltiples usuarios. La integración con ADB (Android Debug Bridge) es el canal a través del cual los scripts envían comandos, instalan APKs y manipulan la interfaz gráfica.

En el ámbito de los videojuegos (caso emblemático *Old School RuneScape*, OSRS), han surgido herramientas especializadas que ilustran técnicas reutilizables en otros contextos fraudulentos:

- **OSMBB (Open Source Mobile Bot, a veces OSMB)** es un cliente de escritorio que se conecta a un emulador de Android y utiliza inyección de código o lectura de memoria para automatizar tareas repetitivas dentro del juego. Funciona estableciendo un puente entre el emulador y un script escrito en Java o Kotlin.
- **WaspScripts** representa la corriente del *color botting*: utiliza la librería **WaspLib** (para Simba 2.0) para buscar colores y patrones en la pantalla, sin tocar el proceso del juego, lo que dificulta la detección por parte de sistemas antitrampas.
- **Runemate** es un bot que se inyecta directamente en el cliente del juego (en este caso, la versión de escritorio) y expone una API para que los scripts interactúen con los objetos del juego. Esta técnica de *injection* es también empleada en otros entornos para manipular aplicaciones bancarias o redes sociales si se logra evadir los mecanismos de ofuscación y root detection.

Desde la óptica defensiva, cualquiera de estos métodos deja trazas. El uso de **permisos de accesibilidad abusivos**, la presencia de binarios con firmas sospechosas, las llamadas a `input tap` o `input swipe` a través de ADB, o la manipulación de la superficie de dibujo, son vectores de detección que las plataformas explotan activamente. Sin embargo, las granjas más avanzadas evitan ADB directo y emplean **microcontroladores externos (Arduino, Teensy)** que emulan eventos de teclado y ratón a nivel de hardware, conectados físicamente al teléfono mediante placas con relés táctiles o interfaces HID, una técnica de evasión cada vez más difícil de contrarrestar.

La técnica es, en esencia, un ataque de **inyección de entrada a nivel de hardware**. Su eficacia radica en que el sistema operativo Android está diseñado para confiar ciegamente en las señales que provienen de un dispositivo de interfaz humana (HID) externo.

A diferencia de los métodos de software (como ADB), que dejan un rastro digital, esta técnica crea una "capa física de engaño" que es extremadamente difícil de detectar. A continuación, te detallo sus mecanismos.

#### 3.2.1 Emulación HID de Pantalla Táctil y Teclado

El núcleo de esta técnica es la capacidad de los microcontroladores para hacerse pasar por periféricos USB estándar. El teléfono, al detectar un dispositivo HID, le otorga control total de la interfaz de usuario de forma nativa.

- **HID de Pantalla Táctil (Digitizador)**: Es la forma más sofisticada. El microcontrolador se anuncia como una pantalla táctil externa, lo que permite inyectar eventos multitáctiles con coordenadas X e Y precisas, simulando toques, deslizamientos y pellizcos. El **USB HID Touchscreen es una tecnología con soporte nativo en el kernel de Android**, no un exploit, por lo que su detección es extremadamente compleja. El proyecto `libaoa_hid` es un ejemplo de librería en C que realiza precisamente esta tarea a través del protocolo AOA 2.0, sin necesidad de root, ADB o una app complementaria en el teléfono.
- **HID de Teclado (BadUSB)**: Es la forma más simple y común. El microcontrolador se presenta como un teclado USB y puede inyectar pulsaciones de teclas a una velocidad sobrehumana. Se utiliza para automatizar la escritura de texto, navegar por menús o ejecutar macros complejas.

#### 3.2.2 Implementación Práctica: Microcontroladores y Herramientas

- **Arduino con DigiCombo**: La librería **DigiCombo** permite a una placa Digispark (ATtiny85) emular **simultáneamente un teclado y una pantalla táctil** ante el teléfono. Puede controlar el scrolling, introducir patrones de desbloqueo e incluso hacer zoom. Al basarse en el estándar HID, no requiere ni root ni la depuración USB activada.
- **Teensy**: Las placas Teensy, con sus potentes bibliotecas de emulación USB, pueden configurarse para actuar como teclados, ratones, joysticks o pantallas táctiles, ofreciendo un control mucho más rápido y fiable que los métodos de software.
- **Otras Técnicas Avanzadas**: Dispositivos como **HackStar** o el **ESP32** con soporte HID permiten un control de automatización a nivel de hardware extremadamente preciso y con muy baja latencia. También se ha documentado el uso de hardware **ESP32-S3** para emular pantallas táctiles a través de Bluetooth (BLE HID), inyectando coordenadas a las aplicaciones mediante el protocolo estándar.

#### 3.2.3 Métodos Alternativos: El Contacto Físico

Aunque la inyección HID es la técnica estrella, existen otros métodos que interactúan directamente con la superficie de la pantalla:

- **Relés Táctiles**: Se trata de pequeños actuadores electromecánicos que realizan un toque físico real sobre la pantalla. Aunque rudimentarios, son imposibles de detectar por software.
- **Estiletes Capacitivos**: Brazos robóticos en miniatura con una punta conductora tocan la pantalla en coordenadas programadas. Es una solución mecánica, lenta pero indetectable a nivel lógico.

### 3.2.4 ¿Por Qué es Tan Difícil de Contrarrestar?

La efectividad de esta técnica reside en su naturaleza misma:

- **Capa de Confianza**: El sistema operativo confía inherentemente en los eventos de un dispositivo de entrada por hardware, pues asume que provienen de un humano. No existe una distinción nativa entre una pulsación real y una generada por un microcontrolador HID.
- **Indetectabilidad por Software**: A diferencia de los comandos ADB, que pueden ser auditados por las aplicaciones de seguridad, las señales HID de bajo nivel son extremadamente difíciles de rastrear o interceptar desde el espacio de usuario.
- **Baja Latencia**: Al evitar la pila de software intermedia, la inyección a nivel de hardware es casi instantánea, permitiendo automatizar tareas que requieren reflejos sobrehumanos.

# 4. Capa de red: la ilusión de la ubicuidad

## 4.1 Proxies residenciales rotativos

Un solo router doméstico con una IP de centro de datos sería detectado instantáneamente. Las granjas utilizan **proxies residenciales**, es decir, direcciones IP que pertenecen a ISP de consumidores reales. Estas IPs se obtienen a través de redes peer-to-peer (como Honeygain o Peer2Profit) o mediante la inclusión de SDKs en apps legítimas que ceden el tráfico de fondo del usuario sin su conocimiento pleno. Servicios comerciales como **Bright Data, Oxylabs o Smartproxy** ofrecen pools de millones de IPs residenciales listas para rotar bajo demanda.

En el contexto de una granja, un software como **Superproxy** (citado en las fuentes) actúa como un middleware que asigna a cada dispositivo, o a cada perfil de app, una IP residencial específica y la mantiene el tiempo necesario para que la cuenta parezca coherente geográficamente. La rotación se programa para simular movilidad humana: una misma cuenta puede aparecer un día en Dallas, al siguiente en Seattle, con cambios de IP que coinciden con franjas horarias y patrones de tráfico móvil.

## 4.2 Túneles VPN y encapsulado

Además de los proxies HTTP/SOCKS5, muchas granjas envuelven todo el tráfico del dispositivo en un túnel VPN que redirige todas las conexiones a través de un servidor intermedio. Esto asegura que incluso las consultas DNS, los WebRTC leaks y las conexiones WebSocket pasen por la IP designada, evitando fugas que revelen la ubicación real de la granja. Herramientas como **OpenVPN, WireGuard o proxies inversos personalizados** se despliegan en el dispositivo a nivel de sistema (con permisos de superusuario) para forzar la transparencia total del enmascaramiento.

Para las granjas alojadas en centros de datos, a menudo se utiliza un esquema de doble salto: el dispositivo se conecta a un proxy residencial a través de un túnel cifrado, y ese proxy, a su vez, sale a internet desde una IP doméstica real. De esta forma, la plataforma objetivo solo ve la IP de un hogar estadounidense, por ejemplo, sin rastro alguno del datacenter de origen.

---

# 5. Capa de inteligencia: la irrupción de los LLMs locales

### 5.1 Generación de contenido verosímil

El último salto cualitativo es la integración de modelos de lenguaje de gran tamaño (LLMs) directamente en la infraestructura de la granja. Antes, los comentarios de los bots se limitaban a frases genéricas copiadas y pegadas o a cadenas de Markov poco sofisticadas. Ahora, un solo operador puede generar, en tiempo real, respuestas contextuales que imitan el tono, la jerga y las posturas ideológicas del público objetivo.

Las fuentes consultadas apuntan al uso de **Llama 3 70B** y **Dolphin** (una versión ajustada de Mixtral, o un fine-tune del propio Llama 3, sin las restricciones de seguridad típicas) ejecutados localmente en servidores con GPU dentro de la misma granja o a través de una API interna. La elección del despliegue local es deliberada: reduce la latencia, evita el coste de APIs comerciales y, crucialmente, elude los filtros de contenido de proveedores como OpenAI o Anthropic.

## 5.2 Arquitectura de integración

El flujo típico en una granja de bots que genera contenido:

1. Un **script monitoriza** en tiempo real las tendencias de X (Twitter), Reddit o portales de noticias mediante scraping de APIs no oficiales.
2. Un **orquestador** (por ejemplo, un servidor Python con Celery) selecciona entre los temas detectados aquellos alineados con los objetivos de la campaña de manipulación.
3. Se envía un *prompt* estructurado al LLM local, que incluye el tema, el sentimiento deseado y ejemplos de estilo. El servidor que aloja el modelo (ej. vLLM o llama.cpp sirviendo a través de API REST) devuelve un comentario breve y natural.
4. El texto generado se inyecta en la cola de tareas de **GenFarmer** o de un script a medida, que lo asigna al dispositivo correspondiente junto con las coordenadas del botón de publicar.
5. El agente en el teléfono pega el texto y pulsa el botón, completando la interacción.

La sofisticación puede llegar al punto de que varios bots mantengan conversaciones entre sí, simulando un debate genuino. Un primer bot publica una afirmación polémica, dos bots responden con puntos de vista opuestos, y el sistema LLM se asegura de que los diálogos no sean calcados, variando la estructura sintáctica y el léxico. Para un observador externo —o incluso para los sistemas de detección basados únicamente en la repetición de patrones—, la conversación resulta indistinguible de un intercambio humano real.

## 5.3 Evasión de la detección textual

Los analistas de seguridad no solo deben preocuparse por la calidad lingüística. El uso de LLMs locales introduce nuevas posibilidades de detección:

- **Marcas de agua esteganográficas**: Los investigadores están desarrollando técnicas para insertar patrones estadísticos imperceptibles en el texto generado. Si el modelo local no aplica deliberadamente una contramedida, los sistemas de la plataforma podrían identificar esas marcas si logran acceder a las probabilidades de token o a la entropía del texto.
- **Perplexidad y burstiness**: Los textos generados tienden a tener una distribución de perplejidad más uniforme que los humanos. Herramientas como GPTZero o los clasificadores internos de X analizan estas métricas. Las granjas responden ajustando parámetros como la temperatura, el top-k sampling o el repetition penalty, e incluso pueden intercalar errores tipográficos deliberados o modismos locales para humanizar la salida.
- **Huella digital del modelo**: Cada LLM tiene una “firma” única en la elección de vocabulario y en las transiciones sintácticas. Entrenar clasificadores que aprendan estas huellas específicas de Llama 3 o Dolphin es una línea de defensa activa en los laboratorios de seguridad de las grandes plataformas.

---

# 6. Infraestructura de monetización y servicios auxiliares

Para cerrar el círculo, la granja necesita monetizar su tráfico o cumplir el objetivo de la campaña. Las fuentes verificadas revelan varios modelos:

- **Instalación de apps de ingresos pasivos**: Se instalan masivamente aplicaciones como **Mode Earn App, S'mores o 101 Sweets** que pagan fracciones de céntimo por ver anuncios, completar encuestas o mantener la pantalla encendida. La automatización reproduce anuncios 24/7, generando beneficios agregados sustanciales.
- **Fraude publicitario y de clics**: Los dispositivos navegan por sitios web que alojan anuncios de display y simulan clics mediante scripts de coordenadas, defraudando a anunciantes. Aquí el uso de proxies residenciales es fundamental para superar la verificación geográfica y de audiencia.
- **Manipulación de redes sociales y desinformación**: Las cuentas falsas creadas y mantenidas por la granja amplifican mensajes políticos, atacan a adversarios o inflan artificialmente la popularidad de productos. Se ha documentado el papel de **Telegram** como centro de mando para la difusión de narrativas; un informe de 2025 señalaba que el 76 % de los artículos de desinformación sobre ciertos conflictos se distribuían a través de canales de Telegram antes de ser replicados por los bots en otras plataformas.
- **Verificación de cuentas y SMS**: Servicios como **TextVerified** o **SMSPool** son esenciales para sortear los muros de registro con número de teléfono. Su coste ($0.30–$5 por verificación) es asumible dentro del modelo de negocio, aunque las granjas más sofisticadas internalizan el proceso con sus propios módems GSM y SIMs vírgenes.

---

# 7. Estrategias de detección y defensa: lecciones para el analista

Entender la arquitectura anterior permite extraer vectores de detección que las plataformas y los equipos de seguridad pueden explotar.

## 7.1 Huellas del hardware y del emulador

Aunque se usen teléfonos físicos, la repetición de ciertos campos en el *fingerprint* del dispositivo delata a una granja. Los sistemas de defensa monitorizan:

- La combinación de **Build.MANUFACTURER, Build.MODEL, versión de Android y parches de seguridad**. Una granja mal configurada puede mostrar miles de dispositivos idénticos con el mismo build incremental, algo que rara vez ocurre en la naturaleza.
- La **presencia de herramientas de automatización**: binarios como `su`, `magisk`, `xposed`, aplicaciones con permisos de accesibilidad no justificados, o procesos con nombres como `genfarmer`, `osmb`, `script` son indicios claros. Muchas plataformas ejecutan un escaneo en el lado del cliente antes de confiar en el dispositivo (por ejemplo, SafetyNet de Google).
- **Sensores y telemetría**: Acelerómetro, giroscopio, barómetro, campo magnético. Un dispositivo que nunca se mueve (todos los ejes a valores constantes) o que reporta exactamente la misma variación de luz ambiente es sospechoso. Las granjas más avanzadas intentan emular estas señales con datos sintéticos inyectados a través de módulos Xposed o Magisk, pero la perfección de la emulación es difícil de lograr.

## 7.2 Anomalías de red

- **Rotación de IP contranatura**: Un usuario real no cambia de IP cada 10 minutos ni salta de un estado a otro en cuestión de segundos. Los modelos de ML entrenados con los datos de geolocalización y franjas horarias pueden puntuar la plausibilidad de cada evento.
- **Patrones de tráfico**: La comunicación con endpoints de anuncios, SDKs de analíticas y pasarelas de verificación de anuncios puede seguir patrones rígidos. El fraude de clics, por ejemplo, genera ráfagas de solicitudes HTTP a las URLs de redireccionamiento de anuncios que nunca van seguidas de la carga completa de la página de destino, un patrón bien conocido por los sistemas antifraude.
- **Flujos DNS y SNI**: El uso de proxies residenciales a menudo deja rastros en las consultas DNS (por ejemplo, resolución de dominios del propio proveedor de proxies). El análisis del tráfico de red mediante inspección profunda de paquetes en las CDNs de las plataformas puede revelar estas relaciones.

## 7.3 Comportamiento no humano

Más allá de la calidad del texto, la interacción con la UI ofrece una ventana de detección privilegiada:

- **Coordenadas de toque y presión**: Un script que toca siempre en el mismo píxel con la misma duración de pulso y la misma área de contacto es fácilmente detectable. Las granjas modernas usan aleatorización de coordenadas, pero aun así las distribuciones estadísticas de los gestos no coinciden con las de un humano.
- **Velocidad de navegación y tiempos de reacción**: La automatización puede completar flujos (registro, like, comentario, logout) en milisegundos donde un humano tardaría varios segundos. Los sistemas de defensa cronometran las interacciones y las comparan con modelos de comportamiento humano.
- **Patrones de actividad circadiana**: Una cuenta que publica 24 horas al día, todos los días, sin pausas de sueño, es un fuerte indicio. Las granjas programan ciclos de inactividad simulada, pero la ausencia de variaciones estacionales (vacaciones, fines de semana) sigue siendo una pista.# Hardware: Dispositivos y Estructura Física

# 1. Introducción: el ecosistema de la automatización a escala

Las granjas de bots han evolucionado desde simples scripts en servidores hacia sofisticadas infraestructuras híbridas que combinan miles de dispositivos móviles reales, redes proxy residenciales rotativas, emuladores de Android y modelos de lenguaje de última generación. El **Bad Bot Report 2024** de Imperva reveló que casi la mitad del tráfico de internet ya no es humano, y que los bots maliciosos representan casi un tercio del total. Detrás de ese tráfico se oculta un entramado técnico meticulosamente diseñado para imitar el comportamiento humano, evadir sistemas antifraude y operar a una escala que desborda los mecanismos de defensa tradicionales.

Este artículo disecciona la infraestructura de las granjas modernas, desde los dispositivos físicos hasta la integración de modelos de lenguaje, siguiendo un enfoque de ingeniería inversa aplicada a la ciberdefensa.

# 2. Capa física: el ejército de silicio

## 2.1 Modelos de dispositivos y racks especializados

A diferencia de los bots puramente basados en servidor, las granjas actuales recurren masivamente a teléfonos reales para heredar las huellas digitales de hardware, sensores y radios de comunicación que las plataformas (Google, Facebook, X) consideran como señales de legitimidad. Los modelos elegidos responden a un equilibrio entre coste, disponibilidad y requisitos técnicos mínimos.

- **Samsung Galaxy S8, S8+ y S10** aparecen de forma recurrente porque ofrecen un rendimiento aceptable, soporte para las versiones de Android requeridas por las apps objetivo y un precio de segunda mano muy competitivo cuando se adquieren en lotes.
- **Google Pixel 2** es citado como dispositivo de entrada idóneo para granjas de monetización pasiva. Cumple sobradamente los requisitos de línea base (Android 7.0+, 16 GB de almacenamiento, 2 GB de RAM) y su naturaleza “stock Android” simplifica la automatización.
- **Xiaomi** completa el abanico en operaciones que buscan la máxima escala; sus modelos de gama media y baja son populares en granjas localizadas en Asia.

Estos teléfonos no se amontonan sobre una mesa, sino que se montan en **placas o racks metálicos que agrupan 20, 40 o hasta más dispositivos por unidad**. Soluciones comerciales como **Box Phone Farm** (parte del ecosistema GenFarmer) ofrecen bandejas modulares con ventilación forzada controlable remotamente, hubs USB integrados y conmutadores de carga. Estas bandejas permiten apilar varias unidades verticalmente y conectarlas en cascada a un solo equipo de control mediante conexiones LAN o USB.

En las estanterías de granjas que operan en países asiáticos, además, se han documentado sistemas de **aislamiento térmico** y refrigeración por aire acondicionado dedicado para disipar el calor generado por miles de dispositivos funcionando 24/7, un detalle que subraya la dimensión industrial de estas operaciones.

## 2.2 Conectividad, alimentación y tarjetas SIM

Mantener encendidos y conectados miles de terminales exige una infraestructura eléctrica robusta: **hubs de carga masiva con múltiples puertos USB de alta potencia** y, en instalaciones más profesionales, fuentes de alimentación industriales que convierten la corriente de forma centralizada.

La conectividad de red es uno de los pilares más sensibles. Se documentan dos configuraciones típicas:

1. **Conexión directa por cable Ethernet mediante adaptadores USB-Ethernet o hubs con puerto RJ45**. Así se evita la latencia y la inestabilidad del WiFi, y se facilita la gestión centralizada del tráfico. El componente **GenRouter**, por ejemplo, se anuncia como un router optimizado para granjas, capaz de manejar el flujo de datos de cientos de dispositivos y de integrarse con software de control como GenLogin y GenFarmer.
2. **WiFi con múltiples puntos de acceso y VLANs segmentadas** para separar el tráfico por país o por campaña, a menudo combinado con proxies.

En cuanto a la verificación de identidad, ciertas plataformas (especialmente Facebook) exigen números de teléfono reales para validar cuentas. Aquí entran en juego las **placas con ranuras para tarjetas SIM físicas**, que permiten insertar SIMs de operadores locales o internacionales y programar la recepción de SMS de verificación. Estas placas se conectan mediante módems GSM integrados o vía hubs SIM conectados por USB al PC de control. Servicios como **TextVerified** o **SMSPool** externalizan esta función, proporcionando números temporales a bajo coste, pero los operadores de granjas a gran escala a menudo integran sus propias baterías de SIMs para reducir costes recurrentes y aumentar la fiabilidad.

---

# 3. Capa de control: orquestación centralizada

### 3.1 Plataformas de gestión de flotas

El verdadero cerebro de una granja es el software que permite manejar cientos o miles de dispositivos desde una única consola. **GenFarmer** es el ejemplo paradigmático encontrado en las fuentes: un ecosistema que se presenta abiertamente como solución para “MMO” (Make Money Online) y automatización de marketing. Su arquitectura típica consta de:

- **Servicio en la nube (Backend)**: Gestiona las licencias, almacena scripts, coordina tareas programadas y recibe telemetría de los dispositivos.
- **Cliente de escritorio (Controlador)**: Instalado en un PC con Windows o macOS, se conecta al servicio en la nube y a los dispositivos locales (vía ADB sobre USB o TCP/IP). Desde aquí se definen los flujos de trabajo, se replican comandos y se visualiza en tiempo real la pantalla de cualquier terminal.
- **Agentes en los dispositivos**: Pequeñas aplicaciones o binarios que se instalan en cada teléfono (a menudo con permisos de accesibilidad) y que ejecutan las órdenes de automatización: abrir apps, deslizar, tocar coordenadas, introducir texto, leer SMS, etc.

Otro enfoque es el **control puramente remoto mediante paneles web** (Cloud Control), donde los dispositivos físicos están alojados en centros de datos o en racks alquilados a terceros (“phone farm as a service”). Los operadores ni siquiera tocan físicamente los teléfonos; alquilan capacidad de automatización por horas o por campañas.

### 3.2 Técnicas de automatización y emuladores

Para tareas que no requieren un sensor de huella dactilar ni una cámara real, los atacantes recurren a **emuladores de Android** ejecutados en servidores. Esto abarata drásticamente el coste, ya que un solo servidor puede albergar docenas de instancias virtuales. Emuladores como **BlueStacks, LDPlayer, Nox o Memu** permiten clonar configuraciones, cambiar identificadores de dispositivo (IMEI, Android ID, MAC) y, mediante scripts de automatización, simular la actividad de múltiples usuarios. La integración con ADB (Android Debug Bridge) es el canal a través del cual los scripts envían comandos, instalan APKs y manipulan la interfaz gráfica.

En el ámbito de los videojuegos (caso emblemático *Old School RuneScape*, OSRS), han surgido herramientas especializadas que ilustran técnicas reutilizables en otros contextos fraudulentos:

- **OSMBB (Open Source Mobile Bot, a veces OSMB)** es un cliente de escritorio que se conecta a un emulador de Android y utiliza inyección de código o lectura de memoria para automatizar tareas repetitivas dentro del juego. Funciona estableciendo un puente entre el emulador y un script escrito en Java o Kotlin.
- **WaspScripts** representa la corriente del *color botting*: utiliza la librería **WaspLib** (para Simba 2.0) para buscar colores y patrones en la pantalla, sin tocar el proceso del juego, lo que dificulta la detección por parte de sistemas antitrampas.
- **Runemate** es un bot que se inyecta directamente en el cliente del juego (en este caso, la versión de escritorio) y expone una API para que los scripts interactúen con los objetos del juego. Esta técnica de *injection* es también empleada en otros entornos para manipular aplicaciones bancarias o redes sociales si se logra evadir los mecanismos de ofuscación y root detection.

Desde la óptica defensiva, cualquiera de estos métodos deja trazas. El uso de **permisos de accesibilidad abusivos**, la presencia de binarios con firmas sospechosas, las llamadas a `input tap` o `input swipe` a través de ADB, o la manipulación de la superficie de dibujo, son vectores de detección que las plataformas explotan activamente. Sin embargo, las granjas más avanzadas evitan ADB directo y emplean **microcontroladores externos (Arduino, Teensy)** que emulan eventos de teclado y ratón a nivel de hardware, conectados físicamente al teléfono mediante placas con relés táctiles o interfaces HID, una técnica de evasión cada vez más difícil de contrarrestar.

---

# 4. Capa de red: la ilusión de la ubicuidad

### 4.1 Proxies residenciales rotativos

Un solo router doméstico con una IP de centro de datos sería detectado instantáneamente. Las granjas utilizan **proxies residenciales**, es decir, direcciones IP que pertenecen a ISP de consumidores reales. Estas IPs se obtienen a través de redes peer-to-peer (como Honeygain o Peer2Profit) o mediante la inclusión de SDKs en apps legítimas que ceden el tráfico de fondo del usuario sin su conocimiento pleno. Servicios comerciales como **Bright Data, Oxylabs o Smartproxy** ofrecen pools de millones de IPs residenciales listas para rotar bajo demanda.

En el contexto de una granja, un software como **Superproxy** (citado en las fuentes) actúa como un middleware que asigna a cada dispositivo, o a cada perfil de app, una IP residencial específica y la mantiene el tiempo necesario para que la cuenta parezca coherente geográficamente. La rotación se programa para simular movilidad humana: una misma cuenta puede aparecer un día en Dallas, al siguiente en Seattle, con cambios de IP que coinciden con franjas horarias y patrones de tráfico móvil.

### 4.2 Túneles VPN y encapsulado

Además de los proxies HTTP/SOCKS5, muchas granjas envuelven todo el tráfico del dispositivo en un túnel VPN que redirige todas las conexiones a través de un servidor intermedio. Esto asegura que incluso las consultas DNS, los WebRTC leaks y las conexiones WebSocket pasen por la IP designada, evitando fugas que revelen la ubicación real de la granja. Herramientas como **OpenVPN, WireGuard o proxies inversos personalizados** se despliegan en el dispositivo a nivel de sistema (con permisos de superusuario) para forzar la transparencia total del enmascaramiento.

Para las granjas alojadas en centros de datos, a menudo se utiliza un esquema de doble salto: el dispositivo se conecta a un proxy residencial a través de un túnel cifrado, y ese proxy, a su vez, sale a internet desde una IP doméstica real. De esta forma, la plataforma objetivo solo ve la IP de un hogar estadounidense, por ejemplo, sin rastro alguno del datacenter de origen.

---

# 5. Capa de inteligencia: la irrupción de los LLMs locales

### 5.1 Generación de contenido verosímil

El último salto cualitativo es la integración de modelos de lenguaje de gran tamaño (LLMs) directamente en la infraestructura de la granja. Antes, los comentarios de los bots se limitaban a frases genéricas copiadas y pegadas o a cadenas de Markov poco sofisticadas. Ahora, un solo operador puede generar, en tiempo real, respuestas contextuales que imitan el tono, la jerga y las posturas ideológicas del público objetivo.

Las fuentes consultadas apuntan al uso de **Llama 3 70B** y **Dolphin** (una versión ajustada de Mixtral, o un fine-tune del propio Llama 3, sin las restricciones de seguridad típicas) ejecutados localmente en servidores con GPU dentro de la misma granja o a través de una API interna. La elección del despliegue local es deliberada: reduce la latencia, evita el coste de APIs comerciales y, crucialmente, elude los filtros de contenido de proveedores como OpenAI o Anthropic. Un modelo sin censura puede generar mensajes que inciten al odio, desinformación o cualquier contenido tóxico sin que un guardarraíl automático lo bloquee.

### 5.2 Arquitectura de integración

El flujo típico en una granja de bots que genera contenido:

1. Un **script monitoriza** en tiempo real las tendencias de X (Twitter), Reddit o portales de noticias mediante scraping de APIs no oficiales.
2. Un **orquestador** (por ejemplo, un servidor Python con Celery) selecciona entre los temas detectados aquellos alineados con los objetivos de la campaña de manipulación.
3. Se envía un *prompt* estructurado al LLM local, que incluye el tema, el sentimiento deseado y ejemplos de estilo. El servidor que aloja el modelo (ej. vLLM o llama.cpp sirviendo a través de API REST) devuelve un comentario breve y natural.
4. El texto generado se inyecta en la cola de tareas de **GenFarmer** o de un script a medida, que lo asigna al dispositivo correspondiente junto con las coordenadas del botón de publicar.
5. El agente en el teléfono pega el texto y pulsa el botón, completando la interacción.

La sofisticación puede llegar al punto de que varios bots mantengan conversaciones entre sí, simulando un debate genuino. Un primer bot publica una afirmación polémica, dos bots responden con puntos de vista opuestos, y el sistema LLM se asegura de que los diálogos no sean calcados, variando la estructura sintáctica y el léxico. Para un observador externo —o incluso para los sistemas de detección basados únicamente en la repetición de patrones—, la conversación resulta indistinguible de un intercambio humano real.

### 5.3 Evasión de la detección textual

Los analistas de seguridad no solo deben preocuparse por la calidad lingüística. El uso de LLMs locales introduce nuevas posibilidades de detección:

- **Marcas de agua esteganográficas**: Los investigadores están desarrollando técnicas para insertar patrones estadísticos imperceptibles en el texto generado. Si el modelo local no aplica deliberadamente una contramedida, los sistemas de la plataforma podrían identificar esas marcas si logran acceder a las probabilidades de token o a la entropía del texto.
- **Perplexidad y burstiness**: Los textos generados tienden a tener una distribución de perplejidad más uniforme que los humanos. Herramientas como GPTZero o los clasificadores internos de X analizan estas métricas. Las granjas responden ajustando parámetros como la temperatura, el top-k sampling o el repetition penalty, e incluso pueden intercalar errores tipográficos deliberados o modismos locales para humanizar la salida.
- **Huella digital del modelo**: Cada LLM tiene una “firma” única en la elección de vocabulario y en las transiciones sintácticas. Entrenar clasificadores que aprendan estas huellas específicas de Llama 3 o Dolphin es una línea de defensa activa en los laboratorios de seguridad de las grandes plataformas.

---

# 6. Infraestructura de monetización y servicios auxiliares

Para cerrar el círculo, la granja necesita monetizar su tráfico o cumplir el objetivo de la campaña. Las fuentes verificadas revelan varios modelos:

- **Instalación de apps de ingresos pasivos**: Se instalan masivamente aplicaciones como **Mode Earn App, S'mores o 101 Sweets** que pagan fracciones de céntimo por ver anuncios, completar encuestas o mantener la pantalla encendida. La automatización reproduce anuncios 24/7, generando beneficios agregados sustanciales.
- **Fraude publicitario y de clics**: Los dispositivos navegan por sitios web que alojan anuncios de display y simulan clics mediante scripts de coordenadas, defraudando a anunciantes. Aquí el uso de proxies residenciales es fundamental para superar la verificación geográfica y de audiencia.
- **Manipulación de redes sociales y desinformación**: Las cuentas falsas creadas y mantenidas por la granja amplifican mensajes políticos, atacan a adversarios o inflan artificialmente la popularidad de productos. Se ha documentado el papel de **Telegram** como centro de mando para la difusión de narrativas; un informe de 2025 señalaba que el 76 % de los artículos de desinformación sobre ciertos conflictos se distribuían a través de canales de Telegram antes de ser replicados por los bots en otras plataformas.
- **Verificación de cuentas y SMS**: Servicios como **TextVerified** o **SMSPool** son esenciales para sortear los muros de registro con número de teléfono. Su coste ($0.30–$5 por verificación) es asumible dentro del modelo de negocio, aunque las granjas más sofisticadas internalizan el proceso con sus propios módems GSM y SIMs vírgenes.

---

# 7. Estrategias de detección y defensa: lecciones para el analista

Entender la arquitectura anterior permite extraer vectores de detección que las plataformas y los equipos de seguridad pueden explotar.

### 7.1 Huellas del hardware y del emulador

Aunque se usen teléfonos físicos, la repetición de ciertos campos en el *fingerprint* del dispositivo delata a una granja. Los sistemas de defensa monitorizan:

- La combinación de **Build.MANUFACTURER, Build.MODEL, versión de Android y parches de seguridad**. Una granja mal configurada puede mostrar miles de dispositivos idénticos con el mismo build incremental, algo que rara vez ocurre en la naturaleza.
- La **presencia de herramientas de automatización**: binarios como `su`, `magisk`, `xposed`, aplicaciones con permisos de accesibilidad no justificados, o procesos con nombres como `genfarmer`, `osmb`, `script` son indicios claros. Muchas plataformas ejecutan un escaneo en el lado del cliente antes de confiar en el dispositivo (por ejemplo, SafetyNet de Google).
- **Sensores y telemetría**: Acelerómetro, giroscopio, barómetro, campo magnético. Un dispositivo que nunca se mueve (todos los ejes a valores constantes) o que reporta exactamente la misma variación de luz ambiente es sospechoso. Las granjas más avanzadas intentan emular estas señales con datos sintéticos inyectados a través de módulos Xposed o Magisk, pero la perfección de la emulación es difícil de lograr.

### 7.2 Anomalías de red

- **Rotación de IP contranatura**: Un usuario real no cambia de IP cada 10 minutos ni salta de un estado a otro en cuestión de segundos. Los modelos de ML entrenados con los datos de geolocalización y franjas horarias pueden puntuar la plausibilidad de cada evento.
- **Patrones de tráfico**: La comunicación con endpoints de anuncios, SDKs de analíticas y pasarelas de verificación de anuncios puede seguir patrones rígidos. El fraude de clics, por ejemplo, genera ráfagas de solicitudes HTTP a las URLs de redireccionamiento de anuncios que nunca van seguidas de la carga completa de la página de destino, un patrón bien conocido por los sistemas antifraude.
- **Flujos DNS y SNI**: El uso de proxies residenciales a menudo deja rastros en las consultas DNS (por ejemplo, resolución de dominios del propio proveedor de proxies). El análisis del tráfico de red mediante inspección profunda de paquetes en las CDNs de las plataformas puede revelar estas relaciones.

### 7.3 Comportamiento no humano

Más allá de la calidad del texto, la interacción con la UI ofrece una ventana de detección privilegiada:

- **Coordenadas de toque y presión**: Un script que toca siempre en el mismo píxel con la misma duración de pulso y la misma área de contacto es fácilmente detectable. Las granjas modernas usan aleatorización de coordenadas, pero aun así las distribuciones estadísticas de los gestos no coinciden con las de un humano.
- **Velocidad de navegación y tiempos de reacción**: La automatización puede completar flujos (registro, like, comentario, logout) en milisegundos donde un humano tardaría varios segundos. Los sistemas de defensa cronometran las interacciones y las comparan con modelos de comportamiento humano.
- **Patrones de actividad circadiana**: Una cuenta que publica 24 horas al día, todos los días, sin pausas de sueño, es un fuerte indicio. Las granjas programan ciclos de inactividad simulada, pero la ausencia de variaciones estacionales (vacaciones, fines de semana) sigue siendo una pista.
