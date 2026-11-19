# Evidencia 15 — Cronología y respuestas a Dirección

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 17


## Cómo sacar los datos

```sql
SELECT num, left(linea, 200) AS linea FROM dba.bitacora
WHERE linea ~ '(parameter|connection authorized|FATAL|ERROR|statement:)'
ORDER BY num;

-- Eventos de la tabla de auditoría
SELECT id_evento, momento, usuario_sesion, aplicacion, ip_cliente, operacion, id_cliente
FROM auditoria.cambios_clientes ORDER BY id_evento;
```

## Cronología

| # | Fecha y hora (CST) | Usuario | Cliente | Tipo de evento | Qué ocurrió | ¿Exitoso? | Fuente |
|---|---|---|---|---|---|---|---|
| 1 | 2026-11-17 19:00:17.627 | postgres | N/A | Configuración | ALTER SYSTEM SET log_connections = on (pgAdmin 4 - CONN:6989817, desde ::1) | Sí | postgresql-2026-11-17_000000.log, línea 40 |
| 2 | 2026-11-18 14:51:09.967 | app_ventas | N/A | Acceso | Intento de conexión desde 127.0.0.1 con contraseña incorrecta (se repite a las 14:51:09.994) | No | bitácora 17 y 20 |
| 3 | 2026-11-18 14:51:16.125 | usuario_inexistente | N/A | Acceso | Intento de conexión con un rol que no existe, desde 127.0.0.1 (se repite a las 14:51:16.153) | No | bitácora 23 y 27 |
| 4 | 2026-11-18 14:51:20.709 | app_ventas | N/A | Acceso | Conexión exitosa (app-ventas-web, 127.0.0.1); desconexión a las 14:51:20.752 (0.067 s) | Sí | bitácora 32 y 33 |
| 5 | 2026-11-18 15:06:15.141 | postgres | N/A | Privilegio | REVOKE UPDATE ON clientes FROM soporte_ana; a las 15:06:18 GRANT UPDATE ON clientes TO app_ventas | Sí | bitácora 40 y 41 |
| 6 | 2026-11-18 15:06:20.143 | postgres | N/A | Estructura | ALTER TABLE clientes ADD COLUMN segmento | Sí | bitácora 42 |
| 7 | 2026-11-18 15:15:33.195 | app_ventas | id 3 | Datos | UPDATE del teléfono a 6679990000 (appventas-web, 127.0.0.1) | Sí | bitácora 76 |
| 8 | 2026-11-18 15:18:14.683 | postgres | N/A | Configuración | ALTER SYSTEM SET log_statement = 'mod'; se repite a las 15:20:58.731 | Sí | bitácora 80 y 81 |
| 9 | 2026-11-18 15:30:41.805 | soporte_ana | id 5 | Error | DELETE rechazado: permission denied for table clientes | No | bitácora 125 y 126 |
| 10 | 2026-11-18 15:59:34.955 | app_ventas | id 3 | Datos | UPDATE del RFC a GAMA780912ZZZ; la auditoría conserva rfc_antes GAMA780912XYZ | Sí | bitácora 266; id_evento 1 |
| 11 | 2026-11-18 16:23:55.822 | postgres | id 6 | Datos | INSERT y DELETE de Pedro Ruiz | Sí | bitácora 333; id_evento 3 y 4 |
| 12 | 2026-11-18 16:34:43.595 | app_ventas | N/A | Estructura | Intento de DISABLE TRIGGER rechazado (must be owner of table clientes), tras un DELETE rechazado sobre auditoria.cambios_clientes | No | bitácora 372 a 376 |
| 13 | 2026-11-18 16:35:05.178 | postgres | N/A | Estructura | DISABLE TRIGGER trg_auditoria_clientes | Sí | bitácora 379 |
| 14 | 2026-11-18 16:35:07.084 | postgres | id 3 | Datos | UPDATE del RFC a GAMA780912GHI con el disparador desactivado; no generó evento de auditoría | Sí | bitácora 380 (sin id_evento) |
| 15 | 2026-11-18 16:35:09.103 | postgres | N/A | Estructura | ENABLE TRIGGER trg_auditoria_clientes | Sí | bitácora 381 |
| 16 | 2026-11-18 16:40:54.821 | postgres | ids 1 a 5 | Datos | UPDATE masivo (Actividad 15): 5 filas auditadas; se repite a las 16:42:21.160 | Sí | id_evento 7 a 11 y 12 a 16 |
| 17 | 2026-11-18 16:49:37.446 a 16:51:27.235 | postgres | N/A | Acceso | 18 intentos de conexión fallidos desde ::1 (contraseña incorrecta) | No | bitácora 424 a 478 |

