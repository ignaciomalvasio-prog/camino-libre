# Rutas Córdoba — mapa de reportes en ruta

Convierte los mensajes de un grupo de WhatsApp en un sitio web con **presencia
policial** y **choques** sobre las rutas de la provincia, con expiración
automática y zoom hasta nivel de calle.

## Estructura

```
generar_sitio.py      genera los datos del sitio
parser_reportes.py    lee el chat, clasifica y geocodifica
cordoba_rutas.py      gazetteer: localidades, coordenadas, alias, trazados
ejemplo_chat.txt      chat de prueba
sitio/
  index.html          la web (esto es lo que se publica)
  datos/datos.js      datos generados
```

`generar_mapa.py` y `mapa.html` quedan del prototipo anterior: producen un único
archivo autocontenido, sin necesidad de publicar nada. Sirven si solo querés
mirarlo local.

## Uso

**1. Exportar el chat.** WhatsApp → grupo → ⋮ → Más → Exportar chat →
**Sin archivos adjuntos**.

**2. Generar los datos.**

```bash
python3 generar_sitio.py chat.txt
```

**3. Abrir `sitio/index.html`.** Funciona con doble click. Si Windows te lo abre
con el Bloc de notas, es la asociación de archivo: click derecho → Abrir con →
Chrome.

Para probar sin tu chat real:

```bash
python3 generar_sitio.py ejemplo_chat.txt --ahora 2026-08-11T12:00:00
```

## Publicar en GitHub Pages

1. Crear un repo nuevo, público, por ejemplo `rutas-cordoba`.
2. Subir el **contenido de `sitio/`** a la raíz del repo — o sea `index.html` y
   la carpeta `datos/`. No subas la carpeta `sitio` entera, o la URL te queda
   con un nivel de más.
3. En el repo: Settings → Pages → Source: `Deploy from a branch` → Branch:
   `main`, carpeta `/ (root)` → Save.
4. A los dos o tres minutos queda en `https://TUUSUARIO.github.io/rutas-cordoba`.

**Para actualizar**: corré `generar_sitio.py`, reemplazá `datos/datos.js` en el
repo y listo. `index.html` no cambia salvo que toques el diseño.

## Cómo funciona

| Paso | Detalle |
|---|---|
| **Lectura** | Formatos de export de Android e iOS, une mensajes multilínea, descarta avisos del sistema |
| **Clasificación** | `CAMINERA` (caminera, control, alcoholemia, radar, gendarmería, retén…) y `CHOQUE` (choque, vuelco, despiste, siniestro…). Si aparecen los dos, gana choque |
| **Levantes** | Detecta *"ya se fue"*, *"levantaron"*, *"camino libre"*, *"no hay caminera"* y da de baja el reporte previo del mismo tipo a menos de 8 km |
| **Geocodificación** | Ruta + localidad → punto exacto (confianza alta). Ruta + km → interpolación sobre el trazado (media). Solo localidad o solo ruta → aproximado (media / baja) |
| **Vigencia** | Caminera 2 h, choques 4 h. Después quedan en gris como histórico |

Cobertura: RN9 norte y sur, RN19, RN20 (incluye Altas Cumbres), RN36, RN38,
RN158, RP5, E-53 y E-55, con ~75 localidades y sus alias coloquiales
("la 9", "carlos paz", "vcp", "el cuadrado").

Sobre el chat de ejemplo: 23 mensajes → 13 reportes, cero falsos positivos.

## La web

- Zoom hasta nivel 19 — se ven carriles, banquina y colectoras.
- Dos capas: Calles (OpenStreetMap) y Relieve (OpenTopoMap, útil en Punilla y
  Altas Cumbres).
- Click en un reporte de la lista → vuela hasta ahí. Click en una ruta →
  encuadra la ruta entera. Escape → vuelve a toda la provincia.
- Responsive: en el celular el mapa queda arriba y el panel abajo.

Los datos van en un `.js` y no en un `.json` a propósito: así el sitio también
funciona abriendo el archivo directamente, donde `fetch()` de un `.json` sería
bloqueado por CORS.

## Ajustes habituales

**Vocabulario propio.** Cada grupo tiene sus códigos. Agregá los términos en
`KW_CAMINERA`, `KW_CHOQUE` y `KW_LEVANTE` dentro de `parser_reportes.py`.

**Más localidades.** En `LOCALIDADES` con coordenadas y alias, y sumalas al
tramo correspondiente en `RUTAS`.

**Tiempos de vigencia.** En `VIGENCIA`, arriba de `parser_reportes.py`.

## Limitaciones

Las coordenadas son del **centro urbano** de cada localidad, no del punto exacto
sobre la ruta. A escala provincial no se nota; con zoom fuerte sí — el punto cae
en el pueblo y no sobre el asfalto. Se corrige bajando la geometría real de cada
ruta desde OpenStreetMap con una consulta Overpass (`ref=RN9`, etc.) y
proyectando el punto sobre la línea.

Los kilometrajes están calibrados a ojo, no son datos de Vialidad.

El parser es de reglas, no de IA: entiende las formas comunes de reportar pero
se le escapan mensajes muy elípticos ("ojo ahí en el puente"). Una mejora clara
es pasarle los mensajes no clasificados a un modelo de lenguaje.

## Para que sea tiempo real

Esto procesa un export, o sea que es un snapshot. Para que se actualice solo, el
camino corto es un **bot de Telegram** en paralelo al grupo de WhatsApp: API
gratis y abierta, soporta ubicación nativa, y el mismo parser sirve sin cambios.
La alternativa es WhatsApp Business API, que funciona pero cobra por
conversación y requiere aprobación de Meta.

## Nota legal

Los datos son generados por los usuarios del grupo, no provienen de Google Maps
ni de Waze — cuyos términos prohíben expresamente extraer contenido para armar
bases derivadas. Al ser un proyecto abierto conviene tener presente que informar
controles de alcoholemia específicamente es un punto sensible en algunas
jurisdicciones; vale chequear la normativa provincial antes de difundirlo
masivamente.
