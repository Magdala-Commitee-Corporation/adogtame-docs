# ADOGTAME — Guía de manejo de roles

> **Basado en:** ERS v2.1.0 (13/11/2026): §3 *Actores del sistema*, §4 *Casos de uso*, RN-04, RN-08, RN-11, RN-12, RN-14 a RN-22, RN-28, RN-36, RN-38, RN-41 y RN-42, y RNF-04. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Material de **apoyo** del instrumento `adog_MER.md` (entidades `ROL` y `USUARIO_ROL`). Define **qué puede hacer cada tipo de usuario** y cómo se asigna y cambia el rol, **sin** consultas SQL ni código. Complementa `solicitudes_verificacion.md`.

---

## 1. Principios

1. **El rol es el tipo de cuenta.** Hay cuatro: MIEMBRO, RESCATISTA, REFUGIO y ADMINISTRADOR. Un visitante no tiene rol porque no tiene cuenta.
2. **Un usuario tiene un solo rol.** Nunca se combinan. El modelo permite varias filas por usuario, pero la aplicación no debe crearlas jamás. Esta regla no la garantiza la base de datos: la garantiza la aplicación.
3. **Las capacidades no son roles.** «Adoptante», «Publicador» y «Usuario registrado» (actores abstractos del ERS) agrupan acciones; se obtienen según el rol (sección 3).
4. **Todo se autoriza en el servidor** (RNF-04). Ocultar una opción en pantalla no sustituye la verificación.
5. **Ante la duda, se deniega.** Un usuario sin rol, o con más de uno, no puede ejecutar ninguna acción (sección 8).
6. **El rol no lo edita nadie** (RN-04, RN-08): ni el usuario ni el administrador. Solo cambia por los flujos de la sección 6.

---

## 2. Catálogo de roles

| Rol | Actor del ERS | Cómo se obtiene | Perfil que tiene | Distintivo |
|---|---|---|---|---|
| **MIEMBRO** | Persona | Registro público (RF-05) o creación por un administrador (RF-26) | Datos personales (`PERSONA`) | — |
| **RESCATISTA** | Rescatista | Aprobación de una solicitud de verificación (RF-25) o creación por un administrador (RF-26, RN-38) | Datos personales (`PERSONA`) | Insignia y presencia en el listado de rescatistas |
| **REFUGIO** | Refugio | Aprobación de una solicitud de refugio (RF-25) o creación por un administrador (RF-26, RN-38) | Datos institucionales (`REFUGIO`) | Insignia y presencia en el listado de refugios |
| **ADMINISTRADOR** | Administrador | Creación por otro administrador (RF-26) o despliegue inicial del sistema (RN-12) | Datos personales (`PERSONA`) | — |

El rol RESCATISTA conserva **todas** las funciones de MIEMBRO. No existe registro público de administradores ni de refugios (RN-12, RN-35).

---

## 3. Capacidades por rol

Los actores abstractos del ERS se resuelven así:

| Capacidad (actor abstracto) | MIEMBRO | RESCATISTA | REFUGIO | ADMINISTRADOR |
|---|---|---|---|---|
| **Usuario registrado** (tener sesión iniciada) | Sí | Sí | Sí | Sí |
| **Adoptante** (test, recomendaciones, solicitar adopción) | Sí | Sí | **No** (RN-28, RN-42) | Sí |
| **Publicador** (publicar y gestionar perros) | Sí | Sí | Sí | Sí (RN-20) |

---

## 4. Matriz de acciones

Cubre los 24 casos de uso del ERS. «Sí» significa que el rol puede ejecutarla; las condiciones de la sección 5 se aplican además.

