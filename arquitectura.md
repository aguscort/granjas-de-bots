La arquitectura de una granja de bots moderna, especialmente en su versión de **"cajas" (Box Phone Farm)**, se divide en una infraestructura física altamente eficiente y una estructura lógica diseñada para la evasión de detecciones.

### **1. Arquitectura Física (Hardware)**

La base física está diseñada para la escalabilidad y la reducción de riesgos operativos como el sobrecalentamiento.

*   **Nivel de Dispositivos:** Se utilizan **placas base (motherboards)** de teléfonos reales, predominantemente **Samsung S8, S8+ o S10**, así como modelos de Xiaomi [3, 4]. En las granjas de cajas, se retiran las pantallas y las baterías para ahorrar espacio y evitar incendios por hinchamiento de celdas.
*   **Suministro de Energía:** En lugar de cargadores individuales, se emplea una **fuente de poder DC centralizada** (típicamente de 5V/40A o 60A) que alimenta las placas directamente.
*   **Chasis y Soporte:** Las placas se montan en **racks o racks industriales** dentro de cajas metálicas que suelen agrupar **20 o 40 dispositivos** por unidad.
*   **Conectividad Física:** Se utilizan **hubs USB de 20 a 21 puertos** para conectar todos los dispositivos a una computadora de control (PC) [7, 11].
*   **Gestión Térmica:** La estructura incluye **ventiladores industriales o sistemas de refrigeración activa** integrados para mantener la estabilidad del hardware funcionando 24/7.
*   **Componentes Opcionales:** Algunas placas incluyen **ranuras para tarjetas SIM físicas**, necesarias para verificaciones de SMS en plataformas como Facebook.

### **2. Arquitectura Lógica (Software y Red)**

Esta capa permite que miles de dispositivos operen como si fueran usuarios humanos independientes, distribuidos geográficamente.

*   **Capa de Control Centralizado:** El software principal (como **GenFarmer** o sistemas de **Cloud Control**) actúa como el cerebro, permitiendo gestionar hasta 1,000 dispositivos desde una sola PC y replicar comandos en masa.
*   **Capa de Identidad (Fingerprinting):** Se utilizan navegadores **Anti-detect** (como **GenLogin**) para crear perfiles con huellas digitales únicas (User Agent, WebGL, fuentes), haciendo que cada cuenta parezca estar en un dispositivo distinto.
*   **Capa de Red (Ofuscación):** 
    *   **GenRouter:** Hardware especializado que cambia parámetros de red como la MAC, DNS y SSID, creando un entorno de red "limpio" para cada grupo de dispositivos.
    *   **Proxies Residenciales:** Se integran aplicaciones como **Superproxy** para asignar IPs residenciales rotativas a cada teléfono, evitando que las plataformas detecten que el tráfico proviene de un centro de datos.
*   **Capa de Automatización e IA:**
    *   **Scripts No-Code:** Sistemas de arrastrar y soltar para programar acciones como dar likes, comentar o navegar.
    *   **Modelos de Lenguaje (LLM):** Integración de IAs como **Llama 3** o **Dolphin** (basado en Mistral) para generar comentarios humanos y evadir filtros de detección de patrones robóticos.
*   **Verificación Externa:** Uso de APIs de servicios de terceros (como **Text Verified**) para obtener números virtuales y completar procesos de registro que requieren SMS.

Esta arquitectura permite que un solo operador maneje una red inmensa de cuentas donde, según investigaciones, solo **1 de cada 80 interacciones** podría ser de una persona real en entornos saturado.
