# Informe — LAB-01 Bitácoras y auditoría del dato en PostgreSQL

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Informe final


## 1. Despliegue y situación inicial

El laboratorio se hizo en una instalación local de PostgreSQL 16 (ver README), con la base `comercial` y la tabla `clientes` de la Unidad II. Con la configuración inicial (`evidencias/01`), el servidor no registraba conexiones (`log_connections = off`), no registraba sentencias (`log_statement = none`) y el prefijo de línea solo traía hora y PID. Por eso no permitía responder a Dirección: tras el cambio del RFC de María García no había manera de saber quién, cuándo, desde dónde ni qué valor tenía antes (`evidencias/03`). Los permisos de la Unidad II permitían el cambio (la aplicación tenía `UPDATE`), pero el control de acceso no produce evidencia.

## 2. Configuración aplicada

| Parámetro | Valor | Aplicación | Justificación |
|---|---|---|---|
| logging_collector | on | Reinicio | El servidor administra su propia bitácora y se lee con `pg_current_logfile()` en cualquier plataforma |
| log_connections / log_disconnections | on | Recarga | Registrar accesos exitosos, fallidos y duración de sesiones |
| log_line_prefix | `%m [%p] user=%u db=%d app=%a client=%h ` | Recarga | Sin usuario, base, aplicación y cliente una línea no dice quién ni desde dónde |
| log_statement | ddl y después mod | Recarga | `ddl` para privilegios y estructura; `mod` para modificaciones de datos |

Los cambios se hicieron con `ALTER SYSTEM`, y se comprobó el origen con `pg_settings.source`. La verificación se hizo en una consulta posterior a `pg_reload_conf()`, porque la recarga es asíncrona.

## 3. Eventos observados

| Tipo | Ejemplo interpretado | Evidencia |
|---|---|---|
| Acceso | `FATAL: password authentication failed` para `app_ventas` desde 127.0.0.1: prueba que alguien presentó ese nombre con una clave incorrecta, no que fuera la aplicación | 05 |
| Privilegio | `GRANT UPDATE ON clientes TO app_ventas`, registrado con `ddl` | 06 |
| Estructura | `ALTER TABLE clientes ADD COLUMN segmento` | 06 |
| Datos | `UPDATE clientes SET telefono = ...` con usuario, hora, IP y aplicación, solo con `mod` | 07 |
| Error | `permission denied for table clientes` con su sentencia, por `soporte_ana` | 08 |
| Configuración | `parameter "log_statement" changed to "mod"` registrado por el proceso principal | 04 |

## 4. Auditoría del dato

Se creó `auditoria.cambios_clientes` (esquema sin acceso para PUBLIC) con momento, usuario de sesión y efectivo, aplicación, IP, operación, id del cliente y los valores antes y después en JSONB, y el disparador `trg_auditoria_clientes` (AFTER INSERT/UPDATE/DELETE, por fila).

La función es `SECURITY DEFINER` para que `app_ventas` pueda provocar el registro sin tener privilegios sobre la auditoría, y tiene `search_path` fijo (`pg_catalog, public`) para que nadie pueda sustituir objetos con privilegios del propietario. Dentro de ella `current_user` es el propietario, por eso se registra también `session_user`, que identifica quién provocó el cambio.

En el incidente auditado, el cambio `GAMA780912XYZ` → `GAMA780912ZZZ` quedó con `usuario_sesion = app_ventas`, aplicación `app-ventas-web` e IP `127.0.0.1`, y con la consulta de campos cambiados se obtuvo el valor anterior para restaurarlo (`evidencias/11`).

## 5. Limitaciones

- **Lecturas:** ni la bitácora con `mod` ni el disparador registran los `SELECT`; la lectura del RFC por `soporte_ana` no dejó rastro (`evidencias/14`). Se requiere `log_statement = all` (mucho volumen y datos sensibles en la bitácora), una función de acceso o `pgaudit`.
- **Volumen:** un UPDATE sin WHERE produce una línea de bitácora pero una fila de auditoría por cada registro afectado.
- **Administrador con control total:** pudo desactivar el disparador, modificar el RFC y reactivarlo sin dejar fila de auditoría; solo quedó la bitácora (`evidencias/13`). Ningún mecanismo dentro del servidor protege contra quien lo controla por completo.
- **Identidad:** la bitácora registra el rol, no a la persona; si varias personas comparten `app_ventas` no se distinguen. `application_name` lo declara el cliente y es fácil de falsificar.

## 6. Riesgos de la bitácora

- **Secretos expuestos:** la contraseña del rol temporal quedó en texto claro y borrar el rol no la quita. Se retiró al exportar y se comprobó con una búsqueda.
- **Pérdida:** vive en el servidor; la rotación, un reinicio o una reinstalación pueden hacerla desaparecer. Se exportó al repositorio y se calculó un SHA-256 (`evidencias/bitacora-lab01.sha256`).
- **Acceso de terceros:** contiene datos personales dentro de las sentencias, direcciones internas y nombres de usuarios, así que su lectura (`dba.bitacora`) se limitó al DBA.
- El hash permite detectar modificaciones, no impedirlas; debe guardarse fuera del servidor.

## 7. Cronología y conclusión

La cronología está en `evidencias/15-cronologia.md`. Muestra que para los cambios hechos antes de configurar la bitácora (Actividades 3 y 7 con `ddl`) **no hay evidencia**, y que se declaran como huecos y no se estiman.

**Conclusión:** con la configuración final sí puede demostrarse quién cambió un dato sensible y qué valor tenía antes, siempre que el cambio ocurra con la bitácora y el disparador activos y que no los manipule un administrador con control total. Los cambios anteriores a la configuración no pueden reconstruirse, las lecturas no quedan registradas y el cambio hecho con el disparador desactivado solo se detecta por la bitácora. Para cerrar esos huecos hacen falta una copia externa de la bitácora, separación de funciones entre quien opera y quien audita y, para lecturas, un mecanismo adicional como `pgaudit`.

## 8. Preguntas de reflexión

1. **¿Por qué no puede configurarse la auditoría después de un incidente?** Porque la auditoría solo registra lo que ocurre mientras está activa; lo que pasó antes no tiene hora, usuario ni valor y no puede reconstruirse (Actividad 3).
2. **¿Por qué que la bitácora la administre el servidor?** Porque con `logging_collector` el servidor decide dónde se escribe, funciona igual en cualquier plataforma, permite leerla con funciones del motor y la evidencia queda bajo control del servidor y no de cómo se desplegó.
3. **¿Por qué el prefijo importa tanto como `log_statement`?** El nivel decide *qué* se registra; el prefijo decide si la línea dice quién, en qué base, desde qué aplicación y desde dónde. Sin `%u`, `%d`, `%a` y `%h`, la sentencia queda sin responsable.
4. **Registrar la sentencia o el cambio en el dato:** la sentencia es el texto SQL ejecutado, sin valores anteriores; el cambio en el dato guarda el estado antes y después de cada registro, lo que permite revertir y ver qué campo cambió.
5. **Qué ofrece `pg_stat_activity` y qué la bitácora:** `pg_stat_activity` muestra el estado actual (sesiones y consultas en curso) pero no conserva nada una vez que la sesión termina; la bitácora conserva la historia (conexiones, errores, sentencias) pero no el estado actual.
6. **¿Quién audita al administrador?** Debe auditarlo alguien ajeno a su control: enviar la bitácora a un sistema externo gestionado por otra área, hacer revisiones periódicas independientes y exigir aprobaciones para cambios en la auditoría. Dentro del propio servidor no hay un mecanismo confiable.
