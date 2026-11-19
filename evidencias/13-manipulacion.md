# Evidencia 13 — Intentar manipular la auditoría

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 14

## Comando ejecutado

```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "DELETE FROM auditoria.cambios_clientes" -c "ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes"
```
```sql
ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes;
UPDATE clientes SET rfc = 'GAMA780912GHI' WHERE id = 3;
ALTER TABLE clientes ENABLE TRIGGER trg_auditoria_clientes;
SELECT count(*) FROM auditoria.cambios_clientes;
SELECT linea FROM dba.bitacora
WHERE linea ~ 'statement: (ALTER TABLE clientes (DIS|EN)ABLE|UPDATE clientes SET rfc = .GAMA780912GHI)'
ORDER BY num;
```

## Resultado obtenido

```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "DELETE FROM auditoria.cambios_clientes" -c "ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes"
>> TRIGGER trg_auditoria_clientes"
ERROR:  permission denied for schema auditoria
LÍNEA 1: DELETE FROM auditoria.cambios_clientes
                     ^
ERROR:  must be owner of table clientes
```

**Conteo de la tabla de auditoria**

| count |
| :--- |
| 4 |

**Líneas de bitácora:**

| num | linea |
| :--- | :--- |
| 375 | `2026-11-18 16:34:43.595 CST [31180] user=app_ventas db=comercial app=app-ventas-web client=127.0.0.1 LOG:  statement: ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes` |
| 379 | `2026-11-18 16:35:05.178 CST [24228] user=postgres db=comercial app=pgAdmin 4 - CONN:5123666 client=::1 LOG:  statement: ALTER TABLE clientes DISABLE TRIGGER trg_auditoria_clientes;` |
| 380 | `2026-11-18 16:35:07.084 CST [24228] user=postgres db=comercial app=pgAdmin 4 - CONN:5123666 client=::1 LOG:  statement: UPDATE clientes SET rfc = 'GAMA780912GHI' WHERE id = 3;` |
| 381 | `2026-11-18 16:35:09.103 CST [24228] user=postgres db=comercial app=pgAdmin 4 - CONN:5123666 client=::1 LOG:  statement: ALTER TABLE clientes ENABLE TRIGGER trg_auditoria_clientes;` |

## Interpretación

**Resultado:** `app_ventas` no pudo borrar la auditoría ni desactivar el disparador (no es propietario). El administrador sí pudo desactivarlo, modificar el RFC y reactivarlo; el conteo de la tabla de auditoría no aumentó, de modo que ese cambio no quedó auditado a nivel de dato. La bitácora, en cambio, registra el `DISABLE TRIGGER`, el `UPDATE` y el `ENABLE TRIGGER` con `user=postgres`, y también el intento fallido de `app_ventas`.

1. **¿Qué evidencia permite detectar un cambio sin auditar?** La comparación entre ambas fuentes: la bitácora muestra un `UPDATE` sobre `clientes` que no tiene su fila correspondiente en la tabla de auditoría, y las líneas `DISABLE TRIGGER` y `ENABLE TRIGGER` delimitan el periodo sin auditoría.
2. **Control administrativo que reduciría el riesgo:** separar a quien opera la base de quien custodia la evidencia (enviar la bitácora a un sistema externo administrado por otra área), exigir autorización de dos personas para cambios en el mecanismo de auditoría y revisar periódicamente la bitácora contra la tabla de auditoría.
3. **Relación con la separación de funciones (Unidad II):** quien tiene control total sobre el servidor puede desactivar su propia auditoría. Por eso las funciones de operación y de auditoría deben recaer en personas o áreas distintas, igual que se separan los roles de DBA, aplicación y consulta.
