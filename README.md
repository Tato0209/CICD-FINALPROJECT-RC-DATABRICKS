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