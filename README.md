# lab-iam-junior · Gestión de identidades y accesos (IAM) en Active Directory con evidencias para auditoría

> **⚠️ CASO SIMULADO.** `a.baja` es un usuario ficticio creado para practicar un proceso de baja (*offboarding*). **No corresponde a ninguna persona real.** El dominio `iamlab.local` es un laboratorio personal en VirtualBox, sin datos de ninguna empresa.

Laboratorio de operaciones IAM de nivel 1: altas, cambios y bajas de usuarios en Active Directory y permisos asignados por grupos (RBAC), con una regla que se aplica a todo: **cada acción deja una evidencia que un auditor puede comprobar sin tener que preguntarme**.

## En 30 segundos

| Qué se demuestra | Cómo | Evidencia |
|---|---|---|
| **Alta con mínimo privilegio** (Joiner) | `a.baja` creado en `OU=Ventas` y añadido solo a `GG-Ventas` | Captura 01 · [evidencias](evidencias/README.md) |
| **Baja y revocación** (Leaver) | Cuenta deshabilitada y retirada de `GG-Ventas` | Captura 02 · [evidencias](evidencias/README.md) |
| **Trazabilidad en menos de 24 h** | Hora de la solicitud en el registro frente a la hora de los eventos 4725 y 4729 de Windows | Captura 03 · [evidencias](evidencias/README.md) · [registro](registros/registro-solicitudes-acceso.csv) |
| **Revisión de accesos** | Miembros de `GG-Ventas` comparados con las altas vigentes | [docs/03](docs/03-ISO27001-2022-control-de-accesos-y-evidencias.md#revisión-de-accesos-de-gg-ventas) |
| **RBAC** | Los permisos se dan a grupos que representan un rol, nunca a usuarios sueltos | [docs/02](docs/02-RBAC-y-minimo-privilegio.md) |
| **ISO/IEC 27001:2022** | Controles A.5.16, A.5.18 y A.8.15 (en la versión 2013: A.9.2 y A.12.4) | [docs/03](docs/03-ISO27001-2022-control-de-accesos-y-evidencias.md) |

> **Estado de las evidencias:** las tres capturas se publicarán en [`evidencias/`](evidencias/README.md), donde se detalla qué debe verse en cada una y qué pregunta de auditoría responde.

## Por qué hice este lab

Trabajo en operaciones, en remoto, en una empresa de eficiencia energética. Mi día a día es gestionar expedientes y reclamaciones con plazo: registrar cada solicitud, hacer el seguimiento hasta el cierre y mantener la documentación localizable en Excel y SharePoint.

Una petición de alta o de baja de accesos es el mismo tipo de trabajo: llega una solicitud, se ejecuta dentro de plazo, se registra y se puede demostrar después. Este laboratorio aplica esa forma de trabajar a la gestión de identidades.

## Entorno

- **Dominio:** `iamlab.local`, con un controlador de dominio Windows Server en VirtualBox.
- **Unidad organizativa:** `OU=Ventas,DC=iamlab,DC=local`.
- **Grupos de seguridad globales:** `GG-Ventas` (rol Ventas) y `GG-Ventas-RO` (rol de solo lectura).
- **Usuario de prueba:** `a.baja`, simulado. Su campo *Descripción* en AD lo indica, de modo que el aviso aparece en todas las capturas.
- **Herramientas:** Usuarios y equipos de Active Directory, Visor de eventos de Windows, VirtualBox y un registro de solicitudes en CSV que se abre en Excel.

## Casos

| # | Caso | Estado | Detalle |
|---|---|---|---|
| 1 | Joiner: alta de `a.baja` en Ventas | ✅ Ejecutado en el lab · captura pendiente de publicar | [docs/01](docs/01-Joiner-Mover-Leaver.md#joiner-alta) |
| 2 | Leaver: baja y revocación en menos de 24 h | ✅ Ejecutado en el lab · capturas pendientes de publicar | [docs/01](docs/01-Joiner-Mover-Leaver.md#leaver-baja) |
| 3 | Revisión de accesos de `GG-Ventas` | 📄 Procedimiento escrito · primera ejecución pendiente de registrar (`SIM-003`) | [docs/03](docs/03-ISO27001-2022-control-de-accesos-y-evidencias.md#revisión-de-accesos-de-gg-ventas) |
| 4 | Mover: cambio de rol | 📄 Procedimiento escrito, sin evidencia todavía | [docs/01](docs/01-Joiner-Mover-Leaver.md#mover-cambio-de-rol) |

## Mapeo con ISO/IEC 27001:2022

| Control (2022) | Qué exige, en resumen | Qué hago en el lab | Equivalencia 2013 |
|---|---|---|---|
| **A.5.16** Gestión de identidades | Gestionar el ciclo de vida completo de las identidades | Alta, cambio y baja de `a.baja`, con cada paso registrado | A.9.2.1 |
| **A.5.18** Derechos de acceso | Conceder, revisar, modificar y retirar los accesos según la política de control de acceso | Alta solo en `GG-Ventas`, revisión del grupo y retirada en la baja | A.9.2.2, A.9.2.5, A.9.2.6 |
| **A.8.15** Registros de eventos | Generar, conservar y proteger los registros de actividad | Eventos 4720, 4728, 4725 y 4729 del registro de Seguridad | A.12.4.1 a A.12.4.3 |

La versión 2013 de la norma dejó de ser válida para certificar el 31 de octubre de 2025. Este repositorio usa la numeración de 2022 e indica la antigua solo como referencia.

## Por qué esto importa en entornos industriales (OT)

Este laboratorio es un entorno IT (Active Directory). Mi relación con OT es de criterio, no de experiencia en planta. Lo que traslado:

- **En OT manda la disponibilidad.** Una cuenta que nadie revoca, como la de un técnico de un proveedor que ya terminó su trabajo, es un riesgo directo para la continuidad del proceso. Por eso el objetivo de baja en menos de 24 horas y la revisión periódica.
- **Cuentas nominales, no compartidas.** Si varias personas usan la misma cuenta, el registro no permite saber quién hizo qué. En este lab cada acción queda asociada a una cuenta y a una hora.
- **Marco normativo.** La Directiva NIS2 (UE) 2022/2555 incluye las políticas de control de acceso entre las medidas de gestión de riesgos de ciberseguridad (art. 21.2.i) y sitúa la energía entre los sectores de alta criticidad (anexo I).

## Límites del laboratorio

- Un solo controlador de dominio, sin usuarios reales y sin conexión con ningún sistema de producción.
- Solicitudes y aprobaciones simuladas. En una empresa llegarían por ticket desde RRHH o desde el responsable del área.
- Ejecuto y verifico yo misma. En un entorno real la verificación la haría otra persona (segregación de funciones, ISO/IEC 27001:2022 A.5.3).
- Solo herramientas nativas de Windows Server: no se usa ninguna plataforma de gobierno de identidades (IGA) ni de accesos privilegiados (PAM).
- La revisión de accesos tiene definida una frecuencia mensual; la primera ejecución de prueba se registra como `SIM-003`.

## Próximas iteraciones

- Caso Mover con evidencia (`04-cambio-de-rol.png`).
- Carpeta compartida de Ventas con permisos NTFS asignados a `GG-Ventas` (modificar) y `GG-Ventas-RO` (lectura).
- Cuenta de un proveedor externo con fecha de caducidad (*La cuenta expira*) dentro de un grupo de acceso remoto para mantenimiento: el caso más parecido a un acceso en OT.

## Estructura del repositorio

```
lab-iam-junior/
├── README.md
├── docs/
│   ├── 01-Joiner-Mover-Leaver.md
│   ├── 02-RBAC-y-minimo-privilegio.md
│   └── 03-ISO27001-2022-control-de-accesos-y-evidencias.md
├── evidencias/
│   └── README.md
├── registros/
│   └── registro-solicitudes-acceso.csv
└── .gitignore
```

---

Ana María Carreño · Murcia · [LinkedIn](https://www.linkedin.com/in/anacarrenonline)
