# ADOGTAME — Microservicios

> **Derivado de:** `adog_ARCH.md` (arquitectura y comunicación), `adog_MER.md` (datos), `adog_CLS.md` (funciones), `adog_RF.md`, `adog_RN.md`, `adog_RNF.md` y `adog_CU.md` (requisitos), `adog_REC.md` (recomendación) y el material de apoyo `support/dev/manejo_roles.md` y `support/dev/solicitudes_verificacion.md`. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse; si difiere de `adog_ARCH.md` en una decisión de arquitectura, prevalece `adog_ARCH.md`.
> Describe **cada uno de los 4 microservicios** (`Auth`, `Dogs`, `Requests` y `Recommendation`): qué datos posee, qué operaciones ofrece, qué reglas aplica, a quién llama y cómo se prueba, y los **flujos que cruzan servicios**. Es una especificación **preliminar** para comenzar la construcción: las rutas, los nombres y los tipos exactos se fijan en el contrato de cada servicio durante el incremento de cimientos (`planning/Adogtame_planificacion_v2.xlsx`).
> Convenciones: los servicios y las rutas llevan nombre en inglés; los textos al usuario, en español. «Registrado» es cualquier usuario con sesión; «Adoptante» es un usuario registrado con rol MIEMBRO, RESCATISTA o ADMINISTRADOR (no REFUGIO); «Publicador» es el usuario registrado que publica un perro (`manejo_roles.md` §3).

---

## 1. Visión general

| Microservicio | Ruta base | Entidades que posee (esquema) | Módulos internos | Requisitos funcionales |
|---|---|---|---|---|
| `Auth` | `/auth` | `USUARIO`, `ROL`, `USUARIO_ROL`, `PERSONA`, `REFUGIO`, `TOKEN_CORREO`, `SESION` (`auth`) | cuentas, sesiones, perfiles, administración de usuarios | RF-03 a RF-05, RF-07 a RF-11, RF-26 a RF-28, RF-31, RF-34 |
| `Dogs` | `/dogs` | `PERRO`, `FOTO_PERRO`, `RAZA`, `COLOR`, `FAVORITO` (`dogs`) | perros, catálogos, favoritos | RF-01, RF-02, RF-12, RF-19 a RF-21, RF-23, RF-29, RF-30, RF-32, RF-33 |
| `Requests` | `/requests` | `SOLICITUD_ADOPCION`, `SOLICITUD_VERIFICACION` (`requests`) | adopción, verificación | RF-06, RF-16 a RF-18, RF-22, RF-24, RF-25 |
| `Recommendation` | `/recommendation` | `PERFIL_ADOPTANTE` (`recommendation`) | test, recomendaciones | RF-13 a RF-15 |

Cada servicio es una función AWS Lambda con despliegue propio y es el único que lee y escribe su esquema. Las llamadas entre servicios son síncronas (flechas punteadas):

```mermaid
flowchart LR
    AUTH["Auth"]
    DOGS["Dogs"]
    REQS["Requests"]
    REC["Recommendation"]

    REQS -.-> AUTH
    REQS -.-> DOGS
    DOGS -.-> REQS
    DOGS -.-> AUTH
    AUTH -.-> DOGS
    AUTH -.-> REQS
    AUTH -.-> REC
    REC -.-> DOGS
```

---

## 2. Convenciones comunes

### 2.1. Identidad de cada petición

Todas las rutas públicas pasan por el autorizador de API Gateway, que valida la sesión contra `Auth` y entrega al servicio la **identidad** de la petición:

| Dato | Descripción |
|---|---|
| `idUsuario` | Usuario con sesión; vacío para un visitante. |
| `idPersona` / `idRefugio` | Perfil del usuario (uno solo, según su rol). |
| `rol` | `MIEMBRO`, `RESCATISTA`, `REFUGIO` o `ADMINISTRADOR`; vacío para un visitante. |

El servicio **nunca confía en un rol enviado por el cliente**. Cada operación declara qué roles la ejecutan y, cuando aplica, la condición de propiedad del recurso (RNF-04, `manejo_roles.md` §5). Ante un usuario sin rol o con más de uno se deniega toda acción y se registra la anomalía.

### 2.2. Rutas y formato

- API REST con JSON, una ruta base por servicio. Las fechas van en formato ISO 8601 y los identificadores son UUID.
- Las rutas de este documento son **orientativas**: sirven para dimensionar el trabajo y se confirman en el contrato.
- Las operaciones internas (entre servicios) no se exponen por API Gateway (sección 7).

### 2.3. Contrato de errores

