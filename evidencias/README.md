# Evidencias

> **⚠️ CASO SIMULADO.** `a.baja` es un usuario ficticio de laboratorio. No es una persona real. Todas las capturas son del dominio de pruebas `iamlab.local`.

Seis capturas de una única ejecución del laboratorio el **26/10/2026**, en Windows Server 2022 con la interfaz en inglés. Las horas son las del reloj de la máquina virtual, pasadas a formato de 24 horas. Ninguna imagen está editada.

Juntas responden a las tres preguntas de cualquier auditoría de accesos: **qué se concedió, qué se retiró y qué dice el sistema**.

## Línea temporal del 26/10/2026

| Hora | Qué ocurre | Evidencia |
|---|---|---|
| 14:20 | `a.baja` en `OU=Ventas`, habilitada; existen `GG-Ventas` y `GG-Ventas-RO` | `01-alta-joiner-OU-Ventas.png` |
| 14:21 | *Member Of*: `Domain Users` y `GG-Ventas` | `01-alta-joiner-memberof-GG-Ventas.png` |
| 14:25:49 | Evento 4725: cuenta de usuario deshabilitada | `03-auditoria-eventos-4725-4729.png` |
| 14:26:28 | Evento 4729: miembro retirado de un grupo global de seguridad | `03-auditoria-eventos-4725-4729.png` |
| 14:28 | *Member Of*: solo `Domain Users` | `02-baja-leaver-memberof-revocada.png` |
| 14:30 | *Account is disabled* marcada | `02-baja-leaver-Account-is-disabled.png` |
| 16:47:39 | Evento 4725 adicional, sin atribuir (H-03) | `03-auditoria-eventos-4725-4729.png` |
| 16:47 | `a.baja` con el icono de cuenta deshabilitada en la OU | `02-baja-leaver-OU-deshabilitada.png` |
| 17:00 | Captura del Event Viewer filtrado | `03-auditoria-eventos-4725-4729.png` |

**Lectura de auditor:** `GG-Ventas` aparece en la captura de las 14:21 y ya no en la de las 14:28, y entre ambas solo hay un evento 4729 (14:26:28). La secuencia es coherente. Lo que la captura 03 no demuestra por sí sola es que los eventos correspondan a `a.baja`, porque no se ve la cuenta de destino (hallazgo H-03).

## Capturas

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
- **Hallazgo H-01:** también se ve *Password never expires* marcada. Contradice el procedimiento de alta y afecta a A.5.17 (Información de autenticación).

### 02-baja-leaver-OU-deshabilitada.png · 16:47

- **Qué se ve:** la OU `Ventas` con `a.baja` mostrando el icono de cuenta deshabilitada (flecha hacia abajo) y el aviso de caso simulado en *Description*.
- **Pregunta de auditoría:** ¿el estado final es visible para cualquiera que revise la OU?
- **Control:** A.5.18 Derechos de acceso.

### 03-auditoria-eventos-4725-4729.png · 17:00

- **Qué se ve:** *Event Viewer → Windows Logs → Security* filtrado. Aparecen tres eventos: 4725 a las 14:25:49, 4729 a las 14:26:28 y 4725 a las 16:47:39. Debajo, el detalle de un 4725 (*A user account was disabled*).
- **Pregunta de auditoría:** ¿cuándo se ejecutó la baja y lo confirma el sistema, no solo la operadora?
- **Control:** A.8.15 Registros de eventos.
- **Hallazgo H-03:** la captura no muestra la cuenta de destino (*Target Account*) ni quién ejecutó la acción (*Subject*), no incluye los eventos del alta (4720 y 4728) y el 4725 de las 16:47:39 está sin atribuir.

## Hallazgos detectados en las evidencias

| ID | Resumen | Estado |
|---|---|---|
| H-01 | *Password never expires* marcada en `a.baja` | Abierto |
| H-02 | Hora de la solicitud de baja no anotada: el plazo de 24 h no se puede demostrar | Abierto |
| H-03 | Captura 03 sin cuenta de destino, sin eventos de alta y con un 4725 sin atribuir | Abierto |

El detalle y las acciones correctoras están en el [README principal](../README.md#hallazgos-de-la-autoauditoría).

## Reglas para nuevas capturas

- Reloj de la máquina virtual visible.
- Aviso de caso simulado visible en *Description* cuando la captura muestre la cuenta.
- Solo la ventana de la máquina virtual: sin escritorio del equipo anfitrión, notificaciones ni datos personales reales.
- Sin retoques: una evidencia editada deja de ser evidencia.
- En el Event Viewer, el detalle del evento debe mostrar *Subject* y *Target Account*.
