# ADOGTAME — Reglas de negocio

> **Basado en:** ERS v2.1.0 (13/11/2026), §5 *Reglas de negocio*. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Reúne las **55 reglas de negocio** (RN-01 a RN-55) agrupadas por módulo. Cada requisito funcional cita en su ficha (ERS §6) las reglas que le aplican. La validación de formato de los datos no se trata aquí: está en la fila *Entradas* de cada RF (ERS §6).
> Sobre las columnas: **Excepción/acción** indica qué ocurre o qué mensaje muestra el sistema cuando la regla se incumple; «N/A» significa que la regla no define una excepción.

| Módulo | Reglas | Sección del ERS |
|---|---|---|
| Cuentas y acceso | RN-01 a RN-13, RN-54, RN-55 | §5.1 |
| Perfiles, visibilidad y favoritos | RN-14 a RN-19 | §5.2 |
| Perros y catálogos | RN-20 a RN-27 | §5.3 |
| Adopción | RN-28 a RN-34 | §5.4 |
| Verificación | RN-35 a RN-38 | §5.5 |
| Recomendación | RN-39 a RN-53 | §5.6 |

---

## 1. Cuentas y acceso (ERS §5.1)

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-01 | El correo es único en todo el sistema: no puede repetirse entre cuentas activas, cuentas eliminadas ni solicitudes de refugio pendientes. | El sistema muestra "El correo ya está registrado" (en el registro de refugio, "Ya existe una solicitud pendiente con este correo" si la coincidencia es con una solicitud). |
| RN-02 | Solo pueden iniciar sesión las cuentas con correo confirmado y no eliminadas. | Correo sin confirmar: "Debes confirmar tu correo antes de iniciar sesión". Cuenta eliminada: "Usuario o contraseña incorrectos". |
| RN-03 | Los mensajes de inicio de sesión y de recuperación de contraseña no revelan si un correo está registrado. | Inicio de sesión: "Usuario o contraseña incorrectos" en todos los casos. Recuperación: el mismo mensaje exista o no el correo. |
| RN-04 | El tipo de cuenta no se puede modificar después de creada, ni por el usuario ni por el administrador. | El campo no se ofrece en los formularios de edición y el servidor rechaza el cambio. |
| RN-05 | El correo de una cuenta solo cambia cuando el usuario lo confirma desde el nuevo correo; hasta entonces, la cuenta usa el correo actual para iniciar sesión, recuperar la contraseña y en su perfil. | Enlace usado, caducado o reemplazado: "El enlace no es válido o ha caducado". |
| RN-06 | Una cuenta tiene como máximo un cambio de correo pendiente; una nueva solicitud invalida el enlace anterior. | El enlace anterior responde "El enlace no es válido o ha caducado". |
| RN-07 | El correo pendiente de confirmación no se reserva; su unicidad se verifica al solicitar el cambio y otra vez al confirmarlo. | Si otra cuenta lo registró entretanto: "El correo ya está registrado" y no se realiza el cambio. |
| RN-08 | Solo el propio usuario cambia su correo y su contraseña; el administrador no puede modificarlos ni ver la contraseña de ningún usuario. | Los campos no se ofrecen en la gestión de usuarios y el servidor rechaza el cambio. |
| RN-09 | La eliminación de una cuenta es lógica y solo un administrador puede revertirla; el correo de una cuenta eliminada no queda disponible para un nuevo registro. | N/A |
| RN-10 | Al eliminar una cuenta se ocultan sus perros, sus solicitudes pendientes enviadas y recibidas pasan a rechazadas y su test se borra. Al restaurarla, sus perros vuelven a mostrarse, pero las solicitudes no se reabren y el test no se recupera. | N/A |
| RN-11 | Siempre debe existir al menos un administrador activo, y un administrador no puede eliminar su propia cuenta. | "Un administrador no puede eliminar su propia cuenta" (RF-11), "No puedes eliminar tu propia cuenta" o "El sistema debe tener al menos un administrador activo" (RF-28). |
| RN-12 | Las cuentas de administrador solo las crea otro administrador o el despliegue inicial del sistema. | No existe registro público de administradores. |
| RN-13 | El enlace para definir la contraseña de un refugio aprobado o de una cuenta creada por un administrador también confirma su correo; si caduca, la contraseña se define con «Recuperar contraseña». | Enlace caducado: "El enlace no es válido o ha caducado". |
| RN-54 | Al cambiar la contraseña se cierran las demás sesiones abiertas de la cuenta y se mantiene la actual; al recuperarla se cierran todas las sesiones. | Las sesiones cerradas deben volver a iniciar sesión con la nueva contraseña. |
| RN-55 | Una sesión se cierra automáticamente tras 30 minutos sin actividad (RNF-25). | En la siguiente acción, el sistema muestra "Tu sesión expiró. Inicia sesión nuevamente". |

---

