# Gestion Logica: GenFarmer, OSMBB y Dolphin

---

# Disección técnica de tres pilares de la automatización moderna

## Introducción

Cuando se analiza el fenómeno de las granjas de dispositivos, tres nombres aparecen de forma recurrente en foros, tiendas de software y conversaciones sobre automatización a escala: **GenFarmer** (el ecosistema integral de gestión de flotas móviles), **OSMBB** (el bot especializado en juegos móviles con emulador) y **Dolphin** (el modelo de lenguaje sin restricciones que insufla "inteligencia" a los bots). Comprender a fondo su arquitectura, sus protocolos de comunicación y sus puntos débiles es fundamental para cualquier analista de ciberseguridad que quiera detectar, atribuir o neutralizar el tráfico no humano.

Este artículo desmenuza cada una de estas herramientas desde una óptica estrictamente técnica y defensiva.

---

## 1. GenFarmer: el sistema nervioso de una flota de mil dispositivos

GenFarmer no es simplemente una aplicación; es un **ecosistema de cuatro componentes** (Box Phone Farm, GenFarmer, GenRouter y GenLogin) diseñado para gestionar, automatizar y monitorizar flotas de dispositivos Android físicos con mínima intervención humana. Su mercado declarado es el "MMO" (Make Money Online) y el marketing digital, pero su arquitectura es idéntica a la de cualquier herramienta de gestión empresarial de dispositivos (MDM) reutilizada con fines espurios.

### 1.1 Arquitectura hardware-software

- **Box Phone Farm**: es un rack metálico con ventiladores, puertos USB y bandejas para anclar entre 10 y cientos de teléfonos. Incluye un microcontrolador integrado que permite controlar remotamente la velocidad de los ventiladores y alternar entre conexión LAN y USB. Desde el punto de vista forense, este hardware no deja huella en el dispositivo móvil, ya que simplemente suministra alimentación y conectividad.
- **GenRouter**: un router personalizado (probablemente basado en OpenWrt) que prioriza el tráfico de los dispositivos, permite crear VLANs y agrupa las IPs internas bajo una única salida WAN gestionable. El verdadero valor añadido es su integración con el backend en la nube de GenFarmer, sincronizando la topología de red con las campañas de automatización.
- **GenFarmer (software de control)**: es una aplicación de escritorio (Windows/macOS) que se comunica con un backend en la nube y con cada dispositivo mediante ADB (Android Debug Bridge) sobre TCP/IP o USB. Su interfaz muestra en una cuadrícula la pantalla de todos los teléfonos simultáneamente (screen mirroring vía minicap o scrcpy integrado), permitiendo al operador lanzar comandos a uno, a varios o a todos los dispositivos a la vez.

### 1.2 Mecanismos de automatización en los dispositivos

Para que GenFarmer pueda automatizar acciones, es necesario instalar una **aplicación agente** en cada teléfono. Esta app solicita permisos de accesibilidad (AccessibilityService) y, en versiones más invasivas, permisos de superusuario (root) o utiliza el daemon `adbd` con privilegios elevados.

El flujo de automatización típico sigue estos pasos:

1. El operador graba una macro (clic en coordenadas, texto, esperas) o importa un script en formato JSON/XML que describe los gestos.
2. El controlador de escritorio envía la orden al backend en la nube, que la retransmite al agente en el dispositivo seleccionado.
3. El agente ejecuta la acción mediante uno de estos métodos:
    - **Input injection a través de AccessibilityService**: realiza gestos y toques simulando un servicio de accesibilidad legítimo. Esta técnica no requiere root y es la más extendida porque sortea muchas detecciones simples. El evento se inyecta a nivel de framework, por lo que la app objetivo recibe un `MotionEvent` con fuente `TOUCH_SCREEN` y apariencia humana si los parámetros (presión, tamaño, duración) se aleatorizan.
    - **ADB shell input tap/swipe**: más simple pero más detectable, ya que muchas apps monitorizan la presencia de `adb shell input`. GenFarmer evita esta vía salvo para instalación de APKs o ajustes del sistema.
    - **Llamadas directas a `InputManager.injectInputEvent()`** mediante reflexión o con privilegios de sistema (si el dispositivo está rooteado). Es más rápido y más difícil de detectar desde la app de usuario, pero modificar el sistema deja rastros (particiones modificadas, binarios `su`).
