# Extracción de datos — accidentes-transito-chile

Este documento describe instrucciones prácticas y reproducibles para extraer datos desde las dos fuentes públicas relacionadas con accidentes de tránsito en Chile:

- https://www.conaset.cl/programa/observatorio-datos-estadistica/
- https://mapas-conaset.opendata.arcgis.com/

Objetivo: proporcionar un procedimiento consistente para automatizar el pipeline de extracción, validar y publicar artefactos (CSV / Parquet / GeoJSON) localmente y en S3 para consumo por pipelines de transformación.

---

## Checklist de requisitos

- [ ] Entorno Python 3.9+ (recomendado 3.10/3.11)
- [ ] Entorno virtual `.venv` activo
- [ ] `requirements.txt` actualizado con dependencias mínimas
- [ ] Credenciales AWS configuradas localmente (`aws configure`) si se usará S3
- [ ] Directorio `accidentes-transito-chile/pipelines/` creado

---

## 1) Estructura de salida recomendada (local)

Organiza los artefactos extraídos en carpetas con partición por fecha para idempotencia y trazabilidad:

```
accidentes-transito-chile/
└── data_raw/
    ├── conaset/
    │   └── raw_YYYYMMDD.csv
    ├── arcgis/
    │   └── features_YYYYMMDD.geojson
    └── manifest/    # JSON con metadatos de extracción (origen, timestamp, records, checksum)
```

En S3 (si aplica): `s3://<bucket>/accidentes-transito-chile/raw/<origen>/year=YYYY/month=MM/day=DD/`.

---

## 2) Prerrequisitos e instalación (PowerShell)

```powershell
# crear y activar venv
python -m venv .venv
./.venv/Scripts/Activate.ps1

# actualizar pip
python -m pip install --upgrade pip

# instalar dependencias mínimas (añadir al requirements.txt)
pip install requests pandas geopandas pyarrow shapely tenacity boto3

# opcional: si sólo trabajas con CSV y GeoJSON puedes evitar geopandas
# pip install requests pandas pyarrow tenacity boto3
```

Añade las dependencias a `requirements.txt` cuando las uses.

---

## 3) Fuente A — CONASET (observatorio)

La página del Observatorio de CONASET contiene enlaces a datasets y reportes. Hay dos enfoques para extraer:

A) Descarga directa (si hay enlaces CSV/Excel)
B) Scraping ligero del HTML para localizar enlaces

### 3.1 Recomendación

- Buscar en la página la sección de "descarga de datos" o enlaces directos a CSV/Excel.
- Si existe un enlace directo a CSV: usar `requests` para descargar y guardar con nombre con fecha.
- Si no hay enlace directo: identificar la URL de origen del dataset (a veces apuntan a ArcGIS Open Data — ver sección 4).

### 3.2 Ejemplo de script para descarga directa (CSV/Excel)

```python
# extract_conaset.py
import requests
from pathlib import Path
from datetime import datetime
import hashlib
import json

OUT_DIR = Path("./data_raw/conaset")
OUT_DIR.mkdir(parents=True, exist_ok=True)

CSV_URL = "<COLOCAR_URL_CSV_DIRECTO_AQUI>"  # actualizar con el enlace real

def save_with_manifest(content: bytes, filename: Path, source: str):
    filename.write_bytes(content)
    manifest = {
        "source": source,
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "size": len(content),
        "sha256": hashlib.sha256(content).hexdigest()
    }
    manifest_path = filename.with_suffix(filename.suffix + ".manifest.json")
    manifest_path.write_text(json.dumps(manifest, indent=2))

if __name__ == "__main__":
    resp = requests.get(CSV_URL, timeout=30)
    resp.raise_for_status()
    date_tag = datetime.utcnow().strftime("%Y%m%d")
    out_file = OUT_DIR / f"conaset_raw_{date_tag}.csv"
    save_with_manifest(resp.content, out_file, CSV_URL)
    print("Guardado:", out_file)
```

Notas:
- Rellenar `CSV_URL` con el enlace encontrado en la página del observatorio.
- Este script es idempotente por fecha; para ejecuciones múltiples en el mismo día podrías añadir hora o un hash del contenido.

---

## 4) Fuente B — ArcGIS Open Data (mapas-conaset.opendata.arcgis.com)

El portal ArcGIS Open Data publica capas espaciales. La forma correcta de extraer es usar la API REST del servicio de Feature Layer.

Pasos para localizar la URL del servicio:
1. Navega a la capa en `mapas-conaset.opendata.arcgis.com` y busca el botón `API` o `Datos` / `Exportar`.
2. Obtendrás una URL base con un patrón parecido a:
   `https://<host>/arcgis/rest/services/<folder>/<serviceName>/FeatureServer/<layerId>`
3. Para descargar todos los registros en GeoJSON, usar el endpoint `/query` con parámetros:
   - where=1=1
   - outFields=*
   - f=geojson
   - (opcional) resultRecordCount / offset para paginar

