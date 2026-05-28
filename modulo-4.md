**MÓDULO 4: GESTIÓN DE IDENTIDAD Y RED**

**Duración:** 4 horas (con pausa intermedia)
**Objetivo del módulo:** Al finalizar esta sesión, seréis capaces de desvelar cómo el ecosistema Gen construye identidades digitales falsas creíbles y cómo enmascara la red para evadir bloqueos, identificando los puntos débiles que permiten la detección.

---

**1. INTRODUCCIÓN AL MÓDULO (15 minutos)**

*El profesor comienza.*

En los módulos anteriores, hemos diseccionado el agente APK y los protocolos de control. Hoy vamos a abordar dos cuestiones que son el corazón del engaño: ¿cómo consigue GenFarmer que cada cuenta parezca un usuario real y distinto? Y ¿cómo evita que todas esas cuentas se bloqueen por provenir de la misma red?

Estamos hablando de la **gestión de identidad** y la **gestión de red**. Sin estas dos capas, todo el tinglado se derrumba. Podéis tener mil teléfonos ejecutando toques perfectos, pero si todos comparten la misma IP o el mismo navegador, las plataformas os detectan en minutos.

GenFarmer no trabaja solo en esto. Recordad del Módulo 1 que el ecosistema Gen incluye **GenLogin**, el navegador anti-detección. Hoy veremos cómo estas dos herramientas se complementan para crear identidades digitales coherentes y cómo se gestionan los proxies a nivel de dispositivo.

Dividiré el módulo en tres grandes bloques: primero, GenLogin y la creación de perfiles de navegador; segundo, la gestión de proxies en el dispositivo; y tercero, la gestión del Advertising ID y otras señales de identidad móvil. En cada bloque, identificaremos los indicadores de compromiso y las estrategias de detección.

Empecemos.

---

**2. GENLOGIN Y LA CREACIÓN DE PERFILES DE NAVEGADOR (75 minutos)**

*El profesor proyecta una diapositiva con una captura de pantalla de la interfaz de GenLogin.*

**2.1 ¿Qué es GenLogin y por qué es necesario? (15 minutos)**

GenLogin es un navegador anti-detección. Pero decir solo eso es quedarse corto. Es una herramienta que permite crear **perfiles de navegador** con huellas digitales únicas y personalizables, diseñados para no ser detectados por los sistemas antifraude de las plataformas.

Pensad en lo que hace vuestro navegador cuando visitáis una web. Sin que lo sepáis, está enviando decenas de datos: el User-Agent, la resolución de pantalla, las fuentes instaladas, la salida de WebGL y Canvas, la zona horaria, el idioma, los codecs de audio, los plugins... Todo esto compone vuestra **huella digital** (fingerprint). Es como vuestro DNI digital.

Si intentáis crear 100 cuentas de Facebook desde el mismo navegador, Facebook verá 100 cuentas con la misma huella digital. Bloqueo instantáneo. Necesitáis que cada cuenta parezca provenir de un ordenador distinto, en una ciudad distinta, con una configuración distinta.

Para eso sirve GenLogin. Genera perfiles de navegador que simulan dispositivos reales: ordenadores Windows, Mac, diferentes versiones de Chrome y Firefox, con todas las variables de fingerprint ajustadas para ser coherentes entre sí. Por ejemplo, si configuráis un perfil como "usuario español en Windows 10 con Chrome 120", GenLogin se asegura de que el User-Agent, las fuentes, la zona horaria y el idioma coincidan con ese perfil. Si ponéis un User-Agent de Windows pero las fuentes de Mac, hay incoherencia y la plataforma lo detecta.

**2.2 Parámetros de fingerprint gestionados por GenLogin (20 minutos)**

*El profesor muestra una tabla detallada en la diapositiva.*

Vamos a desglosar los parámetros que GenLogin modifica o falsifica. Los conoceréis como investigadores porque son exactamente los mismos que debéis monitorizar en vuestros sistemas de detección:

1. **User-Agent**: cadena que identifica navegador, versión y sistema operativo. GenLogin permite elegir entre decenas de combinaciones predefinidas o personalizarlas. Lo importante es la coherencia: el User-Agent debe coincidir con el sistema operativo real de la máquina.

2. **WebGL y Canvas Fingerprint**: cada GPU genera una firma única al renderizar gráficos WebGL y Canvas. GenLogin intercepta estas APIs y devuelve una firma sintética, diferente para cada perfil. Es uno de los parámetros más difíciles de falsificar sin que se note, porque la firma debe ser matemáticamente plausible.

