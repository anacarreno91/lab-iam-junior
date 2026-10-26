# ISO/IEC 27001:2022: control de accesos, revisión y evidencias

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

## Nota sobre la versión de la norma

Este lab usa la numeración de **ISO/IEC 27001:2022**. En la versión de 2013 estos controles eran A.9.2.x y A.12.4.x; esa versión dejó de ser válida para certificar el 31 de octubre de 2025, aunque muchas guías la siguen citando. La equivalencia está en la tabla.

| Control (2022) | Qué exige, en resumen | Equivalencia 2013 | Dónde se ve en este repositorio |
|---|---|---|---|
| **A.5.16** Gestión de identidades | Gestionar el ciclo de vida completo de las identidades | A.9.2.1 | [docs/01](01-Joiner-Mover-Leaver.md) y el [registro](../registros/registro-solicitudes-acceso.csv) |
| **A.5.18** Derechos de acceso | Conceder, revisar, modificar y retirar los accesos según la política de control de acceso | A.9.2.2, A.9.2.5, A.9.2.6 | Capturas [01](../evidencias/01-alta-joiner-memberof-GG-Ventas.png) y [02](../evidencias/02-baja-leaver-memberof-revocada.png); revisión de este documento (pendiente) |
| **A.8.15** Registros de eventos | Generar, conservar y proteger los registros de actividad | A.12.4.1 a A.12.4.3 | Captura [03](../evidencias/03-auditoria-eventos-4725-4729.png) |

## Qué pediría un auditor y dónde está

| Pregunta del auditor | Respuesta en este repositorio |
|---|---|
| ¿Hubo solicitud y aprobación? | [Registro](../registros/registro-solicitudes-acceso.csv): columnas `solicitante` y `aprobador` (simulados) |
| ¿Se concedió lo aprobado y nada más? | Captura 01 Member Of (14:21): `Domain Users` y `GG-Ventas`, nada más |
| ¿Se retiró el acceso a tiempo? | Ejecución a las 14:25:49 (4725) y 14:26:28 (4729). La hora de la solicitud no se anotó, así que el plazo todavía no se puede demostrar (H-02) |
| ¿Lo confirma el sistema, no solo la operadora? | Captura 03: eventos 4725 y 4729 generados por Windows, coherentes con las capturas de antes (14:21) y después (14:28). La captura no muestra la cuenta de destino; la atribución a `a.baja` se completará con la línea temporal exportada (H-03) |
| ¿Se revisan los accesos de forma periódica? | Procedimiento escrito (abajo); primera ejecución pendiente (`SIM-003`) |

## Revisión de accesos de GG-Ventas

**Procedimiento:**

1. Frecuencia mensual.
2. Comparo los miembros actuales de `GG-Ventas` con las altas vigentes del registro.
3. Cualquier miembro sin un alta vigente que lo justifique es una desviación.
4. Anoto la fecha, el resultado y, si las hay, las desviaciones y su tratamiento.

**Ejecución en el lab:** pendiente. La primera revisión se registrará como `SIM-003`, con el resultado en la columna `observaciones` del registro.

## Ejercicio de mesa: ¿y si la revisión encuentra una cuenta huérfana?

*Supuesto no ejecutado:* la revisión mensual detecta que `a.baja` sigue en `GG-Ventas` tres días después de su fecha de baja.

1. Retiro el acceso en ese momento y lo registro como desviación.
2. Compruebo si la cuenta se usó después de la fecha de baja: eventos de inicio de sesión de `a.baja` en el registro de Seguridad (4768, solicitud de ticket Kerberos, en el controlador de dominio; 4624 en el equipo donde se inició sesión).
3. Si hubo uso, lo escalo como posible incidente de seguridad; si no, lo documento como desviación sin impacto.
4. Busco la causa: ¿no llegó la solicitud?, ¿se ejecutó a medias?, ¿falta un paso en el procedimiento?
5. Cierro con la acción correctora y lo compruebo en la siguiente revisión.