Todos los servicios responden los errores con la misma estructura: un código estable, un mensaje en español y, si aplica, el detalle por campo.

| Estado HTTP | Código | Cuándo |
|---|---|---|
| 400 | `VALIDACION` | Datos inválidos o faltantes (con el detalle por campo). |
| 401 | `NO_AUTENTICADO` | Sin sesión, sesión inválida o expirada. |
| 403 | `SIN_PERMISO` | El rol o la propiedad del recurso no lo permite. |
| 404 | `NO_ENCONTRADO` | Recurso inexistente **o eliminado** (misma respuesta, RNF-07). |
| 409 | `CONFLICTO` | Duplicado o estado que no lo permite (por ejemplo «La solicitud ya fue resuelta»). |
| 429 | `LIMITE_EXCEDIDO` | Exceso de peticiones (lo responde API Gateway o WAF, RNF-23). |
| 502/504 | `DEPENDENCIA_NO_DISPONIBLE` | Otro servicio o un proveedor externo no respondió a tiempo. |
| 500 | `ERROR_INTERNO` | Error no previsto; sin detalles internos al cliente. |

Los mensajes de negocio son los del ERS (por ejemplo «El correo ya está registrado»).

### 2.4. Llamadas internas

Resumen de `adog_ARCH.md` §4.3, que se aplica a todas las llamadas de la sección 7:

- Invocación síncrona de Lambda mediante el cliente interno de la librería compartida; el servicio llamado comprueba que el llamante es otro servicio de Adogtame.
- Tiempo límite inicial de 3 s por llamada; reintento acotado **solo en operaciones seguras de repetir**.
- Toda escritura que cruza servicios lleva una **clave de idempotencia** (por ejemplo el id de la solicitud) y se puede repetir sin duplicar efectos.
- Se propaga el identificador de petición y la identidad; máximo **dos saltos** por petición del usuario y datos de otros dominios pedidos **en lote**.
- Un endpoint invocado por otro servicio **no vuelve a llamar al invocante**.
- En operaciones que cruzan servicios, primero se aplica el efecto en el servicio de la regla más fuerte y **al final** se cambia el estado propio; un fallo a mitad se recupera repitiendo la operación (sección 8).

### 2.5. Estructura interna de cada servicio

Programación funcional (sin clases): manejador de Lambda → funciones de aplicación (orquestan, autorizan y controlan la transacción en su esquema) → funciones de dominio (reglas puras) y funciones de acceso a datos (consultas parametrizadas u ORM, RNF-20). Los adaptadores (almacenamiento, correo, IA) y el cliente interno se reciben como parámetro desde la librería compartida. Cada servicio usa un usuario de base de datos con permisos solo sobre su esquema.

### 2.6. Datos que no se guardan

Se calculan al consultar: la edad del perro, la lista de recomendaciones (RN-47), la espera tras un rechazo (se calcula con `fecha_resolucion`) y los perfiles públicos (`adog_MER.md` §3).

---

## 3. Microservicio `Auth`

### 3.1. Propósito y límites

Es dueño de **quién es cada usuario y qué puede hacer**: cuentas, credenciales, sesiones, enlaces de correo, perfiles (de persona y de refugio) y roles. Emite la identidad de cada petición. **No** conoce perros, solicitudes ni tests: cuando una baja de cuenta debe afectarlos, se lo pide al servicio dueño.

### 3.2. Datos

| Entidad | Notas |
|---|---|
| `USUARIO`, `ROL`, `USUARIO_ROL` | Un solo rol por usuario (lo garantiza la aplicación). El rol solo cambia por MIEMBRO → RESCATISTA al aprobar una verificación (`manejo_roles.md` §6). |
| `PERSONA`, `REFUGIO` | Perfil de la cuenta: personal para MIEMBRO, RESCATISTA y ADMINISTRADOR; institucional para REFUGIO. |
| `TOKEN_CORREO`, `SESION` | Se guardan solo como hash. Los enlaces son de un solo uso (10 min; 24 h en la confirmación de registro) y las sesiones se cierran a los 30 min sin actividad. |

Invariantes: todo usuario tiene exactamente un rol; el perfil corresponde al rol; siempre hay al menos un administrador activo (RN-11).

### 3.3. Operaciones públicas

