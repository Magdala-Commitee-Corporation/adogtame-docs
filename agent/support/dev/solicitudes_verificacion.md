# ADOGTAME — Guía de implementación: solicitudes de verificación

> **Basado en:** ERS v2.1.0 (13/11/2026): RF-06, RF-24, RF-25 y RF-26 (§6), RN-01, RN-13 y RN-35 a RN-38 (§5), criterios CRV-01 y CRV-02, y RNF-04, RNF-06, RNF-08, RNF-10, RNF-11 y RNF-13 (§7). Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Es un material de **apoyo** del instrumento `adog_MER.md` (entidad `SOLICITUD_VERIFICACION`). Explica **cómo debe comportarse** la base de datos y la aplicación para cumplir los requisitos, **sin** consultas SQL ni código. Las restricciones aquí descritas se trasladarán al script DBML. Los roles y sus permisos están en `manejo_roles.md`.

---

## 1. Propósito y alcance

Una **solicitud de verificación** es el registro con el que un administrador decide si una persona pasa a ser **rescatista verificado** o si una institución obtiene una **cuenta de refugio**. Ambos casos comparten la misma entidad, pero se comportan distinto porque en uno el solicitante **ya tiene cuenta** y en el otro **todavía no**.

| | Solicitud de rescatista | Solicitud de refugio |
|---|---|---|
| Requisito que la crea | RF-24 | RF-06 |
| Quién la envía | Una persona con sesión iniciada | Un visitante sin cuenta |
| ¿Existe `USUARIO` al solicitar? | Sí | **No** |
| Datos del formulario | Ya están en `PERSONA`; no se copian | Se guardan en la solicitud (`datos_registro`) |
| Qué ocurre al aprobar | Su rol cambia de MIEMBRO a RESCATISTA | Se crean la cuenta (rol REFUGIO) y el perfil del refugio |
| Resolución | RF-25 (flujo alterno A9) | RF-25 (flujo principal) |

Las cuentas de rescatista y de refugio **creadas por un administrador** (RF-26) **no pasan por una solicitud** (RN-38).

---

## 2. Datos de la solicitud

### 2.1 Columnas y qué significan según el tipo

| Columna | Rescatista | Refugio |
|---|---|---|
| `tipo` | `rescatista` | `refugio` |
| `estado` | `pendiente`, `aprobada` o `rechazada` | Ídem |
| `id_usuario` | **Siempre** el solicitante | Vacío mientras esté pendiente; al aprobar, la cuenta recién creada. Vacío si se rechaza |
| `correo_solicitante` | No se usa | Correo del refugio mientras esté pendiente; vacío al resolver |
| `datos_registro` | No se usa | Datos del formulario mientras esté pendiente; vacío al resolver |
| `id_admin_revisor` | Vacío hasta resolver | Ídem |
| `fecha_solicitud` | Siempre | Siempre |
| `fecha_resolucion` | Vacía hasta resolver | Ídem |

### 2.2 Por qué el correo es una columna y no parte del JSON

La unicidad del correo (RN-01) debe garantizarse **a nivel de base de datos**, entre solicitudes pendientes. Esa restricción no puede apoyarse en el interior de un JSON de forma portable entre motores. Por eso `correo_solicitante` es una columna independiente y **no se repite** dentro de `datos_registro`.

### 2.3 Contenido de `datos_registro`

Solo para solicitudes de refugio. Contiene los datos del formulario de RF-06, **excepto el correo**:

| Dato | Obligatorio | Validación (ERS RF-06) |
|---|---|---|
| Nombre de la institución | Sí | Máx. 120 caracteres |
| Teléfono | Sí | 10 dígitos, inicia con 09 |
| Ubicación | Sí | Máx. 255 caracteres |
| Fecha de creación de la institución | No | No futura |
| Página web | No | URL válida, solo http o https (RNF-22) |
| Descripción | No | Máx. 500 caracteres |
| Clave de la foto de perfil | No | JPG, JPEG o PNG, máx. 2 MB; se guarda la **clave** del almacenamiento, no una URL (RNF-13) |

Notas:

