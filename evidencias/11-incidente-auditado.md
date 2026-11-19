# Evidencia 11 — Función, disparador e incidente auditado

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividades 11 y 12

## Comando ejecutado

```sql
-- sql/05-funcion-disparador.sql
SELECT tgname, tgenabled, tgrelid::regclass FROM pg_trigger WHERE NOT tgisinternal;
SELECT linea FROM dba.bitacora WHERE linea ~ 'statement: (CREATE|ALTER)' ORDER BY num;
```
```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912ZZZ' WHERE id = 3"
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial" -c "SELECT * FROM auditoria.cambios_clientes"
```
```sql
-- sql/06-campos-modificados.sql
\x
SELECT id_evento, momento, usuario_sesion, usuario_efectivo, aplicacion, ip_cliente, operacion, id_cliente
FROM auditoria.cambios_clientes ORDER BY id_evento;
\x

-- Que campo cambio: compara antes y despues campo por campo
SELECT a.id_evento, a.momento, a.usuario_sesion, a.ip_cliente,
       d.key AS campo,
       a.valores_antes ->> d.key AS antes,
       d.value #>> '{}' AS despues
FROM auditoria.cambios_clientes a
CROSS JOIN LATERAL jsonb_each(a.valores_despues) d
WHERE a.operacion = 'UPDATE'
  AND a.valores_antes -> d.key IS DISTINCT FROM d.value
ORDER BY a.id_evento;
```

## Resultado obtenido

