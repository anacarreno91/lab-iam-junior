# RBAC y principio de mínimo privilegio

> **Caso simulado:** `a.baja` es un usuario ficticio de laboratorio. No es una persona real.

## La regla

Los permisos se asignan a **grupos que representan un rol**, nunca a usuarios concretos. Una persona tiene un acceso solo porque pertenece al grupo de su rol.

Por qué importa en operación y en auditoría:

- **Revisar es posible:** la lista de miembros de un grupo dice exactamente quién tiene ese acceso.
- **Revocar es completo:** sacar a alguien del grupo le retira todo lo que ese rol le daba, sin tener que buscar permisos sueltos.

## Roles del lab

| Grupo | Tipo | Rol | Estado |
|---|---|---|---|
| `GG-Ventas` | Seguridad, ámbito global | Personal de Ventas | ✅ Creado y en uso ([01 OU](../evidencias/01-alta-joiner-OU-Ventas.png)) |
| `GG-Ventas-RO` | Seguridad, ámbito global | Consulta (solo lectura) | ✅ Creado ([01 OU](../evidencias/01-alta-joiner-OU-Ventas.png)), sin uso todavía |

El ámbito global de `GG-Ventas` lo confirma el propio registro: al retirar a `a.baja`, Windows generó el evento 4729, que es el de los grupos globales de seguridad. En un grupo local de dominio habría sido el 4733.

**Nomenclatura:** `GG-` indica grupo global y `-RO` (*read only*) indica solo lectura. Sigue el modelo clásico de Microsoft para Active Directory (AGDLP): las cuentas se incluyen en grupos globales por rol, y los permisos sobre los recursos se conceden a grupos locales de dominio que contienen esos grupos globales. En este lab está implementado el primer nivel: cuentas en grupos globales.

## Mínimo privilegio aplicado a `a.baja`

| Momento | Member Of | Estado de la cuenta | Evidencia |
|---|---|---|---|
| Tras el alta | `Domain Users`, `GG-Ventas` | Habilitada | [01 Member Of](../evidencias/01-alta-joiner-memberof-GG-Ventas.png) (14:21) |
| Tras la baja | `Domain Users` | Deshabilitada | [02 Member Of](../evidencias/02-baja-leaver-memberof-revocada.png) (14:28) y [02 Account](../evidencias/02-baja-leaver-Account-is-disabled.png) (14:30) |

`Domain Users` es el grupo principal de todas las cuentas de AD y no se puede quitar sin asignar otro (explicado en [docs/01](01-Joiner-Mover-Leaver.md#leaver-baja)).

## Pendiente

- Asignar permisos NTFS sobre una carpeta compartida de Ventas: `GG-Ventas` con permiso de modificar y `GG-Ventas-RO` con permiso de lectura. Hasta entonces, el rol se demuestra por la pertenencia al grupo, no por el acceso a un recurso.
- Dar uso a `GG-Ventas-RO` con el caso Mover.
