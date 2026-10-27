# Evidencias

> **⚠️ CASO SIMULADO.** `a.baja` es un usuario ficticio de laboratorio. No es una persona real. Todas las capturas son del dominio de pruebas `iamlab.local`.

Nueve capturas de Windows Server 2022 (interfaz en inglés): seis de la ejecución del ciclo el **26/10/2026** y tres de la autoauditoría del **27/10/2026**. Las horas son las del reloj de la máquina virtual, pasadas a formato de 24 horas. Ninguna imagen está editada.

Juntas responden a las tres preguntas de cualquier auditoría de accesos: **qué se concedió, qué se retiró y qué dice el sistema**.

## Línea temporal

Eventos del registro de Seguridad filtrados por `a.baja` ([03-linea-temporal-a.baja.png](03-linea-temporal-a.baja.png)), cruzados con la hora de cada captura.

| Fecha y hora | Qué ocurre | Evidencia |
|---|---|---|
| 26/10 14:00:03 | Cuenta creada (4720) y habilitada (4722), con tres 4738 de atributos iniciales. `whenCreated` confirma la hora | `03-linea-temporal-a.baja.png` |
| 26/10 14:14:59 | Cuenta modificada (4738; el detalle del cambio no está capturado) | `03-linea-temporal-a.baja.png` |
| 26/10 14:17:26 | Añadida a un grupo global de seguridad (4728): `GG-Ventas`, su único grupo de negocio | `03-linea-temporal-a.baja.png` |
| 26/10 14:20 | `a.baja` en `OU=Ventas`, habilitada; existen `GG-Ventas` y `GG-Ventas-RO` | `01-alta-joiner-OU-Ventas.png` |
| 26/10 14:21 | *Member Of*: `Domain Users` y `GG-Ventas` | `01-alta-joiner-memberof-GG-Ventas.png` |
| 26/10 14:25:49 | Cuenta deshabilitada (4725, con su 4738) | `03-linea-temporal-a.baja.png` · `03-auditoria-eventos-4725-4729.png` |
| 26/10 14:26:28 | Retirada del grupo global (4729) | `03-linea-temporal-a.baja.png` · `03-auditoria-eventos-4725-4729.png` |
| 26/10 14:28 | *Member Of*: solo `Domain Users` | `02-baja-leaver-memberof-revocada.png` |
| 26/10 14:30 | *Account is disabled* marcada | `02-baja-leaver-Account-is-disabled.png` |
| 26/10 16:47:39 | Deshabilitación reaplicada sobre la cuenta ya deshabilitada (4725, con su 4738). No hay ningún 4722 entre las 14:25 y las 16:47: la cuenta no se rehabilitó | `03-linea-temporal-a.baja.png` |
| 26/10 16:47 | `a.baja` con el icono de cuenta deshabilitada en la OU | `02-baja-leaver-OU-deshabilitada.png` |
| 26/10 17:00 | Captura del Event Viewer filtrado | `03-auditoria-eventos-4725-4729.png` |
| 27/10 08:37:52 | Corrección de H-01: *Password never expires* desmarcada (4738) | `03-linea-temporal-a.baja.png` |
| 27/10 08:40 | `Get-ADUser` antes (`True`) y después (`False`) de la corrección | `04-correccion-H01-PasswordNeverExpires-False.png` |
| 27/10 08:43 | Revisión `SIM-003`: `GG-Ventas` sin miembros | `04-revision-SIM-003-GG-Ventas-vacia.png` |
| 27/10 09:05 | Exportación de la línea temporal y de `whenCreated` | `03-linea-temporal-a.baja.png` |

**Lectura de auditor:** cada captura de estado encaja con un evento del sistema. `GG-Ventas` se añade a las 14:17:26 y aparece en la captura de las 14:21; se retira a las 14:26:28 y ya no está en la de las 14:28. El único 4722 de toda la línea temporal es el de la creación, así que después de la baja la cuenta no volvió a estar habilitada en ningún momento.

## Capturas del 26/10/2026: ejecución del ciclo

### 01-alta-joiner-OU-Ventas.png · 14:20

- **Qué se ve:** Active Directory Users and Computers con la OU `Ventas` abierta: `a.baja` (habilitada) y los grupos de seguridad `GG-Ventas` y `GG-Ventas-RO`. La columna *Description* muestra el aviso de caso simulado.
- **Pregunta de auditoría:** ¿la cuenta existe donde dice el procedimiento y está identificada?
- **Control:** A.5.16 Gestión de identidades.

### 01-alta-joiner-memberof-GG-Ventas.png · 14:21

- **Qué se ve:** *a.baja Properties → Member Of*: `Domain Users` (grupo principal) y `GG-Ventas`. Nada más.
- **Pregunta de auditoría:** ¿se concedió solo lo aprobado?
- **Control:** A.5.18 Derechos de acceso.
- **Por qué es crítica:** es la línea base. Sin saber con certeza qué accesos se dieron, no se puede asegurar después que la baja los retiró todos.

### 02-baja-leaver-memberof-revocada.png · 14:28

