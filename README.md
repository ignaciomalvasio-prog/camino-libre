# Rutas Córdoba

Mapa colaborativo de presencia policial y siniestros viales en las rutas de la
provincia de Córdoba, armado a partir de los reportes de un grupo de WhatsApp.

**Sitio: https://ignaciomalvasio-prog.github.io/rutas-cordoba/**

## Qué hay en este repo

| Archivo | Qué es |
|---|---|
| `index.html` | La web. Mapa, filtros y listado de reportes |
| `datos.js` | Los reportes geolocalizados. Es lo único que cambia al actualizar |
| `README.md` | Esto |

Las herramientas que generan `datos.js` (`generar_sitio.py`, `parser_reportes.py`
y `cordoba_rutas.py`) se corren aparte y no forman parte del sitio publicado.

## Cómo se actualiza

| Paso | Acción |
|---|---|
| Exportar | WhatsApp → grupo → ⋮ → Más → Exportar chat → **Sin archivos** |
| Generar | `python3 generar_sitio.py chat.txt` produce un `datos.js` nuevo |
| Publicar | Reemplazar `datos.js` en este repo. Pages republica solo en un par de minutos |

`index.html` no se toca salvo que cambie el diseño.

## Cómo funciona

| Paso | Detalle |
|---|---|
| Lectura | Formatos de export de Android e iOS, une mensajes multilínea, descarta avisos del sistema |
| Clasificación | `CAMINERA` (caminera, control, alcoholemia, radar, gendarmería, retén…) y `CHOQUE` (choque, vuelco, despiste, siniestro…). Si aparecen los dos, gana choque |
| Levantes | Detecta "ya se fue", "levantaron", "camino libre", "no hay caminera" y da de baja el reporte previo del mismo tipo a menos de 8 km |
| Geocodificación | Ruta + localidad → punto exacto. Ruta + km → interpolación sobre el trazado. Solo localidad o solo ruta → aproximado |
| Vigencia | Caminera 2 h, choques 4 h. Después quedan en gris como histórico |

Cobertura: RN9 norte y sur, RN19, RN20 (incluye Altas Cumbres), RN36, RN38,
RN158, RP5, E-53 y E-55, con unas 75 localidades y sus alias coloquiales
("la 9", "carlos paz", "vcp", "el cuadrado").

Sobre el chat de ejemplo: 23 mensajes, 13 reportes detectados, cero falsos positivos.

## La web

Zoom hasta nivel 19, o sea que se ven carriles y banquina. Dos capas: Calles
(OpenStreetMap) y Relieve (OpenTopoMap, útil en Punilla y Altas Cumbres). Tocar
un reporte del listado vuela hasta ahí; tocar una ruta la encuadra entera;
Escape vuelve a toda la provincia. En el celular el mapa queda arriba y el panel
abajo.

Los datos van en un `.js` y no en un `.json` a propósito: así el sitio también
funciona abriendo `index.html` desde el disco, donde `fetch()` de un `.json`
sería bloqueado por CORS.

## Limitaciones

Las coordenadas son del **centro urbano** de cada localidad, no del punto exacto
sobre la ruta. A escala provincial no se nota; con zoom fuerte sí, porque el
punto cae en el pueblo y no sobre el asfalto. Se corrige bajando la geometría
real de cada ruta desde OpenStreetMap con una consulta Overpass (`ref=RN9`) y
proyectando el punto sobre la línea.

Los kilometrajes están calibrados de forma aproximada, no son datos de Vialidad.

El parser es de reglas, no de IA: entiende las formas comunes de reportar pero
se le escapan mensajes muy elípticos ("ojo ahí en el puente").

## Para que sea tiempo real

Hoy procesa un export, o sea que es una foto del momento. Para que se actualice
solo, el camino corto es un bot de Telegram en paralelo al grupo: la API es
gratis, soporta ubicación nativa y el mismo parser sirve sin cambios. La
alternativa es WhatsApp Business API, que funciona pero cobra por conversación y
requiere aprobación de Meta.

## Nota

Los reportes son de usuarios y no están verificados. Los datos los genera la
comunidad del grupo; no provienen de Google Maps ni de Waze, cuyos términos
prohíben extraer contenido para armar bases derivadas.
