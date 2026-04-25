# 🔥 FuegoAR — Monitor de Incendios en Argentina

Plataforma web de monitoreo satelital de incendios forestales en tiempo real y análisis histórico para Argentina. Desarrollado como herramienta pública y portfolio GIS profesional.

**[→ Ver la aplicación en vivo](https://gerabarion-hash.github.io/fuego-ar/)**

---

## ¿Qué hace?

**Mapa en vivo**
- Focos activos detectados por los satélites NASA VIIRS (NOAA-20 y NOAA-21), actualizados cada 3-6 horas
- Filtros por confiabilidad (Alta / Nominal / Baja) y potencia radiativa del fuego (FRP en MW)
- Verificación satelital inmediata con Sentinel-2 en color real (RGB) e infrarrojo de onda corta (SWIR)
- Compartir focos por WhatsApp, Facebook, Instagram o enlace directo con permalink georreferenciado

**Casos históricos documentados**
- 4 incendios de gran magnitud en la Patagonia argentina (2024–2026)
- Imágenes Sentinel-2 durante el incendio y post-fuego, seleccionadas por baja nubosidad
- Mapa de severidad dNBR con clasificación USGS (Baja / Moderada-baja / Moderada-alta / Alta)
- Análisis NBR, NDVI y dNBR procesados con Google Earth Engine

---

## Casos históricos incluidos

| Caso | Provincia | Superficie | Inicio |
|---|---|---|---|
| Los Manzanos — PN Nahuel Huapi | Río Negro | ~11.475 ha | Dic 2024 |
| Lanín / Valle Magdalena — PN Lanín | Neuquén | ~24.100 ha | Ene 2025 |
| Puerto Madryn — Estepa costera | Chubut | — | Ene 2025 |
| PN Los Alerces "El Centinela" | Chubut | ~6.924 ha (parque) | Dic 2025 |

---

## Stack técnico

| Componente | Tecnología |
|---|---|
| Frontend | HTML5 + Leaflet.js (sin frameworks) |
| Focos activos | NASA FIRMS API — VIIRS NOAA-20/21 NRT |
| Imágenes satelitales | Sentinel Hub WMS + Copernicus ESA |
| Catálogo de imágenes | Element84 Earth Search (STAC) |
| Análisis histórico | Google Earth Engine — `COPERNICUS/S2_SR_HARMONIZED` |
| Índices espectrales | NBR, dNBR, NDVI con máscara SCL |
| Clasificación severidad | Escala USGS (Key & Benson 2006) |
| Servicio de rasters | titiler + Cloudflare R2 |
| Hosting | GitHub Pages |
| Límite geográfico | GeoJSON propio (punto-en-polígono client-side) |

---

## Metodología de análisis

El análisis de severidad de cada incendio sigue el protocolo estándar USGS:

```
dNBR = NBR_prefuego − NBR_postfuego
NBR  = (B8 − B12) / (B8 + B12)   [Sentinel-2 L2A]
```

**Ventanas temporales:**
- Pre-fuego: 30–45 días antes del inicio (estado basal de la vegetación)
- Post-fuego: 15–25 días después del control (daño consolidado antes de revegetación)

**Máscara SCL:** se excluyen píxeles de agua, nieve, nubes y sombras antes de calcular los índices — crítico en la cordillera patagónica donde lagos y glaciares generan falsos positivos.

**Clasificación USGS de 6 niveles:**

| Clase | dNBR | Descripción |
|---|---|---|
| 1 | < -0.10 | Rebrote / sin efecto |
| 2 | -0.10 a 0.10 | Sin quema detectable |
| 3 | 0.10 a 0.27 | Baja severidad |
| 4 | 0.27 a 0.44 | Moderada-baja |
| 5 | 0.44 a 0.66 | Moderada-alta |
| 6 | > 0.66 | Alta severidad |

---

## Estructura del repositorio

```
fuego-ar/
├── index.html          # Aplicación completa (single-file)
├── argentina.geojson   # Límite país para filtrado client-side
├── casos.json          # Datos de casos históricos (editable sin tocar código)
└── GEE/
    └── Codigos para GEE/
        ├── GEE_LosManzanos2024.js
        ├── GEE_Lanin2025.js
        ├── GEE_PuertoMadryn2025.js
        └── GEE_LosAlerces2025.js
```

---

## Cómo agregar un caso histórico

1. Procesá el incendio con el script GEE correspondiente (carpeta `GEE/`)
2. Subí los TIFFs de severidad a Cloudflare R2
3. Agregá una entrada en `casos.json`:

```json
"nuevo_caso": {
  "nombre": "Nombre del incendio",
  "lugar": "Parque / Provincia",
  "lat": -38.000, "lng": -70.000, "zoom": 11,
  "dur":  { "fecha": "YYYY-MM-DD", "label": "DD mes YYYY", "ventana": 2 },
  "post": { "fecha": "YYYY-MM-DD", "label": "DD mes YYYY", "ventana": 2 },
  "sev": {
    "archivo": "NombreArchivo_Severidad_USGS.tif",
    "bounds": [[lat_min, lng_min], [lat_max, lng_max]],
    "w": 900, "h": 600
  }
}
```

4. Agregá la tarjeta correspondiente en el panel HTML de `index.html`
5. `git add . && git commit -m "agrega caso X" && git push origin master:main`

---

## Datos de API

- **NASA FIRMS:** `https://firms.modaps.eosdis.nasa.gov/api/`
- **Sentinel Hub WMS:** instancia propia en Copernicus Data Space
- **Element84 Earth Search:** `https://earth-search.aws.element84.com/v1/`
- **titiler:** `https://titiler.xyz/cog/`

---

## Autor

**Gera Barion** — GIS & Geospatial Web Developer  
Portfolio: [gerabarion-hash.github.io/fuego-ar](https://gerabarion-hash.github.io/fuego-ar/)  
GitHub: [github.com/gerabarion-hash](https://github.com/gerabarion-hash)

---

*Datos de focos activos: NASA FIRMS — Fire Information for Resource Management System*  
*Imágenes satelitales: Copernicus Sentinel-2, ESA — datos procesados con Google Earth Engine*
