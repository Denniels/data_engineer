# 🧠 Portafolio Data Engineer - Daniel Andrés Mardones Sanhueza

Este repositorio contiene una colección de proyectos reales y complejos que demuestran habilidades avanzadas en ingeniería de datos, desde la adquisición física de datos hasta el análisis estadístico, la toma de decisiones y el despliegue en la nube.

## 🎯 Objetivos del Portafolio

- Aplicar análisis exploratorio (EDA) y estadística avanzada para encontrar insights valiosos.
- Mostrar dominio en SQL avanzado (funciones de ventana, CTEs, agregaciones).
- Tomar decisiones basadas en datos reales, con impacto en el negocio.
- Elegir arquitecturas adecuadas: Data Lakes, Data Warehouses, bases relacionales/no relacionales.
- Desplegar soluciones completas en AWS utilizando la capa gratuita.
- Documentar y versionar todo el proceso con Git y GitHub.
- Mantener buenas prácticas de organización, modularidad y escalabilidad.

---

## 🗺️ Roadmap de Desarrollo

| Fase | Proyecto | Estado | Objetivo |
|------|----------|--------|----------|
| ✅ | `iot-jetson-nano` | Ejecutando | Conectar a la base local, extraer datos y prototipar análisis |
| 🟡 | `accidentes-transito-chile` | En desarrollo | EDA, SQL avanzado, visualización y despliegue en AWS |
| ⏳ | `dashboard-indicadores-economicos` | Pendiente | Integración de APIs, análisis económico y visualización |
| ⏳ | `transporte-publico-santiago` | Pendiente | Análisis geoespacial, clustering y optimización de rutas |

---

## 📁 Estructura General del Repositorio

```text
data_engineer/
├── frontend/                  # Frontend React (Vite) portafolio unificado
│   ├── src/                   # Componentes, páginas, hooks, servicios
│   ├── public/                # Assets estáticos
│   └── README.md              # Guía específica frontend
├── accidentes-transito-chile/
│   ├── data/                  # Datos crudos / procesados locales (no versionar grandes)
│   ├── notebooks/             # Exploración y prototipos
│   ├── src/                   # Código modular reutilizable (transform, utils)
│   ├── pipelines/             # Ingesta / limpieza / enriquecimiento (scripts orquestables)
│   ├── sql/                   # Consultas y vistas analíticas
│   ├── infra/                 # IaC / configuración AWS (S3, Glue, Athena, RDS si aplica)
│   └── README.md
├── dashboard-indicadores-economicos/
│   ├── data/
│   ├── notebooks/
│   ├── src/
│   ├── pipelines/             # Jobs de extracción de APIs y normalización
│   ├── api/                   # Endpoints (Lambda/FastAPI empaquetable) opcional
│   ├── infra/                 # Definición de tablas DynamoDB / buckets S3
│   └── README.md
├── transporte-publico-santiago/
│   ├── data/
│   ├── notebooks/
│   ├── src/
│   ├── pipelines/             # Procesamiento geoespacial, clustering
│   ├── sql/                   # PostGIS / análisis espacial
│   ├── infra/                 # Scripts provisión (RDS PostGIS futuro / S3 GeoJSON)
│   └── README.md
├── iot-jetson-nano/
│   ├── data_ingestion/        # Captura desde dispositivos (serial, MQTT, etc.)
│   ├── data_processing/       # Limpieza y estructuración inicial
│   ├── database/              # Conexión / esquemas PostgreSQL local
│   ├── notebooks/             # Exploraciones y validaciones
│   ├── src/                   # Funciones reutilizables
│   ├── pipelines/             # Flujo batch / sincronización a S3
│   ├── api/                   # Exposición de datos (handler Lambda / FastAPI opcional)
│   ├── tunnel/                # Config Cloudflare Tunnel
│   ├── aws_sync/              # Estrategia migración datos a la nube
│   ├── systemd_services/      # Servicios para automatizar en Jetson
│   └── README.md
├── requirements.txt           # Dependencias Python (inicialmente vacío)
├── .venv/                     # (Ignorado) Entorno virtual local
└── README.md                  # Documentación general
```

> 🧩 *Cada proyecto puede extenderse con nuevos directorios según se requiera: pruebas unitarias, documentación técnica, dashboards adicionales, etc.*

---

## 📊 Análisis y Arquitectura por Proyecto

