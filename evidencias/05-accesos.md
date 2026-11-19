# Evidencia 05 — Accesos exitosos y fallidos

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  
**Actividad:** Actividad 5

## Comando ejecutado

```powershell
psql "postgresql://app_ventas:incorrecta@127.0.0.1/comercial" -c "SELECT 1"
psql "postgresql://usuario_inexistente:x@127.0.0.1/comercial" -c "SELECT 1"
psql "postgresql://app_ventas:'contraseña'@127.0.0.1/comercial?application_name=app-ventas-web" -c "SELECT count(*) FROM clientes"
```
```sql
SELECT linea FROM dba.bitacora WHERE linea LIKE '%client=127.0.0.1%' ORDER BY num;
```

## Resultado obtenido

```
    linea                                                                                                            
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 2026-11-18 20:35:56.085 CST [18460] user=[desconocido] db=[desconocido] app=[desconocido] client=127.0.0.1 LOG:  conexi¾n recibida: host=127.0.0.1 port=56863\r
 2026-11-18 20:35:56.091 CST [18460] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 FATAL:  la autentificaci¾n password fall¾ para el usuario ½app_ventas╗\r
 2026-11-18 20:35:56.091 CST [18460] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 DETALLE:  La conexi¾n coincidi¾ con la lÝnea 87 de pg_hba.conf: ½host    all             all             127.0.0.1/32            scram-sha-256╗\r
 2026-11-18 20:36:06.300 CST [23004] user=[desconocido] db=[desconocido] app=[desconocido] client=127.0.0.1 LOG:  conexi¾n recibida: host=127.0.0.1 port=56887\r
 2026-11-18 20:36:06.307 CST [23004] user=usuario_inexistente db=comercial app=[desconocido] client=127.0.0.1 FATAL:  la autentificaci¾n password fall¾ para el usuario ½usuario_inexistente╗\r
 2026-11-18 20:36:06.307 CST [23004] user=usuario_inexistente db=comercial app=[desconocido] client=127.0.0.1 DETALLE:  No existe el rol ½usuario_inexistente╗.\r
 2026-11-18 20:36:14.168 CST [3488] user=[desconocido] db=[desconocido] app=[desconocido] client=127.0.0.1 LOG:  conexi¾n recibida: host=127.0.0.1 port=56897\r
 2026-11-18 20:36:14.175 CST [3488] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 FATAL:  la autentificaci¾n password fall¾ para el usuario ½app_ventas╗\r
 2026-11-18 20:36:14.175 CST [3488] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 DETALLE:  La conexi¾n coincidi¾ con la lÝnea 87 de pg_hba.conf: ½host    all             all             127.0.0.1/32            scram-sha-256╗\r
 2026-11-18 20:36:29.793 CST [10704] user=[desconocido] db=[desconocido] app=[desconocido] client=127.0.0.1 LOG:  conexi¾n recibida: host=127.0.0.1 port=56903\r
 2026-11-18 20:36:29.800 CST [10704] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 LOG:  conexi¾n autenticada: identidad=½app_ventas╗ mÚtodo=scram-sha-256 (C:/Program Files/PostgreSQL/14/data/pg_hba.conf:87)\r
 2026-11-18 20:36:29.800 CST [10704] user=app_ventas db=comercial app=[desconocido] client=127.0.0.1 LOG:  conexi¾n autorizada: usuario=app_ventas base_de_datos=comercial nombre_de_aplicaci¾n=app-ventas-web\r
         SELECT linea FROM dba.bitacora WHERE linea LIKE '%client=127.0.0.1%' ORDER BY num;\r
(13 filas)
```

## Interpretación

### Lectura de la línea del intento fallido de `app_ventas`

Llena la columna "Valor" con los datos de **tu** línea de bitácora:

| Campo | Valor (de mi bitácora) | Significado |
|---|---|---|
| Fecha y hora | 2026-11-18 20:36:29.800 | Momento exacto del intento, con milisegundos |
| PID | 10704 | Proceso del servidor que atendió esa conexión; agrupa las líneas de la misma sesión |
| Usuario | app_ventas | Nombre de usuario con que se intentó entrar |
| Base de datos | comercial | Base a la que se intentó conectar |
| Aplicación | [unknown] | No se había asignado `application_name` en esa etapa |
| Cliente | 127.0.0.1 | Dirección desde donde llegó la conexión |
| Severidad | FATAL | La conexión se terminó |
| Mensaje | password authentication failed | Contraseña incorrecta |
| Regla aplicada | pg_hba.conf, línea [87] | Regla de `pg_hba.conf` con la que coincidió la conexión (aparece en el DETAIL) |

### Preguntas

1. **¿Prueba que la aplicación intentó entrar?** No. Solo prueba que alguien presentó el nombre `app_ventas` con una contraseña incorrecta desde esa dirección; pudo ser la aplicación con una credencial vencida, una persona o un atacante que conoce el nombre de usuario.
2. **Rol inexistente:** la bitácora agrega `DETAIL: Role "usuario_inexistente" does not exist`. No se manda al cliente para no revelar si un usuario existe (enumeración de usuarios); el mensaje al cliente es el mismo en ambos casos, pero el administrador sí necesita saber la causa real.
3. **`app=[unknown]` y luego `app=app-ventas-web`:** `application_name` es un parámetro de la sesión que se aplica una vez que la conexión fue autorizada; mientras el servidor registra autenticación y autorización, el valor aún no está asignado. Al desconectarse ya lo está. En el intento fallido, además, no se envió ningún `application_name`.
4. **Patrón de fuerza bruta:** muchas líneas `FATAL: password authentication failed` en poco tiempo, para el mismo usuario o con muchos usuarios distintos, desde la misma dirección. Consulta que cuenta los fallos por usuario y cliente:

```sql
SELECT substring(linea from 'user=([^ ]+)')   AS usuario,
       substring(linea from 'client=([^ ]+)') AS cliente,
       count(*) AS intentos_fallidos
FROM dba.bitacora
WHERE linea LIKE '%FATAL:%password authentication failed%'
GROUP BY 1, 2
ORDER BY intentos_fallidos DESC;
```