| # | Caso de uso | Visitante | MIEMBRO | RESCATISTA | REFUGIO | ADMINISTRADOR |
|---|---|:-:|:-:|:-:|:-:|:-:|
| 1 | Explorar y filtrar perros | Sí | Sí | Sí | Sí | Sí |
| 2 | Ver perfil del perro | Sí | Sí | Sí | Sí | Sí |
| 3 | Ver listados y perfiles públicos | Sí | Sí | Sí | Sí | Sí |
| 4 | Registrarse como persona | Sí | — | — | — | — |
| 5 | Solicitar registro como refugio | Sí | — | — | — | — |
| 6 | Iniciar sesión | (previo a tener sesión) | Sí | Sí | Sí | Sí |
| 7 | Recuperar contraseña | (previo a tener sesión) | Sí | Sí | Sí | Sí |
| 8 | Guardar perros en favoritos | — | Sí | Sí | Sí | Sí |
| 9 | Ver datos de contacto de usuarios | — | Sí | Sí | Sí | Sí |
| 10 | Administrar perfil | — | Sí | Sí | Sí | Sí (no puede eliminar su cuenta) |
| 11 | Realizar test | — | Sí | Sí | — | Sí |
| 12 | Revisar recomendaciones de perros | — | Sí | Sí | — | Sí |
| 13 | Solicitar adopción | — | Sí | Sí | — | Sí |
| 14 | Revisar mis solicitudes de adopción | — | Sí | Sí | — | Sí |
| 15 | Solicitar verificación como rescatista | — | **Sí** | — | — | — |
| 16 | Registrar perro en adopción | — | Sí | Sí | Sí | Sí |
| 17 | Gestionar información del perro | — | Sí (propios) | Sí (propios) | Sí (propios) | Sí (propios) |
| 18 | Gestionar solicitudes de adopción | — | Sí (de sus perros) | Sí (de sus perros) | Sí (de sus perros) | Sí (de sus perros) |
| 19 | Cambiar estado de adopción del perro | — | Sí (propios) | Sí (propios) | Sí (propios) | Sí (propios) |
| 20 | Gestionar verificaciones | — | — | — | — | Sí |
| 21 | Gestionar usuarios | — | — | — | — | Sí |
| 22 | Gestionar perros | — | — | — | — | Sí (consultar y eliminar) |
| 23 | Gestionar catálogos de formularios | — | — | — | — | Sí |
| 24 | Seleccionar y explicar recomendaciones | — | — | — | — | — |

Notas:

- Los casos 4 y 5 son de quien **no tiene cuenta**; un usuario con sesión no los ejecuta.
- El caso 24 lo ejecuta el **Servicio de IA**, un sistema externo. No es un rol ni tiene cuenta.
- El caso 15 es exclusivo de MIEMBRO: ni el rescatista (ya lo es), ni el refugio, ni el administrador (RN-36) pueden solicitarlo.

---

## 5. Condiciones además del rol

El rol es necesario pero no siempre suficiente. Cada acción puede exigir también lo siguiente.

### 5.1 Sobre la cuenta

- Para tener sesión: correo confirmado y cuenta no eliminada (RN-02).
- Una cuenta eliminada responde como inexistente (RN-16, RNF-07).

### 5.2 Sobre el recurso (propiedad)

| Acción | Condición |
|---|---|
| Editar un perro, cambiar su estado | Solo su publicador (RN-22). El administrador **no** puede editar ni restaurar perros ajenos |
| Eliminar un perro | Su publicador o un administrador (RN-22) |
| Aceptar o rechazar una solicitud de adopción | Solo el publicador del perro y mientras esté pendiente (RNF-04, RN-31) |
| Cancelar una solicitud de adopción | Solo el adoptante que la envió y mientras esté pendiente (RNF-04, RN-31) |
| Solicitar adopción | El perro debe estar disponible y no ser propio (RN-28) |
| Ver o modificar el test | Solo su dueño; nadie más lo ve, ni el administrador (RN-41) |
| Cambiar correo o contraseña | Solo el propio usuario (RN-08) |

### 5.3 Sobre el estado del sistema

- Un perro adoptado no admite solicitudes y se muestra con la etiqueta «Adoptado» (RN-17).
- La verificación de rescatista requiere no tener otra solicitud pendiente ni un rechazo de menos de 5 minutos (RN-36).

---

## 6. Ciclo de vida del rol

### 6.1 Asignación inicial

| Flujo | Rol asignado |
|---|---|
| Registro público (RF-05) | MIEMBRO |
| Solicitud de refugio aprobada (RF-25) | REFUGIO |
| Creación por un administrador (RF-26) | El rol del tipo elegido: MIEMBRO, RESCATISTA, REFUGIO o ADMINISTRADOR |
| Despliegue inicial | ADMINISTRADOR (RN-12) |

La cuenta, su perfil (personal o institucional) y su rol se crean **juntos**: nunca debe existir una cuenta sin rol ni un perfil que no corresponda a su rol.

