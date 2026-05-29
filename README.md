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
╭─────────────────────────────────────────────────────────────────────╮
│  §1 · EL FENÓMENO                        ░ concepto · usos · política  │
╰─────────────────────────────────────────────────────────────────────╯
```

|  | Archivo | Contenido |
| :-: | --- | --- |
| ▢ | [tipos-de-granjas.md](tipos-de-granjas.md) | Granja tradicional vs. granja de cajas: placas base "desnudas", alimentación DC centralizada, refrigeración y control integrado. |
| ▢ | [usos.md](usos.md) | Catálogo de usos: monetización pasiva, *play-to-earn*, inflado de métricas, astroturfing político, reseñas falsas, *scalping*, *scraping* y *credential stuffing*. |
| ▢ | [ecosistema-mmo.md](ecosistema-mmo.md) | El "ecosistema MMO" (*Make Money Online*) y el modelo de tres patas GenLogin / GenFarmer / GenRouter. |
| ▢ | [amplificación-de-narrativas.md](amplificaci%C3%B3n-de-narrativas.md) | La amplificación como táctica: espiral del silencio, asociación repetitiva y el salto que introdujeron los LLM. |
| ▢ | [consensus-reality.md](consensus-reality.md) | La "realidad de consenso": tesis Surkov, elección de "realidades ideales" y parálisis cívica por agotamiento informativo. |

```
╭─────────────────────────────────────────────────────────────────────╮
│  §2 · INFRAESTRUCTURA Y TÉCNICA             ▣ hardware · red · software │
╰─────────────────────────────────────────────────────────────────────╯
```

|  | Archivo | Contenido |
| :-: | --- | --- |
| ⚙ | [arquitectura.md](arquitectura.md) | Arquitectura física y lógica: control centralizado, *fingerprinting*, ofuscación de red, capa de IA y verificación externa. |
| ⚙ | [infraestructura-fisica.md](infraestructura-fisica.md) | Artículo extenso: dispositivos y racks, energía y SIMs, orquestación, emuladores, inyección HID por hardware, proxies, LLM locales y detección. |
| ⚙ | [infraestructura-critica-de-microdispositvos.md](infraestructura-critica-de-microdispositvos.md) | Índice tipo manual de ingeniería de *box phone farms* en seis partes (planificación, hardware, red, software, IA, seguridad) más apéndices. |
| ⚙ | [comparativa-dispositivo-vs-disimulación.md](comparativa-dispositivo-vs-disimulaci%C3%B3n.md) | Tablas: teléfonos reales vs. emuladores vs. simuladores, GeeLark vs. GenFarmer, y coste físico vs. nube. |
| ⚙ | [automatizacion.md](automatizacion.md) | Automatización móvil legítima: de ADB a Appium, con código, escalado a flotas y la línea que separa el QA ético del fraude. |
| ⚙ | [gestion-logica.md](gestion-logica.md) | Los tres pilares: GenFarmer (flotas), OSMBB (inyección sobre emuladores) y Dolphin (LLM sin restricciones). |

```
╭─────────────────────────────────────────────────────────────────────╮
│  §3 · CURSO FORENSE                       ⌬ GenFarmer · 6 módulos · 24h │
╰─────────────────────────────────────────────────────────────────────╯

      [M1]──[M2]──[M3]──[M4]──[M5]──[M6]──▶ ✦ certificación
       │     │     │     │     │     │
     visión APK  control id+red flujos defensa
```

|  | Archivo | Contenido |
| :-: | --- | --- |
| ✦ | [genfarmer-curso.md](genfarmer-curso.md) | Programa del curso: 24 h en 6 módulos, prerrequisitos, objetivos y evaluación final. Enfoque defensivo y forense. |
| ① | [modulo-1.md](modulo-1.md) | Visión general del ecosistema Gen (GenLogin, GenFarmer, GenRouter, GenPlay) y la cadena de suplantación de identidad. |
| ② | [modulo-2.md](modulo-2.md) | Análisis forense del agente APK: obtención de muestra, ingeniería inversa estática y dinámica, y extracción de IoCs. |
| ③ | [modulo-3.md](modulo-3.md) | Protocolos de control y métodos de inyección táctil ordenados por detectabilidad y huella forense. |
| ④ | [modulo-4.md](modulo-4.md) | Gestión de identidad y red: *fingerprinting*, proxies (VPNService vs. iptables) y manipulación del Advertising ID. |
| ⑤ | [modulo-5.md](modulo-5.md) | Flujos típicos (registro masivo, fraude CPA, amplificación) y su reconstrucción forense. |
| ⑥ | [modulo-6.md](modulo-6.md) | Defensa multicapa (dispositivo, red, cuenta) y neutralización: estrangular servicios, acción legal e intercambio de IoCs. |
| ✎ | [material-complementario.md](material-complementario.md) | Tarea de certificación, soluciones razonadas y examen tipo test de 20 preguntas con su clave. |


```
        ╔═══════════════════════════════════════════════════════════╗
        ║  ⚠  Material con fines de investigación, análisis forense  ║
        ║     y divulgación. La automatización sobre dispositivos    ║
        ║     ajenos o cuentas de terceros vulnera los términos de   ║
        ║     servicio y, según la jurisdicción, la ley.             ║
        ╚═══════════════════════════════════════════════════════════╝

              .─────────────────────────────────────────────.
             (   aguscort/granjas-de-bots · MIT · 2026        )
              `─────────────────────────────────────────────'
```