3. **Fonts (fuentes tipográficas)**: la lista de fuentes instaladas varía según el sistema operativo y las aplicaciones. GenLogin configura una lista de fuentes realistas para cada perfil.

4. **Resolución de pantalla y color depth**: ancho, alto y profundidad de color. GenLogin puede simular monitores 1080p, 4K, etc.

5. **Zona horaria e idioma**: `Intl.DateTimeFormat().resolvedOptions().timeZone` debe coincidir con la IP de salida. Si un perfil dice estar en Madrid pero su zona horaria es UTC-5 (Nueva York), hay una incoherencia que los sistemas detectan.

6. **WebRTC**: puede filtrar la IP real incluso detrás de un proxy. GenLogin bloquea o modifica WebRTC para que devuelva la IP del proxy, no la real.

7. **Cookies, localStorage y sessionStorage**: GenLogin almacena estos datos de forma aislada para cada perfil, de modo que cada cuenta tenga su propio espacio de almacenamiento.

8. **AudioContext fingerprint**: similar al Canvas, el subsistema de audio genera una huella única. GenLogin la falsifica.

9. **Plugins y MIME types**: lista de plugins del navegador (Flash, PDF, etc.) y tipos MIME soportados.

**Ejercicio práctico (15 minutos):**
Abrid el navegador Chrome y visitad `browserleaks.com/canvas` o `amiunique.org`. Anotad vuestra huella digital real. Luego, abrid el perfil de GenLogin que os he proporcionado en la máquina virtual del laboratorio (simulado). Visitad la misma página y comparad las huellas. ¿Qué parámetros han cambiado? ¿Son coherentes entre sí? Identificad al menos tres diferencias y explicad cómo podríais usar esa información para detectar un perfil falso.

**2.3 Integración de GenLogin con GenFarmer (25 minutos)**

*El profesor muestra un diagrama de flujo.*

GenLogin se ejecuta en el PC del operador, no en los teléfonos. Pero la integración con GenFarmer es crucial para transferir sesiones. ¿Cómo se hace?

**Flujo típico:**

1. El operador crea un perfil en GenLogin con una huella digital específica.
2. Dentro de ese perfil, navega manualmente (o mediante automatización) hasta la plataforma objetivo (por ejemplo, Facebook) y crea una cuenta nueva. Durante este proceso, la plataforma almacena cookies de sesión en el navegador del perfil.
3. Una vez creada la cuenta en el navegador, el operador quiere transferir esa sesión al dispositivo móvil para que GenFarmer la gestione. Para ello:
   - **Opción A (cookies):** GenLogin exporta las cookies del perfil en formato JSON o Netscape. GenFarmer, a través de su agente, inyecta esas cookies en el `WebView` de la app objetivo (por ejemplo, la app de Facebook). Esto requiere que la app use `WebView` para la autenticación, o que el agente abra una vista web interna y cargue las cookies antes de redirigir a la app nativa.
   - **Opción B (backup de app):** GenFarmer puede restaurar una copia de seguridad de la app objetivo que ya contiene la sesión iniciada. Esto se hace mediante `adb backup` y `adb restore`, o copiando los directorios de datos de la app (`/data/data/com.facebook.katana/shared_prefs/`) si el dispositivo tiene root.
   - **Opción C (automatización del login):** directamente, GenFarmer abre la app en el teléfono, pega el usuario y contraseña desde el portapapeles (generados en GenLogin), y completa el login de forma automatizada.

**Para el defensor, ¿qué implica esto?**

- La transferencia de cookies desde un entorno de escritorio a uno móvil puede dejar rastros. Las cookies de sesión tienen atributos como `SameSite` y `Secure` que pueden comportarse de forma extraña al moverse de contexto.
- Si observáis que una cuenta recién creada en un navegador de escritorio inicia sesión segundos después desde un dispositivo móvil, y además las cookies coinciden, hay una correlación sospechosa. La diferencia de fingerprint entre el navegador y el dispositivo móvil es un indicio.

**Ejercicio práctico (15 minutos):**
En el laboratorio, tenéis una cookie de sesión exportada de GenLogin (formato JSON) y una app de prueba que he creado (simula una red social). Instalad la app en el emulador, inyectad la cookie usando un script de ADB (os lo proporciono) y comprobad que la app reconoce la sesión. Documentad el proceso: ¿qué archivos se modifican en el dispositivo? ¿Qué metadatos de la cookie (dominio, caducidad, SameSite) podrían delatar la transferencia?

*Pausa de 15 minutos.*