4. Las capturas de pantalla se transmiten comprimidas (MJPEG/H.264) al panel de control para verificación humana ocasional.

### 1.3 Sistema de cuentas y proxies

GenFarmer integra un gestor de cuentas que asigna a cada dispositivo un perfil compuesto por:

- Un **nombre de cuenta** (usuario/contraseña) generado por el operador.
- Una **huella digital de dispositivo** (fingerprint) configurable: IMEI, Android ID, Advertising ID, dirección MAC WiFi.
- Una **dirección IP de salida** proporcionada por un proxy SOCKS5 o HTTP, a menudo importado desde servicios externos. El agente fuerza el enrutamiento de la app objetivo a través del proxy mediante `iptables` (requiere root) o mediante un túnel VPN local que encapsula solo el tráfico de la aplicación elegida (técnica de split-tunneling con un `VpnService` propio).

La integración con el router GenRouter permite además asignar proxies a nivel de VLAN, reduciendo la carga en cada dispositivo.

### 1.4 API en la nube y riesgos de seguridad

El backend en la nube expone una API REST que sincroniza scripts, actualiza licencias y recibe telemetría. Esto implica que una brecha en ese servidor podría exponer la actividad de todas las granjas conectadas, incluyendo qué aplicaciones están siendo automatizadas y desde qué IPs de salida. Para un analista, monitorizar los dominios y certificados asociados a GenFarmer podría revelar servidores de mando y control.

---

## 2. OSMBB: inyección de código sobre emuladores de Android

OSMBB (Open Source Mobile Bot, a veces abreviado OSMB) es un bot de escritorio dirigido principalmente a **Old School RuneScape (OSRS)** en su versión móvil, ejecutada dentro de un emulador de Android. Aunque el objetivo es un videojuego, las técnicas que emplea son extrapolables a la automatización de aplicaciones de banca, redes sociales o mensajería si se modificara su módulo de interacción.

### 2.1 Contexto: ¿por qué emuladores?

Las aplicaciones bancarias y muchas redes sociales están blindadas contra emuladores (SafetyNet, hardware-backed key attestation), pero aún es posible ejecutarlas en emuladores parcheados o versiones antiguas. En el mundo del game botting, los emuladores como BlueStacks, LDPlayer o MEmu son el estándar porque permiten escalar horizontalmente: un solo servidor puede alojar 20-30 instancias de Android, cada una con su propio bot.

### 2.2 Modelo cliente-servidor de OSMBB

OSMBB se compone de dos partes:

- **Cliente de escritorio (OSMBB.exe)**: escrito probablemente en C# o Java, proporciona la interfaz gráfica donde se cargan los scripts (en forma de plugins .class o .jar) y se muestra el estado de cada bot.
- **Agente móvil (APK auxiliar)**: instalado dentro del emulador, se comunica con el cliente de escritorio mediante **ADB forward** o un socket local tunelizado a través de `adb reverse`. El agente inyecta su lógica en el proceso del juego mediante una combinación de **reflexión (reflection)** y **manipulación del árbol de accesibilidad** (similar a un servicio de accesibilidad), aunque los detalles exactos dependen de la versión.

La comunicación entre el cliente de escritorio y el agente suele seguir un protocolo binario propietario sobre TCP, encapsulado en el canal ADB. Esto permite enviar comandos como "hacer clic en (X,Y)", "leer texto de la ventana", "obtener inventario" y "caminar a la posición (X,Y,Z)".

### 2.3 Técnica de inyección y evasión de antitrampas

OSMBB entra dentro de la categoría de **bots de inyección**, a diferencia de los bots de color (como veremos en Wasp). La inyección consiste en forzar la carga de una librería nativa (.so) o un archivo dex dentro del proceso del juego, ganando acceso directo a sus estructuras de memoria y a sus métodos. Para OSRS, los bots inyectados pueden leer directamente las coordenadas del jugador, los IDs de los objetos, el estado del inventario y las interacciones con NPCs sin necesidad de analizar la pantalla.

Desde el punto de vista defensivo, la detección se basa en:

