# Evidencia 14 — Límites: lecturas y volumen

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 15

## Comando ejecutado

```powershell
psql "postgresql://soporte_ana:'contraseña'@127.0.0.1/comercial" -c "SELECT nombre, rfc, telefono FROM clientes WHERE id = 3"
```
```sql
UPDATE clientes SET segmento = 'regular';
SELECT count(*) AS lineas_bitacora FROM dba.bitacora WHERE linea LIKE '%statement: UPDATE clientes SET segmento%';
SELECT operacion, count(*) FROM auditoria.cambios_clientes GROUP BY operacion ORDER BY operacion;
SELECT count(*) AS lecturas_soporte FROM dba.bitacora WHERE linea LIKE '%user=soporte_ana%SELECT nombre, rfc%';
```

## Resultado obtenido

```
| lineas_bitacora |
| :--- |
| 2 |

| operacion | count |
| :--- | :--- |
| `DELETE` | 2 |
| `INSERT` | 2 |
| `UPDATE` | 12 |

| lecturas_soporte |
| :--- |
| 0 |
```

## Interpretación

**Resultado esperado y significado:** el UPDATE masivo produjo **una** línea en la bitácora y **una fila por registro** (cinco) en la tabla de auditoría. La lectura del RFC por `soporte_ana` no dejó rastro: no existe disparador para `SELECT` y `mod` no registra consultas.

1. **Con un millón de registros y un UPDATE sin WHERE:** la bitácora seguiría teniendo una sola línea, pero la tabla de auditoría recibiría un millón de filas (cada una con el registro antes y después en JSONB). Eso significa mucho almacenamiento, una transacción larga y lenta, y la tabla de auditoría crece muy rápido.
2. **¿Quién autoriza la depuración de `auditoria.cambios_clientes`?** El responsable de los datos junto con seguridad o cumplimiento, nunca el DBA por su cuenta; el DBA solo la ejecuta cuando haya una autorización y un registro de ello.
3. **¿Por qué `log_statement = 'all'` convierte la bitácora en un depósito de datos sensibles?** Porque registra el texto completo de todas las sentencias, y ese texto contiene los valores (RFC, teléfonos, correos en los `WHERE` o `INSERT`) y a veces contraseñas; la bitácora termina siendo una copia sin control de datos personales.

| Alternativa para registrar lecturas | Ventaja | Desventaja |
|---|---|---|
| `log_statement = 'all'` | No requiere instalar nada | Registra todas las consultas de todas las tablas |
| Entregar el dato mediante una función que registre su uso | Control preciso | Requiere cambiar la aplicación |
| `pgaudit` con auditoría de objeto (Parte C) | Registra lecturas por tabla y columna | Requiere instalar una extensión |
