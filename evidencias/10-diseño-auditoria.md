# Evidencia 10 — Diseño de la tabla de auditoría

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 10

## Comando ejecutado

```sql
CREATE SCHEMA auditoria;
REVOKE ALL ON SCHEMA auditoria FROM PUBLIC;

CREATE TABLE auditoria.cambios_clientes ( id_evento BIGSERIAL PRIMARY KEY,
 momento TIMESTAMPTZ NOT NULL DEFAULT now(),
 usuario_sesion TEXT NOT NULL DEFAULT session_user,
 usuario_efectivo TEXT NOT NULL DEFAULT current_user,
 aplicacion TEXT DEFAULT current_setting('application_name'),
 ip_cliente INET DEFAULT inet_client_addr(),
 operacion TEXT NOT NULL,
 id_cliente BIGINT,
 valores_antes JSONB,
 valores_despues JSONB
);
```

## Resultado obtenido

```
| nspname | nspacl |
| :--- | :--- |
| `auditoria` | `["postgres=UC/postgres"]` |

| column_name | data_type | is_nullable | column_default |
| :--- | :--- | :--- | :--- |
| `id_evento` | `bigint` | `NO` | `nextval('auditoria.cambios_clientes_id_evento_seq'::regclass)` |
| `momento` | `timestamp with time zone` | `NO` | `now()` |
| `usuario_sesion` | `text` | `NO` | `SESSION_USER` |
| `usuario_efectivo` | `text` | `NO` | `CURRENT_USER` |
| `aplicacion` | `text` | `YES` | `current_setting('application_name'::text)` |
| `ip_cliente` | `inet` | `YES` | `inet_client_addr()` |
| `operacion` | `text` | `NO` | |
| `id_cliente` | `bigint` | `YES` | |
| `valores_antes` | `jsonb` | `YES` | |
| `valores_despues` | `jsonb` | `YES` | |
```

## Interpretación

### Diseño de los campos

| Campo | Tipo | ¿Qué pregunta responde? | ¿Puede ser nulo? | ¿Por qué? |
|---|---|---|---|---|
| id_evento | BIGSERIAL | Identifica cada evento y fija su orden | No | Es la llave primaria |
| momento | TIMESTAMPTZ | ¿Cuándo ocurrió? | No | Todo evento tiene un instante; el valor por defecto es `now()` |
| usuario_sesion | TEXT | ¿Quién lo hizo? | No | `session_user` siempre existe y es el rol con que se autenticó |
| usuario_efectivo | TEXT | ¿Con qué privilegios se ejecutó? | No | `current_user` siempre existe (dentro del disparador será el propietario de la función) |
| aplicacion | TEXT | ¿Desde qué aplicación? | Sí | Lo declara el cliente y puede venir vacío |
| ip_cliente | INET | ¿Desde qué equipo? | Sí | Es `NULL` si la conexión es por socket local |
| operacion | TEXT | ¿Qué ocurrió? (INSERT, UPDATE, DELETE) | No | Todo evento es de un tipo |
| id_cliente | BIGINT | ¿A qué registro afectó? | Sí | Permite buscar el historial de un cliente; en eventos sin fila asociada podría no existir |
| valores_antes | JSONB | ¿Qué valor tenía antes? | Sí | No existe en un INSERT |
| valores_despues | JSONB | ¿Qué valor quedó? | Sí | No existe en un DELETE |

### Preguntas

1. **¿Por qué `TIMESTAMPTZ` y no `TIMESTAMP`?** Porque guarda un instante absoluto y no depende de la zona horaria de la sesión ni del horario de verano; así se puede comparar sin ambigüedad con la bitácora, que registra con zona.
2. **¿Por qué el registro completo en JSONB?** Para conservar el estado completo de la fila aunque la tabla `clientes` cambie después (se agreguen o quiten columnas), sin modificar la tabla de auditoría, y para poder restaurar un registro entero, por ejemplo uno eliminado.
3. **¿Qué implica que la tabla de auditoría contenga RFC?** Que es una copia de datos personales y debe clasificarse y protegerse igual o más que `clientes`: acceso solo para DBA o auditor, sin permisos para la aplicación, respaldos protegidos y una política de retención definida.