- **Qué se ve:** *Member Of* con solo `Domain Users` y *Primary group: Domain Users*. `GG-Ventas` ya no está.
- **Pregunta de auditoría:** ¿se retiraron todos los accesos de negocio?
- **Control:** A.5.18 Derechos de acceso.
- **Nota:** `Domain Users` no se puede quitar porque es el grupo principal (ver [docs/01](../docs/01-Joiner-Mover-Leaver.md#leaver-baja)).

### 02-baja-leaver-Account-is-disabled.png · 14:30

- **Qué se ve:** pestaña *Account* con *Account is disabled* marcada y *Account expires: Never*. Al fondo, la pantalla de bienvenida de Microsoft Edge de la máquina virtual, sin datos personales.
- **Pregunta de auditoría:** ¿la cuenta puede iniciar sesión?
- **Control:** A.5.18 Derechos de acceso.
- **Hallazgo H-01:** también se ve *Password never expires* marcada. Corregido el 27/10/2026 (ver `04-correccion-H01-PasswordNeverExpires-False.png`).

### 02-baja-leaver-OU-deshabilitada.png · 16:47

- **Qué se ve:** la OU `Ventas` con `a.baja` mostrando el icono de cuenta deshabilitada (flecha hacia abajo) y el aviso de caso simulado en *Description*.
- **Pregunta de auditoría:** ¿el estado final es visible para cualquiera que revise la OU?
- **Control:** A.5.18 Derechos de acceso.

### 03-auditoria-eventos-4725-4729.png · 17:00

- **Qué se ve:** *Event Viewer → Windows Logs → Security* filtrado, con tres eventos: 4725 a las 14:25:49, 4729 a las 14:26:28 y 4725 a las 16:47:39. Debajo, el detalle de un 4725 (*A user account was disabled*).
- **Pregunta de auditoría:** ¿cuándo se ejecutó la baja y lo confirma el sistema, no solo la operadora?
- **Control:** A.8.15 Registros de eventos.
- **Limitación:** no se ve la cuenta de destino. La atribución a `a.baja` la aporta la línea temporal del 27/10 (H-03, cerrado).

## Capturas del 27/10/2026: autoauditoría

### 03-linea-temporal-a.baja.png · 09:05

- **Qué se ve:** PowerShell con `Get-WinEvent -FilterHashtable` sobre el registro de Seguridad, filtrado por los ID 4720, 4722, 4725, 4728, 4729 y 4738 y por el texto `a.baja`, ordenado por fecha, con la descripción de cada evento. Debajo, `Get-ADUser` con `whenCreated` (26/10/2026 14:00:03).
- **Pregunta de auditoría:** ¿qué le pasó a esta cuenta, cuándo y en qué orden, según el sistema?
- **Control:** A.8.15 Registros de eventos y A.5.16 Gestión de identidades.
- **Nota:** en la parte superior se ve un intento anterior con `-MaxEvents 200`. Ese comando solo leía los 200 eventos más recientes del registro y por eso no encontraba los del día anterior. La línea temporal válida es la tabla con la columna *Evento*.

### 04-correccion-H01-PasswordNeverExpires-False.png · 08:40

- **Qué se ve:** `Get-ADUser a.baja` con *PasswordNeverExpires: True*, la corrección con `Set-ADUser -PasswordNeverExpires $false` y la misma consulta con *False*.
- **Pregunta de auditoría:** ¿se corrigió el hallazgo y hay prueba del antes y del después?
- **Control:** A.5.17 Información de autenticación.
- **Nota:** la primera línea, con una `\` delante de `Set-ADUser`, es un error de tecleo. El registro muestra un único 4738 el 27/10 (08:37:52), así que el cambio se aplicó una sola vez.

### 04-revision-SIM-003-GG-Ventas-vacia.png · 08:43

- **Qué se ve:** la misma consola y, al final, `Get-ADGroupMember GG-Ventas`, que no devuelve ningún miembro.
- **Pregunta de auditoría:** ¿alguien conserva un acceso que ya no le corresponde?
- **Control:** A.5.18 Derechos de acceso (revisión).
- **Resultado:** sin desviaciones. Ninguna cuenta dada de baja sigue en `GG-Ventas`.

## Estado de los hallazgos

| ID | Resumen | Estado |
|---|---|---|
| H-01 | *Password never expires* marcada en `a.baja` | ✅ Cerrado el 27/10/2026 |
| H-02 | Hora de la solicitud de baja no anotada: el plazo de 24 h no se puede demostrar | ⚠️ Abierto: se cierra en el próximo caso |
| H-03 | Captura del Event Viewer sin cuenta de destino ni eventos de alta, y un 4725 sin atribuir | ✅ Cerrado el 27/10/2026 |

El detalle está en el [README principal](../README.md#hallazgos-de-la-autoauditoría).

## Reglas para nuevas capturas

- Reloj de la máquina virtual visible.
- Aviso de caso simulado visible en *Description* cuando la captura muestre la cuenta.
- Solo la ventana de la máquina virtual: sin escritorio del equipo anfitrión, notificaciones ni datos personales reales.
- Sin retoques: una evidencia editada deja de ser evidencia.
- En el Event Viewer, el detalle del evento debe mostrar *Subject* y *Target Account*.
- Antes de ejecutar un cambio, la solicitud se anota en el registro con su hora.