- **Integridad del proceso**: el antitrampas de Jagex (RuneLite/OSRS) escanea `/proc/self/maps` en busca de librerías no autorizadas o regiones de memoria con permisos `rwx`.
- **Firma de la APK**: la app del juego verifica su propia firma y la del emulador. Los emuladores parcheados intentan suplantar las firmas, pero las técnicas de atestación (`key attestation`) dificultan cada vez más este engaño.
- **Patrones de entrada**: aunque el bot puede aleatorizar clics, la secuencia de acciones (pausas entre clics, orden de interacción con objetos) sigue siendo demasiado perfecta. Los sistemas modernos utilizan modelos de machine learning entrenados con sesiones humanas reales para puntuar la naturalidad del juego.

### 2.4 Herramientas complementarias: WaspScripts y el color botting

Para esquivar las detecciones de inyección, la comunidad ha desarrollado alternativas de **color botting**, como **WaspScripts** (basado en Simba 2.0). En lugar de inyectarse, analiza la pantalla píxel a píxel buscando colores predefinidos que identifican objetos del juego, y envía eventos de ratón mediante llamadas a la API de Windows o a través de `adb shell input` con coordenadas calculadas. La principal ventaja es que no modifica el proceso del juego, por lo que es indetectable por escáneres de memoria. La desventaja es una menor precisión y velocidad. WaspLib proporciona funciones como `findColors`, `mouseBox`, `waitColor` que los scripts combinan para navegar por los menús.

La existencia de estas dos aproximaciones (inyección vs. color) muestra la escalada continua entre atacantes y defensores, una dinámica que se reproduce en cualquier entorno de automatización fraudulenta.

---

## 3. Dolphin: el cerebro generativo sin censura

Dolphin no es una herramienta de automatización en sí, sino un modelo de lenguaje (LLM) ajustado para funcionar **sin las restricciones de seguridad** que incorporan modelos como GPT-4 o Claude. La versión más citada, **Dolphin 2.9**, se basa en la arquitectura Llama 3 de Meta (8B y 70B parámetros) y ha sido fine-tuneada con un conjunto de datos curado para eliminar sesgos de alineamiento y rechazar menos peticiones. Esto lo convierte en el motor ideal para generar contenido que otras IA se negarían a producir: discursos de odio, desinformación, mensajes de acoso o simplemente texto indistinguible del humano para cuentas falsas.

### 3.1 Arquitectura del modelo y ejecución local

Dolphin 2.9 se distribuye en formato GGUF (cuantizado) para ser ejecutado localmente con **llama.cpp**, o en formato safetensors para servidores de inferencia como **vLLM** o **Text Generation Inference**. Las granjas típicas optan por la ejecución local porque:

- **Elimina la dependencia de APIs externas** que podrían registrar las consultas o imponer límites de uso.
- **Reduce la latencia** al residir el modelo en la misma LAN que los dispositivos automatizados.
- **Garantiza la ausencia de filtros de contenido**, ya que el operador controla totalmente el pipeline.

En un servidor con una GPU como la NVIDIA RTX 4090, el modelo Dolphin 8B cuantizado a 4 bits puede generar aproximadamente 100 tokens por segundo, suficiente para alimentar cientos de bots que publiquen un comentario por minuto. El modelo 70B requiere múltiples GPUs, pero su calidad de texto es notablemente superior y más difícil de detectar automáticamente.

### 3.2 Integración en el ciclo de automatización

La integración típica sigue este esquema:

1. Un script de Python (orquestador) recibe una solicitud: "necesito un comentario de 20 palabras, tono enojado, sobre la subida de impuestos, simulando ser un votante de clase media".
2. El script construye un prompt estructurado que incluye instrucciones de sistema ("Eres un ciudadano español descontento...") y ejemplos de estilo.
3. El prompt se envía a la API REST de vLLM/llama.cpp (por ejemplo, `POST <http://llm-server:8000/generate`>).
4. El servidor devuelve el texto generado.
5. El orquestador inyecta ese texto en el dispositivo adecuado (vía GenFarmer, por ejemplo) y ordena la publicación.

Pseudo-código del orquestador:

```python
import requests
from genfarmer_api import assign_task  # API ficticia

def generar_comentario(tema, sentimiento):
    prompt = f"[INST] Eres un usuario de redes sociales. {sentimiento}. Escribe un comentario breve sobre: {tema}. Usa jerga coloquial. [/INST]"
    response = requests.post("<http://192.168.1.50:8000/generate>", json={
        "prompt": prompt,
        "max_tokens": 60,
        "temperature": 0.8
    })
    return response.json()["text"]

comentario = generar_comentario("subida de impuestos", "Estás muy enfadado")
assign_task(device_id="samsung_s8_42", action="comment", text=comentario)
```