**2.4 Implicaciones forenses y de detección (15 minutos)**

*El profesor retoma.*

Para nosotros, los defensores, GenLogin no es invencible. Tiene puntos débiles:

- **Coherencia del fingerprint:** aunque GenLogin se esfuerza, a veces falla. Por ejemplo, la lista de fuentes puede no coincidir exactamente con la esperada para el sistema operativo declarado. Herramientas como FingerprintJS o los sistemas internos de las plataformas analizan estas incoherencias.
- **Huella de la herramienta:** la propia presencia de GenLogin deja rastros en el tráfico. Algunas versiones añaden cabeceras HTTP personalizadas o tienen patrones de negociación TLS ligeramente diferentes. El JA3 fingerprint del cliente TLS puede delatar que se está usando un navegador modificado.
- **Comportamiento de navegación:** un perfil recién creado que inmediatamente realiza acciones de registro sin historial de navegación previo es sospechoso. Las plataformas analizan el "comportamiento de calentamiento" (warming up).
- **Cookie stuffing y sesiones clonadas:** si detectáis que una misma cookie de sesión aparece en múltiples dispositivos (porque el operador la exportó y la reutilizó), habéis pillado una clonación de sesiones. Esto es un indicador de compromiso de altísima fidelidad.

---

**3. GESTIÓN DE PROXIES EN EL DISPOSITIVO (75 minutos)**

*El profesor muestra una diapositiva con la arquitectura de red del ecosistema Gen.*

Ahora pasamos a la capa de red en el dispositivo móvil. Recordad: cada teléfono de la granja debe salir a internet con una IP distinta, idealmente residencial, y esa IP debe ser coherente con la geolocalización del perfil.

**3.1 Opción A: sin root – VPNService (30 minutos)**

Cuando el dispositivo no tiene permisos de superusuario, GenFarmer utiliza la API `VpnService` de Android para crear un túnel VPN local. Esta API está pensada para aplicaciones de VPN legítimas (como NordVPN o ExpressVPN), pero el agente la usa con fines de enmascaramiento.

**Funcionamiento técnico:**

1. El agente solicita al usuario (o fuerza mediante el servicio de accesibilidad) que acepte la creación de una VPN local. Aparece el icono de llave en la barra de estado.
2. Una vez activada, el agente crea una interfaz de red virtual (`tun0`). Todo el tráfico del dispositivo puede ser redirigido a través de esta interfaz.
3. La aplicación `VpnService` recibe los paquetes IP, los encapsula y los envía a un servidor proxy remoto (SOCKS5 o HTTP) a través de una conexión TCP. El proxy, a su vez, reenvía el tráfico a internet con su propia IP.
4. GenFarmer puede configurar un *split-tunneling* rudimentario: solo redirige el tráfico de las apps objetivo (por ejemplo, Instagram, Facebook), mientras que el tráfico del sistema (actualizaciones de Google, telemetría del agente) sale por la IP real de la granja. Esto se consigue filtrando por UID de la aplicación en el método `addRoute()`.

**Implicaciones forenses:**

- **Presencia de `tun0`:** un análisis de interfaces de red (`ip addr show` o `ifconfig`) revela una interfaz `tun0` que no debería estar ahí en un teléfono normal.
- **Icono de VPN activa:** visualmente, el usuario puede ver el icono de llave. En una granja, esto no importa, pero en un dispositivo de una víctima real podría ser una pista.
- **Análisis de tráfico:** si capturáis tráfico en el dispositivo (con `tcpdump`), veréis que las conexiones de la app objetivo van dirigidas a la IP del proxy, no a los servidores de la plataforma. La IP de destino es el proxy, y el SNI (en TLS) puede estar ofuscado o ser el del servidor real.
- **Detección por la plataforma:** algunas apps detectan la presencia de una VPN y niegan el servicio. Otras analizan si la IP de salida coincide con la IP del túnel. Si hay discrepancia, es sospechoso.

**Ejercicio práctico (15 minutos):**
En el emulador, activad una VPN de prueba (podéis usar la app de ejemplo de Android Studio). Monitorizad las interfaces con `adb shell ip addr` antes y después. Luego, con mitmproxy, inspeccionad el tráfico para ver cómo se encapsula. Documentad los cambios en la tabla de enrutamiento y en las reglas iptables (si las hay). ¿Podríais escribir un script que detecte automáticamente la presencia de una VPN no autorizada?

**3.2 Opción B: con root – iptables (30 minutos)**

*El profesor muestra una terminal con reglas iptables.*

