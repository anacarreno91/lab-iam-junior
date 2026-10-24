# Evidencias

> **⚠️ CASO SIMULADO.** `a.baja` es un usuario ficticio de laboratorio. No es una persona real. Todas las capturas son del dominio de pruebas `iamlab.local`.

**Estado:** capturas pendientes de publicar. Este documento fija qué debe verse en cada una antes de subirla.

Tres capturas, una por momento del ciclo de vida. Juntas responden a las tres preguntas de cualquier auditoría de accesos: **qué se concedió, qué se retiró y qué dice el sistema**.

| Archivo | Qué prueba | Pregunta del auditor que responde |
|---|---|---|
| `01-alta-a.baja-GG-Ventas.png` | Estado de la cuenta tras el alta | ¿Se concedió solo lo aprobado? |
| `02-baja-a.baja-deshabilitada.png` | Estado de la cuenta tras la baja | ¿Se retiró todo? |
| `03-log-eventos-4725-4729.png` | Registro de Seguridad de Windows | ¿Cuándo y quién? ¿Lo confirma el sistema? |

## Reglas para las tres capturas

- **Hora visible:** el reloj de Windows de la máquina virtual debe aparecer en la captura.
- **Aviso visible:** el campo *Descripción* de `a.baja` dice que es un usuario simulado, así que el aviso queda a la vista en las capturas 01 y 02 sin añadir nada a la imagen.
- **Solo la máquina virtual:** recortada a la ventana de VirtualBox, sin el escritorio del equipo anfitrión, notificaciones, correo ni ningún dato personal real.
- **Sin retoques:** ni recortes dentro de la imagen, ni marcas, ni ediciones. Una evidencia editada deja de ser evidencia.
- **Nombre exacto del archivo:** la documentación se refiere a cada captura por estos nombres.

## 01 · Alta: `01-alta-a.baja-GG-Ventas.png`

**Qué debe verse**

- *Usuarios y equipos de Active Directory* con la OU `Ventas` abierta y `a.baja` en la lista, con su *Descripción*.
- Encima, *Propiedades de a.baja → Miembro de*, con exactamente dos grupos: `Usuarios del dominio` y `GG-Ventas`.
- El reloj de la máquina virtual.

**Por qué es crítica:** es la línea base. Demuestra que el acceso concedido coincide con el aprobado (mínimo privilegio) y es la referencia para comprobar después que la baja lo retiró todo. Si no se sabe con certeza qué accesos se dieron, no se puede asegurar que se han quitado; en un entorno industrial, ese hueco es exactamente por donde se queda una cuenta olvidada.

## 02 · Baja: `02-baja-a.baja-deshabilitada.png`

**Qué debe verse**

- `a.baja` en la OU `Ventas` con el icono de cuenta deshabilitada (flecha hacia abajo) y la *Descripción* actualizada sin perder el aviso: `Usuario simulado de laboratorio · Baja SIM-002 - AAAA-MM-DD`.
- *Propiedades de a.baja → Miembro de*, con solo `Usuarios del dominio` (grupo principal, no se puede quitar; ver [docs/01](../docs/01-Joiner-Mover-Leaver.md#leaver-baja)).
- El reloj de la máquina virtual.

**Por qué es crítica:** demuestra el estado final: la cuenta no puede iniciar sesión y no conserva grupos de negocio. Es lo que evita una identidad huérfana, es decir, la cuenta de alguien que ya no está pero sigue activa y con accesos. En OT, ese tipo de cuenta (por ejemplo, la de un técnico externo que terminó su trabajo) es una vía de entrada que nadie vigila y un riesgo directo para la continuidad del proceso.

## 03 · Log: `03-log-eventos-4725-4729.png`

**Cómo se obtiene:** *Visor de eventos → Registros de Windows → Seguridad → Filtrar registro actual* → en el campo de identificadores de evento: `4720,4728,4725,4729`.

**Qué debe verse**

- La lista filtrada con los cuatro eventos y su fecha y hora: el ciclo completo de `a.baja` de un vistazo (4720 cuenta creada, 4728 añadida a grupo, 4725 deshabilitada, 4729 quitada del grupo).
- El detalle del evento 4725 abierto, donde se lean la *cuenta de destino* (`a.baja`) y la cuenta que ejecutó la acción (*Sujeto*).

**Por qué es crítica:** las capturas 01 y 02 muestran un estado; el registro muestra **cuándo** y **quién**, y lo genera Windows, no la persona que opera. Comparando la hora del evento 4725 con la hora de la solicitud en el [registro](../registros/registro-solicitudes-acceso.csv) se demuestra el plazo de menos de 24 horas. Después de un incidente en una planta, este registro es lo que permite reconstruir quién retiró qué acceso y en qué momento.

## Notas para reproducir las evidencias

- **Si no aparecen los eventos:** la auditoría de administración de cuentas puede estar desactivada. Se comprueba con `auditpol /get /category:*` en una consola como administrador; hay que tener activadas en éxito las subcategorías de administración de cuentas de usuario y de grupos de seguridad (directiva *Default Domain Controllers Policy*). Los eventos solo se registran desde que la auditoría está activa: si estaba desactivada, se repite el caso y se anotan en el registro las fechas nuevas.
- **Si aparecen 4732 y 4733 en lugar de 4728 y 4729:** el grupo se creó como local de dominio en vez de global. Hay que corregir el ámbito de `GG-Ventas` o ajustar la documentación para que diga lo que realmente hay.