```
| tgname | tgenabled | tgrelid |
| :--- | :--- | :--- |
| `trg_auditoria_clientes` | `O` | `clientes` |

**lineas de la bitacora**

| num | linea |
| :--- | :--- |
| 40 | `2026-11-18 15:06:15.141 CST [34320] user=postgres db=comercial app=pgAdmin 4 - CONN:8090263 client=::1 LOG:  statement: REVOKE UPDATE ON clientes FROM soporte_ana;` |
| 43 | `2026-11-18 15:06:28.960 CST [34320] user=postgres db=comercial app=pgAdmin 4 - CONN:8090263 client=::1 LOG:  statement: CREATE ROLE rol_temporal LOGIN PASSWORD '<REDACTED>';` |
| 86 | `2026-11-18 15:24:00.479 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 LOG:  statement: REVOKE UPDATE ON clientes FROM soporte_ana;` |
| 93 | `2026-11-18 15:24:03.578 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 LOG:  statement: REVOKE UPDATE ON clientes FROM soporte_ana;` |
| 98 | `2026-11-18 15:24:12.586 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 LOG:  statement: CREATE ROLE rol_temporal LOGIN PASSWORD '<REDACTED>';` |
| 145 | `2026-11-18 15:44:29.335 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: CREATE SCHEMA auditoria;` |
| 146 | `2026-11-18 15:44:30.936 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: REVOKE ALL ON SCHEMA auditoria FROM PUBLIC;` |
| 147 | `2026-11-18 15:44:32.945 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: CREATE TABLE auditoria.cambios_clientes ( id_evento BIGSERIAL PRIMARY KEY,` |
| 163 | `2026-11-18 15:47:48.923 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: CREATE SCHEMA auditoria;` |
| 166 | `2026-11-18 15:47:51.433 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: REVOKE ALL ON SCHEMA auditoria FROM PUBLIC;` |
| 197 | `2026-11-18 15:49:29.882 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: CREATE SCHEMA auditoria;` |
| 200 | `2026-11-18 15:49:36.643 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: REVOKE ALL ON SCHEMA auditoria FROM PUBLIC;` |
| 201 | `2026-11-18 15:49:40.691 CST [9312] user=postgres db=comercial app=pgAdmin 4 - CONN:792644 client=::1 LOG:  statement: CREATE TABLE auditoria.cambios_clientes ( id_evento BIGSERIAL PRIMARY KEY,` |
| 253 | `2026-11-18 15:55:15.707 CST [34280] user=postgres db=comercial app=pgAdmin 4 - CONN:5137444 client=::1 LOG:  statement: REVOKE ALL ON FUNCTION auditoria.registrar_cambio_cliente() FROM PUBLIC;` |
| 277 | `2026-11-18 16:11:18.201 CST [34280] user=postgres db=comercial app=pgAdmin 4 - CONN:5137444 client=::1 LOG:  statement: CREATE OR REPLACE FUNCTION auditoria.registrar_cambio_cliente()` |
| 300 | `2026-11-18 16:11:20.801 CST [34280] user=postgres db=comercial app=pgAdmin 4 - CONN:5137444 client=::1 LOG:  statement: REVOKE ALL ON FUNCTION auditoria.registrar_cambio_cliente() FROM PUBLIC;` |
| 301 | `2026-11-18 16:11:23.025 CST [34280] user=postgres db=comercial app=pgAdmin 4 - CONN:5137444 client=::1 LOG:  statement: CREATE TRIGGER trg_auditoria_clientes` |   

**Salida de los dos psql de app_ventas**

```powershell
PS C:\Users\zaydr> psql "postgresql://app_ventas:Ventas_Practica_2026@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912ZZZ' WHERE id = 3"
UPDATE 1

PS C:\Users\zaydr> psql "postgresql://app_ventas:Ventas_Practica_2026@127.0.0.1/comercial" -c "SELECT * FROM auditoria.cambios_clientes"
ERROR:  permission denied for schema auditoria
LÍNEA 1: SELECT * FROM auditoria.cambios_clientes
```

**Registro auditoria**

| id_evento | momento | usuario_sesion | usuario_efectivo | aplicacion | ip_cliente | operacion | id_cliente |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `2026-11-18 15:59:34.955456-06` | `app_ventas` | `postgres` | `appventas-web` | `127.0.0.1` | `UPDATE` | 3 |
| 2 | `2026-11-18 16:13:22.109663-06` | `app_ventas` | `postgres` | `app-ventas-web` | `127.0.0.1` | `UPDATE` | 3 |

**Campo modificado:**

| id_evento | momento | usuario_sesion | ip_cliente | campo | antes | despues |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | `2026-11-18 15:59:34.955456-06` | `app_ventas` | `127.0.0.1` | `rfc` | `GAMA780912XYZ` | `GAMA780912ZZZ` |

## Interpretación

### Preguntas de la Actividad 11

1. **¿Por qué `REVOKE ALL ON FUNCTION ... FROM PUBLIC` si el disparador seguirá funcionando para `app_ventas`?** Por mínimo privilegio: por defecto todo rol recibe `EXECUTE` sobre una función nueva, y una función `SECURITY DEFINER` no debe quedar al alcance de cualquiera. El disparador sigue funcionando porque el permiso de ejecución se verifica al crear el disparador, no cada vez que se dispara.
2. **¿Qué pasaría sin `SECURITY DEFINER`?** La función correría con los privilegios de `app_ventas`, que no tiene acceso al esquema `auditoria`; el `INSERT` en la auditoría fallaría con `permission denied for schema auditoria` y, como el error cancela la sentencia, el UPDATE de la aplicación también fallaría.
3. **¿Quedó registrada la creación del esquema, la tabla, la función y el disparador?** Sí, porque son sentencias `CREATE` (DDL) y con `log_statement = mod` (que incluye `ddl`) se registran.

### Preguntas de la Actividad 12

1. **Hipótesis frente a resultado:** mi hipótesis (que sin `SECURITY DEFINER` fallaría por permisos) se confirma por contraste: con `SECURITY DEFINER` el UPDATE de `app_ventas` se ejecutó y quedó auditado, aunque `app_ventas` no puede leer ni escribir en `auditoria` (lo comprueba el `permission denied` al consultarla).
2. **UPDATE para restaurar el valor anterior** (según `valores_antes` del evento 1):

```sql
UPDATE clientes SET rfc = 'GAMA780912XYZ' WHERE id = 3;
```

3. **¿`aplicacion` o `ip_cliente`?** Tiene más valor `ip_cliente`: la obtiene el servidor de la conexión y el cliente no puede elegirla fácilmente. `aplicacion` (`application_name`) lo declara el propio cliente y es muy fácil de falsificar; sirve para distinguir aplicaciones legítimas, no como prueba de identidad.

`usuario_efectivo` aparece como `postgres` porque dentro de una función `SECURITY DEFINER`, `current_user` es el propietario de la función; por eso la auditoría usa `session_user` para saber quién provocó realmente el cambio.
