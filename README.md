Markdown# 🛡️ Proyecto Final: CI/CD Databricks - Monitoreo y Prevención de Fraude Bancario

![Azure Databricks](https://img.shields.io/badge/Azure_Databricks-FF3621?style=for-the-badge&logo=Databricks&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![Apache Spark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)

---

## 📌 1. Descripción del Proyecto

Este repositorio contiene la implementación end-to-end de un pipeline de datos en arquitectura **Lakehouse** construido sobre **Azure Databricks** y desplegado de forma automatizada mediante integración y despliegue continuo (**CI/CD**) con **GitHub Actions**.

El objetivo del proyecto es procesar transacciones bancarias en tiempo real/lote, calcular scores y niveles de riesgo de fraude, aplicar reglas de gobernanza mediante **Unity Catalog** y consumir los resultados en un **Lakeview Dashboard** interactivo.

---

## 🏗️ 2. Arquitectura de la Solución

El pipeline sigue el patrón de diseño **Arquitectura Medallion** (Bronze $\rightarrow$ Silver $\rightarrow$ Gold) integrado con **Unity Catalog** (`catalog_au`):

```text
                               ┌────────────────────────┐
                               │   Fuentes Data Lake    │
                               └───────────┬────────────┘
                                           │
                                           ▼
                               ┌────────────────────────┐
                               │  1. Prep. Ambiente     │
                               └───────────┬────────────┘
                                           │
                    ┌──────────────────────┴──────────────────────┐
                    ▼                                             ▼
       ┌────────────────────────┐                    ┌────────────────────────┐
       │ 2a. Ingest Clientes    │                    │ 2b. Ingest Transacciones│
       └───────────┬────────────┘                    └───────────┬────────────┘
                   │                                             │
                   └──────────────────────┬──────────────────────┘
                                          ▼
                               ┌────────────────────────┐
                               │   Capa Bronze (Raw)    │
                               └───────────┬────────────┘
                                           │
                                           ▼
                               ┌────────────────────────┐
                               │  3. Transform (Silver) │
                               └───────────┬────────────┘
                                           │
                                           ▼
                               ┌────────────────────────┐
                               │   4. Load (Gold)       │
                               └───────────┬────────────┘
                                           │
                                           ▼
                               ┌────────────────────────┐
                               │ 5. Grants & Gobernanza │
                               └───────────┬────────────┘
                                           │
                                           ▼
                               ┌────────────────────────┐
                               │  Lakeview Dashboard    │
                               └────────────────────────┘
🛠️ 3. Tecnologías UtilizadasNube: AzurePlataforma Data/AI: Azure Databricks (Unity Catalog, Databricks Workflows, Lakeview Dashboards)Orquestación & CI/CD: GitHub Actions (Databricks REST API v2.0 / v2.1)Procesamiento de Datos: PySpark / Spark SQLLenguajes: Python, SQL, YAML, Bash📂 4. Estructura del RepositorioPlaintextCICD-FINALPROJECT-RC-DATABRICKS/
│
├── .github/
│   └── workflows/
│       └── deploy-notebook.yml       # Pipeline CI/CD automatizado en GitHub Actions
│
├── proceso/
│   ├── 1.Preparacion_Ambiente.sql    # Configuración e infraestructura base
│   ├── 2.Ingest_clientes_data.py     # Ingesta de datos de clientes (Bronze)
│   ├── 2.Ingest_Transacciones.py     # Ingesta de transacciones (Bronze)
│   ├── 3.Transform.py                # Limpieza y enriquecimiento de datos (Silver)
│   ├── 4.Load.py                     # Cálculo de reglas de riesgo y persistencia (Gold)
│   └── 5.Grants_Medallion.sql        # Control de accesos y permisos en Unity Catalog
│
├── Monitoreo y Prevención de Fraude Bancario.lvdash.json  # Definición Lakeview Dashboard
└── README.md                         # Documentación del proyecto
🔄 5. Flujo del Pipeline CI/CD (GitHub Actions)El archivo .github/workflows/deploy-notebook.yml ejecuta las siguientes etapas automatizadas en cada push a la rama main:Checkout & Setup: Descarga el código e instala las dependencias (jq, curl).Export de Notebooks: Exporta dinámicamente los cuadernos desde el Workspace de Desarrollo (DATABRICKS_ORIGIN_HOST).Deploy de Notebooks: Importa los cuadernos en formato SOURCE hacia la ruta destino /py/scripts/yml en Producción (DATABRICKS_DEST_HOST).Verificación & Limpieza: Busca si existe una versión previa del Workflow WF_ADB y la elimina.Detección de Cluster: Obtiene el cluster_id del cluster existente cluster_SD.Creación del Databricks Workflow: Genera mediante la API /api/2.1/jobs/create el Workflow multitarea con sus parámetros y dependencias.Ejecución y Monitoreo: Inicia el trabajo mediante /api/2.1/jobs/run-now y realiza monitoreo continuo (polling) hasta obtener el estado TERMINATED (SUCCESS).
⚡6. Workflow WF_ADB en DatabricksEl trabajo multitarea orquestado incluye:TareaNotebookDescripciónParámetrosPreparacion_Ambiente1.Preparacion_AmbienteInicialización de storage y external locationssrcData1, srcData2Ingest_clientes_data2.Ingest_clientes_dataCarga de clientes a capa Bronzecontainer, catalogo, esquema, srcData1Ingest_Transacciones2.Ingest_TransaccionesCarga de transacciones a capa Bronzecontainer, catalogo, esquema, srcData2Transform3.TransformUnificación, deduplicación y reglas Silvercatalogo, esquema_source, esquema_sinkLoad4.LoadGeneración de capa Gold con nivel/score de riesgocatalogo, esquema_source, esquema_sinkGrants5.Grants_MedallionAsignación de privilegios SELECT/USE CATALOGN/A

📊 7. Dashboard de Analítica de Fraude (Databricks Lakeview)El dashboard consume la tabla/vista unificada catalog_au.golden.golden_analisis_fraude (fraude_main):  
🎯 KPIs Principales (Counters)Total Transacciones Procesadas: Conteo global de operaciones.  Monto Total Procesado (S/): Volumen total financiero analizado.  Transacciones en Alerta Alta: Total de operaciones marcadas para Revisión Inmediata.  Monto Total en Riesgo Alto (S/): Suma total de dinero expuesto en nivel de riesgo Alto. 

📈 Visualizaciones ClaveDistribución por Nivel de Riesgo: Anillo (Pie Chart) con la proporción Bajo, Medio y Alto.  Monto Acumulado por Banco y Nivel de Riesgo: Barras apiladas (Stacked Bar Chart).  Top Departamentos en Revisión Inmediata: Barras horizontales por región geográfica.  Monto vs Score de Riesgo por Prioridad: Dispersión (Scatter Plot) de volumen operado vs. puntaje.  Transacciones por Banco y Estado de Fraude: Comparativo de severidad por entidad financiera.  % de Fraude por Banco: Medida de efectividad comparada contra la línea promedio nacional. 

🎛️ Filtros Globales DinámicosBanco (:banco)  Departamento (:departamento)  Nivel de Riesgo (:nivel_riesgo)  Prioridad de Atención (:prioridad)

🚀 8. Instrucciones de Configuración y DesplieguePrerrequisitosAzure Databricks Workspace con Unity Catalog habilitado.Cluster activo en el workspace destino llamado cluster_SD.