# Evidencia 06 — Cambios de privilegios y de estructura

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 6

## Comando ejecutado

```sql
REVOKE UPDATE ON clientes FROM soporte_ana;
GRANT UPDATE ON clientes TO app_ventas;
ALTER TABLE clientes ADD COLUMN segmento VARCHAR(20);
CREATE ROLE rol_temporal LOGIN PASSWORD 'contraseña';
DROP ROLE rol_temporal;
SELECT linea FROM dba.bitacora WHERE linea LIKE '%statement:%' ORDER BY num;
```

## Resultado obtenido

```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 2026-11-18 20:15:15.280 CST [23020] SENTENCIA:  SELECT num, linea FROM dba.bitacora ORDER BY num DESC LIMIT 10;\r
 2026-11-18 20:15:56.965 CST [23020] SENTENCIA:  CREATE SCHEMA dba;\r
 2026-11-18 20:15:56.970 CST [23020] SENTENCIA:  CREATE VIEW dba.bitacora AS SELECT t.n AS num, t.linea FROM regexp_split_to_table(pg_read_file(pg_current_logfile()), E'\n') WITH ORDINALITY AS t(linea, n) WHERE t.linea <> '';\r
 2026-11-18 20:15:56.979 CST [23020] SENTENCIA:  SELECT num, linea FROM dba.bitacora ORDER BY num DESC LIMIT 10;\r
 2026-11-18 20:28:04.877 CST [21608] SENTENCIA:  psql "postgresql://app_ventas:43829600@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912XXX' WHERE id = 3"\r
 2026-11-18 20:28:10.943 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
 2026-11-18 20:28:46.224 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
 2026-11-18 20:28:46.227 CST [21608] SENTENCIA:  2026-11-18 20:28:10.943 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
 2026-11-18 20:28:55.718 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
 2026-11-18 20:29:03.871 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
 2026-11-18 20:36:45.523 CST [23020] user=postgres db=comercial app=psql client=::1 SENTENCIA:  psql "postgresql://app_ventas:incorrecta@127.0.0.1/comercial" -c "SELECT 1"\r
 2026-11-18 20:43:12.500 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: REVOKE UPDATE ON clientes FROM soporte_ana;\r
 2026-11-18 20:43:12.501 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: GRANT UPDATE ON clientes TO app_ventas;\r
 2026-11-18 20:43:12.502 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: ALTER TABLE clientes ADD COLUMN segmento VARCHAR(20);\r
 2026-11-18 20:43:12.507 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: CREATE ROLE rol_temporal LOGIN PASSWORD 'Temporal_Practica_2026';\r
 2026-11-18 20:43:12.513 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: DROP ROLE rol_temporal;\r
(16 filas)
```

## Interpretación

1. **Tipo de evento:** `REVOKE` y `GRANT` son eventos de **privilegio**; `ALTER TABLE ... ADD COLUMN` es un evento de **estructura**.
2. **¿Por qué `GRANT` y `REVOKE` quedan con `log_statement = 'ddl'`?** Porque PostgreSQL incluye en la categoría `ddl` las sentencias de definición (CREATE, ALTER, DROP) y también GRANT, REVOKE y COMMENT.
3. **¿Quién debería poder leer la bitácora?** Solo el DBA y el responsable de auditoría. La sentencia `CREATE ROLE rol_temporal ... PASSWORD '...'` dejó la contraseña en texto claro y eliminar el rol no la borra del archivo. Esto se relaciona con la gestión de secretos de la Unidad II: las credenciales no deben viajar en sentencias SQL en claro (se usa `\password`, que envía solo el resumen SCRAM), y cualquier secreto expuesto debe considerarse comprometido y cambiarse. La bitácora, por tanto, se trata como un archivo sensible.