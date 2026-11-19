# Evidencia 01 — Configuración de registro por defecto

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 1

## Comando ejecutado

```sql
SELECT name, setting, source FROM pg_settings
WHERE name IN ('log_destination','logging_collector','log_directory','log_filename',
 'log_connections','log_disconnections','log_line_prefix',
 'log_statement','log_min_error_statement','log_timezone') ORDER BY name;
```

## Resultado obtenido

```
          name           |            setting             |       source
-------------------------+--------------------------------+--------------------
 log_connections         | off                            | default
 log_destination         | stderr                         | configuration file
 log_directory           | log                            | default
 log_disconnections      | off                            | default
 log_filename            | postgresql-%Y-%m-%d_%H%M%S.log | default
 log_line_prefix         | %m [%p]                        | default
 log_min_error_statement | error                          | default
 log_statement           | none                           | default
 log_timezone            | America/Mexico_City            | configuration file
 logging_collector       | on                             | configuration file
(10 filas)
```

## Interpretación

1. **¿Se registra quién se conecta?** No. `log_connections` está en `off`, así que las conexiones no dejan línea en la bitácora.
2. **¿Se registran los UPDATE?** No. `log_statement = none` no registra ninguna sentencia.
3. **¿Incluye usuario o IP?** No. El prefijo por defecto `%m [%p]` solo trae la hora y el PID; sin `%u`, `%d` y `%h` no se sabe quién ni desde dónde.
4. **Zona horaria:** la que indica `log_timezone` en mi resultado. Importa porque al reconstruir una cronología hay que comparar la bitácora con otras fuentes (la tabla de auditoría usa `TIMESTAMPTZ`, la aplicación, el reporte de facturación). Si las zonas no coinciden, el orden de los eventos puede ser equivocado.
5. **Con `logging_collector = off`** decide el mecanismo con que se desplegó el servidor (servicio, script de arranque o contenedor), porque PostgreSQL solo escribe en stderr y alguien más decide a dónde va.

Nota: en mi instalación algunos valores pueden venir ya modificados por el instalador; el `source` de cada fila lo indica (`default` o `configuration file`).