Si el dispositivo está rooteado, GenFarmer puede usar un método mucho más sigiloso: `iptables`. No necesita crear una VPN visible, sino que modifica directamente las reglas de firewall del kernel para redirigir el tráfico.

**Funcionamiento técnico:**

1. El agente (con permisos de superusuario) ejecuta comandos como:
   ```bash
   iptables -t nat -A OUTPUT -p tcp --dport 443 -m owner --uid-owner 10087 -j DNAT --to-destination <proxy_ip>:<proxy_port>
   ```
   Donde `10087` es el UID de la app objetivo (por ejemplo, Instagram). Esto redirige todo el tráfico HTTPS de esa app hacia el proxy.
2. Adicionalmente, puede haber reglas `SNAT` o `MASQUERADE` para reescribir la IP de origen.
3. Para tráfico HTTP, se puede usar un proxy transparente con `TPROXY` o `REDIRECT`.

Este método es más sigiloso porque:
- No crea una interfaz `tun0` visible.
- No muestra el icono de VPN.
- El tráfico parece salir directamente por la interfaz WiFi/móvil, pero a nivel de kernel es redirigido.

**Implicaciones forenses:**

- **Inspección de reglas iptables:** si el dispositivo está bajo nuestro control (análisis forense), podemos volcar las reglas con `iptables -t nat -L -v`. Veremos reglas de redirección hacia IPs de proxies. La presencia de reglas `DNAT` que apuntan a un puerto no estándar es sospechosa.
- **Monitorización de procesos:** el comando `iptables` debe ejecutarse. Un sistema de detección puede monitorizar las llamadas a `execve` de `iptables` mediante Frida o auditd.
- **Tráfico de red:** en una captura, veremos que la app objetivo se conecta a una IP (el proxy) en lugar de a los servidores legítimos. Si la IP de destino es conocida por ser un proxy residencial, es una bandera roja.

**Comparativa entre ambos métodos:**

| Característica | VPNService (sin root) | iptables (con root) |
|----------------|------------------------|----------------------|
| Visibilidad | Icono VPN, interfaz `tun0` | Ninguna en UI, solo reglas de firewall |
| Detectabilidad por apps | Media (detectan VPN) | Baja (a menos que auditen iptables) |
| Requisitos | Aceptación del usuario | Root |
| Rendimiento | Ligera sobrecarga por encapsulado | Casi nativo |

**Ejercicio práctico (15 minutos):**
En el emulador rooteado, ejecutad el script que os proporciono (simula la activación de iptables por GenFarmer). Inspeccionad las reglas antes y después con `adb shell iptables -t nat -L`. Identificad la IP del proxy y el UID de la app objetivo. Luego, lanzad tráfico desde la app objetivo y capturadlo con `tcpdump`. ¿Veis la redirección? Documentad vuestras observaciones.

**3.3 Rotación de IP y asignación de proxies (15 minutos)**

GenFarmer no asigna una IP fija a cada dispositivo durante toda su vida útil. Las va rotando para simular movilidad humana. El panel de control mantiene una lista de proxies (obtenidos de proveedores como Bright Data, Oxylabs o ProxySmart) y los asigna dinámicamente.

**Criterios de rotación:**
- **Por tiempo:** cada 30-120 minutos, se cambia la IP para simular que el usuario se ha movido a otro lugar.
- **Por bloqueo:** si la plataforma objetivo devuelve un error de autenticación o un CAPTCHA, el agente puede cambiar automáticamente de proxy.
- **Por acción:** antes de realizar una acción crítica (por ejemplo, una compra fraudulenta), se asigna un proxy "limpio" de alta reputación.

**Detección de la rotación:**
- Un usuario real no cambia de IP cada 30 minutos ni salta de una ciudad a otra en segundos. Los modelos de ML de las plataformas puntúan la verosimilitud geográfica de los cambios de IP.
- Si monitorizáis las IPs de inicio de sesión de una cuenta, una rotación demasiado rápida o geográficamente imposible (Madrid, luego Berlín, luego Madrid en 5 minutos) es un indicio de proxy rotativo.

---

**4. GESTIÓN DEL ADVERTISING ID Y OTRAS SEÑALES DE IDENTIDAD (30 minutos)**

*El profesor cambia de diapositiva.*

Hasta ahora hemos hablado de navegadores e IPs. Pero en el ecosistema móvil, la identidad no se limita a eso. El **Advertising ID (AAID)** es el identificador principal que las apps y los anunciantes utilizan para rastrear a los usuarios y atribuir instalaciones. Para el fraude, es crítico restablecer este ID después de cada acción fraudulenta, de modo que la siguiente instalación parezca de un usuario nuevo.

