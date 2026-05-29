```
  ▄████  ██▀███   ▄▄▄       ███▄    █  ▄▄▄██▀▀▀ ▄▄▄        ██████
 ██▒ ▀█▒▓██ ▒ ██▒▒████▄     ██ ▀█   █    ▒██   ▒████▄    ▒██    ▒
▒██░▄▄▄░▓██ ░▄█ ▒▒██  ▀█▄  ▓██  ▀█ ██▒   ░██   ▒██  ▀█▄  ░ ▓██▄
░▓█  ██▓▒██▀▀█▄  ░██▄▄▄▄██ ▓██▒  ▐▌██▒▓██▄██▓  ░██▄▄▄▄██   ▒   ██▒
░▒▓███▀▒░██▓ ▒██▒ ▓█   ▓██▒▒██░   ▓██░ ▓███▒    ▓█   ▓██▒▒██████▒▒
 ░▒   ▒ ░ ▒▓ ░▒▓░ ▒▒   ▓▒█░░ ▒░   ▒ ▒  ▒▓▒▒░    ▒▒   ▓▒█░▒ ▒▓▒ ▒ ░
              ██████  ▓█████     ▄▄▄▄    ▒█████  ▄▄▄█████▓  ██████
            ▒██    ▒  ▓█   ▀    ▓█████▄ ▒██▒  ██▒▓  ██▒ ▓▒▒██    ▒
            ░ ▓██▄    ▒███      ▒██▒ ▄██▒██░  ██▒▒ ▓██░ ▒░░ ▓██▄
              ▒   ██▒ ▒▓█  ▄    ▒██░█▀  ▒██   ██░░ ▓██▓ ░   ▒   ██▒
            ▒██████▒▒ ░▒████▒   ░▓█  ▀█▓░ ████▓▒░  ▒██▒ ░ ▒██████▒▒
            ▒ ▒▓▒ ▒ ░ ░░ ▒░ ░   ░▒▓███▀▒░ ▒░▒░▒░   ▒ ░░   ▒ ▒▓▒ ▒ ░
```
```
        ╔═══════════════════════════════════════════════════════════╗
        ║   B O X   P H O N E   F A R M   ·   v.GenFarmer            ║
        ╠═══════════════════════════════════════════════════════════╣
        ║  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐       ║
        ║  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ░ PSU ║
        ║  ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤ 5V40A ║
        ║  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▣ USB ║
        ║  ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤ ×20   ║
        ║  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ≋ FAN ║
        ║  ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤ 24/7  ║
        ║  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ▢  │ ⌁ SIM ║
        ║  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘       ║
        ╚═══════════════════════════════════════════════════════════╝
       "1 de cada 80 interacciones podría ser de una persona real."
```

# granjas-de-bots

Documentación generada por una investigación sobre las **granjas de bots** y, en
particular, las *box phone farms*: su arquitectura física, su gestión lógica
(GenFarmer, OSMBB, Dolphin), sus usos, su papel en la amplificación de narrativas
y un curso de análisis forense para detectarlas y neutralizarlas.

Este archivo funciona como índice del repositorio.

```
   ┌──────────┐   ┌────────────┐   ┌───────────┐   ┌──────────────────┐
   │ GenLogin │──▶│ GenFarmer  │──▶│ GenRouter │──▶│  X · IG · FB · TT │
   │ identidad│   │ flota +LLM │   │  proxies  │   │   (plataformas)   │
   └──────────┘   └────────────┘   └───────────┘   └──────────────────┘
      huella         dispositivo        IP              objetivo
```

```
═══════════════════════════════════════════════════════════════════════
   §1 · EL FENÓMENO  ░ concepto · usos · dimensión política
═══════════════════════════════════════════════════════════════════════
```

| Archivo | Contenido |
| --- | --- |
| [tipos-de-granjas.md](tipos-de-granjas.md) | Diferencia entre la granja tradicional (teléfonos completos en estanterías) y la granja de cajas (*box phone farm*): placas base "desnudas", alimentación DC centralizada, refrigeración y control integrado. |
| [usos.md](usos.md) | Catálogo de usos: monetización pasiva y *play-to-earn*, inflado de métricas, astroturfing político (estrategia de las tres fronteras), reseñas falsas, *scalping*, *scraping* y *credential stuffing*. |
| [ecosistema-mmo.md](ecosistema-mmo.md) | El concepto de "ecosistema MMO" (*Make Money Online*) y el modelo de tres patas GenLogin / GenFarmer / GenRouter. |
| [amplificación-de-narrativas.md](amplificaci%C3%B3n-de-narrativas.md) | La amplificación de narrativas como táctica de manipulación: espiral del silencio, asociación repetitiva y el salto de calidad que introdujeron los LLM. |
| [consensus-reality.md](consensus-reality.md) | La "realidad de consenso" o posverdad: la tesis Surkov, la elección de "realidades ideales" y el objetivo de parálisis cívica mediante el agotamiento informativo. |