### 1. `accidentes-transito-chile`
- **Objetivo**: Identificar patrones de siniestros viales y proponer medidas preventivas.
- **Fuente de Datos**: Dataset público CONASET (descarga periódica → staging S3).
- **Pipeline**: Ingesta (descarga), validación esquemas, normalización, enriquecimiento geográfico opcional, publicación parquet particionado.
- **SQL / Analytics**: Ventanas para recurrencia (hora, comuna), severidad, segmentación temporal.
- **Arquitectura Inicial**: S3 (raw/processed), vistas analíticas en Athena (Glue catálogo) y opcional RDS para consultas transaccionales limitadas.
- **Frontend**: Consumo de archivo agregado (JSON/Parquet resumido) servido vía bucket público o endpoint Lambda.

---

### 2. `dashboard-indicadores-economicos`
- **Objetivo**: Analizar evolución y correlaciones de indicadores macro.
- **Fuente de Datos**: APIs Banco Central, INE y otras públicas.
- **Pipeline**: Jobs programados (EventBridge → Lambda) que extraen, homogenizan unidades, calculan variaciones y almacenan versiones (S3 + DynamoDB para últimas métricas).
- **Modelo de Datos**: Series temporales normalizadas (base 100), granularidad mensual y anual.
- **Exposición**: Endpoint ligero (Lambda) para último snapshot + archivos históricos en S3.
- **Frontend**: Gráficas interactivas consumiendo endpoints JSON.

---

### 3. `transporte-publico-santiago`
- **Objetivo**: Derivar métricas de eficiencia y demanda en rutas.
- **Datos**: Logs GPS, horarios planificados, zonas y paraderos.
- **Pipeline**: Ingesta batch (CSV o APIs), limpieza coordenadas, map matching (futuro), generación de GeoJSON agregados (frecuencia, puntualidad), clustering de puntos de congestión.
- **Procesamiento Espacial**: Uso local de PostGIS / GeoPandas; en nube se publican sólo resultados optimizados (GeoJSON) en S3.
- **Frontend**: Mapas interactivos (leaflet/MapLibre) cargando capas GeoJSON desde S3.

---

### 4. `iot-jetson-nano`
- **Objetivo**: Capturar y analizar telemetría de sensores físicos.
- **Datos**: Temperatura, luz y otros (ESP32 / Arduino) via serial o MQTT.
- **Pipeline**: Ingesta continua a PostgreSQL local → batch export a archivos (Parquet) → sincronización a S3 (processed) → agregaciones temporales (resample) → detección de anomalías.
- **Exposición**: API ligera (FastAPI/Lambda) para métricas recientes + archivos históricos en S3.
- **Infra Local**: Servicios systemd para robustez en Jetson + túnel seguro (Cloudflare) para acceso eventual.
- **Migración a Nube**: Export incremental programado (cron / systemd timer) hacia S3.

---

## 🧰 Tecnologías Utilizadas

- **Lenguajes**: Python, SQL, TypeScript (frontend)
- **Datos / Almacenamiento**: S3 (raw/processed), PostgreSQL (local / RDS futuro), DynamoDB (últimos snapshots), Athena/Glue (consulta ad-hoc), GeoJSON (capas espaciales)
- **Procesamiento**: Lambda (ingestas programadas), scripts batch, PostgreSQL / PostGIS local, Pandas / PyArrow / GeoPandas
- **Frontend**: React (Vite), Tailwind CSS
- **APIs**: FastAPI (local) / Lambda handlers (producción ligera)
- **Infra / DevOps**: AWS (S3, CloudFront, EventBridge, IAM, Lambda), Cloudflare Tunnel, GitHub Actions (futuro), Docker (desarrollo selectivo)
- **Observabilidad**: CloudWatch Logs (retención corta), métricas básicas Lambda

---

## 🧪 Entorno Local de Desarrollo

### Python (Data Engineering)
Se usará un entorno virtual aislado para todas las dependencias de los proyectos de datos.

Pasos (PowerShell Windows):

```powershell
# Crear entorno virtual
python -m venv .venv

# Activar
./.venv/Scripts/Activate.ps1

# Actualizar pip
python -m pip install --upgrade pip

# Instalar dependencias (archivo inicialmente vacío)
pip install -r requirements.txt
```

Para agregar una nueva librería:
```powershell
pip install nombre_libreria
pip freeze | Select-String nombre_libreria >> requirements.txt
```
(Mantenemos el archivo ordenado manualmente más adelante.)