Ejemplo de URL:
```
https://<host>/arcgis/rest/services/.../FeatureServer/0/query?where=1%3D1&outFields=*&f=geojson
```

### 4.1 Ejemplo de script para extraer de ArcGIS (paginación segura)

```python
# extract_arcgis.py
import requests
from pathlib import Path
from datetime import datetime
import json
from tenacity import retry, stop_after_attempt, wait_exponential

OUT_DIR = Path("./data_raw/arcgis")
OUT_DIR.mkdir(parents=True, exist_ok=True)

FEATURE_URL = "<COLOCAR_URL_FEATURE_LAYER_QUERY_BASE_AQUI>"  # ejemplo: https://.../FeatureServer/0/query

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def fetch_geojson(params):
    resp = requests.get(FEATURE_URL, params=params, timeout=60)
    resp.raise_for_status()
    return resp.json()

def paginate_and_save():
    # La API ArcGIS puede devolver todo si no hay límite, pero si hay limitaciones hay que paginar con resultOffset/resultRecordCount
    params = {
        "where": "1=1",
        "outFields": "*",
        "f": "geojson",
        # "resultRecordCount": 1000,
        # "resultOffset": 0
    }

    geojson = fetch_geojson(params)
    date_tag = datetime.utcnow().strftime("%Y%m%d")
    out_file = OUT_DIR / f"arcgis_features_{date_tag}.geojson"
    out_file.write_text(json.dumps(geojson))
    print("Guardado:", out_file)

if __name__ == "__main__":
    paginate_and_save()
```

Notas:
- Si la capa limita el número de registros, debes usar `resultOffset` y `resultRecordCount` para paginar. Alternativamente usa el parámetro `where` por rangos (por fecha) si existe un campo timestamp.
- Revisa `features` en la respuesta para contar registros y construir registro de manifest.

---

## 5) Buenas prácticas para automatización

- Idempotencia: siempre guardar con `year/month/day` o `YYYYMMDD` y generar manifiesto con checksum.
- Retries: usar `tenacity` o lógica de reintento exponencial para llamadas HTTP.
- Timeouts: usar timeouts en `requests` para evitar colas infinitas.
- Logging: escribir logs estructurados (nivel INFO/ERROR) y guardar salida de ejecución en `pipelines/logs/`.
- Validación de esquema: tras descarga, validar columnas / tipos esenciales antes de aceptar dataset.
- Testing: crear un test minimal (pytest) que valide que el extractor devuelve >0 registros en un entorno de prueba.

---

## 6) Ejemplo de pipeline simple (bash / PowerShell)

Extrae, valida y sube a S3 (ejemplo PowerShell):

```powershell
# activar venv
./.venv/Scripts/Activate.ps1

# ejecutar extractores
python pipelines/extract_conaset.py
python pipelines/extract_arcgis.py

# validar (script local) -> pipelines/validate.py
python pipelines/validate.py --input data_raw/conaset/conaset_raw_$(Get-Date -Format yyyyMMdd).csv

# subir a S3
aws s3 sync data_raw s3://mi-bucket/accidentes-transito-chile/raw --exclude "*.manifest.json" --metadata direct=true
```

Nota: ajustar `aws s3 sync` con políticas de versión y control de acceso.

---

## 7) Integración con AWS (despliegue en free tier)

- Frontend: subir build a S3 + CloudFront.
- Datos: S3 para raw/processed.
- Automatización: usar una EC2 t2.micro o GitHub Actions como runner para ejecutar extracciones programadas si no quieres usar Lambda para ejecuciones periódicas.
- Si usas Lambda: empaqueta los extractores o crea una función que invoque un job de extracción desde un container (podría requerir más trabajo si dependes de geopandas).

Recomendación: para extracción inicial conservar ejecuciones locales / runner GitHub Action; cuando el extractor sea ligero (solo requests + pandas) migrar a Lambda.

---

## 8) Plantilla de manifest (ejemplo)

```json
{
  "source": "conaset",
  "extracted_at": "2025-09-09T12:00:00Z",
  "file": "conaset_raw_20250909.csv",
  "records": 12345,
  "size_bytes": 1234567,
  "sha256": "..."
}
```

---

## 9) Siguientes pasos sugeridos

1. Recolectar las URLs directas en la página del Observatorio CONASET y rellenar `CSV_URL` en `extract_conaset.py`.
2. Localizar el Feature Service URL en ArcGIS Open Data y colocarlo en `FEATURE_URL` en `extract_arcgis.py`.
3. Añadir estos scripts reales en `pipelines/` y crear pruebas unitarias básicas.
4. Definir el job de CI (GitHub Actions) o planificar ejecución en un runner local/EC2.

---

Si quieres, creo los scripts `pipelines/extract_conaset.py` y `pipelines/extract_arcgis.py` en el repo y actualizo `requirements.txt` con las librerías necesarias; dime si procedo y si prefieres que use `geopandas` (más pesado) o sólo `requests/pandas` + `pyarrow`.