- El JSON es una **instantánea**: no es la fuente de verdad. Una vez aprobada la solicitud, los datos vigentes del refugio están en `REFUGIO`.
- La base de datos **no valida el interior del JSON**. La validación recae en la aplicación (ver 5.1 y 5.3).
- Conviene que el JSON incluya un número de versión de su estructura, por si el formulario cambia en el futuro.

### 2.4 Restricciones que debe garantizar la base de datos

**Consistencia entre columnas** (en lenguaje natural):

1. Una solicitud de rescatista siempre tiene `id_usuario`, y nunca tiene `correo_solicitante` ni `datos_registro`.
2. Una solicitud de refugio **pendiente** siempre tiene `correo_solicitante` y `datos_registro`, y no tiene `id_usuario`.
3. Una solicitud de refugio **aprobada** siempre tiene `id_usuario`, y no tiene `correo_solicitante` ni `datos_registro`.
4. Una solicitud de refugio **rechazada** no tiene `id_usuario`, `correo_solicitante` ni `datos_registro`.
5. Una solicitud **pendiente** no tiene `id_admin_revisor` ni `fecha_resolucion`; una **resuelta** tiene ambos.

**Unicidades entre solicitudes pendientes** (índices únicos que solo consideran las filas pendientes):

| Regla | Qué debe ser único mientras esté pendiente |
|---|---|
| RN-36 | Una sola solicitud de rescatista pendiente por persona (`id_usuario`) |
| RN-01 / RF-06 | Un solo correo por solicitud de refugio pendiente, **sin distinguir mayúsculas** |

> Si el motor elegido no admite índices parciales, estas dos reglas deben garantizarse dentro de la misma transacción que crea la solicitud, y documentarse como riesgo de concurrencia.

**Integridad referencial:** `id_usuario` e `id_admin_revisor` apuntan a `USUARIO`. El revisor debe ser un administrador (esto lo valida la aplicación, no la base de datos).

---

## 3. Ciclo de vida

```
          (crear)            (aprobar)
  ─────────────► PENDIENTE ─────────────► APROBADA
                     │
                     │ (rechazar)
                     └──────────────────► RECHAZADA
```

- Una solicitud **solo se resuelve una vez**. Las resueltas no cambian de estado (RN-37: una verificación aprobada no se revoca; si deja de cumplir, se elimina la cuenta, RN-09).
- Las solicitudes pendientes **no caducan** (RN-35).
- **Al resolver (aprobar o rechazar), en una solicitud de refugio se anulan `correo_solicitante` y `datos_registro`.** Motivo: RNF-08 y la LOPDP piden no conservar datos personales innecesarios, y quien solicita como refugio **no tiene cuenta**. Tras aprobar, los datos ya están en `USUARIO` y `REFUGIO`; tras rechazar, no hay motivo para conservarlos.
- `fecha_resolucion` **se conserva siempre**: la espera de 5 minutos tras un rechazo de rescatista (RN-36) se calcula con ella.
- No se registra un motivo de rechazo (RN-37).

---

## 4. Solicitud de rescatista

### 4.1 Crear (RF-24)

**Quién:** un usuario con rol MIEMBRO y sesión iniciada. No pueden solicitarlo quienes tienen el rol REFUGIO, ADMINISTRADOR (RN-36) ni RESCATISTA (precondición de RF-24).

**La aplicación debe:**

1. Autorizar en el servidor según el tipo de cuenta (RNF-04); no basta con ocultar el botón.
2. Verificar que su rol sea MIEMBRO (no RESCATISTA, REFUGIO ni ADMINISTRADOR).
3. Verificar que no tenga una solicitud pendiente. Mensaje: «Ya tienes una solicitud de verificación pendiente».
4. Verificar que su última solicitud rechazada no tenga menos de 5 minutos. Mensaje: «Podrás enviar una nueva solicitud en unos minutos».
5. Registrar la solicitud como pendiente, con `id_usuario` = la persona. No se copian datos.
6. Mostrar «Tu solicitud fue enviada. Recibirás la respuesta en tu correo» y el estado «Verificación pendiente» en su perfil.

**Mientras esté pendiente**, la persona sigue publicando y adoptando con normalidad (RN-36).

