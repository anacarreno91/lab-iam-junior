# Procedimiento Joiner-Mover-Leaver (altas, cambios y bajas)

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

Cada petición sigue el mismo circuito, igual que un expediente: **solicitud registrada → aprobación → ejecución → evidencia → cierre**. Todas las peticiones del lab están en [`registros/registro-solicitudes-acceso.csv`](../registros/registro-solicitudes-acceso.csv). Las horas corresponden a la ejecución del 26/10/2026 y salen del registro de Seguridad ([línea temporal](../evidencias/03-linea-temporal-a.baja.png)).

## Joiner (alta)

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud de alta con la aprobación del responsable de Ventas (simulada) | Fila `SIM-001` del registro |
| 2 | Creo `a.baja` en `OU=Ventas` con el aviso de caso simulado en *Description*. El procedimiento exige contraseña temporal con *User must change password at next logon*; en esta ejecución quedó marcada *Password never expires* (hallazgo H-01, corregido el 27/10/2026) | Evento 4720 (cuenta creada) a las 14:00:03 |
| 3 | Lo añado **solo** a `GG-Ventas` | Evento 4728 (miembro añadido a un grupo global de seguridad) a las 14:17:26 |
| 4 | Compruebo la OU y la pestaña *Member Of*: `Domain Users` y `GG-Ventas` | Capturas [01 OU](../evidencias/01-alta-joiner-OU-Ventas.png) (14:20) y [01 Member Of](../evidencias/01-alta-joiner-memberof-GG-Ventas.png) (14:21) |
| 5 | Cierro la solicitud en el registro | Fila `SIM-001` actualizada |

## Mover (cambio de rol)

> **Estado:** procedimiento escrito, sin ejecutar todavía.

| Paso | Qué hago | Por qué |
|---|---|---|
| 1 | Registro la solicitud con el nuevo rol y la aprobación del responsable del área de destino | El cambio de permisos necesita aprobación previa |
| 2 | Retiro los grupos del rol anterior y añado los del nuevo en la misma intervención | Evitar la acumulación de permisos: quien cambia de puesto no debe conservar los accesos del anterior |
| 3 | Muevo la cuenta a la OU del nuevo departamento, si aplica | Que la estructura de AD refleje la organización real |
| 4 | Compruebo *Member Of*: solo grupos del rol nuevo | Verificación antes del cierre |
| 5 | Cierro la solicitud en el registro | Trazabilidad |

## Leaver (baja)

**Objetivo de plazo: menos de 24 horas** desde la solicitud hasta la revocación. Es el objetivo que fijo en este lab; en una empresa lo define su política de control de acceso.

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud de baja de RRHH (simulada) con fecha y hora | Fila `SIM-002`. En esta ejecución la hora de la solicitud no se anotó (hallazgo H-02) |
| 2 | **Deshabilito** la cuenta (no la borro) | Evento 4725 (cuenta deshabilitada) a las 14:25:49 |
| 3 | La retiro de `GG-Ventas` | Evento 4729 (miembro quitado de un grupo global de seguridad) a las 14:26:28 |
| 4 | Compruebo *Member Of*: solo `Domain Users` | Captura [02 Member Of](../evidencias/02-baja-leaver-memberof-revocada.png) (14:28) |
| 5 | Compruebo *Account is disabled* y el icono de cuenta deshabilitada en la OU | Capturas [02 Account](../evidencias/02-baja-leaver-Account-is-disabled.png) (14:30) y [02 OU](../evidencias/02-baja-leaver-OU-deshabilitada.png) (16:47) |
| 6 | Anoto la ejecución en el registro | Fila `SIM-002` actualizada |

**¿Por qué deshabilitar y no borrar?** Una cuenta deshabilitada no permite iniciar sesión, pero conserva su identificador (SID), sus atributos y su relación con el historial, que es lo que se necesita en una auditoría o una investigación. Si se borrara y hubiera que volver a crearla, sería una cuenta distinta con otro SID. El borrado se hace más tarde, cuando lo indique la política de conservación.

**¿Por qué sigue apareciendo `Domain Users`?** Es el grupo principal (*Primary group*) que Active Directory asigna a todas las cuentas y no se puede quitar sin asignar otro grupo principal. En este lab no tiene permisos sobre recursos de Ventas; lo que impide usar la cuenta es que está deshabilitada.

## Usuario externo (alta con caducidad)

El personal externo, como el técnico de un proveedor, necesita acceso solo mientras dura su trabajo. Por eso su cuenta lleva una **fecha de caducidad**: si nadie pide la baja a tiempo, la cuenta deja de funcionar sola y no se queda huérfana. Ejecutado el 29/10/2026 como `SIM-005`.

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud con su hora **antes de tocar AD** | Línea `SIM-005` a las 07:39:10 en el historial de PowerShell |
| 2 | Creo la OU `Proveedores` y el grupo `GG-Mantenimiento-Externo` | Historial: 07:39:29 |
| 3 | Creo `ext.mantenimiento` en `OU=Proveedores`, con caducidad a 30 días y *User must change password at next logon*. *Password never expires* queda desactivado | Eventos 4720 y 4722 a las 07:44:40 |
| 4 | Lo añado **solo** a `GG-Mantenimiento-Externo` | Evento 4728 a las 07:45:39 |
| 5 | Compruebo el estado y la fecha de caducidad | Capturas [05 alta](../evidencias/05-externo-alta-powershell.png) y [05 caducidad](../evidencias/05-externo-account-expires.png) |
| 6 | Compruebo qué cuentas caducan en los próximos 31 días | `Search-ADAccount -AccountExpiring`, captura [05 eventos](../evidencias/05-externo-caducidad-y-eventos.png) |

Entre la solicitud (07:39:10) y el alta completa (07:45:39) pasaron 6 minutos y 29 segundos.

**¿Por qué la consola dice «End of: 27 de noviembre» y PowerShell, 28/11 a las 7:44?** Es el mismo dato. La cuenta caduca el 28 a esa hora, y la consola de Active Directory lo muestra como «hasta el final del día 27».

**Nota sobre el historial:** el comando de las 07:40:52 no llegó a ejecutarse, porque se pegó dos veces seguido en la misma línea. La cuenta se creó con el de las 07:44:22, como confirma el evento 4720 de las 07:44:40.