## 2. Perfiles, visibilidad y favoritos (ERS §5.2)

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-14 | Todo usuario tiene un perfil público. Solo existen listados navegables de rescatistas y refugios verificados; a los demás perfiles se accede por enlace, sin buscador general de usuarios. | N/A |
| RN-15 | El teléfono y el correo de un usuario solo se muestran a usuarios registrados; las respuestas del sistema a un visitante no incluyen esos datos. | Visitante: el sistema muestra "Inicia sesión para ver los datos de contacto". |
| RN-16 | Un perro eliminado, un perro de un usuario eliminado o el perfil de un usuario eliminado responden 404 y no aparecen en listados, favoritos, perfiles ni recomendaciones; solo el administrador puede consultarlos. | "Perro no encontrado" o "Usuario no encontrado". |
| RN-17 | Un perro adoptado no aparece en el listado público ni en las recomendaciones; se accede a él por su enlace, desde favoritos o desde el perfil de su publicador, con la etiqueta "Adoptado" y sin la opción de solicitar adopción. | Solicitud sobre un perro adoptado: "Este perro ya fue adoptado". |
| RN-18 | Los listados de perros se ordenan por fecha de publicación, de la más reciente a la más antigua. Excepciones: los favoritos, por la fecha en que se marcaron (la más reciente primero), y las recomendaciones, según RN-43 a RN-52. | N/A |
| RN-19 | Cualquier usuario registrado puede guardar perros en favoritos, incluidos los refugios y los administradores. | N/A |

---

## 3. Perros y catálogos (ERS §5.3)

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-20 | Pueden publicar perros las personas, los rescatistas, los refugios y los administradores. | N/A |
| RN-21 | Si se desconoce la fecha de nacimiento del perro, el publicador ingresa una fecha estimada; la edad siempre se calcula a partir de esa fecha. | N/A |
| RN-22 | Solo el publicador edita su perro y cambia su estado de adopción; el administrador solo puede eliminar perros ajenos, sin editarlos ni restaurarlos. | "No tienes permiso para editar este perro", "No tienes permiso para eliminar este perro" o "No tienes permiso para modificar este perro". |
| RN-23 | La eliminación de un perro es lógica e irreversible: las fotos no se borran del almacenamiento, sus solicitudes pendientes pasan a rechazadas y no se registra un motivo ni se avisa al publicador. | N/A |
| RN-24 | Un perro solo puede estar disponible o adoptado. Al volver a disponible conserva su fecha de publicación original y sus solicitudes aceptadas se mantienen como aceptadas. | N/A |
| RN-25 | Un perro puede marcarse como adoptado sin una solicitud aceptada; al hacerlo, sus solicitudes pendientes pasan a rechazadas. | N/A |
| RN-26 | Al publicar o editar un perro solo se ofrecen razas y colores activos; los perros que ya usan un valor desactivado lo conservan, y renombrar un valor cambia cómo se muestra en todos los perros que lo usan. | Raza o color inexistente o inactivo: "Selecciona una raza y un color válidos". |
| RN-27 | Los nombres de cada catálogo son únicos y un valor en uso no puede eliminarse, solo desactivarse. | "El valor ya existe" o "El valor está en uso; solo puede desactivarse". |

---

## 4. Adopción (ERS §5.4)

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-28 | Solo los adoptantes (personas, rescatistas y administradores) solicitan adopciones; los refugios no. No se puede solicitar un perro propio ni uno adoptado. | La opción no se muestra; si se intenta igualmente, el sistema muestra "Este perro ya fue adoptado" o el servidor rechaza la solicitud. |
| RN-29 | Un adoptante tiene como máximo una solicitud pendiente por perro, pero puede tener solicitudes pendientes para varios perros. Tras un rechazo o una cancelación puede volver a solicitar el mismo perro, sin límite de intentos. | "Ya tienes una solicitud pendiente para este perro". |
| RN-30 | El mensaje de la solicitud se genera automáticamente con el nombre del adoptante, el nombre del perro, el teléfono vigente al momento de solicitar y un enlace al perfil del adoptante. | N/A |
| RN-31 | Solo las solicitudes pendientes pueden cancelarse, aceptarse o rechazarse. | "La solicitud ya no está pendiente y no puede cancelarse" o "La solicitud ya no está pendiente". |
| RN-32 | Al aceptar una solicitud, el perro pasa a adoptado y las demás solicitudes pendientes de ese perro pasan a rechazadas. | N/A |
| RN-33 | Las solicitudes no generan notificaciones ni correos, y el publicador no escribe comentarios al aceptar o rechazar; cada parte consulta el estado en su panel. | N/A |
| RN-34 | En el historial del adoptante, una solicitud cuyo perro fue eliminado, o cuyo publicador fue eliminado, muestra el nombre del perro sin enlace a su perfil. | N/A |

---

