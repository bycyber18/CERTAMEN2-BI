# Certamen 2 - Inteligencia de Negocios

## 1. Título del Proyecto y Caso

**Caso 3: Kent Foods**

**Integrantes:** Álvaro Ramírez Silva; Moisés Larenas; Ignacio Miranda; Inteligencia de Negocios paralelo 300.

> **Alcance de esta versión:** el script SQL crea el modelo dimensional descrito abajo. El proyecto SSIS está en estado de plantilla: el paquete incluido no contiene tareas, conexiones ni flujo de datos. Por ello, los KPI se describen como indicadores que se pueden calcular a partir de las métricas del modelo, y el ETL aún requiere implementación.

## 2. Descripción del Problema y Objetivo

El caso Kent Foods aborda el análisis de las ventas registradas en órdenes de alimentos. El objetivo de la Fase 1 es establecer una base analítica que permita consultar las ventas por fecha, cliente, producto, empleado y transportista, además de disponer de métricas de cantidades, importes, descuentos y despacho.

El artefacto implementado en esta fase es `KentFoods_DW`, una base de datos dimensional con esquema estrella. El repositorio todavía no incluye datos cargados ni un proceso ETL funcional; la base queda lista para recibirlos cuando se complete la integración con el origen.

## 3. KPI Definidos

El DDL crea métricas en `H_VENTAS`, pero no define consultas, vistas ni cálculos de KPI. Los siguientes indicadores son derivables de esas columnas y se proponen para el análisis; sus fórmulas y justificaciones deben validarse con el equipo y con las reglas del caso.

| Indicador | Cálculo propuesto | Utilidad para la toma de decisiones |
|---|---|---|
| Venta bruta | Suma de `VENTA_BRUTA` | Permite comparar el valor de las ventas antes de descuentos. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |
| Venta neta | Suma de `VENTA_NETA` | Permite analizar el importe vendido después de aplicar descuentos. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |
| Descuento otorgado | Suma de `VENTA_BRUTA - VENTA_NETA` | Ayuda a cuantificar el efecto de los descuentos sobre las ventas. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |
| Unidades vendidas | Suma de `CANTIDAD` | Facilita comparar el volumen vendido por producto, categoría, cliente y período. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |
| Órdenes procesadas | Conteo distinto de `ID_ORDEN` | Permite seguir la cantidad de órdenes sin confundirla con el número de líneas de detalle. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |
| Tiempo promedio de envío | Promedio de `DIAS_ENVIO`, excluyendo valores `NULL` | Permite comparar el tiempo de despacho por transportista, período y otros atributos disponibles. [VALIDAR JUSTIFICACIÓN CON EL EQUIPO] |

Las expresiones anteriores son propuestas analíticas, no medidas implementadas en el repositorio. La fórmula de `VENTA_BRUTA`, la interpretación/unidad de `DESCUENTO` y la derivación exacta de `DIAS_ENVIO` deben confirmarse al implementar el ETL.

## 4. Modelo Dimensional (Metodología Kimball)

- **Proceso de negocio seleccionado:** ventas de órdenes de alimentos de Kent Foods.
- **Granularidad:** una fila por línea de detalle de orden, identificada por la combinación `OrdenID + ProductoID`. La clave primaria de `H_VENTAS` es compuesta por `ID_ORDEN` e `ID_PRODUCTO`.
- **Dimensiones elegidas:**
  - `D_TIEMPO`: fecha, día, mes, nombre del mes, trimestre y año. El DDL no especifica qué campo de fecha del origen alimenta `ID_TIEMPO`; [CONFIRMAR FECHA DE NEGOCIO, POR EJEMPLO FECHA DE ORDEN].
  - `D_CLIENTE`: código de origen, empresa, contacto, cargo, ciudad y país. `ID_CLIENTE` se documenta como clave entera generada en el ETL.
  - `D_PRODUCTO`: producto, cantidad por unidad, categoría y proveedor, incluyendo identificadores y país del proveedor.
  - `D_EMPLEADO`: nombre, apellido, nombre completo, cargo, fechas de nacimiento y contratación, ciudad y país.
  - `D_TRANSPORTISTA`: nombre del transportista y teléfono.
- **Tabla de hechos y métricas:** `H_VENTAS` relaciona las dimensiones con el identificador de orden y producto. Sus métricas son `PRECIO_UNITARIO`, `CANTIDAD`, `DESCUENTO`, `VENTA_BRUTA` y `VENTA_NETA`; `DIAS_ENVIO` es una métrica de duración que admite `NULL` cuando la orden aún no tiene fecha de envío. Las claves foráneas dimensionales de cliente, empleado, transportista y tiempo también admiten `NULL`.

El modelo se crea en `database/scripts/01_crear_modelo_dimensional.sql`. El script crea la base `KentFoods_DW` si no existe y elimina/recrea las tablas del esquema antes de volver a definirlas.

## 5. Proceso ETL

El proyecto SSIS incluido (`etl/ETL_KentFoods`) no implementa todavía las fases de extracción, transformación ni carga. `ETL_KentFoods.dtsx` contiene un paquete sin ejecutables; el proyecto no declara administradores de conexión, orígenes, destinos, parámetros ni transformaciones. Por lo tanto, no es posible atribuirle actualmente reglas de limpieza o validación ni afirmar que carga `KentFoods_DW`.

Al completar el ETL, el flujo deberá documentarse y configurarse para:

