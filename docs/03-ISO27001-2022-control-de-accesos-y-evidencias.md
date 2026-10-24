# ISO/IEC 27001:2022: control de accesos, revisión y evidencias

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

## Nota sobre la versión de la norma

Este lab usa la numeración de **ISO/IEC 27001:2022**. En la versión de 2013 estos controles eran A.9.2.x y A.12.4.x; esa versión dejó de ser válida para certificar el 31 de octubre de 2025, aunque muchas guías la siguen citando. La equivalencia está en la tabla.

| Control (2022) | Qué exige, en resumen | Equivalencia 2013 | Dónde se ve en este repositorio |
|---|---|---|---|
| **A.5.16** Gestión de identidades | Gestionar el ciclo de vida completo de las identidades | A.9.2.1 | [docs/01](01-Joiner-Mover-Leaver.md) y el registro |
| **A.5.18** Derechos de acceso | Conceder, revisar, modificar y retirar los accesos según la política de control de acceso | A.9.2.2, A.9.2.5, A.9.2.6 | Capturas 01 y 02, revisión de este documento |
| **A.8.15** Registros de eventos | Generar, conservar y proteger los registros de actividad | A.12.4.1 a A.12.4.3 | Captura 03 |

## Qué pediría un auditor y dónde está

| Pregunta del auditor | Respuesta en este repositorio |
|---|---|
| ¿Hubo solicitud y aprobación? | [Registro](../registros/registro-solicitudes-acceso.csv): columnas `solicitante` y `aprobador` (simulados) |
| ¿Se concedió lo aprobado y nada más? | Captura 01: solo `GG-Ventas` |
| ¿Se retiró el acceso a tiempo? | Hora de la solicitud en el registro frente a la hora del evento 4725 en la captura 03 |
| ¿Lo confirma el sistema, no solo la operadora? | Captura 03: eventos generados por Windows |
| ¿Se revisan los accesos de forma periódica? | Revisión de `GG-Ventas` (abajo) y fila `SIM-003` del registro |

## Revisión de accesos de GG-Ventas

**Procedimiento:**

1. Frecuencia mensual.
2. Comparo los miembros actuales de `GG-Ventas` con las altas vigentes del registro.
3. Cualquier miembro sin un alta vigente que lo justifique es una desviación.
4. Anoto la fecha, el resultado y, si las hay, las desviaciones y su tratamiento.

**Ejecución en el lab:** la primera revisión de prueba se hace después de la baja de `a.baja` y se registra como `SIM-003`; el resultado se anota en la columna `observaciones` del registro.

## Ejercicio de mesa: ¿y si la revisión encuentra una cuenta huérfana?

*Supuesto no ejecutado:* la revisión mensual detecta que `a.baja` sigue en `GG-Ventas` tres días después de su fecha de baja.

1. Retiro el acceso en ese momento y lo registro como desviación.
2. Compruebo si la cuenta se usó después de la fecha de baja: eventos de inicio de sesión de `a.baja` en el registro de Seguridad (4768, solicitud de ticket Kerberos, en el controlador de dominio; 4624 en el equipo donde se inició sesión).
3. Si hubo uso, lo escalo como posible incidente de seguridad; si no, lo documento como desviación sin impacto.
4. Busco la causa: ¿no llegó la solicitud?, ¿se ejecutó a medias?, ¿falta un paso en el procedimiento?
5. Cierro con la acción correctora y lo compruebo en la siguiente revisión.