### Node / React (Frontend del Portafolio)

1. Verificar instalación:
```powershell
node --version
npm --version
```
2. Si no está instalado: descargar LTS desde https://nodejs.org/
3. Inicializar el frontend (pendiente de ejecutar, documentado):
```powershell
cd frontend
npm create vite@latest . -- --template react-swc
npm install
npm run dev
```
4. Añadir Tailwind (cuando se cree el scaffold):
```powershell
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

Estrategia: El frontend leerá un archivo de configuración JSON/TS (`projects.config.json` o `projects.ts`) mantenido manualmente para listar proyectos, estados y enlaces a documentación, además de endpoints/data sources.

---

## 🚀 Roadmap de Despliegue (AWS Free Tier)

Objetivo: Minimizar costos usando capa gratuita y arquitectura desacoplada, manteniendo portafolio central + demos específicas.

### Fase 0 – Preparación
- [ ] Crear Budget con alarma (Billing > Budgets) ≤ 5 USD.
- [ ] Crear usuario IAM para despliegue (políticas: `AmazonS3FullAccess` limitado a bucket específico, `CloudFrontFullAccess`, `AWSLambda_FullAccess` mínimo + logs, `AmazonDynamoDBFullAccess` limitado por tabla, `AmazonRDSFullAccess` si aplica). Luego restringir.
- [ ] Elegir región principal (ej: `us-east-1`).
- [ ] Configurar credenciales locales (`aws configure`).
- [ ] Crear archivo `infrastructure/README.md` (pendiente) para registrar recursos.

### Fase 1 – Frontend Portafolio (React Estático)
Opción inicial: S3 (hosting estático) + CloudFront (OAC) + dominio opcional posterior.

Pasos:
- [ ] Inicializar proyecto React en `frontend/`.
- [ ] Agregar Tailwind y estructura de componentes (Layout, Header, ProjectCard, FilterBar).
- [ ] Crear `src/data/projects.ts` (o `.json`) con metadatos: slug, nombre, descripción corta, tecnologías, estado, enlaces (README sección anchor), tipo de despliegue.
- [ ] Implementar build: `npm run build` genera `dist/`.
- [ ] Crear bucket S3: nombre `portfolio-denniels` (sin mayúsculas). Habilitar hosting estático.
- [ ] Subir build: `aws s3 sync dist/ s3://portfolio-denniels --delete`.
- [ ] Crear distribución CloudFront apuntando al bucket (origin access control) + cache policies.
- [ ] (Opcional) Agregar invalidation paso post-deploy.
- [ ] Añadir GitHub Action futura (pendiente) para desplegar automáticamente en push a `main`.

Alternativa posterior: AWS Amplify Hosting (simplifica CI) — evaluar tras MVP.

### Fase 2 – Backends / APIs (Serverless)
Proyectos que requieren endpoints:
- Accidentes tránsito: Lambda (ETL ligero) + S3 processed + Athena/Glue futuro + API Gateway (resúmenes agregados).
- Indicadores económicos: EventBridge → Lambda → S3 (histórico) + DynamoDB (último valor) + API Gateway lectura.
- Transporte público: Publicación de GeoJSON agregados estáticos S3 (sin backend inicial).
- IoT Jetson: Export batch a S3 + Lambda para métricas recientes.

Checklist general serverless:
- [ ] Definir API contract (input/output JSON).
- [ ] Crear carpeta `infra/lambda/<proyecto>/` con código Python (FastAPI sólo si se empaqueta en container; preferir handler simple).
- [ ] Añadir dependencias específicas en `requirements.txt` (mínimas).
- [ ] Empaquetar: zip con handler + libs vendorizadas si no son puras.
- [ ] Crear función Lambda (128–256MB, timeout ≤ 10s inicial).
- [ ] Configurar API Gateway (REST o HTTP API) + CORS.
- [ ] Pruebas locales (pytest + invocación simulada).

### Fase 3 – Datos y Persistencia
- [ ] S3 buckets por dominio de datos: `data-accidentes`, `data-indicadores`, `data-iot`.
- [ ] Convención de particionado: `s3://data-accidentes/processed/year=YYYY/month=MM/archivo.parquet`.
- [ ] Glue Crawler (cuando se requieran consultas Athena) – posponer hasta tener volumen.
- [ ] RDS (PostgreSQL t3.micro) sólo cuando se justifique análisis relacional persistente.
- [ ] Programar apagado manual cuando no se use (RDS puede detenerse hasta 7 días).