1. **Extraer:** conectarse al origen relacional que proporcione las órdenes, sus detalles y las entidades relacionadas. [INDICAR BASE DE DATOS, SERVIDOR Y TABLAS/VISTAS DE ORIGEN].
2. **Transformar y validar:** revisar nulos y tipos de datos; normalizar cadenas; validar claves y relaciones; generar las claves enteras de cliente; derivar atributos de tiempo, venta bruta, venta neta y días de envío con las reglas acordadas; y decidir cómo registrar filas inválidas.
3. **Cargar:** poblar primero las dimensiones y después `H_VENTAS`, respetando claves primarias y foráneas. Definir el tratamiento de dimensiones existentes, duplicados y ejecuciones repetidas antes de habilitar cargas recurrentes.

Estos puntos son el diseño pendiente del flujo y no describen transformaciones ya presentes en el paquete.

## 6. Tecnologías Utilizadas

- **Microsoft SQL Server:** base de datos destino `KentFoods_DW`; el proyecto SSIS declara como versión de destino `SQLServer2025`.
- **Visual Studio 2022 (solución formato 17):** entorno asociado a `ETL_KentFoods.sln` y `ETL_KentFoods.dtproj`.
- **SQL Server Integration Services (SSIS):** proyecto de implementación del ETL, actualmente en estado de plantilla.
- **Git y GitHub:** control de versiones y alojamiento del repositorio, evidenciados por el historial y las ramas remotas.
- **SQL Server Management Studio (SSMS):** herramienta sugerida para ejecutar y revisar el script SQL; no se registra como dependencia del proyecto.

## 7. Estructura del Repositorio

```text
CERTAMEN2-BI/
├── database/
│   ├── backup/                 # Reservado para respaldos; actualmente contiene .gitkeep
│   └── scripts/
│       └── 01_crear_modelo_dimensional.sql  # Crea KentFoods_DW y el esquema estrella
├── docs/
│   ├── modelo-dimensional/     # Reservado para documentación del modelo; actualmente .gitkeep
│   └── presentacion/           # Reservado para material de presentación; actualmente .gitkeep
├── etl/
│   ├── ETL_KentFoods/              # Solución/proyecto SSIS y paquete de plantilla
│   └── proyecto-ssis/          # Reservado; actualmente contiene .gitkeep
├── olap/                       # Reservado para artefactos OLAP; actualmente .gitkeep
├── powerbi/                    # Reservado para la Fase 2; actualmente .gitkeep
├── .gitignore
└── README.md
```

## 8. Instrucciones de Ejecución

### Crear el modelo dimensional

1. Contar con acceso a una instancia de Microsoft SQL Server con permisos para crear bases de datos y tablas.
2. Abrir `database/scripts/01_crear_modelo_dimensional.sql` en SSMS o en una herramienta compatible.
3. Revisar la instancia seleccionada y ejecutar el script completo.
4. Confirmar la creación de `KentFoods_DW` y de las tablas `D_TIEMPO`, `D_CLIENTE`, `D_PRODUCTO`, `D_EMPLEADO`, `D_TRANSPORTISTA` y `H_VENTAS`.

**Precaución:** el script usa `DROP TABLE IF EXISTS` para eliminar las tablas del modelo antes de crearlas de nuevo. Una nueva ejecución elimina los datos que esas tablas contengan. No ejecutarlo sobre datos que se necesiten conservar sin respaldo previo.

### Abrir y ejecutar el proyecto SSIS

1. Usar Visual Studio 2022 con la extensión **SQL Server Integration Services Projects** instalada.
2. Abrir `etl/ETL_KentFoods/ETL_KentFoods.sln` y revisar la configuración `Development` y la versión de destino de SQL Server.
3. Antes de ejecutar una carga, completar el paquete con conexiones, tareas de flujo de datos, transformaciones, validaciones y destinos; configurar también los accesos al origen y a `KentFoods_DW`.
4. Cuando el flujo esté implementado, seleccionar el paquete y usar **Start** en Visual Studio. Verificar el resultado y los errores en la ventana de ejecución de SSIS.

**Estado actual:** el paquete versionado está vacío, por lo que abrir o iniciar la solución no extrae, transforma ni carga datos. El nombre `ETL_KentFoods` del proyecto difiere del caso `Kent Foods`; [CONFIRMAR SI ES UN NOMBRE PROVISIONAL O SI DEBE RENOMBRARSE].

## 9. Distribución de Aportes

La siguiente atribución se basa únicamente en los commits disponibles en el historial de Git. La identidad aparece como `Alvaro` en los commits de trabajo y como `AlvaroRamirezSilva` en el commit de merge; se presenta unificada como Álvaro Ramírez Silva. El historial no permite confirmar aportes realizados fuera de Git.

| Integrante | Aportes respaldados por commits |
|---|---|
| Álvaro Ramírez Silva | Estructura inicial del repositorio y `.gitignore`; incorporación de la carpeta reservada para Power BI (Fase 2); creación del script SQL del modelo dimensional `KentFoods_DW`. |
| [INSERTAR NOMBRE] | [INSERTAR RESPONSABILIDADES Y/O COMMITS VERIFICABLES]. |
| [INSERTAR NOMBRE] | [INSERTAR RESPONSABILIDADES Y/O COMMITS VERIFICABLES]. |

**Nota:** verificar con el equipo los nombres oficiales de todos los integrantes y completar las contribuciones que no estén representadas en el historial actual.