| Método y ruta | RF | Quién | Qué hace y reglas principales |
|---|---|---|---|
| `POST /auth/register` | RF-05 | Visitante | Valida datos (nombre máx. 120, teléfono de 10 dígitos que inicia con 09, mayor de 18 años) y que el correo sea único **sin pertenecer a ninguna cuenta ni a una solicitud de refugio pendiente**. Crea la cuenta como MIEMBRO y envía el enlace de confirmación. |
| `POST /auth/email/confirm` | RF-05 | Visitante | Confirma el correo con el enlace (un solo uso, 24 h). Sin confirmar no se inicia sesión. |
| `POST /auth/login` | RF-07 | Visitante | Valida credenciales, correo confirmado y cuenta no eliminada; abre la sesión. Los mensajes no revelan si un correo está registrado. |
| `POST /auth/logout` | RF-34 | Registrado | Cierra la sesión actual o todas las del usuario. |
| `POST /auth/password/forgot` | RF-08 | Visitante | Envía un enlace de 10 min (mismo mensaje exista o no el correo). |
| `POST /auth/password/reset` | RF-08 | Visitante | Define la contraseña con el enlace y cierra todas las sesiones. |
| `POST /auth/password/define` | RF-25, RF-26 | Visitante | Define la contraseña de una cuenta creada por aprobación o por un administrador; el mismo enlace confirma el correo (RN-13). |
| `PUT /auth/me/password` | RF-10 | Registrado | Exige la contraseña actual; cierra las demás sesiones y mantiene la actual. |
| `GET` y `PUT /auth/me` | RF-09 | Registrado | Consulta y edita los datos del propio perfil, excepto correo y tipo de cuenta. La foto se guarda por el adaptador de almacenamiento. |
| `POST /auth/me/email` y `POST /auth/me/email/confirm` | RF-31 | Registrado | Pide el cambio con la contraseña actual y envía el enlace al nuevo correo; el correo solo cambia al confirmar. La unicidad se comprueba al pedirlo y al confirmarlo (RN-07). |
| `DELETE /auth/me` | RF-11 | Registrado, excepto ADMINISTRADOR | Advierte y, al confirmar, da de baja la cuenta (sección 8, flujo F6). |
| `GET /auth/rescuers` y `GET /auth/shelters` | RF-03 | Público | Listados de cuentas verificadas (nombre, foto e insignia). |
| `GET /auth/users/{id}` | RF-03, RF-04 | Público | Perfil público con los perros publicados (pedidos a `Dogs`). Teléfono y correo solo con sesión (RN-15). Una cuenta eliminada responde 404. |
| `POST /auth/users` | RF-26 | ADMINISTRADOR | Crea cuentas de persona, rescatista, refugio o administrador; rescatistas y refugios quedan verificados (RN-38) y reciben el enlace para definir su contraseña. Comprueba el correo como en el registro. |
| `GET /auth/users` y `PUT /auth/users/{id}` | RF-27 | ADMINISTRADOR | Busca y edita datos, excepto correo, contraseña y tipo de cuenta. |
| `DELETE /auth/users/{id}` y `POST /auth/users/{id}/restore` | RF-28 | ADMINISTRADOR | Baja o restauración lógica; no puede eliminarse a sí mismo ni dejar el sistema sin administradores (RN-11). |

### 3.4. Operaciones internas que expone

| Operación | La usa | Descripción |
|---|---|---|
| `validarSesion` | Autorizador | Devuelve la identidad (usuario, perfil y rol) y actualiza la última actividad; rechaza sesiones expiradas o de cuentas eliminadas. |
| `obtenerPublicadores` / `obtenerContactoSolicitante` | `Dogs`, `Requests` | Datos de varios usuarios en lote (nombre, foto, tipo de cuenta, verificación y, si hay sesión, contacto). |
| `existeCuentaConCorreo` | `Requests` | Indica si un correo pertenece a una cuenta, activa o eliminada. |
| `cambiarRol` | `Requests` | Cambia MIEMBRO → RESCATISTA, idempotente por el id de la solicitud. |
| `crearCuentaRefugio` | `Requests` | Crea cuenta, perfil y enlace para definir la contraseña en su propia transacción, idempotente por el id de la solicitud; **no** consulta a `Requests` porque ya viene de una solicitud. |

### 3.5. Llamadas que realiza

| A | Para qué |
|---|---|
| `Requests` | Comprobar si un correo tiene una solicitud de refugio pendiente (registro, creación de usuarios y cambio de correo). |
| `Dogs` | Ocultar o mostrar los perros de una cuenta en la baja y la restauración; listar los perros de un perfil público. |
| `Requests` | Rechazar las solicitudes de adopción pendientes de una cuenta dada de baja. |
| `Recommendation` | Borrar el test de una cuenta dada de baja. |

### 3.6. Adaptadores y requisitos

Usa los adaptadores de **correo** (enlaces y avisos) y de **almacenamiento** (foto de perfil). Requisitos transversales: RNF-03 (contraseñas con hash y sal), RNF-04, RNF-05, RNF-06, RNF-07, RNF-08 y RNF-25.

