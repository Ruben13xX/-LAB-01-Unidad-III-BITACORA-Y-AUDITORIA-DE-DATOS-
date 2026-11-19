# Evidencia 03 — El incidente sin bitácora

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 3

## Comando ejecutado

```powershell
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "UPDATE clientes SET rfc = 'GAMA780912XXX' WHERE id = 3"
```
```sql
SELECT linea FROM dba.bitacora WHERE linea ILIKE '%update%' OR linea LIKE '%app_ventas%';
```

## Resultado obtenido

```
[PEGA AQUÍ LA SALIDA REAL DE TU TERMINAL]
```

## Interpretación

1. **¿Se puede demostrar quién modificó el RFC?** No. La consulta no devuelve filas: ni el UPDATE ni la conexión de `app_ventas` quedaron registrados.
2. **¿Se puede demostrar que no lo hizo `soporte_ana`?** Tampoco. Sin registro no hay prueba a favor ni en contra de nadie; solo sabemos que el dato cambió.
3. **¿Los privilegios impidieron el cambio?** No, y no debían: `app_ventas` tiene `UPDATE` sobre `clientes`, así que el cambio era una operación permitida. El control de acceso dice quién *puede* hacer algo, no deja evidencia de lo que *hizo*.
4. **Qué debía estar configurado antes:** `log_connections` y `log_disconnections`, un `log_line_prefix` con usuario, base, aplicación y cliente, `log_statement` en `mod` (o al menos `ddl`) y la tabla de auditoría con su disparador.

Conclusión: la auditoría no se puede configurar después del incidente; lo que no se registró no se puede reconstruir.