## 5. Verificación (ERS §5.5)

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-35 | La cuenta de un refugio se crea solo cuando un administrador aprueba su solicitud. Las solicitudes pendientes no caducan y, si se rechazan, el refugio puede enviar una nueva. | N/A |
| RN-36 | Solo las personas pueden solicitar la verificación como rescatista, con una sola solicitud pendiente a la vez y una espera de 5 minutos tras un rechazo; mientras está pendiente, pueden seguir publicando y adoptando. | "Ya tienes una solicitud de verificación pendiente" o "Podrás enviar una nueva solicitud en unos minutos". |
| RN-37 | El resultado de una verificación se informa por correo, sin motivo en caso de rechazo. Una verificación aprobada no se revoca; si el rescatista o refugio deja de cumplir, se elimina su cuenta (RN-09). | N/A |
| RN-38 | Las cuentas de rescatista y de refugio creadas por un administrador quedan verificadas sin pasar por la solicitud. | N/A |

---

## 6. Recomendación (ERS §5.6)

El funcionamiento completo del módulo (test, fórmula, Servicio de IA y respaldo) está en el instrumento de recomendación (`adog_REC.md`).

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| RN-39 | Las respuestas del test se guardan tal cual, sin calcular un perfil; el comentario opcional no influye en la afinidad y solo lo usa el Servicio de IA. | N/A |
| RN-40 | Cambiar las respuestas reemplaza el test anterior; un test abandonado antes de enviarse no se guarda. | N/A |
| RN-41 | Las respuestas del test son privadas: no las ve otro usuario, incluido el administrador, ni aparecen en el perfil público. | N/A |
| RN-42 | Pueden realizar el test las personas, los rescatistas y los administradores; los refugios no. | La sección no se muestra al refugio y el servidor rechaza la petición. |
| RN-43 | Son candidatos los perros disponibles, no eliminados, de publicadores no eliminados y no publicados por el adoptante. Se excluyen los cachorros si el perro estaría solo más de 8 horas, y los perros con necesidades especiales si el adoptante no las acepta. | Sin candidatos: "Por ahora no hay perros que cumplan tus condiciones. Realiza el test nuevamente o explora el listado de perros". |
| RN-44 | Vivir en departamento o en casa sin patio penaliza a los perros grandes en la afinidad, pero no los excluye. | N/A |
| RN-45 | El tiempo de espera suma afinidad a los perros publicados hace más tiempo; a igual afinidad, va primero el publicado hace más tiempo. | N/A |
| RN-46 | Los pesos y las penalizaciones de la fórmula de afinidad son valores iniciales ajustables en el código. | N/A |
| RN-47 | La lista de recomendaciones se calcula cada vez que se abre la sección y no se guarda. | N/A |
| RN-48 | La afinidad no se muestra como número ni como porcentaje. | N/A |
| RN-49 | Todo perro recomendado muestra «Por qué encaja» (máx. 200 caracteres) y «Lo que debes saber» (máx. 160 caracteres). | Si la IA no los entrega válidos, se usan los textos de plantilla. |
| RN-50 | El Servicio de IA solo elige entre los candidatos enviados; una respuesta inválida o fuera de tiempo se descarta y se muestran los 6 primeros perros por afinidad con textos de plantilla. | El sistema responde "Respuesta del Servicio de IA no válida" y aplica el respaldo. |
| RN-51 | El Servicio de IA recibe como máximo 20 candidatos, nunca el catálogo completo. | N/A |
| RN-52 | El comentario del adoptante y las descripciones de los perros se envían al Servicio de IA como datos, nunca como instrucciones. | Si la respuesta incumple el formato o los límites, se aplica RN-50. |
| RN-53 | El test se borra de forma definitiva, no lógica, al eliminarlo o al eliminarse la cuenta. | N/A |

---

## 7. Criterios mínimos para aprobar una verificacióN

Las siguientes reglas de negocio salen de especificaciones formales que estarán implementadas en el sistema. Se las menciona porque corresponden a un comportamiento esperado por los administradores antes de poder aprobar una solicitud de verificación de rescatista o refugio. Sus detalles no condicionan, ni limitan lo que ya está especificado en los requisitos, casos de uso y reglas de negocio anteriores; por eso tienen hasta otro identificador.

| ID | Regla o condición | Excepción/acción |
|---|---|---|
| CRV-01 | Criterios mínimos para verificar a un rescatista: (a) perfil completo con foto, teléfono y descripción. (b) al menos 1 perro publicado en la plataforma. (c) atender a las llamadas telefónicas entrantes del administrador verificador. (d) evidencia de su labor de rescate en los últimos 6 meses (mediante las llamadas del criterio anterior). | Si falta un criterio, se rechaza la solicitud; puede volver a solicitar tras 5 minutos. |
| CRV-02 | Criterios para verificar un registro de refugio: (a) dirección institucional dentro del DMQ. (b) teléfono institucional confirmado mediante llamada. (c) presencia pública verificable (página web o red social con publicaciones de adopción de los últimos 6 meses). | Si falta un criterio, se rechaza la solicitud; el refugio puede enviar una nueva solicitud. |
