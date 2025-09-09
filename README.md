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
├── accidentes-transito-chile/
│   ├── data/                  # Datos crudos y procesados
│   ├── notebooks/             # EDA, análisis estadístico, SQL avanzado
│   ├── scripts/               # ETL, limpieza, transformaciones
│   ├── sql/                   # Queries avanzadas
│   ├── aws_deployment/        # Infraestructura en AWS (RDS, S3, EC2)
│   ├── streamlit_app/         # Visualización interactiva
│   └── README.md              # Documentación del proyecto
├── dashboard-indicadores-economicos/
│   ├── api_sources/           # Scripts de conexión a APIs públicas
│   ├── data/                  # Datos estructurados
│   ├── notebooks/             # Análisis económico y comparativas
│   ├── fastapi_backend/       # Backend para servir datos
│   ├── streamlit_dashboard/   # Visualización en tiempo real
│   ├── aws_deployment/        # Infraestructura en AWS
│   └── README.md              # Documentación del proyecto
├── transporte-publico-santiago/
│   ├── data/                  # Logs GPS, rutas, horarios
│   ├── notebooks/             # EDA, clustering, análisis espacial
│   ├── scripts/               # ETL y limpieza
│   ├── sql/                   # Consultas con PostGIS
│   ├── streamlit_map/         # Visualización geográfica
│   ├── aws_deployment/        # PostgreSQL + PostGIS en RDS
│   └── README.md              # Documentación del proyecto
├── iot-jetson-nano/
│   ├── data_ingestion/        # Scripts de captura desde ESP32 y Arduino
│   ├── data_processing/       # Limpieza y estructuración
│   ├── database/              # Conexión a PostgreSQL local
│   ├── notebooks/             # Prototipo de análisis local
│   ├── api/                   # FastAPI para exponer datos
│   ├── tunnel/                # Configuración de Cloudflare Tunnel
│   ├── streamlit_frontend/    # Dashboard en Streamlit Cloud
│   ├── aws_sync/              # Estrategia para migrar datos a AWS
│   ├── systemd_services/      # Archivos de servicio para Jetson Nano
│   └── README.md              # Documentación del proyecto
└── README.md                  # Documentación general del portafolio
```

> 🧩 *Cada proyecto puede extenderse con nuevos directorios según se requiera: pruebas unitarias, documentación técnica, dashboards adicionales, etc.*

---

## 📊 Análisis y Arquitectura por Proyecto

### 1. `accidentes-transito-chile`
- **Objetivo**: Identificar patrones de siniestros viales y proponer medidas preventivas.
- **Datos**: Dataset público de CONASET.
- **Análisis**: EDA, correlaciones, regresión logística.
- **SQL**: Funciones de ventana para recurrencia por zona/hora.
- **Arquitectura**: PostgreSQL en RDS, Data Lake en S3, visualización en Streamlit.

---

### 2. `dashboard-indicadores-economicos`
- **Objetivo**: Visualizar y analizar indicadores como IPC, PIB, desempleo.
- **Datos**: APIs públicas del Banco Central e INE.
- **Análisis**: Comparativas históricas, correlaciones económicas.
- **SQL**: Agregaciones por periodo, normalización de unidades.
- **Arquitectura**: FastAPI + PostgreSQL + Streamlit + AWS Lambda.

---

### 3. `transporte-publico-santiago`
- **Objetivo**: Optimizar rutas y frecuencias según demanda.
- **Datos**: Logs GPS, horarios, zonas.
- **Análisis**: Clustering, densidad, rutas críticas.
- **SQL**: PostGIS para análisis espacial.
- **Arquitectura**: PostgreSQL + PostGIS en RDS, visualización geográfica.

---

### 4. `iot-jetson-nano`
- **Objetivo**: Analizar datos físicos de sensores distribuidos.
- **Datos**: Temperatura y luz desde ESP32 y Arduino.
- **Análisis**: Series temporales, correlaciones, anomalías.
- **SQL**: Consultas estructuradas en PostgreSQL local.
- **Arquitectura**: Jetson Nano + PostgreSQL + API + Cloudflare Tunnel + Streamlit Cloud.
- **Desarrollo local**:
  - Crear conexión desde tu PC a la base PostgreSQL de la Jetson.
  - Extraer datos para prototipar en notebooks.
  - Diseñar estrategia para migrar datos a AWS (RDS o S3).

---

## 🧰 Tecnologías Utilizadas

- **Lenguajes**: Python, SQL
- **Bases de datos**: PostgreSQL, PostGIS, DynamoDB (si aplica)
- **Cloud**: AWS (RDS, S3, Lambda, EC2), Cloudflare Tunnel
- **Frameworks**: FastAPI, Streamlit
- **Herramientas**: Git, GitHub, Docker, systemd

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