### 3.3 Por qué la ausencia de censura es crítica

Los modelos comerciales (OpenAI, Anthropic, Google) incorporan capas de seguridad (RLHF, filtros de toxicidad) que bloquean peticiones dañinas. Dolphin ha sido fine-tuneado explícitamente para eliminar esa alineación; su tarjeta de modelo lo anuncia como "uncensored" y "compliant to any request". Esto significa que puede generar:

- Desinformación política sin mostrar ambigüedad ni advertencias.
- Suplantación de identidad: redactar mensajes imitando a una persona real, con su estilo y conocimientos.
- Contenido extremista o incendiario que los modelos comerciales rechazarían de plano.

Para un analista de seguridad, un comentario que contiene frases que un LLM alineado nunca emitiría puede ser un indicio de que se ha usado un modelo local sin censura, aunque la detección aún es un reto abierto.

### 3.4 Huellas digitales del texto generado

Aunque el contenido sea coherente, los modelos de lenguaje dejan rastros estadísticos:

- **Perplejidad baja y uniforme**: los humanos presentan picos de perplejidad al cambiar de idea o introducir conceptos imprevistos; los LLMs mantienen una perplejidad más estable. Herramientas como GPTZero o los detectores internos de X se basan en esta propiedad.
- **Falta de conocimiento contextual del mundo real**: el modelo puede afirmar que "el presidente Sánchez dimitió ayer" cuando ese hecho no ha ocurrido, porque su ventana de entrenamiento está congelada. Contrastar afirmaciones con bases de conocimiento en tiempo real puede delatar al bot.
- **Ausencia de errores humanos genuinos**: los humanos cometemos erratas, autocorrecciones, saltos de línea extraños. Los LLMs escriben demasiado bien. Para contrarrestarlo, las granjas introducen post-procesado: inserción de errores tipográficos, repetición de caracteres ("nooo"), uso inconsistente de mayúsculas, etc.

---

## 4. Interconexión de las tres herramientas en una operación real

Estas piezas no funcionan aisladas. Una granja avanzada podría combinarlas de la siguiente manera:

- **GenFarmer** gestiona 200 teléfonos Samsung S8 conectados al Box Phone Farm y al GenRouter, cada uno con una IP residencial de EE.UU. gracias a un proxy SOCKS5 inyectado mediante un `VpnService` modificado.
- **OSMBB** (adaptado) se ejecuta en 20 emuladores dentro del mismo servidor que aloja el LLM, automatizando el registro masivo de cuentas de Gmail porque el cliente de juego no es el objetivo, sino el formulario de registro; la técnica de inyección sobre el WebView del navegador móvil permite rellenar campos y resolver CAPTCHAs textuales mediante el LLM.
- **Dolphin 70B**, corriendo en 4 GPUs A6000, genera los correos de verificación, las respuestas a preguntas de seguridad y, más tarde, el contenido que esas cuentas publicarán en redes sociales.

Este escenario, aunque hipotético, es técnicamente factible con las herramientas analizadas y explica por qué las plataformas luchan por erradicar el tráfico no humano: el adversario dispone de un conjunto de piezas de Lego altamente especializadas, fáciles de adquirir y de integrar.

---

## 5. Conclusión: la importancia de desmontar las herramientas del adversario

La transparencia con la que se venden y documentan herramientas como GenFarmer, OSMBB o Dolphin es un arma de doble filo. Para el atacante, reduce la barrera de entrada; para el defensor, proporciona un mapa detallado de capacidades y, sobre todo, de limitaciones.

Conocer que GenFarmer depende en última instancia de ADB y de un agente con permisos de accesibilidad nos indica que un análisis estático de las apps instaladas en busca de servicios de accesibilidad no declarados puede revelar su presencia. Saber que OSMBB inyecta librerías nativas nos permite monitorizar `/proc/self/maps` en entornos controlados. Y comprender los patrones de perplejidad de Dolphin nos guía hacia detectores de texto generado más afinados.

La ciberseguridad en el ámbito de la automatización fraudulenta no consiste en prohibir la tecnología, sino en conocerla mejor que quien le da un mal uso. Este análisis técnico espera haber contribuido precisamente a eso.