### 3.7. Pruebas clave

Un solo uso y caducidad de los enlaces; cierre de sesión a los 30 min y en todos los dispositivos; unicidad del correo incluyendo cuentas eliminadas y solicitudes de refugio pendientes; el administrador no puede eliminarse ni dejar el sistema sin administradores; el rol solo cambia por la vía de la aprobación; una cuenta eliminada responde 404.

---

## 4. Microservicio `Dogs`

### 4.1. Propósito y límites

Es dueño de **los perros y todo lo que los rodea**: publicación, fotos, estado de adopción, catálogos de raza y color, listados con filtros y favoritos. **No** conoce a los usuarios más allá de su identificador: los datos del publicador los pide a `Auth`. **No** gestiona solicitudes: al cambiar el estado de un perro avisa a `Requests`.

### 4.2. Datos

| Entidad | Notas |
|---|---|
| `PERRO` | Estados disponible y adoptado, borrado lógico irreversible, fecha de publicación que se conserva al volver a disponible. |
| `FOTO_PERRO` | De 1 a 5 fotos por perro; la de orden 1 es la principal. Guarda la clave del archivo, no la imagen. |
| `RAZA`, `COLOR` | Catálogos; un valor en uso solo se desactiva. |
| `FAVORITO` | Clave compuesta de usuario y perro; el usuario es una referencia lógica a `Auth`. |

Un perro se oculta cuando su publicador es dado de baja (RN-16). Como `Dogs` no puede consultar a los usuarios, la baja lo comunica por la llamada interna y `Dogs` lo conserva como una marca propia (decisión abierta de la sección 9).

### 4.3. Operaciones públicas

| Método y ruta | RF | Quién | Qué hace y reglas principales |
|---|---|---|---|
| `GET /dogs` | RF-01 | Público | Lista los perros disponibles en tarjetas y los filtra por raza, edad, tamaño, género, pelaje, color, esterilizado, fecha de publicación y nombre. Excluye eliminados y perros de cuentas dadas de baja. |
| `GET /dogs/{id}` | RF-02, RF-29 | Público; ADMINISTRADOR ve también eliminados | Ficha con fotos, datos físicos, información médica y publicador (pedido a `Auth`). Un adoptado se muestra con «Adoptado»; un eliminado o de una cuenta dada de baja responde 404. |
| `POST /dogs` | RF-19 | Registrado | Publica el perro como disponible: valida los datos (peso de 0,5 a 90 kg; fecha de nacimiento no futura ni de más de 20 años), raza y color activos y fotos validadas por su contenido. Si falla el almacenamiento, no queda nada incompleto (RNF-11). |
| `POST /dogs/{id}/photos` | RF-19, RF-20 | Publicador | Sube **una foto por petición**; valida contenido, genera el nombre, quita metadatos (RNF-22). |
| `PUT /dogs/{id}` | RF-20 | Publicador (el administrador no edita perros ajenos) | Edita los datos del perro. |
| `DELETE /dogs/{id}` | RF-21, RF-29 | Publicador o ADMINISTRADOR | Pide confirmación; marca el perro como eliminado y rechaza sus solicitudes pendientes (flujo F5). Un administrador no avisa al publicador. |
| `GET /dogs/mine` | RF-32 | Registrado | «Mis perros»: disponibles y adoptados, del más reciente al más antiguo, con las acciones de editar, eliminar y cambiar de estado. |
| `PUT /dogs/{id}/status` | RF-23 | Publicador | Marca adoptado o devuelve a disponible; al marcar adoptado rechaza las solicitudes pendientes (flujo F5). Al volver a disponible conserva la fecha de publicación y las solicitudes aceptadas. |
| `PUT` y `DELETE /dogs/{id}/favorite` | RF-12 | Registrado | Marca o desmarca el favorito. |
| `GET /dogs/favorites` | RF-33 | Registrado | Favoritos del último agregado al primero; incluye adoptados y excluye eliminados. |
| `GET /dogs/catalogs/breeds` y `/colors` | RF-19, RF-30 | Público | Lista los valores activos (el administrador ve todos). |
| `POST`, `PUT` y `DELETE` en `/dogs/catalogs/breeds` y `/colors` | RF-30 | ADMINISTRADOR | Agrega, renombra, activa, desactiva o elimina valores; sin nombres repetidos; un valor en uso no se elimina. |

### 4.4. Operaciones internas que expone

