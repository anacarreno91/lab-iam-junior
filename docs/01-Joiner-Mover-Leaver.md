# Procedimiento Joiner-Mover-Leaver (altas, cambios y bajas)

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

Cada petición sigue el mismo circuito, igual que un expediente: **solicitud registrada → aprobación → ejecución → evidencia → cierre**. Todas las peticiones del lab están en [`registros/registro-solicitudes-acceso.csv`](../registros/registro-solicitudes-acceso.csv).

## Joiner (alta)

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud de alta con la aprobación del responsable de Ventas (simulada) | Fila `SIM-001` del registro, con fecha y hora |
| 2 | Creo `a.baja` en `OU=Ventas` con una contraseña temporal y cambio obligatorio en el primer inicio de sesión. En *Descripción* indico que es un usuario simulado | Evento 4720 (cuenta de usuario creada) |
| 3 | Lo añado **solo** a `GG-Ventas` | Evento 4728 (miembro añadido a un grupo global de seguridad) |
| 4 | Compruebo la pestaña *Miembro de* | Captura 01 (ver [evidencias](../evidencias/README.md)) |
| 5 | Cierro la solicitud con la hora de ejecución | Registro actualizado |

## Mover (cambio de rol)

> **Estado:** procedimiento escrito; la evidencia llegará en la próxima iteración del lab.

| Paso | Qué hago | Por qué |
|---|---|---|
| 1 | Registro la solicitud con el nuevo rol y la aprobación del responsable del área de destino | Sin aprobación no hay cambio |
| 2 | Retiro los grupos del rol anterior y añado los del nuevo en la misma intervención | Evitar la acumulación de permisos: quien cambia de puesto no debe conservar los accesos del anterior |
| 3 | Muevo la cuenta a la OU del nuevo departamento, si aplica | Que la estructura de AD refleje la organización real |
| 4 | Compruebo *Miembro de*: solo grupos del rol nuevo | Verificación antes del cierre |
| 5 | Cierro la solicitud en el registro | Trazabilidad |

## Leaver (baja)

**Objetivo de plazo: menos de 24 horas** desde la solicitud hasta la revocación. Es el objetivo que fijo en este lab; en una empresa lo define su política de control de acceso.

| Paso | Qué hago | Qué queda como evidencia |
|---|---|---|
| 1 | Registro la solicitud de baja de RRHH (simulada) con fecha y hora | Fila `SIM-002` del registro |
| 2 | **Deshabilito** la cuenta (no la borro) | Evento 4725 (cuenta de usuario deshabilitada) |
| 3 | La retiro de `GG-Ventas` y de cualquier otro grupo de negocio | Evento 4729 (miembro quitado de un grupo global de seguridad) |
| 4 | Actualizo *Descripción* sin quitar el aviso: `Usuario simulado de laboratorio · Baja SIM-002 - AAAA-MM-DD` | Visible en la captura 02 |
| 5 | Compruebo el estado final | Capturas 02 y 03 (ver [evidencias](../evidencias/README.md)) |
| 6 | Anoto la hora de ejecución, calculo las horas transcurridas y marco si está dentro de plazo | Registro actualizado |

**¿Por qué deshabilitar y no borrar?** Una cuenta deshabilitada no permite iniciar sesión, pero conserva su identificador (SID), sus atributos y su relación con el historial, que es lo que se necesita en una auditoría o una investigación. Si se borrara y hubiera que volver a crearla, sería una cuenta distinta con otro SID. El borrado se hace más tarde, cuando lo indique la política de conservación.

**¿Por qué sigue apareciendo "Usuarios del dominio"?** Es el grupo principal que Active Directory asigna a todas las cuentas y no se puede quitar sin asignar otro grupo principal. En este lab no tiene permisos sobre recursos de Ventas; lo que impide usar la cuenta es que está deshabilitada.
