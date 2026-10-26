# Procedimiento Joiner-Mover-Leaver (altas, cambios y bajas)

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

Cada petición sigue el mismo circuito, igual que un expediente: **solicitud registrada → aprobación → ejecución → evidencia → cierre**. Todas las peticiones del lab están en [`registros/registro-solicitudes-acceso.csv`](../registros/registro-solicitudes-acceso.csv). Las horas corresponden a la ejecución del 26/10/2026.

## Joiner (alta)

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud de alta con la aprobación del responsable de Ventas (simulada) | Fila `SIM-001` del registro |
| 2 | Creo `a.baja` en `OU=Ventas` con el aviso de caso simulado en *Description*. El procedimiento exige contraseña temporal con *User must change password at next logon*; en esta ejecución quedó marcada *Password never expires* (hallazgo H-01) | Evento 4720 (cuenta creada), pendiente de exportar (H-03) |
| 3 | Lo añado **solo** a `GG-Ventas` | Evento 4728 (miembro añadido a un grupo global de seguridad), pendiente de exportar (H-03) |
| 4 | Compruebo la OU y la pestaña *Member Of*: `Domain Users` y `GG-Ventas` | Capturas [01 OU](../evidencias/01-alta-joiner-OU-Ventas.png) (14:20) y [01 Member Of](../evidencias/01-alta-joiner-memberof-GG-Ventas.png) (14:21) |
| 5 | Cierro la solicitud en el registro | Fila `SIM-001` actualizada |

## Mover (cambio de rol)

> **Estado:** procedimiento escrito, sin ejecutar todavía.

| Paso | Qué hago | Por qué |
|---|---|---|
| 1 | Registro la solicitud con el nuevo rol y la aprobación del responsable del área de destino | Sin aprobación no hay cambio |
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
