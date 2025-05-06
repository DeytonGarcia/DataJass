# 📊 Consulta SQL para Determinar Meses Pagados y Adeudados por Usuario

Este script en SQL (BigQuery) tiene como objetivo identificar para cada usuario registrado en la tabla `users` de la base de datos `jass-inn.data_2025`, cuál fue el último mes que pagó su cuota y qué meses tiene adeudados hasta el mes actual. Esta información es clave para monitorear pagos, generar reportes administrativos, y tomar decisiones respecto a la gestión de usuarios de servicios comunitarios como agua o mantenimiento.

---
## 📝 ¿Qué hace este código?

**Paso 1 (`WITH datos`)**:  
Extrae el primer mes con estado `'P'` (pagado) desde diciembre hacia enero, utilizando `UNNEST` y `STRUCT` para mapear cada mes con su estado. Esto se hace para identificar el último mes que fue pagado por el usuario.

**Paso 2 (consulta principal):**

- Muestra los campos personales: nombre, DNI, zona.  
- Muestra el último mes pagado (o `"NINGUNO"` si no hay ninguno).  
- Determina el mes actual con `EXTRACT(MONTH FROM CURRENT_DATE())`.  
- Construye una lista de meses adeudados desde el mes siguiente al último pagado hasta el mes actual, en orden, separados por guiones.

---

## 📌 Uso práctico

Este reporte puede utilizarse para:

- 🧾 Notificar deudas pendientes mes a mes.  
- 📈 Visualizar el comportamiento de pago por zonas.  
- 📬 Automatizar mensajes de cobro.  
- 🔍 Hacer auditorías o reportes mensuales.

---

## 💻 Código SQL Explicado

```sql
WITH datos AS (
  SELECT 
    NOMBRES_Y_APELLIDOS,
    DNI,
    ZONA,
    (
      SELECT mes
      FROM UNNEST([
        STRUCT(DIC AS estado, 'DICIEMBRE' AS mes),
        STRUCT(NOV AS estado, 'NOVIEMBRE'),
        STRUCT(OCT AS estado, 'OCTUBRE'),
        STRUCT(SEP AS estado, 'SEPTIEMBRE'),
        STRUCT(AGO AS estado, 'AGOSTO'),
        STRUCT(JUL AS estado, 'JULIO'),
        STRUCT(JUN AS estado, 'JUNIO'),
        STRUCT(MAY AS estado, 'MAYO'),
        STRUCT(ABR AS estado, 'ABRIL'),
        STRUCT(MAR AS estado, 'MARZO'),
        STRUCT(FEB AS estado, 'FEBRERO'),
        STRUCT(ENE AS estado, 'ENERO')
      ])
      WHERE estado = 'P'
      LIMIT 1
    ) AS ultimo_mes_pagado_nombre
  FROM 
    jass-inn.data_2025.users
)

SELECT 
  NOMBRES_Y_APELLIDOS,
  DNI,
  ZONA,
  COALESCE(ultimo_mes_pagado_nombre, 'NINGUNO') AS ultimo_mes_pagado,
  
  CASE EXTRACT(MONTH FROM CURRENT_DATE())
    WHEN 1 THEN 'ENERO'
    WHEN 2 THEN 'FEBRERO'
    WHEN 3 THEN 'MARZO'
    WHEN 4 THEN 'ABRIL'
    WHEN 5 THEN 'MAYO'
    WHEN 6 THEN 'JUNIO'
    WHEN 7 THEN 'JULIO'
    WHEN 8 THEN 'AGOSTO'
    WHEN 9 THEN 'SEPTIEMBRE'
    WHEN 10 THEN 'OCTUBRE'
    WHEN 11 THEN 'NOVIEMBRE'
    WHEN 12 THEN 'DICIEMBRE'
  END AS mes_actual,
  
  ARRAY_TO_STRING(
    ARRAY(
      SELECT mes_nombre
      FROM UNNEST([
        STRUCT(1 AS mes_num, 'ENERO' AS mes_nombre),
        STRUCT(2 AS mes_num, 'FEBRERO'),
        STRUCT(3 AS mes_num, 'MARZO'),
        STRUCT(4 AS mes_num, 'ABRIL'),
        STRUCT(5 AS mes_num, 'MAYO'),
        STRUCT(6 AS mes_num, 'JUNIO'),
        STRUCT(7 AS mes_num, 'JULIO'),
        STRUCT(8 AS mes_num, 'AGOSTO'),
        STRUCT(9 AS mes_num, 'SEPTIEMBRE'),
        STRUCT(10 AS mes_num, 'OCTUBRE'),
        STRUCT(11 AS mes_num, 'NOVIEMBRE'),
        STRUCT(12 AS mes_num, 'DICIEMBRE')
      ]) 
      WHERE mes_num > COALESCE(
        CASE ultimo_mes_pagado_nombre
          WHEN 'ENERO' THEN 1
          WHEN 'FEBRERO' THEN 2
          WHEN 'MARZO' THEN 3
          WHEN 'ABRIL' THEN 4
          WHEN 'MAYO' THEN 5
          WHEN 'JUNIO' THEN 6
          WHEN 'JULIO' THEN 7
          WHEN 'AGOSTO' THEN 8
          WHEN 'SEPTIEMBRE' THEN 9
          WHEN 'OCTUBRE' THEN 10
          WHEN 'NOVIEMBRE' THEN 11
          WHEN 'DICIEMBRE' THEN 12
          ELSE 0
        END
      )
      AND mes_num <= EXTRACT(MONTH FROM CURRENT_DATE())
      ORDER BY mes_num
    ), ' - '
  ) AS meses_adeudados_lista

FROM datos
LIMIT 100;