### Fase 4 – Observabilidad y Optimización
- [ ] Activar CloudWatch Logs para Lambdas y establecer retención (7–14 días inicial).
- [ ] Configurar métricas personalizadas (invocaciones, errores, duración) si escala.
- [ ] Revisar Cost Explorer semanalmente.

### Fase 5 – Automatización (CI/CD)
- [ ] GitHub Action: lint + build frontend.
- [ ] GitHub Action: deploy S3 + invalidation CloudFront (usar `aws-actions/configure-aws-credentials`).
- [ ] (Opcional) Pipeline empaquetado Lambda con hash de contenido.

### Seguridad / Buenas Prácticas
- Bloquear acceso público directo a bucket estático usando CloudFront OAC.
- Principio de privilegios mínimos en IAM (refinar después del MVP).
- No almacenar secretos en repositorio (usar Parameter Store / Secrets Manager si aparece necesidad).
- Activar MFA en cuenta root y no usar root para despliegues.

---

## 📦 Estructura Planificada Frontend (Detalle)

```text
frontend/
├── src/
│   ├── components/          # Reutilizables (ProjectCard, Tag, Layout)
│   ├── pages/               # Rutas (Home, ProyectoDetalle)
│   ├── data/                # projects.[ts|json]
│   ├── hooks/               # Hooks personalizados (useProjects, useFilters)
│   ├── styles/              # Tailwind config adicional
│   └── main.tsx             # Entry point
├── public/                  # favicon, logo, manifest
└── index.html               # Template Vite
```

MVP Features:
- Listado filtrable (estado, tecnología, tipo datos).
- Tarjetas: nombre, descripción corta, estado, tecnologías.
- Página detalle (cargará metadata extendida del JSON/TS; futura extracción parcial de README).
- Botones: Código, Docs, Dataset/Endpoint.

---

## ♻️ Estrategia de Evolución (Notebooks → Pipelines)
1. Exploración en `notebooks/` valida supuestos y define transformaciones.
2. Refactor a funciones puras en `src/` (testables, idempotentes).
3. Orquestación mínima en `pipelines/` (scripts que encadenan ingest → transform → publish). Más adelante integración con Step Functions / MWAA si escala.
4. Publicación de artefactos estables: Parquet particionado, GeoJSON, métricas agregadas en JSON.
5. Exposición (si aplica): endpoints serverless o archivos estáticos listos para consumo frontend.

## ✅ Próximos Pasos Inmediatos
1. [ ] Crear entorno virtual `.venv` y confirmar `requirements.txt` versionado.
2. [ ] Inicializar `frontend/` (scaffold Vite + React + Tailwind).
3. [ ] Crear `frontend/src/data/projects.ts` con metadata inicial de los 4 proyectos.
4. [ ] Definir plantilla estándar de README para proyectos (pendiente de creación).
5. [ ] Configurar presupuesto AWS + credenciales IAM locales.
6. [ ] Implementar build y primer despliegue manual S3 + CloudFront.
7. [ ] Diseñar esquema de particionado S3 por dominio (accidentes, indicadores, iot, transporte).

---

---

## 🤝 Colaboraciones

Este repositorio no tiene licencia. Si deseas colaborar, mejorar o contratar, puedes contactarme directamente.

---

## 📬 Contacto

Si deseas ponerte en contacto conmigo, puedes hacerlo a través de las siguientes plataformas:

<table>
  <tr>
    <td><img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/github/github-original.svg" width="24" alt="GitHub"/></td>
    <td><a href="https://github.com/Denniels">Denniels</a></td>
  </tr>
  <tr>
    <td><img src="https://cdn.jsdelivr.net/gh/devicons/devicon/icons/linkedin/linkedin-original.svg" width="24" alt="LinkedIn"/></td>
    <td><a href="https://www.linkedin.com/in/daniel-andres-mardones-sanhueza-27b73777/">Daniel Andrés Mardones Sanhueza</a></td>
  </tr>
  <tr>
    <td><img src="https://upload.wikimedia.org/wikipedia/commons/4/4e/Mail_%28iOS%29.svg" width="24" alt="Email"/></td>
    <td><a href="mailto:d.mardones.s@gmail.com">d.mardones.s@gmail.com</a></td>
  </tr>
</table>

Estoy abierto a colaboraciones, consultas y oportunidades laborales.
