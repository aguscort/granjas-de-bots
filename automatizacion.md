# Automatización industrial: de ADB a Appium

## 1. El porqué de la automatización controlada

Automatizar un dispositivo Android o iOS no es intrínsecamente malicioso. De hecho, las propias Google y Apple proporcionan frameworks oficiales para que los desarrolladores automaticen pruebas, garantizando la calidad del software. Los mismos pilares técnicos (ADB, Appium, gestores de dispositivos) se usan tanto para fines legítimos como para los ilegítimos; la diferencia está en **el propósito, el consentimiento y el entorno**.

Casos de uso perfectamente legales que cubriremos aquí:

- **Pruebas de regresión y funcionales** en aplicaciones móviles propias o de clientes que han contratado tus servicios de QA.
- **Automatización de tareas repetitivas** en dispositivos de tu propiedad: por ejemplo, sincronizar configuraciones, instalar aplicaciones en flotas de terminales corporativos, o recolectar datos de sensores para un proyecto de investigación.
- **CI/CD móvil**: integrar pruebas automáticas en pipelines de Jenkins, GitHub Actions o GitLab CI.
- **Investigación de seguridad autorizada**: ejecutar análisis de tráfico, comportamiento de apps o fuzzing sobre tu propio software en un entorno aislado.

Quedan fuera, por tanto, la creación de cuentas falsas, la suplantación de identidad, el fraude de clics, la manipulación de redes sociales y cualquier actividad que viole los ToS de plataformas de terceros.

## 2. Preparación del entorno de automatización

### 2.1 Dispositivos físicos vs. emuladores

Para testing serio, se recomienda combinar ambos:

- **Dispositivos físicos**: imprescindibles para validar rendimiento real, consumo de batería, interacción con sensores (GPS, cámara, acelerómetro) y conectividad móvil. Adquiere modelos representativos de tu público objetivo (Google Pixel, Samsung Galaxy, gama media con Android Go, etc.). Asegúrate de que son de tu propiedad y están borrados de cuentas personales.
- **Emuladores**: útiles para pruebas rápidas en CI. El emulador oficial de Android Studio permite crear dispositivos virtuales (AVD) con distintas versiones de API y configuraciones de hardware. Para flotas más grandes, herramientas como **Genymotion** o **Android Emulator containers** ofrecen gestión centralizada.

Mantén los dispositivos en una red WiFi dedicada o en una VLAN aislada si manejas datos sensibles. Conéctalos a tu PC mediante USB 2.0/3.0 y, si la escala lo requiere, usa **hubs USB alimentados** para evitar caídas de tensión. A diferencia de las granjas maliciosas, no necesitas ocultar nada: la transparencia es tu mejor aliada.

### 2.2 Activación de opciones de desarrollo y depuración USB

En cada dispositivo físico:

1. Ve a **Ajustes → Información del teléfono → Número de compilación** y pulsa 7 veces para activar las opciones de desarrollo.
2. En **Opciones de desarrollo**, habilita **Depuración USB** y, si tu flujo lo requiere, **Instalación vía USB** y **Permitir screen overlay en configuraciones**.
3. Conecta el dispositivo al PC. Acepta el diálogo de autorización de depuración USB (marca "Permitir siempre" para este equipo).
4. Verifica la conexión con el comando:
Deberías ver el dispositivo listado como `device`. Si aparece `unauthorized`, comprueba que aceptaste el diálogo en el móvil.
    
    ```bash
    adb devices
    ```
    

### 2.3 Instalación del Android SDK y herramientas de línea de comandos

Descarga el **SDK Platform Tools** (que incluye ADB, fastboot, etc.) desde la web oficial de Android. Añade la carpeta `platform-tools` a tu PATH. ADB (Android Debug Bridge) es el canal de comunicación fundamental; mediante él podrás instalar apps, capturar logs, simular eventos y ejecutar comandos shell.

```bash
# Ejemplo: instalar una APK de prueba
adb install mi_app.apk

# Capturar logcat en tiempo real
adb logcat

# Listar paquetes instalados
adb shell pm list packages
```

## 3. Automatización con Appium: el estándar multiplataforma

**Appium** es un framework open source que sigue el protocolo WebDriver (W3C) para automatizar aplicaciones nativas, híbridas y web móvil. Soporta Android (a través del UiAutomator2) e iOS (XCUITest), permitiendo escribir pruebas en múltiples lenguajes: Python, Java, JavaScript (Node.js), Ruby, C#... Su arquitectura cliente-servidor lo hace ideal para integrar en pipelines de CI.

### 3.1 Instalación del servidor Appium

La forma más sencilla es mediante npm:

```bash
npm install -g appium
appium driver install uiautomator2   # para Android
```

Si prefieres una GUI, Appium Inspector te ayuda a localizar elementos visualmente.

Para Android necesitarás también Java JDK (8 o superior), Android SDK y las variables de entorno `ANDROID_HOME` configuradas.

### 3.2 Primer script: abrir una app y realizar una acción

Supongamos que tienes una app de ejemplo (puede ser una app propia en desarrollo) con el paquete `com.ejemplo.mi_app` y queremos automatizar la pulsación de un botón con ID `boton_saludo`. Usaremos Python con el cliente Appium-Python-Client.

Instala las dependencias:

```bash
pip install Appium-Python-Client
```

Crea un archivo `test_basico.py`:

```python
from appium import webdriver
from appium.webdriver.common.appiumby import AppiumBy
import time

# Capacidades deseadas (desired capabilities)
desired_caps = {
    "platformName": "Android",
    "appium:automationName": "UiAutomator2",
    "appium:deviceName": "R58M34XKZBN",   # Se obtiene con 'adb devices'
    "appium:appPackage": "com.ejemplo.mi_app",
    "appium:appActivity": ".MainActivity",
    "appium:noReset": True,              # No reiniciar estado de la app
    "appium:newCommandTimeout": 300
}

# Conectar al servidor Appium (debe estar corriendo en otra terminal con 'appium')
driver = webdriver.Remote('<http://127.0.0.1:4723>', desired_caps)

try:
    # Esperar a que el elemento esté presente
    saludo_btn = driver.find_element(AppiumBy.ID, "com.ejemplo.mi_app:id/boton_saludo")
    saludo_btn.click()
    time.sleep(1)
    # Verificar resultado (ejemplo simplificado)
    texto_resultado = driver.find_element(AppiumBy.ID, "com.ejemplo.mi_app:id/etiqueta_resultado")
    assert texto_resultado.text == "Hola, mundo"
    print("Prueba superada")
finally:
    driver.quit()
```

Ejecuta en una terminal el servidor:

```bash
appium --log-level info
```

Y en otra:

```bash
python test_basico.py
```

Si todo está correcto, verás cómo el botón se pulsa automáticamente y la prueba pasa.

### 3.3 Localización de elementos

Appium ofrece varias estrategias para encontrar elementos de la UI:

- `AppiumBy.ID`: el recurso-id de Android (`com.ejemplo:id/mi_id`).
- `AppiumBy.ACCESSIBILITY_ID`: la etiqueta `content-desc` (muy estable, recomendada).
- `AppiumBy.XPATH`: rutas XPath (flexible pero más lento; evitar si hay alternativa).
- `AppiumBy.CLASS_NAME`: la clase Java (ej. `android.widget.Button`).
- `AppiumBy.ANDROID_UIAUTOMATOR`: usando UiSelector de UiAutomator (potente para búsquedas complejas).

Ejemplo con UiAutomator:

```python
elemento = driver.find_element(AppiumBy.ANDROID_UIAUTOMATOR,
    'new UiSelector().text("Enviar").className("android.widget.Button")')
```

Para explorar la jerarquía de la pantalla usa **Appium Inspector** o el comando ADB:

```bash
adb shell uiautomator dump
```

Esto genera un XML que puedes inspeccionar para extraer los selectores adecuados.

## 4. Escalado y robustez: de un dispositivo a una flota

En un entorno de QA profesional, necesitas ejecutar las mismas pruebas en múltiples dispositivos simultáneamente. Appium lo soporta mediante la creación de sesiones paralelas.

### 4.1 Arquitectura multi-dispositivo con pytest

Puedes aprovechar `pytest` con la librería `pytest-xdist` o gestionar tú mismo los dispositivos.

Ejemplo de gestión de varios dispositivos con un pool:

```python
import subprocess
import threading
from appium import webdriver

dispositivos = ["R58M34XKZBN", "emulator-5554", "192.168.1.45:5555"]  # ADB sobre TCP/IP

def ejecutar_prueba(serial):
    caps = {
        "platformName": "Android",
        "appium:automationName": "UiAutomator2",
        "appium:deviceName": serial,
        "appium:app": "/ruta/a/mi_app.apk",   # se reinstala cada vez
        "appium:fullReset": True
    }
    driver = webdriver.Remote('<http://localhost:4723/wd/hub>', caps)
    try:
        # ... tu lógica de prueba ...
        pass
    finally:
        driver.quit()

threads = []
for d in dispositivos:
    t = threading.Thread(target=ejecutar_prueba, args=(d,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
```

