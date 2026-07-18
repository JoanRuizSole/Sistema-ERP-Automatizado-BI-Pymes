# 📊 Ecosistema Financiero, Fiscal y de BI Automatizado para PYMES y Autónomos

![Banner Arquitectura](images/1_arquitectura_general.png)

## 🎯 Sobre el Proyecto
Este proyecto es una solución integral de **Automatización de Procesos de Negocio (BPA)** y **Business Intelligence (BI)** desarrollada para resolver el caos contable, la entrada manual de facturas y la incertidumbre fiscal al cierre de trimestre que sufren pequeñas empresas y trabajadores autónomos.

A diferencia de los ERPs tradicionales (altos costes de licencia, rigidez y complejas curvas de aprendizaje), este ecosistema ha sido diseñado desde cero combinando herramientas en la nube altamente escalables y accesibles: **n8n** (orquestación), **Google Sheets** (almacenamiento relacional) y **Google Looker Studio** (visualización ejecutiva).

---

## 🔗 Demo Interactiva
👉 **[Ver el Dashboard Interactivo en Vivo (Looker Studio - Acceso de Solo Lectura)](https://datastudio.google.com/reporting/136a952a-a515-4dc3-b1b4-024aaa6130a5)**

*(Nota: La demostración funciona en un entorno seguro de solo lectura con datos anonimizados para garantizar la integridad de la aplicación y el cumplimiento de privacidad RGPD).*

---

## 🔐 Nota sobre Propiedad Intelectual y Arquitectura Privada
Este repositorio actúa como un **Estudio de Caso Arquitectónico (Tech Showcase)**. 

Debido a que este sistema constituye el núcleo operativo de una solución comercial SaaS/Consultoría y para prevenir alteraciones de seguridad en las bases de datos de demostración, **los workflows de backend en n8n (.JSON) y las credenciales de enrutamiento API se mantienen en un repositorio privado**. 

A continuación, se documenta la ingeniería del dato, la lógica de automatización y la estructura de visualización implementada.

---

## 🏗️ Arquitectura del Sistema (End-to-End)

El ecosistema está construido en una arquitectura de tres capas desacopladas, garantizando modularidad, cero mantenimiento manual y procesamiento en tiempo real:

### 1. Capa de Ingesta y Lógica de Negocio (`n8n`)
El backend del proyecto reemplaza la picadura manual de datos mediante un flujo automatizado de nodos conectados en **n8n**:
- **Recepción (Trigger):** Captura de datos transaccionales de facturación a través de webhooks y formularios de entrada.
- **Procesamiento Lógico y Sanitización:** Los bloques lógicos del flujo extraen metadatos clave (CIF/NIF, Cliente/Proveedor, Concepto), calculan las bases imponibles y determinan automáticamente los porcentajes de IVA vigentes según la tipología documental.
- **Enrutamiento Condicional:** El sistema evalúa el tipo de transacción y la inyecta mediante API en las tablas normalizadas correspondientes, diferenciando ingresos de gastos de forma autónoma.

![Flujo Backend n8n](images/2_flujo_n8n.png)
*Vista general de la arquitectura del flujo lógica y de categorización implementada en el backend con n8n.*

---

### 2. Capa de Almacenamiento Relacional (`Google Sheets`)
Actúa como una base de datos ligera, en la nube y optimizada. Para evitar la degradación de rendimiento típica de hojas de cálculo saturadas, la base de datos se ha diseñado bajo modelos relacionales sencillos con tablas normalizadas (`📈 BASE - INGRESSOS` y `📊 BASE - DESPESES`), alimentadas exclusivamente a través del backend automatizado sin intervención humana en celda.

---

### 3. Capa de Analítica, BI y Fiscalidad (`Looker Studio`)
El panel de visualización interactivo está diseñado bajo una filosofía de **UI/UX Directiva y Cero Mantenimiento**: en lugar de sobrecargar la pantalla con datos irrelevantes, presenta métricas accionables listas para la toma de decisiones financieras y fiscales.

![Dashboard de Fiscalidad](images/3_dashboard_fiscalidad.png)
*Módulo fiscal automatizado con previsión de cuota de IVA trimestral y cálculo de reserva de liquidez del 25%.*

**Funcionalidades directivas clave:**
- **🏛️ Módulo de Previsión Fiscal Automática:** Cálculo inmediato en tiempo real de las cuotas de IVA trimestral a pagar o compensar (cruce de IVA repercutido vs. soportado). Incorpora el KPI de alerta de **"Colchón Fiscal"**, un algoritmo que calcula y bloquea visualmente el 25% del Beneficio Neto como provisión intocable para el pago del Impuesto de Sociedades o IRPF a final de año.
- **🏆 Ránking Pareto de Clientes (Top Tier):** Gráfico directivo de facturación acumulada que permite identificar al segundo qué clientes concentran la rentabilidad real de la empresa, aislando variaciones estacionales.
- **📈 Salud Operativa en Tiempo Real:** Seguimiento instantáneo del Beneficio Bruto Anual sin necesidad de cierres contables manuales.

![Ránking de Clientes](images/4_ranking_clientes.png)
*Identificación visual en tiempo real del Top Clientes por volumen de facturación.*

---

## 🧠 Retos Técnicos Resueltos y Decisiones de Ingeniería

Durante la construcción del ecosistema, se resolvieron importantes desafíos de modelado en Business Intelligence:

### 1. Resolución de Asincronía en Fechas mediante *Blended Data*
**El Reto:** Al combinar tablas independientes de Ingressos y Despeses en Looker Studio mediante uniones externas (*Full Outer Joins*), las diferencias en los días exactos de facturación provocaban asincronía temporal, generando el error común de "barras fantasma", duplicidades de agregación y meses desaparecidos en los gráficos combinados.
**La Solución Tecnológica:** Se descartaron las uniones por importes monetarios en crudo. En su lugar, se implementó una normalización temporal estricta mediante el uso de campos calculados con transformaciones `FORMAT_DATETIME("%Y-%m", Data)` y truncamientos (`DATE_TRUNC`). Al estandarizar las fechas en claves uniformes de *Mes/Año* y *Trimestre* desde el origen, el motor de Looker Studio logra un cruce relacional exacto y estable.

### 2. Optimización de UI/UX: Evolución hacia el "Cero Mantenimiento"
**El Reto:** Los gráficos evolutivos mes a mes con múltiples series de datos temporales suelen generar altas tasas de fallos visuales y requieren mantenimiento constante cuando las fuentes de datos sufren interrupciones puntuales en periodos vacíos.
**La Solución Tecnológica:** Se aplicó una refactorización hacia una interfaz ejecutiva de alto impacto. Se eliminaron los gráficos temporales propensos a roturas en favor de tarjetas KPI directas (Previsión Fiscal del 25%) y gráficos de agregación por entidades (Top Clientes). Esto garantiza que el cuadro de mando mantenga una disponibilidad del 100% y una lectura limpia e instantánea los 365 días del año.

---

## 🛠️ Stack Tecnológico
- **Orquestación & Backend:** `n8n` (API Routing, Webhooks, Data Parsing & Logic Automation).
- **Base de Datos Relacional:** `Google Sheets` (Tablas Normalizadas en la Nube).
- **Business Intelligence & Visualización:** `Google Looker Studio` (Blended Data, Outer Joins, Advanced Calculated Fields: `IFNULL`, `FORMAT_DATETIME`, `DATE_TRUNC`).
- **Control de Versiones & Showcase:** `Git` / `GitHub`.

---
*© 2026 - Todos los derechos de arquitectura y propiedad intelectual reservados por el autor. Para consultas sobre implementación en entorno comercial, contactar de forma directa.*