| archivo | num | linea |
|---|---|---|
| postgresql-2026-11-17_000000.log | 40 | 2026-11-17 19:00:17.627 CST [12140] user=postgres db=comercial app=pgAdmin 4 - CONN:6989817 client=::1 LOG:  statement: ALTER SYSTEM SET log_connections = on; |
| postgresql-2026-11-18_000000.log | 80 | 2026-11-18 15:18:14.683 CST [35204] user=postgres db=comercial app=pgAdmin 4 - CONN:7734158 client=::1 LOG:  statement: ALTER SYSTEM SET log_statement = 'mod'; |
| postgresql-2026-11-18_000000.log | 81 | 2026-11-18 15:20:58.731 CST [35204] user=postgres db=comercial app=pgAdmin 4 - CONN:7734158 client=::1 LOG:  statement: ALTER SYSTEM SET log_statement = 'mod'; |


## Respuestas a Dirección (solo con la evidencia)

**1. ¿Quién cambió el RFC?**
- **Actividad 3 (primer cambio, a `...XXX`):** No hay evidencia. En ese momento el servidor no registraba conexiones ni sentencias (Actividad 3); el cambio no tiene hora ni usuario demostrable. Es un hueco.
- **Actividad 7 (a `...XYZ`, con `log_statement = ddl`):** No hay evidencia del UPDATE. La bitácora solo muestra la conexión y desconexión de `app_ventas` desde `app-ventas-web` en ese momento, pero no la sentencia; solo se infiere, no se demuestra.
- **Actividad 12 (a `...ZZZ`):** `app_ventas`, desde `app-ventas-web` e IP `127.0.0.1`, evento 1 de la auditoría.
- **Actividad 14 (a `...GHI`):** `postgres` (administrador), según la bitácora (`UPDATE clientes SET rfc = ...GHI` con `user=postgres`); **no** hay fila en la auditoría porque el disparador estaba desactivado.

**2. ¿Quién cambió el teléfono?** `app_ventas`, según la línea `statement: UPDATE clientes SET telefono = '6679990000'` de la bitácora (Actividad 7). El valor anterior del teléfono no consta: ese cambio ocurrió antes de crear el disparador y la bitácora no guarda valores.

**3. ¿Hubo intentos de acceso fallidos?** Sí: uno del usuario `app_ventas` con contraseña incorrecta y otro del usuario `usuario_inexistente` (el rol no existe), ambos desde `127.0.0.1`. Las horas y PID salen de las líneas FATAL de la cronología. No hay evidencia de intentos fallidos *anteriores* al cambio del RFC de la Actividad 3 porque en ese momento `log_connections` estaba apagado.

**4. ¿Qué valor tenía el RFC antes de cada cambio?**
- Actividad 3: No hay evidencia. (El valor `GAMA780912GHI` solo se conoce por el script de carga inicial, no por la bitácora ni la auditoría.)
- Actividad 7: No hay evidencia directa; la bitácora no guarda valores.
- Actividad 12: `GAMA780912XYZ` (campo `valores_antes` del evento 1).
- Actividad 14: No hay evidencia en la auditoría (disparador desactivado); por la secuencia, lo anterior era `GAMA780912ZZZ`, pero es una inferencia y no un registro.

**5. ¿Desde qué aplicación o equipo?** `app-ventas-web` desde `127.0.0.1` para los eventos con evidencia (Actividades 7 y 12). Para el cambio de la Actividad 3 no hay evidencia. Para `postgres` el equipo es la propia máquina (el campo `application_name` lo declara el cliente, por lo que no es prueba fuerte).