### 6.2 Única transición permitida: MIEMBRO → RESCATISTA

- Ocurre cuando un administrador **aprueba** la solicitud de verificación (RF-25, flujo A9).
- El rol RESCATISTA **reemplaza** al rol MIEMBRO. No se agrega: nunca quedan los dos.
- La fecha de asignación del nuevo rol es la de aprobación.
- Rige de inmediato; el usuario no necesita volver a iniciar sesión.
- La persona conserva su perfil, perros, favoritos, test y solicitudes.

No existen otras transiciones: ni de RESCATISTA a MIEMBRO (una verificación aprobada no se revoca, RN-37), ni cambios hacia o desde REFUGIO o ADMINISTRADOR. Si un rescatista o refugio deja de cumplir, se elimina su cuenta (RN-09).

### 6.3 Texto propuesto para RN-04

El ERS dice que el tipo de cuenta «no se puede modificar» (RN-04). Con este diseño hay una excepción, por lo que se propone redactarla así:

> *El tipo de cuenta no se puede modificar después de creada, ni por el usuario ni por el administrador. La única excepción es el paso de MIEMBRO a RESCATISTA, que ocurre exclusivamente cuando un administrador aprueba la solicitud de verificación.*

### 6.4 Eliminar y restaurar

- Eliminar una cuenta no cambia su rol; restaurarla la devuelve con el mismo rol.
- Se conserva el rol aunque la cuenta esté eliminada (no se borra nada, RNF-10).

---

## 7. Restricciones especiales del administrador

- No puede eliminar su propia cuenta (RN-11).
- El sistema siempre debe tener al menos un administrador activo (RN-11).
- No puede cambiar el correo, la contraseña ni el tipo de cuenta de ningún usuario, ni ver contraseñas (RN-04, RN-08).
- No puede editar ni restaurar perros ajenos; solo consultarlos y eliminarlos, sin avisar al publicador (RN-22, RN-23).
- No puede ver el test de otros usuarios (RN-41).
- No puede solicitar la verificación de rescatista (RN-36).
- Sí puede realizar el test, pedir recomendaciones, solicitar adopciones, publicar perros y guardar favoritos (RN-19, RN-20, RN-28, RN-42).

---

## 8. Coherencia del rol

### 8.1 Condiciones que siempre deben cumplirse

1. Todo usuario tiene **exactamente un** rol.
2. Un usuario con rol REFUGIO tiene perfil institucional y **no** datos personales.
3. Un usuario con rol MIEMBRO, RESCATISTA o ADMINISTRADOR tiene datos personales y **no** perfil institucional.
4. Solo un usuario con rol MIEMBRO, RESCATISTA o ADMINISTRADOR puede tener un test.
5. Hay al menos un ADMINISTRADOR activo.

### 8.2 Si no se cumplen

- **Sin rol o con más de uno:** se deniega toda acción y se registra la anomalía para revisión. No se elige un rol «por defecto».
- Debe existir una revisión periódica que detecte los casos 1, 2 y 3, y pruebas de integración que comprueben que cada flujo de creación y de aprobación los respeta.

---

## 9. Visibilidad según el rol

| Información | Visitante | Con sesión (cualquier rol) |
|---|---|---|
| Perros disponibles y perfiles públicos | Sí | Sí |
| Teléfono y correo de publicadores y usuarios | No (RN-15) | Sí |
| Perro adoptado | Solo por enlace, favoritos o perfil del publicador (RN-17) | Ídem |
| Perro o usuario eliminado | No existe (404) | No existe (404); el administrador sí puede consultarlos (RN-16) |
| Respuestas del test | No | Solo su dueño (RN-41) |

---

## 10. Lista de verificación

| Requisito | Dónde se cumple |
|---|---|
| RN-04, RN-08 | Principio 6; 6.2 y 6.3 |
| RN-11 | 7; 8.1 |
| RN-12 | 6.1 |
| RN-20 | 3; 4 (caso 16) |
| RN-22 | 5.2 |
| RN-28, RN-42 | 3; 4 (casos 11 a 14) |
| RN-36 | 4 (caso 15); 7 |
| RN-38 | 2; 6.1 |
| RN-41 | 5.2; 7; 9 |
| RNF-04 | Principios 4 y 5 |
| RNF-10 | 6.4 |