### 4.2 Resolver (RF-25, flujo A9)

- **Aprobar:** el rol del usuario **cambia** de MIEMBRO a RESCATISTA. La asignación reemplaza al rol anterior: nunca quedan los dos a la vez. La fecha de asignación del nuevo rol es la fecha de aprobación. Con el nuevo rol, el usuario muestra la insignia y aparece en el listado de rescatistas. El cambio se hace dentro de la misma operación que marca la solicitud como aprobada y rige de inmediato, sin que la persona deba volver a iniciar sesión. Detalle de la transición en `manejo_roles.md` (sección 6).
- **Rechazar:** solo cambia la solicitud. La persona puede volver a solicitar tras 5 minutos.
- En ambos casos: registrar el administrador y la fecha de resolución, y enviar un correo con el resultado, sin motivo.

---

## 5. Solicitud de refugio

### 5.1 Crear (RF-06)

**Quién:** un visitante, sin sesión.

**La aplicación debe:**

1. Validar los datos del formulario con las reglas de la sección 2.3 y el formato del correo (máx. 120 caracteres).
2. Verificar que el correo **no pertenezca a ninguna cuenta**, activa o eliminada (RN-01). Mensaje: «El correo ya está registrado».
3. Verificar que el correo **no tenga una solicitud pendiente**. Mensaje: «Ya existe una solicitud pendiente con este correo».
4. Guardar la foto en el almacenamiento (si la hay) y obtener su clave. Si el almacenamiento falla, no registrar la solicitud y avisar (RNF-11).
5. Registrar la solicitud como pendiente: tipo refugio, `correo_solicitante` con el correo, `datos_registro` con el resto de los datos, `id_usuario` vacío.
6. Mostrar «Tu solicitud fue enviada. En los próximos días recibirás la respuesta en tu correo».

**No se crea ninguna cuenta ni se pide contraseña** (RN-35). El refugio la define con el enlace que recibe al ser aprobado.

**Efecto sobre otros flujos:** mientras la solicitud esté pendiente, su correo queda **reservado**. Por eso, cualquier flujo que compruebe unicidad de correo debe consultar **también** las solicitudes de refugio pendientes: registro de persona (RF-05), cambio de correo (RF-31, al pedirlo y al confirmarlo, RN-07) y creación de usuarios por un administrador (RF-26).

### 5.2 Aprobar (RF-25, flujo principal)

La aplicación debe ejecutar todo lo siguiente como **una sola operación atómica**; si algún paso falla, no queda nada a medias:

1. Verificar que la solicitud siga **pendiente**. Si ya la resolvió otro administrador: «La solicitud ya fue resuelta» (RF-25, A8).
2. Verificar de nuevo que el correo no pertenezca a ninguna cuenta. La reserva de 5.1 debería impedirlo, pero se comprueba por seguridad.
3. Volver a validar `datos_registro` con las mismas reglas de la sección 2.3, porque la base de datos no las garantiza.
4. Crear la **cuenta**: correo del solicitante, **correo confirmado**, sin contraseña, con el rol REFUGIO.
5. Crear el **perfil del refugio** con los datos de `datos_registro`.
6. Crear el **enlace para definir la contraseña** (un solo uso, 10 minutos; RNF-06). Este enlace también confirma el correo (RN-13).
7. Marcar la solicitud como aprobada, registrar el administrador y la fecha, vincular `id_usuario` con la cuenta creada, y **anular** `correo_solicitante` y `datos_registro`.

**Después de confirmar la operación**, enviar el correo de aprobación con el enlace. Si el correo falla, no se deshace nada: se informa al administrador y la contraseña se puede definir luego con «Recuperar contraseña» (RN-13, RNF-11).

### 5.3 Rechazar (RF-25, flujo A5)

1. Verificar que la solicitud siga pendiente.
2. Marcarla como rechazada, registrar administrador y fecha, y **anular** `correo_solicitante` y `datos_registro`.
3. Enviar el correo de rechazo, **sin motivo** (RN-37).

