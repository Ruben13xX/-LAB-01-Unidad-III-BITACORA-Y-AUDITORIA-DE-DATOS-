# Evidencia 07 — Modificaciones de datos: ddl frente a mod

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 7

## Comando ejecutado

```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912XYZ' WHERE id = 3"
```
```sql
ALTER SYSTEM SET log_statement = 'mod';
SELECT pg_reload_conf();
```
```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET telefono = '6679990000' WHERE id = 3" -c "SELECT nombre, rfc FROM clientes WHERE id = 3"
```
```sql
SELECT linea FROM dba.bitacora
WHERE linea LIKE '%log_statement%' OR linea LIKE '%user=app_ventas%statement:%' ORDER BY num;
```

## Resultado obtenido

```
-----------------------------------------------------------------------------------------------------------------------------------------------
 2026-11-18 20:31:17.674 CST [22780] user= db= app= client= LOG:  el parßmetro ½log_statement╗ fue cambiado a ½ddl╗\r
 2026-11-18 20:48:10.887 CST [23020] user=postgres db=comercial app=psql client=::1 LOG:  sentencia: ALTER SYSTEM SET log_statement = 'mod';\r
 2026-11-18 20:48:31.908 CST [22780] user= db= app= client= LOG:  el parßmetro ½log_statement╗ fue cambiado a ½mod╗\r
(3 filas)
```

## Interpretación

**Análisis:** con `ddl` aparecen la conexión y desconexión de `app_ventas`, pero no el UPDATE. Con `mod` el UPDATE del teléfono sí aparece con usuario, hora, IP, aplicación y sentencia. El SELECT que leyó el RFC no aparece, y la bitácora no muestra el valor anterior del teléfono.

1. **¿Qué nivel habría registrado la lectura del RFC?** `all`. En un servidor con miles de consultas por minuto el costo sería enorme: crecimiento muy rápido de la bitácora, más escritura en disco, más carga y ruido que dificulta encontrar lo importante, además de que la bitácora acumularía datos sensibles.
2. **¿Qué le falta para revertir con certeza?** El valor anterior. La bitácora guarda el texto de la sentencia, no el contenido previo del registro; eso se resuelve con la tabla de auditoría y el disparador.
