# Evidencia 02 — Colector de bitácora y vista dba.bitacora

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 2

## Comando ejecutado

```sql
ALTER SYSTEM SET logging_collector = on;
SELECT name, setting, pending_restart FROM pg_settings WHERE name = 'logging_collector';
-- reinicio del servicio
SELECT name, setting, source FROM pg_settings WHERE name = 'logging_collector';
SELECT pg_current_logfile();
CREATE SCHEMA dba; REVOKE ALL ON SCHEMA dba FROM PUBLIC;
CREATE VIEW dba.bitacora AS ... 
SELECT num, linea FROM dba.bitacora ORDER BY num DESC LIMIT 10;
```

## Resultado obtenido

```
       name        | setting |       source
-------------------+---------+--------------------
 logging_collector | on      | configuration file
(1 fila)


       name        | setting | pending_restart
-------------------+---------+-----------------
 logging_collector | on      | f
(1 fila)
```

## Interpretación

1. **¿Por qué `logging_collector` requiere reinicio y `log_statement` solo recarga?** El colector es un proceso aparte que se crea al arrancar el servidor y que captura el stderr de todos los procesos; eso no se puede cambiar con el servidor ya en marcha. `log_statement` en cambio lo lee cada sesión al recargar la configuración, por lo que basta una señal.
2. **Ventaja y riesgo de leer la bitácora con SQL:** la ventaja es que funciona igual en cualquier plataforma, no depende del sistema operativo ni de permisos sobre el archivo, y se puede filtrar con SQL. El riesgo es que `pg_read_file` puede leer cualquier archivo que el servidor pueda leer, y que la vista expone todo el contenido de la bitácora (incluso secretos) a quien tenga acceso.
3. **¿Quién debería consultar `dba.bitacora`?** Solo el DBA o el responsable de auditoría (superusuario). Por eso se creó el esquema `dba` sin acceso para PUBLIC y no se otorgó a `app_ventas` ni a `soporte_ana`.
