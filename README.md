# LAB-01 — Bitácoras y auditoría del dato en PostgreSQL

**Estudiante:** Jesus Ruben Monzon Garcia  
**Grupo y grado:** 4-3  
**Materia:** Gestión de Seguridad y Fundamentos de Base de Datos  
**Laboratorio:** LAB-01 Bitácoras y auditoría del dato en PostgreSQL  


## Despliegue

- Motor: PostgreSQL 16, instalación nativa local.
- Versión (`SELECT version();`): 14
- Sistema operativo: [TU SISTEMA OPERATIVO]
- Base de datos: `comercial`. Roles: `postgres` (DBA), `app_ventas`, `soporte_ana`.
- Sesión de administrador: `psql -U postgres -d comercial`. Sesión por red: cadena `postgresql://...@127.0.0.1/comercial?application_name=...`.
- Reinicio del servicio: `Restart-Service postgresql-x64-16` (PowerShell como administrador).
- Rutas (`SHOW data_directory`, `config_file`, `hba_file`): ver `evidencias/00` en la salida de `sql/00-preparacion.sql` y la evidencia 01.

## Estructura

```
LAB-01/
├── README.md
├── informe.md
├── sql/            00-preparacion … 06-campos-modificados
└── evidencias/     01 … 15, bitacora-lab01.log, bitacora-lab01.sha256
```

## Índice de evidencias

| Evidencia | Actividad | Tema |
|---|---|---|
| 01 | 1 | Configuración de registro por defecto |
| 02 | 2 | Colector y vista `dba.bitacora` |
| 03 | 3 | Incidente sin bitácora |
| 04 | 4 | Configuración de la bitácora |
| 05 | 5 | Accesos exitosos y fallidos |
| 06 | 6 | Privilegios y estructura |
| 07 | 7 | `ddl` frente a `mod` |
| 08 | 8 | Errores |
| 09 | 9 | `pg_stat_activity` |
| 10 | 10 | Diseño de la tabla de auditoría |
| 11 | 11 y 12 | Disparador e incidente auditado |
| 12 | 13 | Altas y bajas |
| 13 | 14 | Manipulación de la auditoría |
| 14 | 15 | Límites: lecturas y volumen |
| 15a | 16 | Exportación e integridad |
| 15 | 17 | Cronología y respuestas a Dirección |