| Operación | La usa | Descripción |
|---|---|---|
| `consultarPerro` (en lote) | `Requests` | Datos y estado de varios perros; no devuelve eliminados. |
| Perros de un publicador (solo ids) | `Requests` | Para que el publicador vea las solicitudes recibidas. |
| Candidatos | `Recommendation` | Perros disponibles, no propios y compatibles con el filtro del test, con los atributos que usa la fórmula. |
| `marcarAdoptado` | `Requests` | Marca adoptado, idempotente; con origen `Requests` no vuelve a llamarla. |
| `listarPerrosDePublicador` | `Auth` | Perros visibles de un perfil público. No llama de vuelta a `Auth`. |
| `darBajaCuenta` / `restaurarCuenta` | `Auth` | Oculta o muestra los perros de una cuenta. |

### 4.5. Llamadas que realiza

| A | Para qué |
|---|---|
| `Auth` | Datos de los publicadores en lote (ficha, listados). |
| `Requests` | Rechazar las solicitudes pendientes de un perro cuando pasa a adoptado manualmente o se elimina. |

### 4.6. Adaptadores y requisitos

Usa el adaptador de **almacenamiento** (fotos de los perros). Requisitos: RNF-01 (listados en 3 s), RNF-04, RNF-07, RNF-10, RNF-11, RNF-13, RNF-14 y RNF-22.

### 4.7. Pruebas clave

Filtros combinados y vacío; ficha de perro adoptado, eliminado y de cuenta dada de baja; máximo de 5 fotos y rechazo de archivos falsos; solo el publicador edita y cambia de estado; el administrador elimina pero no edita; el valor de catálogo en uso no se elimina; el favorito de un perro eliminado no se lista.

---

## 5. Microservicio `Requests`

### 5.1. Propósito y límites

Es dueño de **las solicitudes que alguien resuelve**: las de adopción (el adoptante pide, el publicador resuelve) y las de verificación (una persona o un refugio piden, un administrador resuelve). Comparten el patrón pendiente → resuelta, pero **se mantienen como dos módulos con dos tablas separadas**; la lógica común de transición de estados vive en la librería compartida. **No** guarda datos de perros ni de usuarios: los pide a `Dogs` y `Auth`.

### 5.2. Datos

| Entidad | Notas |
|---|---|
| `SOLICITUD_ADOPCION` | Estados pendiente, aceptada, rechazada y cancelada. El `mensaje` conserva el teléfono vigente al solicitar y no se actualiza (RN-30). |
| `SOLICITUD_VERIFICACION` | Tipos rescatista y refugio, con las restricciones de coherencia de `solicitudes_verificacion.md` §2.4: una sola pendiente por rescatista y un solo correo pendiente por refugio (sin distinguir mayúsculas); al resolver un refugio se anulan `correo_solicitante` y `datos_registro`. |

### 5.3. Operaciones públicas: módulo de adopción

| Método y ruta | RF | Quién | Qué hace y reglas principales |
|---|---|---|---|
| `POST /requests/adoptions` | RF-16 | Adoptante | Con el id del perro y confirmación, crea una solicitud pendiente con el mensaje (nombre, teléfono vigente y enlace al perfil). El perro debe estar disponible y no ser propio; el solicitante no es REFUGIO y no tiene otra pendiente para ese perro. |
| `GET /requests/adoptions/mine` | RF-17 | Adoptante | Historial con perro, fecha y estado. |
| `POST /requests/adoptions/{id}/cancel` | RF-18 | Adoptante | Cancela una solicitud propia pendiente; si no lo está: «La solicitud ya no está pendiente y no puede cancelarse». |
| `GET /requests/adoptions/received` | RF-22 | Publicador | Solicitudes de sus perros con mensaje, nombre, foto y contacto del interesado (obtiene sus perros de `Dogs` y los datos de los solicitantes de `Auth`). |
| `POST /requests/adoptions/{id}/accept` y `/reject` | RF-22 | Publicador del perro | Solo si está pendiente. Al aceptar, el perro pasa a adoptado y las demás pendientes se rechazan (flujo F4). Al rechazar, solo cambia esa solicitud. Sin comentario ni notificación. |

### 5.4. Operaciones públicas: módulo de verificación

