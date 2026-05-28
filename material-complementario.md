# MATERIAL COMPLEMENTARIO DEL CURSO

---

## 1. TAREA FINAL DE CERTIFICACIÓN

**Enunciado para el alumno:**

Dispones de dos semanas para entregar un informe pericial que responda a las siguientes preguntas, basándote en los ficheros de evidencias que se adjuntan (simulados pero realistas):

- **Archivo 1:** `network_capture.pcap` – Captura de tráfico de red de 30 minutos en el punto de salida de una supuesta granja.
- **Archivo 2:** `device_logs.json` – Registros de eventos de 5 dispositivos (accesibilidad, instalaciones, cambios de AAID).
- **Archivo 3:** `account_events.csv` – 10.000 eventos de una plataforma social (registros, tuits, likes) con metadatos (IP, timestamp, modelo de dispositivo, texto).

**Preguntas a resolver:**

1. ¿Existe una granja operando? Justifica tu respuesta con al menos tres tipos de evidencias extraídas de los ficheros.
2. ¿Cuántos dispositivos componen la granja? ¿Cómo los has identificado?
3. ¿Qué método o métodos de automatización táctil están utilizando? Aporta pruebas de los logs de dispositivo o del tráfico.
4. ¿Qué campaña de fraude están ejecutando? (Registro masivo, fraude CPA, amplificación en redes sociales, o una combinación). Describe el flujo que has reconstruido.
5. ¿Utilizan modelos de lenguaje generativo (LLM)? ¿Qué indicios lo sugieren?
6. Basándote en los hallazgos, propón un plan de defensa concreto para la plataforma afectada, detallando al menos una medida en cada una de las cuatro capas: dispositivo, red, cuenta y acción legal.

**Formato de entrega:** Informe PDF, extensión máxima 15 páginas, con capturas de pantalla de las herramientas utilizadas (Wireshark, jadx, scripts propios, etc.) y referencias a los IoCs encontrados.

---

## 2. SOLUCIONES RAZONADAS DE LOS EJERCICIOS PRÁCTICOS

### Módulo 1 – Ejercicio de identificación de dispositivos de granja

**Datos simulados:** 10 dispositivos, se muestran 3.

| Dispositivo | Build.FINGERPRINT | AccessibilityService | Tiempo conexión USB |
|-------------|-------------------|----------------------|----------------------|
| A | samsung/dreamlte/dreamlte:9/PPR1.180610.011/G950FXXU5DSFB:user/release-keys | com.genfarmer.agent/.MyAccessibilityService | 23h 15m |
| B | samsung/dreamlte/dreamlte:9/PPR1.180610.011/G950FXXU5DSFB:user/release-keys | com.genfarmer.agent/.MyAccessibilityService | 22h 58m |
| C | xiaomi/raphael/raphael:11/RKQ1.200826.002/V12.5.1.0.RFKMIXM:user/release-keys | com.google.android.marvin.talkback/.TalkBackService | 0h 05m |

**Solución:**
- Los dispositivos A y B comparten exactamente el mismo `Build.FINGERPRINT`, lo que indica que son el mismo modelo, misma versión de sistema y mismos parches. Además, ambos tienen activo el servicio de accesibilidad `com.genfarmer.agent`, que coincide con el nombre de paquete del agente GenFarmer analizado. El tiempo de conexión USB es superior a 22 horas, lo que sugiere una conexión permanente a un controlador. **Conclusión: A y B son parte de una granja.**
- El dispositivo C tiene un fingerprint diferente y ejecuta TalkBack (lector de pantalla legítimo para personas con discapacidad visual). Su tiempo de conexión USB es bajo. **Conclusión: C es probablemente un usuario legítimo.**

### Módulo 2 – Identificación de permisos y configuración del AccessibilityService

**Solución esperada:**
- Permisos encontrados en la muestra: `BIND_ACCESSIBILITY_SERVICE`, `SYSTEM_ALERT_WINDOW`, `INTERNET`, `FOREGROUND_SERVICE`, `RECEIVE_BOOT_COMPLETED`.
- Configuración del servicio en `accessibility_service_config.xml`:
  - `accessibilityEventTypes`: `typeAllMask`
  - `accessibilityFeedbackType`: `feedbackGeneric`
  - `canPerformGestures`: `true`
  - `packageNames`: `com.facebook.katana, com.instagram.android, com.twitter.android`
- **¿Podría confundirse con una herramienta legítima?** No. Una herramienta legítima de accesibilidad no limitaría su escucha a apps de redes sociales específicas, ni declararía `canPerformGestures=true` sin una funcionalidad clara de asistencia (como un lector de pantalla que requiere gestos). La combinación de estos factores es un IoC de alta fidelidad.

### Módulo 3 – Método de inyección en código

**Fragmento de código analizado (simulado):**
```java
public void performTouch(int x, int y, int duration) {
    GestureDescription.Builder builder = new GestureDescription.Builder();
    Path path = new Path();
    path.moveTo(x + randomOffset(), y + randomOffset());
    builder.addStroke(new GestureDescription.StrokeDescription(path, 0, duration + randomDelay()));
    dispatchGesture(builder.build(), null, null);
}
```