**4.1 Funcionamiento del AAID (10 minutos)**

El AAID (Google Advertising ID) es un identificador único, reseteable por el usuario, que se almacena en `com.google.android.gms`. Las apps lo obtienen mediante:
```java
AdvertisingIdClient.getAdvertisingIdInfo(context).getId();
```
Si un usuario restablece su AAID desde los ajustes de Google, todas las apps dejan de poder asociar el nuevo AAID con el anterior. Para un granjero, esto es ideal: después de instalar una app y cobrar la comisión, restablecen el AAID y el dispositivo está "limpio" para la siguiente campaña.

**4.2 Métodos de restablecimiento del AAID (10 minutos)**

- **Sin root:** GenFarmer puede automatizar la navegación por los ajustes de Google hasta la opción "Restablecer ID de publicidad" usando `AccessibilityService`. Es lento, pero no requiere permisos especiales.
- **Con root:** directamente, el agente puede manipular la base de datos donde se almacena el AAID (`/data/data/com.google.android.gms/databases/adid.db`) o usar módulos Xposed para falsificar el AAID al vuelo.

**4.3 Implicaciones forenses (10 minutos)**

Un dispositivo que restablece su AAID varias veces al día es, sin duda, una granja. La tasa de restablecimiento de AAID en dispositivos normales es bajísima (quizás una vez cada varios meses, o nunca). Monitorizar la frecuencia de cambio de AAID es una técnica defensiva muy potente.

Google Play Services registra eventos de restablecimiento. Con acceso root, podéis inspeccionar los logs de GMS. Incluso sin root, podéis correlacionar eventos de instalación de apps con cambios de AAID: si veis una cuenta de Google que instala 10 apps nuevas cada hora, y entre cada instalación el AAID cambia, es una granja.

**Otras señales de identidad:**
- **IMEI/MEID** (en dispositivos con módem): GenFarmer no puede cambiar el IMEI sin modificar la partición de radio (arriesgado y a menudo ilegal). Por eso usan teléfonos físicos; cada uno tiene un IMEI único y real. Pero si una granja compra lotes de teléfonos, los IMEIs pueden ser correlacionables (rangos secuenciales).
- **Android ID:** es un identificador único del dispositivo, se restablece con un restablecimiento de fábrica. GenFarmer puede forzar un `adb shell settings put secure android_id <nuevo_id>` si tiene root, o restaurar una copia de seguridad limpia.
- **Números de serie:** se pueden falsificar con módulos Magisk como `MagiskHidePropsConf`.

---

**5. CIERRE DEL MÓDULO Y TAREA (15 minutos)**

*El profesor muestra un resumen.*

Hemos desmontado la fábrica de identidades de Gen. Sabemos cómo GenLogin crea perfiles de navegador coherentes, cómo esos perfiles se transfieren a los dispositivos, cómo se enmascaran las IPs mediante VPNService o iptables, y cómo se restablece el AAID para el fraude.

**Ideas clave que debéis retener:**

1. **La coherencia es la debilidad.** Si un perfil de navegador dice ser de Madrid pero su IP es de Varsovia, hay una incoherencia. Si un dispositivo tiene un AAID que cambia cada hora, hay una incoherencia. Vuestra defensa debe basarse en detectar estas fisuras.
2. **Cada capa deja rastros.** GenLogin deja huellas TLS y de comportamiento de navegación. VPNService deja interfaces `tun0`. iptables deja reglas visibles. El restablecimiento de AAID deja eventos en GMS.
3. **La correlación es el arma definitiva.** No basta con mirar una señal aislada. Debéis correlacionar IP, AAID, fingerprint, cookies y comportamiento. Un sistema de detección de fraude moderno integra todas estas señales en un modelo de riesgo.

**Tarea para el Módulo 5:**
Os entregaré un conjunto de datos simulados de 1.000 sesiones de dispositivos, con información de IP, AAID, fingerprint de navegador (si aplica), y eventos de instalación de apps. Vuestro trabajo es identificar qué sesiones corresponden a una granja y justificar por qué, basándoos en las incoherencias y patrones que hemos visto hoy. Traed vuestro análisis a la próxima sesión.

Además, leed sobre los protocolos de fraude publicitario CPA y click injection. En el Módulo 5, reconstruiremos campañas reales de fraude y analizaremos los flujos de trabajo completos.

**Ronda de preguntas (5 minutos).**

¿Alguna duda? Perfecto. Nos vemos en el Módulo 5.