| Método y ruta | RF | Quién | Qué hace y reglas principales |
|---|---|---|---|
| `POST /requests/verifications/shelter` | RF-06 | Visitante | Valida el formulario del refugio, que el correo no pertenezca a una cuenta (`Auth`) ni tenga otra solicitud pendiente, guarda la foto por el adaptador (si falla, no registra) y crea la solicitud pendiente **sin crear cuenta ni pedir contraseña**. |
| `POST /requests/verifications/rescuer` | RF-24 | Solo MIEMBRO | Crea la solicitud pendiente. «Ya tienes una solicitud de verificación pendiente»; tras un rechazo, espera de 5 minutos: «Podrás enviar una nueva solicitud en unos minutos». Mientras está pendiente la persona sigue publicando y adoptando. |
| `GET /requests/verifications` | RF-25 | ADMINISTRADOR | Lista las solicitudes pendientes. |
| `POST /requests/verifications/{id}/approve` | RF-25 | ADMINISTRADOR | Rescatista: cambia el rol a RESCATISTA. Refugio: crea la cuenta (flujo F2). Solo si está pendiente: «La solicitud ya fue resuelta». |
| `POST /requests/verifications/{id}/reject` | RF-25 | ADMINISTRADOR | Rechaza sin motivo y envía el aviso por correo; en refugio borra correo y datos. El refugio puede reenviar de inmediato. |

### 5.5. Operaciones internas que expone

| Operación | La usa | Descripción |
|---|---|---|
| `rechazarPendientesDelPerro` | `Dogs` | Rechaza las solicitudes pendientes de un perro (adoptado manualmente o eliminado); idempotente y sin aviso. |
| `darBajaCuenta` / `restaurarCuenta` | `Auth` | Rechaza las solicitudes de adopción pendientes enviadas y recibidas por la cuenta; al restaurar no se reabren. El tratamiento de una solicitud de rescatista pendiente está abierto (sección 9). |
| `existeSolicitudRefugioPendiente` | `Auth` | Indica si un correo está reservado por una solicitud de refugio pendiente. |

### 5.6. Llamadas que realiza

| A | Para qué |
|---|---|
| `Auth` | Comprobar si un correo pertenece a una cuenta; cambiar el rol; crear la cuenta del refugio; obtener los datos de los solicitantes en lote. |
| `Dogs` | Comprobar disponibilidad y propiedad del perro; marcarlo adoptado; obtener los datos de varios perros y los perros de un publicador. |

### 5.7. Adaptadores y requisitos

Usa el adaptador de **correo** (resultado de las verificaciones) y de **almacenamiento** (foto de la solicitud de refugio). Requisitos: RNF-04, RNF-06, RNF-08, RNF-11, RNF-14 y RNF-23.

### 5.8. Pruebas clave

Una sola pendiente por perro y adoptante; solo el publicador resuelve y solo lo pendiente; aceptar rechaza las demás; la espera de 5 minutos aplica al rescatista y no al refugio; aprobar un refugio dos veces no duplica la cuenta; la solicitud de refugio rechazada no conserva correo ni datos; el refugio no puede solicitar adopciones.

---

## 6. Microservicio `Recommendation`

### 6.1. Propósito y límites

Es dueño del **test de estilo de vida y de las recomendaciones**. Guarda las respuestas del test, calcula la afinidad con los perros, consulta al Servicio de IA y devuelve 6 recomendaciones, con respaldo de plantillas si la IA falla. **No** guarda las recomendaciones (RN-47) ni conoce a los usuarios: usa la identidad de la petición. Los detalles de la fórmula, el prompt y las plantillas están en `adog_REC.md`.

### 6.2. Datos

| Entidad | Notas |
|---|---|
| `PERFIL_ADOPTANTE` | Respuestas del test de una persona (7 preguntas, con comentario opcional de hasta 150 caracteres). Se borra de forma definitiva (RN-53). `id_persona` es una referencia lógica a `Auth`. |

### 6.3. Operaciones públicas

| Método y ruta | RF | Quién | Qué hace y reglas principales |
|---|---|---|---|
| `GET /recommendation/test` | RF-13 | Adoptante | Devuelve las respuestas guardadas del propio usuario. Nadie más las ve, ni el administrador (RN-41). |
| `PUT /recommendation/test` | RF-13, RF-15 | Adoptante | Valida las respuestas obligatorias y guarda o reemplaza el test. El aviso de privacidad se muestra antes de empezar. Los refugios no realizan el test (RN-42). |
| `DELETE /recommendation/test` | RF-15 | Adoptante | Con confirmación, borra el test de forma definitiva. |
| `GET /recommendation/recommendations` | RF-14 | Adoptante | Genera 6 recomendaciones (flujo F8). Sin test, indica que debe realizarlo. |

### 6.4. Operaciones internas que expone

| Operación | La usa | Descripción |
|---|---|---|
| `darBajaCuenta` | `Auth` | Borra el test de una cuenta dada de baja (RF-11, RF-28). |

### 6.5. Llamadas que realiza