**Solución:**
- El método utiliza `dispatchGesture`, por tanto, el agente se basa en el `AccessibilityService` sin necesidad de root.
- Introduce aleatorización en coordenadas (`randomOffset()`) y en duración (`randomDelay()`), con un rango probable de ±10 píxeles y ±50 ms.
- **Detección:** Aunque aleatoriza, el trazo es una línea recta entre dos puntos sin microtemblores. Un modelo de ML entrenado con trayectorias humanas (que contienen pequeñas curvas) podría detectar la linealidad.

### Módulo 4 – Detección de reglas iptables

**Script simulado:**
```bash
iptables -t nat -L OUTPUT -v
```
**Salida observada:**
```
Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
  123  7890 DNAT       tcp  --  any    any     anywhere             anywhere             tcp dpt:https owner UID match 10087 to:10.0.0.5:3128
```

**Solución:**
- La regla DNAT redirige el tráfico HTTPS (puerto 443) de la app con UID 10087 hacia `10.0.0.5:3128`, que es un proxy (posiblemente Squid o un SOCKS5).
- El UID 10087 corresponde a la app objetivo (por ejemplo, Instagram). El tráfico de esa app no saldrá directamente a internet, sino al proxy, que lo reenviará con su IP.
- **Detección:** Un escaneo periódico de `iptables -t nat -L` por parte de una app de seguridad (con permisos de root) revelaría esta regla. La presencia de reglas `DNAT` no relacionadas con el tethering o VPNs conocidas es un IoC.

### Módulo 5 – Datos de atribución de instalaciones

**Fragmento del dataset:**

| timestamp_clic | timestamp_instalacion | ip_clic | ip_instalacion | aaids | tiempo_tutorial |
|----------------|------------------------|---------|----------------|-------|-----------------|
| 10:00:00.100 | 10:00:00.150 | 185.220.101.1 | 185.220.101.1 | a1b2c3... | 0.5s |
| 10:00:05.200 | 10:00:06.500 | 85.34.12.45 | 85.34.12.45 | d4e5f6... | 45s |
| 10:00:10.300 | 10:00:10.350 | 185.220.101.2 | 185.220.101.2 | g7h8i9... | 0.3s |

**Solución:**
- Las instalaciones con IPs `185.220.101.x` pertenecen a un rango conocido de proxies/Tor (en este caso, un pool de proxies residenciales o de salida de Tor). Son sospechosas.
- El tiempo entre clic e instalación es de solo 50 ms en la primera fila, lo que es físicamente imposible para un humano (click injection).
- El tiempo de tutorial de 0.5s y 0.3s es ridículamente bajo (un humano tardaría al menos 30-60 segundos).
- La instalación con IP `85.34.12.45` tiene tiempos realistas (1.3s de diferencia clic-instalación, 45s de tutorial) y una IP de aspecto residencial normal. Es probablemente legítima.

### Módulo 6 – JA3 fingerprinting

**Solución esperada:**
- Tras extraer los JA3 con `tshark -r captura.pcap -Y "tls.handshake.type==1" -T fields -e tls.handshake.ja3`, se observan dos grupos:
  - `771,4865-4866-4867-49195-49199...` (Chrome en Android, legítimo).
  - `771,4865-4867-49195-49199...` (OkHttp con configuración personalizada, típico de scripts de automatización).
- El segundo grupo carece de ciertos cipher suites que Chrome incluye por defecto, lo que sugiere una pila TLS modificada o un agente de automatización (GenFarmer usando OkHttp o Cronet). Este JA3 puede añadirse a la blacklist de la plataforma.

---

## 3. EXAMEN TIPO TEST (20 PREGUNTAS)

**Instrucciones:** Marca la respuesta correcta. Solo una es válida por pregunta.

**1. ¿Cuál es la función principal de GenRouter en el ecosistema Gen?**
a) Almacenar las cookies de sesión de los navegadores.
b) Gestionar el tráfico de red y asignar proxies a nivel de subred.
c) Ejecutar los scripts de automatización en los dispositivos.
d) Generar huellas digitales de navegador únicas.

**2. ¿Qué permiso de Android es indispensable para que un agente simule toques sin root?**
a) `INTERNET`
b) `CAMERA`
c) `BIND_ACCESSIBILITY_SERVICE`
d) `READ_SMS`

**3. ¿Cuál de los siguientes métodos de inyección táctil deja un rastro visible en `dumpsys accessibility`?**
a) `InputManager.injectInputEvent()` (con root)
b) Emulación HID externa con Arduino
c) `AccessibilityService.dispatchGesture()`
d) `adb shell input tap`

**4. ¿Qué es el AAID?**
a) Un identificador único del chip WiFi.
b) El Advertising ID de Google, reseteable por el usuario.
c) El número de serie de la batería.
d) Un hash del `Build.FINGERPRINT`.

