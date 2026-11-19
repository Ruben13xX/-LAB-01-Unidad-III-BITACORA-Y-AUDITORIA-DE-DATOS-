# Evidencia 08 — Errores y operaciones rechazadas

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 8

## Comando ejecutado

```powershell
psql "postgresql://soporte_ana:'contraseña'@127.0.0.1/comercial" -c "DELETE FROM clientes WHERE id = 5"
```
```sql
DROP ROLE soporte_ana;
SELECT linea FROM dba.bitacora
WHERE linea ~ '(ERROR|STATEMENT|DETAIL): (permission|privileges|role|DELETE|DROP)' ORDER BY num;
```

## Resultado obtenido

```
| linea |
| :--- |
| `2026-11-18 15:27:43.832 CST [20868] user=soporte_ana db=comercial app=psql client=127.0.0.1 ERROR:  permission denied for table clientes` |
| `2026-11-18 15:27:43.832 CST [20868] user=soporte_ana db=comercial app=psql client=127.0.0.1 STATEMENT:  DELETE FROM` |
| `2026-11-18 15:27:56.849 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 ERROR:  role "soporte_ana" cannot be dropped because some objects depend on it` |
| `2026-11-18 15:27:56.849 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 DETAIL:  privileges for table clientes` |
| `2026-11-18 15:27:56.849 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 STATEMENT:  DROP ROLE soporte_ana;` |
| `2026-11-18 15:29:25.110 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 ERROR:  role "soporte_ana" cannot be dropped because some objects depend on it` |
| `2026-11-18 15:29:25.110 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 DETAIL:  privileges for table clientes` |
| `2026-11-18 15:29:25.110 CST [10128] user=postgres db=comercial app=pgAdmin 4 - CONN:6841707 client=::1 STATEMENT:  DROP ROLE soporte_ana;` |
| `2026-11-18 15:30:41.805 CST [20500] user=soporte_ana db=comercial app=psql client=127.0.0.1 ERROR:  permission denied for table clientes` |
| `2026-11-18 15:30:41.805 CST [20500] user=soporte_ana db=comercial app=psql client=127.0.0.1 STATEMENT:  DELETE FROM clientes WHERE id = 5` |

```

## Interpretación

1. **¿Causó daño el intento de `soporte_ana`?** No, PostgreSQL lo rechazó (`permission denied for table clientes`). Debe auditarse igual porque revela una acción no autorizada: puede ser un error, una cuenta comprometida o alguien probando sus límites, y detectarlo a tiempo permite actuar antes de que haya un daño real.
2. **Autenticación vs autorización:** el error de autenticación (Actividad 5) ocurre antes de entrar, cuando el servidor no puede verificar la identidad, y es `FATAL` (se cierra la conexión). El de autorización ocurre con la identidad ya validada, cuando el rol no tiene el privilegio; es `ERROR` y la sesión continúa.
3. **Orden para dar de baja a un usuario:** el `DROP ROLE` falló porque `soporte_ana` aún tiene privilegios sobre `clientes`. Primero hay que revocar o reasignar sus privilegios y objetos (`REVOKE`, `REASSIGN OWNED`, `DROP OWNED`) y solo después eliminar el rol.

Las sentencias que provocan error se registran gracias a `log_min_error_statement = error`, aunque `log_statement` no las incluya.