Si el número de dispositivos crece, considera **Selenium Grid** con Appium (el proyecto Appium ya soporta el registro en un hub Selenium Grid 4) o soluciones comerciales como **Sauce Labs** o **BrowserStack** (si necesitas dispositivos en la nube de forma legal, pagando el servicio). Así tu granja de pruebas será ética y profesional.

### 4.2 Patrón Page Object Model (POM)

Para que las pruebas sean mantenibles, encapsula la lógica de interacción en clases que representan pantallas:

```python
class LoginPage:
    def __init__(self, driver):
        self.driver = driver
        self.usuario_input = (AppiumBy.ID, "com.app:id/usuario")
        self.password_input = (AppiumBy.ID, "com.app:id/password")
        self.login_btn = (AppiumBy.ID, "com.app:id/login")

    def login(self, usuario, password):
        self.driver.find_element(*self.usuario_input).send_keys(usuario)
        self.driver.find_element(*self.password_input).send_keys(password)
        self.driver.find_element(*self.login_btn).click()
        return HomePage(self.driver)
```

Este patrón aísla los cambios de UI y hace las pruebas legibles y reutilizables.

## 5. Automatización avanzada con ADB directamente

Para tareas que no requieren el contexto de una app (como gestionar el sistema, instalar certificados, manipular ficheros), ADB es suficiente y más ligero.

### 5.1 Simular gestos y pulsaciones

Puedes simular toques mediante `input` (aunque Appium es mejor porque respeta la jerarquía UI):

```bash
adb shell input tap 500 1000
adb shell input swipe 300 1000 300 500
adb shell input text "Hola mundo"
```

### 5.2 Capturas de pantalla y grabación

Útil para generar evidencias de pruebas:

```bash
adb exec-out screencap -p > captura.png
adb shell screenrecord /sdcard/video.mp4
# Ctrl+C para detener
adb pull /sdcard/video.mp4 .
```

### 5.3 Integración en scripts Python

La librería `pure-python-adb` permite controlar dispositivos sin depender del binario `adb` del sistema. Sin embargo, para entornos de CI es común invocar `adb` como subproceso.

## 6. Alternativas a Appium: cuándo y por qué

- **Espresso (Android)**: para pruebas unitarias y de UI muy rápidas, con acceso directo al código de la app (necesitas el código fuente). No sirve para automatizar apps de terceros.
- **UI Automator (Android)**: el framework sobre el que se apoya Appium; puedes usarlo directamente en Java/Kotlin para pruebas internas.
- **XCUITest (iOS)**: la alternativa nativa de Apple para pruebas de UI. Appium lo envuelve también.
- **Detox (React Native)**: especializado en apps React Native, con sincronización automática.
- **Maestro**: una herramienta YAML-driven muy sencilla para flujos rápidos; ideal para humo tests sin código.

Elegir la herramienta adecuada depende del tipo de aplicación, la integración con el pipeline y las habilidades del equipo. Ninguna de ellas es ilegítima por sí misma; su uso fuera de los términos de servicio de las apps objetivo es lo que cruza la raya.

## 7. Seguridad y buenas prácticas

Aunque trabajes en un entorno propio, la automatización introduce riesgos que debes mitigar:

- **Datos de prueba**: nunca uses datos reales de usuarios. Genera conjuntos de datos sintéticos o anonimizados.
- **Credenciales**: almacena tokens y contraseñas en gestores de secretos (Vault, variables de entorno cifradas en CI) y nunca las hardcodees.
- **Aislamiento**: si pruebas apps que hacen llamadas de red, configura un proxy de desarrollo (mitmproxy) o usa servidores de staging, nunca producción de terceros.
- **Permisos**: mantén los dispositivos de prueba desconectados de cuentas personales (Google, iCloud) y restringe los permisos de las apps de automatización a lo estrictamente necesario.
- **Limpieza**: al terminar, desinstala las apps de prueba y ejecuta `adb shell pm clear <paquete>` para eliminar datos residuales.

## 8. Conclusión: el poder de la automatización bien empleada

La línea que separa una herramienta de automatización ética de una granja fraudulenta no es técnica, sino de intención y legalidad. Con los conocimientos aquí presentados puedes construir un laboratorio de pruebas robusto, acelerar tus ciclos de desarrollo y mantener la calidad de tus aplicaciones, todo ello respetando la ley y los derechos de terceros.

La automatización móvil es una disciplina apasionante y legítima. Aprovéchala para innovar, no para explotar. Si decides profundizar en alguno de estos frameworks o necesitas ejemplos más específicos (Appium + WebView, testing de notificaciones push, integración con Jenkins), estaré encantado de ampliar la guía.