El refugio puede enviar una nueva solicitud **de inmediato**, sin espera (RN-35). Solo se exige que no tenga otra pendiente con ese correo y que el correo no pertenezca a una cuenta. La espera de 5 minutos aplica únicamente a las solicitudes de rescatista (4.1).

---

## 6. Creación directa por un administrador (RF-26)

- El administrador crea cuentas de rescatista y refugio **sin solicitud**; quedan verificadas (RN-38). La cuenta de rescatista recibe directamente el rol RESCATISTA, sin pasar por MIEMBRO.
- Para refugios, el formulario y las validaciones son los **mismos** de RF-06. Conviene que el paso «validar datos → crear cuenta y perfil → crear enlace de contraseña» sea **la misma lógica** que usa la aprobación (5.2, pasos 2 a 6), para que ambos caminos no se desalineen.
- Debe comprobarse también que el correo no tenga una solicitud pendiente.

---

## 7. Validaciones y mensajes (resumen)

| Situación | Mensaje (ERS) |
|---|---|
| Correo ya registrado | «El correo ya está registrado» |
| Correo con solicitud de refugio pendiente | «Ya existe una solicitud pendiente con este correo» |
| Rescatista con solicitud pendiente | «Ya tienes una solicitud de verificación pendiente» |
| Rescatista con rechazo reciente | «Podrás enviar una nueva solicitud en unos minutos» |
| Solicitud ya resuelta por otro administrador | «La solicitud ya fue resuelta» |
| Enlace de contraseña usado o caducado | «El enlace no es válido o ha caducado» |
| No hay solicitudes por resolver | «No hay solicitudes de verificación pendientes» |

---

## 8. Puntos a decidir o vigilar

| # | Punto | Detalle |
|---|---|---|
| 1 | Aprobación de rescatista | **Decidido:** se reemplaza el rol MIEMBRO por RESCATISTA (4.2). Obliga a acotar RN-04 en el ERS (ver `manejo_roles.md`, sección 6). |
| 2 | Espera tras un rechazo | **Decidido:** el rescatista debe esperar 5 minutos (RN-36); el refugio puede reenviar de inmediato (RN-35). En ambos casos se verifica que no haya una solicitud pendiente. El texto de CRV-02 («tras 5 minutos» para el refugio) en el ERS y en `adog_RN.md` debe corregirse. Como no hay espera para refugios, anular `correo_solicitante` al resolver no afecta nada. Riesgo asumido: el formulario público no tiene límite propio de reenvíos; solo lo mitiga RNF-23. |
| 3 | Solicitud pendiente de un usuario eliminado | RN-10 rechaza las solicitudes de **adopción**, pero no menciona las de verificación. Debe definirse si al eliminar una cuenta su solicitud de rescatista pendiente pasa a rechazada. |
| 4 | Fotos huérfanas | Si una solicitud de refugio se rechaza, su foto queda en el almacenamiento sin referencia. Es aceptable, pero quien opere el almacenamiento debe saberlo. |
| 5 | Criterios de verificación | CRV-01 y CRV-02 los aplica el administrador fuera del sistema (llamadas, revisión de la web). No se guarda evidencia de su cumplimiento. |
| 6 | Auditoría | Solo se registra quién resolvió y cuándo; no hay historial de cambios (coherente con RF-27). |

---

## 9. Lista de verificación

| Requisito | Cómo se cumple |
|---|---|
| RF-06 | 5.1 |
| RF-24 | 4.1 |
| RF-25 | 4.2, 5.2, 5.3 |
| RF-26 | 6 |
| RN-01 | 2.2, 2.4, 5.1 (efecto sobre otros flujos) |
| RN-13 | 5.2 paso 6 y envío posterior |
| RN-35 | 3, 5.1 |
| RN-36 | 2.4 (unicidad), 4.1 |
| RN-37 | 3, 4.2, 5.3 |
| RN-38 | 6 |
| RNF-04 | 4.1 paso 1; solo un administrador resuelve (5.2) |
| RNF-06 | 5.2 paso 6 |
| RNF-08 | 3 (anulación al resolver) |
| RNF-11 | 5.1 paso 4, 5.2 (correo después de confirmar) |
| RNF-13 | 2.3 (clave de foto, no URL) |