| A | Para qué |
|---|---|
| `Dogs` | Obtener los perros candidatos (disponibles, no propios y compatibles). |
| Servicio de IA (adaptador) | Elegir 6 de los 20 mejores y redactar «Por qué encaja» y «Lo que debes saber». |

### 6.6. Adaptadores y requisitos

Usa el adaptador de **IA**. Requisitos: RNF-02 (10 s, 15 s con arranque en frío), RNF-09 (solo respuestas del test y atributos de los perros, nunca identidad ni ubicación), RNF-11, RNF-13, RNF-15, RNF-18 (tope de USD 5 al mes) y RNF-19 (el comentario y las descripciones se envían como datos, nunca como instrucciones).

### 6.7. Pruebas clave

Test incompleto; solo el dueño accede a su test; refugio rechazado; la respuesta de la IA con perros repetidos, ajenos a los candidatos o con textos largos se descarta; falla o demora de la IA devuelve el respaldo dentro del límite; comentarios con instrucciones maliciosas no alteran el resultado.

---

## 7. Catálogo de llamadas internas

| Operación | La expone | La usa | Escritura | Idempotencia |
|---|---|---|---|---|
| `validarSesion` | `Auth` | Autorizador (todos) | No (actualiza la última actividad) | — |
| `obtenerPublicadores` / `obtenerContactoSolicitante` | `Auth` | `Dogs`, `Requests` | No | — |
| `existeCuentaConCorreo` | `Auth` | `Requests` | No | — |
| `cambiarRol` | `Auth` | `Requests` | Sí | Clave: id de la solicitud |
| `crearCuentaRefugio` | `Auth` | `Requests` | Sí | Clave: id de la solicitud |
| `existeSolicitudRefugioPendiente` | `Requests` | `Auth` | No | — |
| `consultarPerro` (lote, perros de un publicador, candidatos) | `Dogs` | `Requests`, `Recommendation` | No | — |
| `marcarAdoptado` | `Dogs` | `Requests` | Sí | Clave: id de la solicitud |
| `listarPerrosDePublicador` | `Dogs` | `Auth` | No | — |
| `rechazarPendientesDelPerro` | `Requests` | `Dogs` | Sí | Clave: id del perro y motivo |
| `darBajaCuenta` / `restaurarCuenta` | `Dogs`, `Requests`, `Recommendation` | `Auth` | Sí | Clave: id del usuario |

Reglas de las llamadas: ver la sección 2.4 y `adog_ARCH.md` §3.2 y §4.3.

---

## 8. Flujos que cruzan servicios

Cada flujo sigue la regla de la sección 2.4 (efecto en el servicio dueño primero, estado propio al final) y se puede repetir ante un fallo.

### F1. Registro con correo reservado

1. `Auth` valida los datos y consulta a `Requests` si el correo tiene una solicitud de refugio pendiente.
2. `Auth` crea la cuenta y el token de confirmación y, después de guardar, envía el correo.
3. Si el correo no sale, la cuenta existe y el usuario puede pedir el reenvío; no se pierden datos.

*Riesgo asumido:* entre la consulta y la creación puede entrar una solicitud de refugio con el mismo correo; se mitiga porque la aprobación vuelve a comprobar que el correo no pertenezca a una cuenta.

### F2. Aprobar una solicitud de refugio

1. `Requests` comprueba que la solicitud siga pendiente y vuelve a validar `datos_registro`.
2. `Requests` llama a `Auth.crearCuentaRefugio` con el id de la solicitud como clave. `Auth`, en una sola transacción propia, crea la cuenta (correo confirmado, sin contraseña, rol REFUGIO), su perfil y el enlace para definir la contraseña (10 min).
3. Con la cuenta creada, `Requests` marca la solicitud como aprobada, registra administrador y fecha, vincula `id_usuario` y anula `correo_solicitante` y `datos_registro`.
4. Se envía el correo de aprobación con el enlace.

*Fallos:* si falla el paso 2, no queda nada y el administrador reintenta. Si falla el paso 3, la solicitud sigue pendiente y reintentar no duplica la cuenta (clave de idempotencia). Si falla el correo, no se deshace nada y la contraseña se define luego con «Recuperar contraseña».
*Correo (a confirmar):* el aviso de aprobación del refugio, que contiene el enlace de `Auth`, lo envía `Auth` al crear la cuenta; los demás resultados (rechazos y aprobación de rescatista) los envía `Requests`.

### F3. Aprobar una solicitud de rescatista

1. `Requests` comprueba que siga pendiente.
2. `Requests` llama a `Auth.cambiarRol` (MIEMBRO → RESCATISTA, idempotente por el id de la solicitud); el rol rige de inmediato.
3. `Requests` marca la solicitud como aprobada y envía el correo del resultado.

