# Evidencia 04 — Configuración de la bitácora

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 4

## Comando ejecutado

```sql
ALTER SYSTEM SET log_connections = on;
ALTER SYSTEM SET log_disconnections = on;
ALTER SYSTEM SET log_line_prefix = '%m [%p] user=%u db=%d app=%a client=%h ';
ALTER SYSTEM SET log_statement = 'ddl';
SELECT pg_reload_conf();
SELECT name, setting, source, sourcefile FROM pg_settings
WHERE name IN ('log_connections','log_line_prefix','log_statement');
SELECT num, linea FROM dba.bitacora ORDER BY num DESC LIMIT 6;
```

## Resultado obtenido

```
      name       |                 setting                 |       source       |                        sourcefile  
-----------------+-----------------------------------------+--------------------+----------------------------------------------------------
 log_connections | off                                     | default            |
 log_line_prefix | %m [%p] user=%u db=%d app=%a client=%h  | configuration file | C:/Program Files/PostgreSQL/14/data/postgresql.auto.conf
 log_statement   | ddl                                     | configuration file | C:/Program Files/PostgreSQL/14/data/postgresql.auto.conf
(3 filas)


 num |                                                                           linea                               
-----+------------------------------------------------------------------------------------------------------------------------------------------------------------
  61 | 2026-11-18 20:31:17.674 CST [22780] user= db= app= client= LOG:  el parßmetro ½log_statement╗ fue cambiado a ½ddl╗\r
  60 | 2026-11-18 20:31:17.674 CST [22780] user= db= app= client= LOG:  el parßmetro ½log_line_prefix╗ fue cambiado a ½%m [%p] user=%u db=%d app=%a client=%h ╗\r
  59 | 2026-11-18 20:31:17.674 CST [22780] LOG:  el parßmetro ½log_disconnections╗ fue cambiado a ½on╗\r
  58 | 2026-11-18 20:31:17.674 CST [22780] LOG:  el parßmetro ½log_connections╗ fue cambiado a ½on╗\r
  57 | 2026-11-18 20:31:17.672 CST [22780] LOG:  se recibi¾ SIGHUP, volviendo a cargar archivos de configuraci¾n\r
  56 | 2026-11-18 20:29:03.871 CST [21608] SENTENCIA:  SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';\r
(6 filas)

 pg_reload_conf
----------------
 t
(1 fila)
```

## Interpretación

**Análisis:** las sentencias `ALTER SYSTEM` no aparecen porque cuando se ejecutaron `log_statement` todavía valía `none`. Sí aparecen las líneas `parameter ... changed to ...`, que son el registro del cambio de configuración. Mi propia sesión de administrador también quedó registrada: consultar la bitácora deja rastro.

1. **¿Por qué lo registra el proceso principal y no mi sesión?** Porque mi sesión solo escribió el valor en `postgresql.auto.conf` y envió la señal de recarga; quien relee la configuración y la aplica es el proceso principal del servidor (postmaster), y por eso es él quien escribe el mensaje.
2. **¿Y si alguien ejecutara `ALTER SYSTEM SET log_statement = 'none'`?** Sí quedaría rastro: si el nivel vigente era `ddl` o `mod`, la sentencia se registra, y después de la recarga el servidor registra `parameter "log_statement" changed to "none"`. Lo que ya no quedaría es nada de lo posterior. Por eso hay que proteger y respaldar la bitácora.
3. **¿Por qué no editar `postgresql.auto.conf` a mano?** Porque el archivo lo gestiona PostgreSQL: un error de sintaxis o un valor inválido puede impedir que se aplique la configuración (o que el servicio arranque), y una edición manual no pasa por las validaciones ni por el registro que sí hace `ALTER SYSTEM`.
