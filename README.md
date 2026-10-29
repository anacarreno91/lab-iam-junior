# lab-iam-junior · Gestión de identidades y accesos (IAM) en Active Directory con evidencias para auditoría

> **⚠️ CASO SIMULADO.** `a.baja` y `ext.mantenimiento` son usuarios ficticios creados para practicar altas y bajas, y el proveedor también es ficticio. **No corresponden a ninguna persona ni empresa real.** El dominio `iamlab.local` es un laboratorio personal en VirtualBox, sin datos de ninguna empresa.

Laboratorio personal para practicar tareas de operación IAM de nivel 1 en Active Directory: altas y bajas de usuarios, permisos asignados por grupos (RBAC) y registro de cada acción con su evidencia (capturas, eventos de Windows y un registro de solicitudes). También incluye la revisión que hice de mi propia ejecución y los fallos que encontré.

## Resumen

| Qué se demuestra | Cómo | Evidencia |
|---|---|---|
| **Alta con mínimo privilegio** (Joiner) | `a.baja` creada a las 14:00:03 (4720) y añadida solo a `GG-Ventas` a las 14:17:26 (4728) | [01 OU](evidencias/01-alta-joiner-OU-Ventas.png) · [01 Member Of](evidencias/01-alta-joiner-memberof-GG-Ventas.png) |
| **Baja y revocación** (Leaver) | Deshabilitada a las 14:25:49 (4725) y retirada de `GG-Ventas` a las 14:26:28 (4729) | [02 Member Of](evidencias/02-baja-leaver-memberof-revocada.png) · [02 Account](evidencias/02-baja-leaver-Account-is-disabled.png) · [02 OU](evidencias/02-baja-leaver-OU-deshabilitada.png) |
| **Trazabilidad completa** | Línea temporal de `a.baja` extraída del registro de Seguridad con PowerShell, coherente con todas las capturas | [03 línea temporal](evidencias/03-linea-temporal-a.baja.png) · [03 eventos](evidencias/03-auditoria-eventos-4725-4729.png) · [cronología](evidencias/README.md#línea-temporal) |
| **Revisión de accesos** | `SIM-003`: `GG-Ventas` sin miembros; ninguna cuenta dada de baja conserva el acceso | [04 revisión](evidencias/04-revision-SIM-003-GG-Ventas-vacia.png) |
| **Usuario externo con caducidad** | `ext.mantenimiento` en `OU=Proveedores`, solo en `GG-Mantenimiento-Externo` y con caducidad a 30 días. Solicitud registrada antes de ejecutar (07:39:10) | [05 alta](evidencias/05-externo-alta-powershell.png) · [05 caducidad](evidencias/05-externo-account-expires.png) · [05 eventos](evidencias/05-externo-caducidad-y-eventos.png) |
| **Autoauditoría** | 3 hallazgos detectados en mis propias evidencias, los 3 cerrados con evidencia | [hallazgos](#hallazgos-de-la-autoauditoría) |
| **RBAC** | Permisos por grupo de rol: `GG-Ventas` y `GG-Ventas-RO` | [docs/02](docs/02-RBAC-y-minimo-privilegio.md) |
| **ISO/IEC 27001:2022** | Controles A.5.16, A.5.18 y A.8.15 (en la versión 2013: A.9.2 y A.12.4) | [docs/03](docs/03-ISO27001-2022-control-de-accesos-y-evidencias.md) |

<p>
<img src="evidencias/01-alta-joiner-memberof-GG-Ventas.png" width="32%" alt="01 · Tras el alta: Domain Users y GG-Ventas">
<img src="evidencias/02-baja-leaver-memberof-revocada.png" width="32%" alt="02 · Tras la baja: solo Domain Users">
<img src="evidencias/03-auditoria-eventos-4725-4729.png" width="32%" alt="03 · Event Viewer: eventos 4725 y 4729">
</p>

## Por qué hice este lab

Trabajo en operaciones, en remoto, en una empresa de eficiencia energética. Mi día a día es gestionar expedientes y reclamaciones con plazo: registrar cada solicitud, hacer el seguimiento hasta el cierre y mantener la documentación localizable en Excel y SharePoint.

Una petición de alta o de baja de accesos es el mismo tipo de trabajo: llega una solicitud, se ejecuta dentro de plazo, se registra y se puede demostrar después. Este laboratorio aplica esa forma de trabajar a la gestión de identidades.

## Entorno

- **Dominio:** `iamlab.local`, con un controlador de dominio Windows Server 2022 en VirtualBox. La interfaz está en inglés, así que la documentación cita los nombres tal como aparecen en pantalla (*Member Of*, *Domain Users*, *Account is disabled*).
- **Unidades organizativas:** `OU=Ventas` (personal interno) y `OU=Proveedores` (personal externo).
- **Grupos de seguridad globales:** `GG-Ventas` (rol Ventas), `GG-Ventas-RO` (rol de solo lectura) y `GG-Mantenimiento-Externo` (proveedor de mantenimiento).
- **Usuarios de prueba:** `a.baja` (empleado) y `ext.mantenimiento` (proveedor externo), ambos simulados. Su campo *Description* en AD lo indica.
- **Herramientas:** Active Directory Users and Computers, Event Viewer, PowerShell (módulo de Active Directory y `Get-WinEvent`), VirtualBox y un registro de solicitudes en CSV que se abre en Excel.

## Casos

| # | Caso | Estado | Detalle |
|---|---|---|---|
| 1 | Joiner: alta de `a.baja` en Ventas | Hecho, con capturas y eventos 4720/4728 | [docs/01](docs/01-Joiner-Mover-Leaver.md#joiner-alta) |
| 2 | Leaver: baja y revocación | Hecho, con capturas y eventos 4725/4729. Plazo no demostrable (H-02) | [docs/01](docs/01-Joiner-Mover-Leaver.md#leaver-baja) |
| 3 | Revisión de accesos de `GG-Ventas` | Hecha el 27/10/2026, sin desviaciones (`SIM-003`) | [docs/03](docs/03-ISO27001-2022-control-de-accesos-y-evidencias.md#revisión-de-accesos-de-gg-ventas) |
| 4 | Mover: cambio de rol | Solo procedimiento escrito, sin ejecutar | [docs/01](docs/01-Joiner-Mover-Leaver.md#mover-cambio-de-rol) |
| 5 | Alta de usuario externo con caducidad | Hecho el 29/10/2026, con la solicitud registrada antes de ejecutar (`SIM-005`) | [docs/01](docs/01-Joiner-Mover-Leaver.md#usuario-externo-alta-con-caducidad) |

## Hallazgos de la autoauditoría

Al revisar mis capturas y el registro de eventos encontré tres fallos en mi propia ejecución:

| ID | Hallazgo | Estado | Cierre |
|---|---|---|---|
| H-01 | `a.baja` tenía marcada *Password never expires*. Contradecía mi procedimiento de alta (contraseña temporal con cambio obligatorio) y, en un entorno real, sería una desviación de la política de contraseñas (A.5.17). | Cerrado el 27/10/2026 | Corregido con `Set-ADUser` (`SIM-004`). Evidencia: estado antes y después en [04 corrección](evidencias/04-correccion-H01-PasswordNeverExpires-False.png) y evento 4738 a las 08:37:52 en la [línea temporal](evidencias/03-linea-temporal-a.baja.png). |
| H-02 | No anoté la hora de la solicitud de baja simulada (`SIM-002`) antes de ejecutarla, así que el plazo de 24 h de esa baja no se puede demostrar con evidencia. | Cerrado el 29/10/2026 | La hora de `SIM-002` no se corrige hacia atrás, porque sería falsear la evidencia. Corregí el procedimiento: en `SIM-005` registré la solicitud a las 07:39:10, antes de cualquier cambio, y el alta quedó completa a las 07:45:39 ([05 alta](evidencias/05-externo-alta-powershell.png)). |
| H-03 | La captura del Event Viewer no mostraba la cuenta de destino ni los eventos del alta, y tenía un segundo 4725 a las 16:47:39 sin explicar. | Cerrado el 27/10/2026 | Línea temporal exportada con PowerShell y filtrada por `a.baja` ([03 línea temporal](evidencias/03-linea-temporal-a.baja.png)). El 4725 de las 16:47:39 es de `a.baja` y no tiene un 4722 (cuenta habilitada) antes: la deshabilitación se volvió a aplicar sobre una cuenta que ya estaba deshabilitada, y no hubo ninguna ventana de acceso. |

## Mapeo con ISO/IEC 27001:2022

| Control (2022) | Qué exige, en resumen | Qué hago en el lab | Equivalencia 2013 |
|---|---|---|---|
| **A.5.16** Gestión de identidades | Gestionar el ciclo de vida completo de las identidades | Alta y baja de `a.baja` y alta con caducidad de `ext.mantenimiento`, con cada paso registrado | A.9.2.1 |
| **A.5.18** Derechos de acceso | Conceder, revisar, modificar y retirar los accesos según la política de control de acceso | Alta solo en `GG-Ventas`, retirada en la baja y revisión del grupo (`SIM-003`) | A.9.2.2, A.9.2.5, A.9.2.6 |
| **A.8.15** Registros de eventos | Generar, conservar y proteger los registros de actividad | Línea temporal de `a.baja` en el registro de Seguridad: 4720, 4722, 4728, 4725, 4729 y 4738 | A.12.4.1 a A.12.4.3 |

La versión 2013 de la norma dejó de ser válida para certificar el 31 de octubre de 2025. Este repositorio usa la numeración de 2022 e indica la antigua solo como referencia.

## Relación con entornos industriales (OT)

El laboratorio es un entorno IT. No tengo experiencia en entornos industriales; lo que aplico de mi formación en ciberseguridad OT es esto:

- En OT la prioridad es la disponibilidad. Una cuenta que no se revoca a tiempo, por ejemplo la de un técnico externo que ya terminó su trabajo, es un riesgo para la continuidad del proceso. Por eso el objetivo de baja en menos de 24 horas, la revisión periódica y, para el personal externo, una fecha de caducidad en la propia cuenta.
- Las cuentas compartidas impiden saber quién hizo cada cambio, así que cada persona debe tener su propia cuenta. En este lab todavía administro con la cuenta Administrator integrada (ver límites).
- La Directiva NIS2 (UE) 2022/2555 incluye las políticas de control de acceso entre las medidas de gestión de riesgos de ciberseguridad (art. 21.2.i) y sitúa la energía entre los sectores de alta criticidad (anexo I).

## Límites del laboratorio

- Un solo controlador de dominio, sin usuarios reales y sin conexión con ningún sistema de producción.
- Solicitudes y aprobaciones simuladas. En una empresa llegarían por ticket desde RRHH o desde el responsable del área.
- Ejecuto y verifico yo misma. En un entorno real la verificación la haría otra persona (segregación de funciones, ISO/IEC 27001:2022 A.5.3).
- Las acciones se hacen con la cuenta Administrator integrada del dominio. En un entorno real se usaría una cuenta de administración nominal, para que cada cambio quede asociado a una persona.
- Solo herramientas nativas de Windows Server: no se usa ninguna plataforma de gobierno de identidades (IGA) ni de accesos privilegiados (PAM).
- Ejecución del ciclo el 26/10/2026; autoauditoría, corrección y primera revisión el 27/10/2026; alta del usuario externo el 29/10/2026.

## Próximas iteraciones

- Caso Mover: un usuario simulado que pasa de `GG-Ventas` a `GG-Ventas-RO`.
- Carpeta compartida de Ventas con permisos NTFS asignados a `GG-Ventas` (modificar) y `GG-Ventas-RO` (lectura).
- Administrar con una cuenta nominal en lugar de la cuenta Administrator integrada.
- Siguiente revisión mensual de `GG-Ventas`.

---

Ana María Carreño · Murcia · [LinkedIn](https://www.linkedin.com/in/anacarrenonline)
