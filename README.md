# Monitor de nivel del tanque — sitio web

Página estática (GitHub Pages) que muestra el nivel del tanque y permite calibrarlo. Lee de ThingSpeak los datos que envía el ESP32.

## Archivos
- `index.html` — la página completa (HTML + CSS + JS, usa Chart.js desde CDN).
- `config.json` — configuración compartida por la web y el ESP32.

## Publicar en GitHub Pages
1. Crea un repositorio **público** y sube `index.html` y `config.json` a la raíz.
2. En el repositorio: **Settings → Pages → Source: Deploy from a branch**, rama `main`, carpeta `/ (root)`.
3. La página queda en `https://<tu-usuario>.github.io/<repositorio>/`.

## Modo demo
Si `canal_datos.id` está vacío, la página usa datos simulados (30 días) y muestra "Modo demo".
También puedes forzarlo con `?demo=1` al final de la URL. En demo, la calibración se aplica solo en pantalla.

## Canales de ThingSpeak (2 de los 4 gratuitos)

**Canal 1 — Datos** (lo escribe solo el ESP32)
| Campo | Contenido |
|---|---|
| Field 1 | Distancia medida por el sensor (cm), **cruda** |
| Field 2 | Modo (siempre 0; el 1 era el antiguo modo calibración por botón) |
| Field 3 | Código de error (0 = OK, 1 = falla del sensor) |

**Canal 2 — Calibración** (lo escribe la web; lo leen la web y el ESP32)
| Campo | Contenido |
|---|---|
| Field 1 | Distancia con tanque vacío (cm) = altura total del tanque (sensor en la tapa) |
| Field 2 | Distancia con tanque lleno (cm) = 25 cm (zona muerta del sensor) |
| Field 3 | Diámetro promedio del tanque (cm) |

Pon los **Channel ID** en `config.json → thingspeak`. Si un canal es privado, pon su **Read API Key** en `read_key`; si es público, déjala vacía.

## Cálculo del nivel (en la web)
- Altura de agua: `h = d_vacío − d`
- Porcentaje: `% = h / (d_vacío − d_lleno) × 100`
- Volumen (tanque cilíndrico): `V [L] = π · (D/2)² · h / 1000`

Como el ESP32 guarda la distancia cruda, recalibrar recalcula todo el histórico.

## Calibración
Es manual, desde la página (el dispositivo no tiene botón). Solo se ingresan dos datos:
1. **Diámetro promedio** del tanque.
2. **Altura total** del tanque, del piso (fondo) a la tapa, donde va el sensor.

El nivel se mide desde el piso. El **100 %** es el agua a **25 cm por debajo de la tapa**, porque el sensor no mide más cerca (zona muerta). La página guarda `vacío = altura total` y `lleno = 25 cm`, así que la altura útil es `altura total − 25 cm`.

**Guardar calibración** escribe los tres valores en una sola fila del canal de calibración; el ESP32 los toma en su siguiente ciclo.

La Write API Key del canal de calibración se escribe en el panel y, si marcas "Recordar", queda solo en tu navegador.

> ⚠️ **Nunca** pongas Write API Keys en `config.json`: el repositorio es público. La clave de escritura del canal de datos vive solo en el firmware del ESP32.

## Qué configura `config.json`
| Clave | Uso |
|---|---|
| `nombre_tanque`, `ubicacion` | Título de la página |
| `umbrales.bajo_pct`, `umbrales.alto_pct` | Líneas de alerta y estado |
| `esp32.intervalo_s` | Ciclo normal del ESP32 en segundos (mínimo 20 s por el límite de ThingSpeak) |
| `esp32.*` (resto) | Parámetros que el ESP32 descarga en cada ciclo |
