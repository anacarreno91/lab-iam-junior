# ISO/IEC 27001:2022: control de accesos, revisión y evidencias

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

## Nota sobre la versión de la norma

Este lab usa la numeración de **ISO/IEC 27001:2022**.

| Control | Qué exige, en resumen | Dónde se ve en este repositorio |
|---|---|---|
| **A.5.16** Gestión de identidades | Gestionar el ciclo de vida completo de las identidades | [docs/01](01-Joiner-Mover-Leaver.md) y el [registro](../registros/registro-solicitudes-acceso.csv) |
| **A.5.18** Derechos de acceso | Conceder, revisar, modificar y retirar los accesos según la política de control de acceso | Capturas [01](../evidencias/01-alta-joiner-memberof-GG-Ventas.png) y [02](../evidencias/02-baja-leaver-memberof-revocada.png); revisión [SIM-003](../evidencias/04-revision-SIM-003-GG-Ventas-vacia.png) |
| **A.8.15** Registros de eventos | Generar, conservar y proteger los registros de actividad | Capturas [03 eventos](../evidencias/03-auditoria-eventos-4725-4729.png) y [03 línea temporal](../evidencias/03-linea-temporal-a.baja.png) |

## Qué pediría un auditor y dónde está

| Pregunta del auditor | Respuesta en este repositorio |
|---|---|
| ¿Hubo solicitud y aprobación? | [Registro](../registros/registro-solicitudes-acceso.csv): columnas `solicitante` y `aprobador` (simulados) |
| ¿Se registra la solicitud antes de actuar? | Desde `SIM-005`, sí: solicitud a las 07:39:10 y alta completa a las 07:45:39 (captura [05 alta](../evidencias/05-externo-alta-powershell.png)) |
| ¿Se concedió lo aprobado y nada más? | Captura 01 Member Of (14:21): `Domain Users` y `GG-Ventas`, nada más |
| ¿Se retiró el acceso a tiempo? | Ejecución a las 14:25:49 (4725) y 14:26:28 (4729). La hora de la solicitud no se anotó, así que el plazo de esta baja no se puede demostrar (H-02). Desde `SIM-005` la solicitud se registra antes de actuar |
| ¿Lo confirma el sistema, no solo la operadora? | Captura 03: eventos 4725 y 4729 generados por Windows, coherentes con las capturas de antes (14:21) y después (14:28). La [línea temporal](../evidencias/03-linea-temporal-a.baja.png) filtrada por `a.baja` confirma que son de esta cuenta e incluye también los eventos del alta (H-03, cerrado) |
| ¿Se revisan los accesos de forma periódica? | Sí: primera revisión el 27/10/2026, sin desviaciones (`SIM-003`, [captura](../evidencias/04-revision-SIM-003-GG-Ventas-vacia.png)) |

## Revisión de accesos de GG-Ventas

**Procedimiento:**

1. Frecuencia mensual.
2. Comparo los miembros actuales de `GG-Ventas` con las altas vigentes del registro.
3. Cualquier miembro sin un alta vigente que lo justifique es una desviación.
4. Anoto la fecha, el resultado y, si las hay, las desviaciones y su tratamiento.

**Ejecución en el lab:** primera revisión el 27/10/2026 a las 08:43 con `Get-ADGroupMember GG-Ventas`. Resultado: el grupo no tiene miembros, así que ninguna cuenta dada de baja conserva el acceso. Sin desviaciones. Registrada como `SIM-003` ([captura](../evidencias/04-revision-SIM-003-GG-Ventas-vacia.png)).

## Ejercicio de mesa: ¿y si la revisión encuentra una cuenta huérfana?

*Supuesto no ejecutado:* la revisión mensual detecta que `a.baja` sigue en `GG-Ventas` tres días después de su fecha de baja.

1. Retiro el acceso en ese momento y lo registro como desviación.
2. Compruebo si la cuenta se usó después de la fecha de baja, con los eventos de inicio de sesión de `a.baja` en el registro de Seguridad.
3. Si hubo uso, lo escalo como posible incidente de seguridad; si no, lo documento como desviación sin impacto.
4. Busco la causa: ¿no llegó la solicitud?, ¿se ejecutó a medias?, ¿falta un paso en el procedimiento?
5. Cierro con la acción correctora y lo compruebo en la siguiente revisión.
