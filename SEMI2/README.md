# Práctica 1: Proceso ETL de la empresa SG-FOOD

**Seminario de Sistemas 2 - Universidad de San Carlos de Guatemala**

Este proyecto implementa un flujo completo de **Extracción, Transformación y Carga (ETL)** para la empresa SG-FOOD, migrando datos transaccionales de ventas y compras desde archivos planos (CSV) hacia un Data Warehouse modelado bajo un esquema de Constelación (Galaxy Schema).

---

## 1. Descripción del Proceso ETL Realizado

El proceso se ha desarrollado utilizando **Python (Pandas)** para la manipulación de datos y **SQL** para la estructura de almacenamiento.

### FASE 1: Extracción (Extract)

- **Fuentes de Datos:** Se cargaron dos archivos CSV: `compras.csv` y `ventas.csv`.
- **Codificación:** Se utilizó codificación `utf-8` para manejar correctamente caracteres especiales del idioma español.

### FASE 2: Transformación (Transform)

Esta fue la etapa más crítica para asegurar la calidad de los datos:

1.  **Limpieza de Fechas (Data Cleaning):**
    - Se detectaron y corrigieron errores de tipografía en el campo `Fecha` (ej. 'Z' en lugar de '2', 'O' en lugar de '0').
    - Se estandarizó el formato a `DD/MM/YYYY` y se generó una llave subrogada numérica (`id_fecha` = YYYYMMDD) para optimizar las consultas.
2.  **Corrección de Integridad de Datos:**
    - **Valores Negativos:** Se detectaron precios y costos con signo negativo. Se aplicó una transformación de **valor absoluto (`abs()`)** para corregir esta inconsistencia.
3.  **Normalización de Dimensiones:**
    - Se unificaron las tablas (`Producto` y `Sucursal`) extrayendo valores únicos de ambos archivos (compras y ventas) para evitar duplicados.
    - Se separaron atributos descriptivos para crear las dimensiones de `Cliente`, `Vendedor` y `Proveedor`.

### FASE 3: Carga (Load)

- Los datos transformados se cargan en un modelo relacional dimensional.
- En el entorno de pruebas (Python), se utilizó **SQLite** en memoria.
- Para el despliegue final, se generó el script DDL compatible con **SQL Server**.

---

## 2. Elección del Modelo Empresarial Implementado

### Tipo de Modelo: Esquema de Constelación

Se seleccionó un esquema de constelación, una arquitectura avanzada de Data Warehouse que permite tener **múltiples tablas de hechos** compartiendo **dimensiones conformadas**.

### Justificación

La elección responde a las necesidades específicas de SG-FOOD:

1.  **Procesos de Negocio Heterogéneos:** Al tener dos procesos opuestos (Ventas = Ingresos, Compras = Egresos), el esquema de constelación permite mantenerlos en tablas de hechos separadas (`fac_ventas` y `fac_compras`) sin mezclar métricas incompatibles, manteniendo la claridad del modelo.
2.  **Análisis Cruzado:** El requerimiento de analizar márgenes (Precio Venta vs. Costo Compra) se resuelve eficientemente gracias a las dimensiones compartidas (`dim_producto` y `dim_fecha`), que actúan como puentes entre las dos tablas de hechos.
3.  **Escalabilidad:** Este modelo permite agregar futuros procesos (ej. Inventario, Nómina) añadiendo nuevas tablas de hechos sin romper la estructura existente.

---

## 3. Detalle de las Características y Tablas

El modelo consta de **2 Tablas de Hechos** y **6 Tablas de Dimensiones**.

### Definición de Tablas

#### Dimensiones (Catálogos)

- **`dim_fecha`**: Calendario maestro (Año, Mes, Trimestre). _Compartida._
- **`dim_producto`**: Catálogo unificado de productos. _Compartida._
- **`dim_sucursal`**: Ubicaciones geográficas (Región, Departamento). _Compartida._
- **`dim_cliente`**: Información de clientes (Tipo: Mayorista/Minorista).
- **`dim_vendedor`**: Fuerza de ventas.
- **`dim_proveedor`**: Suministradores de mercadería.

#### Hechos (Transacciones)

- **`fac_ventas`**:
  - Registra cada venta detallada.
  - **Métricas:** `unidades`, `precioUnitario`, `total_venta`.
- **`fac_compras`**:
  - Registra cada adquisición de inventario.
  - **Métricas:** `unidades`, `costoUnitario`, `total_compra`.

---

## 4. Resultados de Consultas y Pruebas

Se ejecutaron consultas SQL sobre el modelo cargado para validar la integridad de los datos y responder a las preguntas de negocio:

### Consulta 1: Total de compras y ventas por año

- **Objetivo:** Mostrar total de compras y ventas por año.
- **Resultado:** Se validó que las fechas se agruparon correctamente por año (2018-2024), demostrando que la dimensión `dim_fecha` funciona correctamente.

### Consulta 2: Productos con pérdida (Precio Venta < Costo Compra)

- **Objetivo:** Identificar productos donde `Promedio Precio Venta < Promedio Costo Compra`.
- **Resultado:** **0 registros devueltos.**
- **Interpretación:** Tras el análisis de datos, se confirmó que **todos los productos tienen un margen de ganancia positivo**. Ningún producto se vende, en promedio, más barato de lo que se compra. La limpieza de valores negativos (abs) aseguró que este resultado sea real y no por error de datos.

### Consulta 3: Top 5 productos más vendidos por unidades

- **Objetivo:** Productos más vendidos por unidades.
- **Resultado:** Se identificaron exitosamente los productos con mayor rotación, validando la relación entre `fac_ventas` y `dim_producto`.

### Consulta 4: Ingresos por región y año

- **Objetivo:** Ingresos por Región y Año.
- **Resultado:** La consulta cruzó correctamente `fac_ventas` con `dim_sucursal`, permitiendo desgloses por "Metropolitana", "Norte", etc.

### Consulta 5: Proveedores con mayor volumen de compras

- **Objetivo:** Identificar a los proveedores a quienes se les compran más unidades y mayor monto de inversión.
- **Resultado:** Se obtuvo el listado de los 5 proveedores principales, ordenados por volumen de compra, validando la integración correcta de la tabla `fac_compras` con `dim_proveedor`.

---

## Instrucciones de Ejecución

1.  Asegurar que los archivos `compras.csv` y `ventas.csv` estén en la carpeta raíz junto al script o notebook.
2.  Ejecutar el script principal de Python (o todas las celdas del Notebook `.ipynb`) para realizar el proceso ETL.
3.  El sistema generará las tablas en memoria y mostrará los resultados de las consultas directamente en la consola o salida del notebook.