### F4. Aceptar una solicitud de adopción

1. `Requests` comprueba que la solicitud siga pendiente y que quien acepta sea el publicador del perro (dato de `Dogs`).
2. `Requests` llama a `Dogs.marcarAdoptado` con origen `Requests`; `Dogs` no vuelve a llamar a `Requests`.
3. `Requests` marca la solicitud como aceptada y **rechaza por sí mismo** las demás pendientes del perro.

*Fallos:* si falla el paso 3, la solicitud sigue pendiente; al reintentar, `Requests` continúa aunque el perro ya figure como adoptado.

### F5. Adopción manual o eliminación de un perro (nace en `Dogs`)

1. `Dogs` valida que el usuario sea el publicador (o administrador, para eliminar).
2. `Dogs` llama a `Requests.rechazarPendientesDelPerro` (idempotente).
3. `Dogs` cambia el estado del perro (adoptado o eliminado).

### F6. Baja y restauración de una cuenta (RF-11 y RF-28)

1. `Auth` valida la operación (el administrador no se elimina a sí mismo ni deja el sistema sin administradores).
2. `Auth` llama a `Dogs` (ocultar sus perros), `Requests` (rechazar sus solicitudes pendientes) y `Recommendation` (borrar su test), todas idempotentes por el id del usuario.
3. `Auth` marca la cuenta como eliminada y cierra sus sesiones.

La restauración hace lo inverso para los perros (mostrarlos) y no reabre las solicitudes ni recupera el test. *Fallos:* si algún servicio no responde, la cuenta sigue activa y se reintenta.

### F7. Solicitar una adopción

1. `Requests` obtiene de `Dogs` el perro (disponible, no propio) y de `Auth` el nombre y teléfono vigentes del solicitante.
2. `Requests` genera el mensaje y crea la solicitud pendiente.

### F8. Generar recomendaciones

1. `Recommendation` lee el test del usuario y obtiene de `Dogs` los candidatos.
2. Calcula la afinidad y preselecciona los 20 mejores.
3. Consulta a la IA con límite de 10 s (15 s con arranque en frío); valida la respuesta (6 perros o todos si eran 6 o menos, sin repetidos, ajenos al conjunto o con textos fuera de límite).
4. Si la IA falla, tarda o responde algo inválido, usa la fórmula con plantillas. Devuelve 6 recomendaciones y no las guarda.

---

## 9. Decisiones abiertas

| # | Punto | Propuesta o consecuencia |
|---|---|---|
| 1 | Perros ocultos por la baja del publicador | El MER los trata como dato derivado, pero `Dogs` no puede consultar a los usuarios. Se propone que `Dogs` guarde una marca en `PERRO` (por ejemplo `publicador_eliminado`) que actualiza la llamada de baja o restauración. Requiere un ajuste a `adog_MER.md`. |
| 2 | Quién envía cada correo | Propuesta del flujo F2: `Auth` envía el aviso con enlace de la aprobación de refugio; `Requests` envía los demás resultados. |
| 3 | Solicitud de rescatista pendiente de una cuenta dada de baja | `solicitudes_verificacion.md` §8 (punto 3) la deja sin definir; hay que decidir si pasa a rechazada. |
| 4 | Foto de la solicitud de refugio | La guarda `Requests` y la reutiliza `Auth` al crear el perfil; si se rechaza, queda sin referencia (aceptado en `solicitudes_verificacion.md`). |
| 5 | Orden de las fotos al publicar | Crear el perro y luego subir las fotos una por una, o subirlas antes con clave temporal; se fija en el contrato de `Dogs`. |
| 6 | Valores finales | Tiempos límite internos, de la IA y de cada Lambda, tamaño máximo por foto y transporte de la sesión (`adog_ARCH.md` §7). |

---

## 10. Trazabilidad con los requisitos funcionales

| Servicio | RF |
|---|---|
| `Auth` | RF-03, RF-04, RF-05, RF-07, RF-08, RF-09, RF-10, RF-11, RF-26, RF-27, RF-28, RF-31, RF-34 |
| `Dogs` | RF-01, RF-02, RF-12, RF-19, RF-20, RF-21, RF-23, RF-29, RF-30, RF-32, RF-33 |
| `Requests` | RF-06, RF-16, RF-17, RF-18, RF-22, RF-24, RF-25 |
| `Recommendation` | RF-13, RF-14, RF-15 |

Los 34 RF quedan cubiertos por un único servicio dueño; los que cruzan servicios (RF-11, RF-21, RF-22, RF-23, RF-25, RF-28) se describen en los flujos de la sección 8.
