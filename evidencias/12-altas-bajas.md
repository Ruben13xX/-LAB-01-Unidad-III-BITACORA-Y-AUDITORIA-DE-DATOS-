# Evidencia 12 — Altas y bajas

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 13

## Comando ejecutado

```sql
INSERT INTO clientes (nombre, correo, rfc, edad, ciudad, codigo_postal)
VALUES ('Pedro Ruiz','pedro.ruiz@example.test','RUPE950101AAA',30,'Guasave','81000');
DELETE FROM clientes WHERE nombre = 'Pedro Ruiz';
SELECT id_evento, usuario_sesion, aplicacion, ip_cliente, operacion, id_cliente,
       valores_antes ->> 'rfc' AS rfc_antes, valores_despues ->> 'rfc' AS rfc_despues
FROM auditoria.cambios_clientes ORDER BY id_evento;
```

## Resultado obtenido

```
| id_evento | usuario_sesion | aplicacion | ip_cliente | operacion | id_cliente | rfc_antes | rfc_despues |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `app_ventas` | `appventas-web` | `127.0.0.1` | `UPDATE` | 3 | `GAMA780912XYZ` | `GAMA780912ZZZ` |
| 2 | `app_ventas` | `app-ventas-web` | `127.0.0.1` | `UPDATE` | 3 | `GAMA780912ZZZ` | `GAMA780912ZZZ` |
| 3 | `postgres` | `pgAdmin 4 - CONN:5123666` | `::1` | `INSERT` | 6 | | `RUPE950101AAA` |
| 4 | `postgres` | `pgAdmin 4 - CONN:5123666` | `::1` | `DELETE` | 6 | `RUPE950101AAA` | |
```

## Interpretación

1. **¿Dónde queda evidencia de que Pedro Ruiz existió?** En `auditoria.cambios_clientes`: el evento `INSERT` guarda el registro completo en `valores_despues` y el evento `DELETE` lo guarda en `valores_antes`; así se puede reconstruir incluso un registro que ya no existe en `clientes`.
2. **¿Por qué `ip_cliente` está vacía en los eventos 2 y 3?** `ip_cliente` usa `inet_client_addr()`, que devuelve `NULL` cuando la sesión no llega por TCP (socket local). Si mi sesión de administrador se conectó por TCP a localhost, el campo muestra `127.0.0.1` o `::1`; lo que corresponde es lo que se ve en mi resultado.
3. **Conflicto con la política de retención de la Unidad II:** esa política pide eliminar los datos cuando ya no sean necesarios, pero la auditoría conserva una copia de los datos personales del cliente borrado. Debe resolverlo quien define el tratamiento de los datos: el responsable de los datos junto con el área jurídica o de cumplimiento, definiendo cuánto tiempo se conserva la auditoría y con qué justificación (obligación de trazabilidad frente a minimización de datos).