**5. ¿Cuál de estas NO es una técnica de enmascaramiento de red utilizada por GenFarmer?**
a) Creación de una VPN local con `VpnService`.
b) Redirección de tráfico con `iptables` (si hay root).
c) Uso de proxies residenciales rotativos.
d) Envío de tráfico a través de la red Tor exclusivamente.

**6. ¿Qué es GenLogin?**
a) Un emulador de Android.
b) Un navegador anti-detección que genera huellas digitales falsas.
c) Un servicio de verificación por SMS.
d) Un router modificado con OpenWrt.

**7. En un flujo de fraude CPA, ¿qué acción se realiza típicamente después de completar el tutorial de una app?**
a) Apagar el dispositivo.
b) Restablecer el AAID y desinstalar la app.
c) Enviar un SMS de verificación.
d) Cambiar la IP del proxy.

**8. ¿Qué característica de un `AccessibilityService` permite específicamente simular toques?**
a) `android:canTakeScreenshot="true"`
b) `android:accessibilityFeedbackType="feedbackSpoken"`
c) `android:canPerformGestures="true"`
d) `android:packageNames="*"`

**9. ¿Cuál es un indicio forense de que un dispositivo forma parte de una granja física?**
a) Tener muchas fotos almacenadas.
b) Conexión USB permanente y temperatura de batería elevada constante.
c) Usar una red WiFi con contraseña.
d) Tener la última versión de Android.

**10. ¿Qué protocolo de comunicación utiliza GenFarmer para la conexión persistente con el backend en la nube?**
a) FTP
b) SMTP
c) WebSocket o MQTT
d) DNS

**11. ¿Por qué los modelos de lenguaje como Dolphin son atractivos para las granjas?**
a) Porque son más baratos que contratar a un redactor humano.
b) Porque están "sin censura" y pueden generar contenido que otras IA rechazan, como desinformación.
c) Porque se ejecutan exclusivamente en la nube.
d) Porque solo generan texto en inglés.

**12. ¿Qué herramienta forense se usa para monitorizar llamadas a APIs de Android en tiempo real?**
a) Wireshark
b) jadx
c) Frida
d) apktool

**13. En la detección de texto generado por LLM, ¿qué métrica suele ser más baja y uniforme que en el texto humano?**
a) La longitud de las palabras.
b) La perplejidad.
c) El número de emojis.
d) La cantidad de hashtags.

**14. ¿Qué es el click injection?**
a) Un método para aumentar los likes en una publicación.
b) La inyección de un clic fraudulento justo antes de una instalación orgánica para robar la atribución.
c) Una técnica para simular toques con un Arduino.
d) Un ataque de denegación de servicio.

**15. ¿Cuál es una medida defensiva a nivel de dispositivo contra los agentes de automatización?**
a) Bloquear el puerto 80 en el router.
b) Escanear los servicios de accesibilidad activos y detectar `canPerformGestures=true` en apps no legítimas.
c) Desactivar el Bluetooth.
d) Obligar al usuario a usar una VPN.

**16. ¿Qué es un JA3 fingerprint?**
a) La huella digital del sensor de huellas dactilares.
b) Un hash de la negociación TLS que identifica al cliente.
c) El identificador único del chip NFC.
d) La suma de verificación de una APK.

**17. En una campaña de amplificación en redes sociales, ¿qué patrón delata a las cuentas de la granja?**
a) Publican a diferentes horas del día.
b) Publican textos con alta variabilidad semántica.
c) Publican en una ventana temporal estrecha, con textos semánticamente similares y desde IPs de pools de proxies.
d) Solo publican fotos sin texto.

**18. ¿Qué es SIMFaaS?**
a) Un software para gestionar contactos.
b) Un servicio que alquila acceso a granjas de tarjetas SIM para verificación por SMS.
c) Una aplicación de mensajería segura.
d) Un tipo de tarjeta SIM virtual.

**19. ¿Qué información NO se puede obtener de un volcado de tráfico de red de una granja?**
a) Los dominios del backend de mando y control.
b) La frecuencia de los latidos.
c) El contenido exacto de la memoria RAM del dispositivo.
d) Los tipos de comandos enviados (touch, swipe, etc.).

**20. ¿Cuál es la estrategia de defensa más efectiva contra las granjas de bots?**
a) Bloquear todas las IPs de un país.
b) Una defensa multicapa que combine detección en dispositivo, red y cuenta.
c) Confiar en que los usuarios no harán clic en enlaces maliciosos.
d) Usar solo contraseñas seguras.

---

### CLAVE DE RESPUESTAS

| Pregunta | Respuesta | Pregunta | Respuesta |
|----------|-----------|----------|-----------|
| 1 | b | 11 | b |
| 2 | c | 12 | c |
| 3 | c | 13 | b |
| 4 | b | 14 | b |
| 5 | d | 15 | b |
| 6 | b | 16 | b |
| 7 | b | 17 | c |
| 8 | c | 18 | b |
| 9 | b | 19 | c |
| 10 | c | 20 | b |