```
═══════════════════════════════════════════════════════════════════════
   §2 · INFRAESTRUCTURA Y TÉCNICA  ▣ hardware · red · software
═══════════════════════════════════════════════════════════════════════
```

| Archivo | Contenido |
| --- | --- |
| [arquitectura.md](arquitectura.md) | Visión resumida de la arquitectura física (hardware) y lógica (control centralizado, *fingerprinting*, ofuscación de red, capa de IA y verificación externa). |
| [infraestructura-fisica.md](infraestructura-fisica.md) | Artículo extenso sobre la infraestructura: dispositivos y racks, energía y SIMs, orquestación, emuladores, inyección HID por hardware, proxies residenciales, LLM locales y vectores de detección. |
| [infraestructura-critica-de-microdispositvos.md](infraestructura-critica-de-microdispositvos.md) | Índice tipo manual de ingeniería para *box phone farms*, organizado en seis partes (planificación, hardware, red, software, IA, seguridad) más apéndices. |
| [comparativa-dispositivo-vs-disimulación.md](comparativa-dispositivo-vs-disimulaci%C3%B3n.md) | Tablas comparativas: teléfonos reales vs. emuladores vs. simuladores, GeeLark vs. GenFarmer, y costes de granjas físicas frente a la nube. |
| [automatizacion.md](automatizacion.md) | Automatización móvil legítima: de ADB a Appium, con ejemplos de código, escalado a flotas y la línea que separa el QA ético del fraude. |
| [gestion-logica.md](gestion-logica.md) | Disección técnica de los tres pilares de la automatización moderna: GenFarmer (flotas), OSMBB (inyección sobre emuladores) y Dolphin (LLM sin restricciones). |

```
═══════════════════════════════════════════════════════════════════════
   §3 · CURSO FORENSE  ⌬ GenFarmer · 6 módulos · 24 h
═══════════════════════════════════════════════════════════════════════

   [M1]──[M2]──[M3]──[M4]──[M5]──[M6]──▶ [ certificación ]
    │     │     │     │     │     │
  visión APK  control id+red flujos defensa
```

| Archivo | Contenido |
| --- | --- |
| [genfarmer-curso.md](genfarmer-curso.md) | Programa del curso: 24 horas en 6 módulos, prerrequisitos, objetivos y evaluación final. Enfoque defensivo y forense. |
| [modulo-1.md](modulo-1.md) | Visión general del ecosistema Gen (GenLogin, GenFarmer, GenRouter, GenPlay) y la cadena de suplantación de identidad. |
| [modulo-2.md](modulo-2.md) | Análisis forense del agente APK: obtención de la muestra, ingeniería inversa estática y dinámica, y extracción de IoCs. |
| [modulo-3.md](modulo-3.md) | Protocolos de control panel-dispositivo y métodos de inyección táctil ordenados por detectabilidad y huella forense. |
| [modulo-4.md](modulo-4.md) | Gestión de identidad y red: GenLogin y *fingerprinting*, proxies (VPNService vs. iptables) y manipulación del Advertising ID. |
| [modulo-5.md](modulo-5.md) | Flujos de trabajo típicos —registro masivo de cuentas, fraude CPA y amplificación en redes— y su reconstrucción forense. |
| [modulo-6.md](modulo-6.md) | Estrategias de defensa multicapa (dispositivo, red, cuenta) y neutralización: estrangulamiento de servicios, acción legal e intercambio de IoCs. |
| [material-complementario.md](material-complementario.md) | Tarea final de certificación, soluciones razonadas de los ejercicios y examen tipo test de 20 preguntas con su clave. |


        ╔═══════════════════════════════════════════════════════════╗
        ║  ⚠  Material con fines de investigación, análisis forense  ║
        ║     y divulgación. La automatización sobre dispositivos    ║
        ║     ajenos o cuentas de terceros vulnera los términos de   ║
        ║     servicio y, según la jurisdicción, la ley.             ║
        ╚═══════════════════════════════════════════════════════════╝

              .─────────────────────────────────────────────.
             (   aguscort/granjas-de-bots · MIT · 2026        )
              `─────────────────────────────────────────────'
