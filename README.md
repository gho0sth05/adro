# Plataforma Empresarial de Automatización de Inventarios y Sell Out

Plataforma web de arquitectura modular y empresarial diseñada para automatizar el procesamiento de archivos masivos de **Inventario** y **Sell Out**, eliminando la manipulación manual de hojas de cálculo, estandarizando catálogos maestros de clientes y generando reportes consolidados para la toma de decisiones estratégicas en *trade marketing* y supply chain.

---

## 📋 Tabla de Contenido

- [Visión General del Proyecto](#-visión-general-del-proyecto)
- [Arquitectura de Datos por Capas](#-arquitectura-de-datos-por-capas)
- [Caso de Uso Base: Alkosto](#-caso-de-uso-base-alkosto)
- [Stack Tecnológico](#-stack-tecnológico)
- [Flujo de Procesamiento ETL](#-flujo-de-procesamiento-etl)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Guía de Instalación y Despliegue](#-guía-de-instalación-y-despliegue)
- [Hoja de Ruta (Roadmap)](#-hoja-de-ruta-roadmap)

---

## 🎯 Visión General del Proyecto

La plataforma permite a la empresa gestionar de manera unificada y escalable el flujo de información comercial proveniente de aproximadamente **11 grandes clientes retail**:

1. **Alkosto** *(Fase 1 / Implementación Base)*
2. **Crossur**
3. **Falabella**
4. **Farmatodo**
5. **Grupo Éxito**
6. **Mecanelectro**
7. **Pepe Ganga**
8. **Makro**
9. **PriceSmart**
10. **Primark**
11. **OXXO**

### Principales Problemas que Resuelve:
* **Procesamiento manual ineficiente:** Ingesta y limpieza automatizada de archivos `.xlsx` y `.csv` multi-hoja de gran volumen.
* **Desalineación de Maestros:** Homologación en tiempo real de códigos de almacén (`UNINEG` / Puntos de venta) y catálogos de productos (`EAN` / `SKU` / `REF`).
* **Pérdida de Históricos:** Almacenamiento estructurado de transacciones mensuales y cortes de stock sin sobrescribir información pasada.
* **Inconsistencia de Indicadores:** Cálculo automatizado y uniforme de **Días de Inventario (DI)**, **Meses de Inventario (MI)** y alertas de rotación/agotados.

---

## 🏗 Arquitectura de Datos por Capas

El core del sistema implementa un modelo conceptual de datos divididos en 5 capas lógicas para garantizar integridad, auditoría y trazabilidad completa:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        1. RAW DATA (Original)                          │
│     Almacenamiento de archivos intactos (Ventas e Inventario)          │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      2. STAGING DATA (Temporal)                        │
│     Lectura, parsing, limpieza de nulos y normalización inicial         │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                       3. MASTER DATA (Maestros)                        │
│    Cruce con Base de Clientes (PDVs) y SKUs (Catálogo de Productos)   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      4. HISTORICAL DATA (Histórico)                    │
│    Persistencia de evolutivos mensuales en MySQL Server 8.4 LTS        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                     5. PROCESSED DATA (Informe Final)                  │
│    Consolidado listo para consumo en Dashboard y exportación Excel     │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 🛒 Caso de Uso Base: Alkosto

La implementación inicia con la parametrización completa del flujo para **Alkosto**.

### Archivos de Entrada (Raw Data):
* **`ALKOSTO VENTAS.xlsx`**: Unidades vendidas y valor de venta neta por punto de venta y matriz de fechas/días.
* **`ALKOSTO INVENTARIO.xlsx`**: Unidades en inventario (`INV`), tránsito y días de inventario reportados por la cadena.

### Archivos Maestros (Master Data):
* **`BASE CLIENTES ALKOSTO.xlsx`**: Mapeo de unidades de negocio (`UNINEG` como `AK170`, `AKB68`, `AKVEN`), nombres ajustados de PDV, dirección, ciudad, departamento y coordenadas geográficas.
* **`SKUS ALKOSTO.xlsx`**: Mapeo de `EAN ITEM`, descripción corta, `REF`, `EAN 13`, nombre de producto extendido, embalaje por caja, unidad de venta y marca.

### Salida Consolidada (Processed Data):
* **`Sell Out-Inv ALKOSTO.xlsx`**:
  * Pestañas de detalle plano (`Base Venta`, `Base Inventario`).
  * Tablas dinámicas agregadas (`TD Venta`, `TD INV INF`).
  * Módulos ejecutivos de resumen (`Sell Out Mensual`, `Inventario por Marca`, `Inventario por PDV/SKU`, `Inventario para Mercaderistas`).

---

## 🛠 Stack Tecnológico

| Capa / Módulo | Tecnología Seleccionada | Justificación Técnica |
| :--- | :--- | :--- |
| **Base de Datos** | MySQL Server 8.4 LTS | Motor relacional robusto para manejo de históricos e índices masivos. |
| **Administración BD** | DBeaver Community | Cliente SQL estándar para gestión y monitoreo de tablas. |
| **Backend API** | Python + FastAPI | Mapeo asíncrono, velocidad de ejecución y validación de tipos mediante Pydantic. |
| **Procesamiento ETL**| Pandas + OpenPyXL | Manipulación vectorizada de DataFrames e ingesta de libros de cálculo complejos. |
| **Frontend** | React + TypeScript | Interfaz moderna, reactiva, fuertemente tipada y escalable. |
| **Diseño / Prototipado**| Figma | Sistema de diseño empresarial y maquetación de la UI. |
| **IDE & Control** | VS Code + Node.js + GitHub | Entorno estandarizado de desarrollo y control de versiones. |

---

## 🔄 Flujo de Procesamiento ETL

```mermaid
sequenceDiagram
    autonumber
    actor Usuario
    participant Frontend as React UI
    participant Backend as FastAPI Engine
    participant ETL as Pandas ETL Module
    participant DB as MySQL Server
    
    Usuario->>Frontend: Carga archivos ALKOSTO VENTAS e INVENTARIO
    Frontend->>Backend: POST /api/v1/etl/upload (archivos + cliente_id)
    Backend->>DB: Registra metadatos en RAW_FILES y guarda archivo original
    Backend->>ETL: Invoca Pipeline de Procesamiento
    ETL->>DB: Lee tablas Maestras (CLIENTES y SKUS)
    ETL->>ETL: Ejecuta cruces (Merge), limpia duplicados y calcula Días de Inventario
    ETL->>DB: Carga registros validados en STAGING y traslada a HISTORICO
    ETL->>DB: Genera vista consolidada en PROCESSED
    Backend-->>Frontend: Retorna resumen de ejecución, métricas e informe final
    Frontend-->>Usuario: Despliega Dashboard y habilita descarga de Sell Out-Inv ALKOSTO.xlsx

## 🚀 Guía de Instalación y Despliegue

### Requisitos Previos
* MySQL Server 8.4 LTS instalado en puerto `3306`.
* Python 3.10+
* Node.js v18+ y `npm` / `yarn`.

### 1. Base de Datos
1. Abre DBeaver y conéctate a tu servidor MySQL local/remoto.
2. Crea la base de datos vacía:
   ```sql
   CREATE DATABASE db_sellout_platform;
   ```
3. Ejecuta el script de inicialización de esquemas:
   ```bash
   mysql -u root -p db_sellout_platform < database/init_schema.sql
   ```

### 2. Backend (FastAPI)
```bash
cd backend
python -m venv venv
# En Windows (PowerShell):
. env\Scripts ctivate
# En Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```
* API Documentation: `http://localhost:8000/docs`

### 3. Frontend (React + TypeScript)
```bash
cd frontend
npm install
npm start
```
* Aplicación Web: `http://localhost:3000`

---
