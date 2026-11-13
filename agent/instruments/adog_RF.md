# ADOGTAME — Requisitos funcionales

> **Basado en:** ERS v2.1.0 (13/11/2026), §6 *Requisitos funcionales*. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Resume de forma general las **34 funcionalidades** del sistema. Para cada requisito, la ficha completa del ERS §6 (entradas, flujo principal, flujos alternativos, postcondiciones y comentarios) es la especificación detallada. Las reglas de negocio (RN) están en `adog_RN.md`, los requisitos no funcionales (RNF) en `adog_RNF.md` y los casos de uso en `adog_CU.md`.

## Resumen

- **Prioridad (escala MoSCoW):** 18 «Debe tener», 14 «Debería tener» y 2 «Podría tener».
- **Numeración:** RF-31 a RF-34 se agregaron después para no renumerar los demás; pertenecen a los grupos 2 (RF-31, RF-34), 3 (RF-33) y 6 (RF-32).
- **Servicio de IA:** solo interviene en RF-14.
- **Columnas:** *Actor(es)* son los del ERS; *RN* y *RNF* son las reglas de negocio y los requisitos no funcionales que cita la ficha del requisito.

---

## 1. Exploración y perfiles públicos

> ERS §6, RF-01 a RF-04.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-01 | Explorar y filtrar perros | Visitante | Debe tener | El usuario aplica filtros sobre el listado de perros; el sistema consulta los perros disponibles que cumplen los criterios y los muestra en tarjetas. | RN-16, RN-17, RN-18 | RNF-01 |
| RF-02 | Ver perfil del perro | Visitante | Debe tener | El usuario selecciona un perro o accede por su enlace; el sistema muestra sus fotos, datos físicos, información médica destacada y publicador, junto con las acciones disponibles según el usuario. | RN-15, RN-16, RN-17, RN-21 | RNF-01, RNF-07 |
| RF-03 | Ver listados de rescatistas y refugios y perfiles públicos | Visitante | Debería tener | El usuario accede al listado de rescatistas o de refugios, o al perfil de cualquier usuario mediante un enlace; el sistema muestra el listado de cuentas verificadas o el perfil público con sus perros publicados. | RN-14, RN-15, RN-16, RN-18 | RNF-01 |
| RF-04 | Ver datos de contacto | Usuario registrado | Debe tener | El usuario con sesión iniciada abre el perfil de un perro o de un usuario; el sistema muestra además el teléfono y el correo, que se ocultan al visitante. | RN-15 | RNF-04, RNF-08 |

---

## 2. Registro, acceso y perfil propio

> ERS §6, RF-05 a RF-11, RF-31 y RF-34.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-05 | Registrar persona | Visitante | Debe tener | El visitante completa el formulario de registro; el sistema valida los datos y que el correo sea único, crea la cuenta como persona y envía un correo de confirmación, necesario para iniciar sesión. | RN-01, RN-02 | RNF-03, RNF-05, RNF-08 |
| RF-06 | Solicitar registro como refugio | Visitante | Debe tener | El visitante completa el formulario de refugio; el sistema valida los datos y que el correo no esté registrado ni tenga otra solicitud pendiente, y guarda la solicitud sin crear la cuenta hasta que un administrador la apruebe. | RN-01, RN-35 | RNF-05, RNF-08 |
| RF-07 | Iniciar sesión | Usuario registrado | Debe tener | El usuario ingresa su correo y contraseña; el sistema valida las credenciales, que el correo esté confirmado y que la cuenta no esté eliminada, y le da acceso según su tipo de cuenta. | RN-02, RN-03 | RNF-03, RNF-04, RNF-05 |
| RF-08 | Recuperar contraseña | Usuario registrado | Debería tener | El usuario ingresa su correo; el sistema envía un enlace de un solo uso válido por 10 minutos y, al abrirlo, permite definir una nueva contraseña. | RN-03, RN-13, RN-54 | RNF-03, RNF-06, RNF-25 |
| RF-09 | Actualizar datos del perfil | Usuario registrado | Debería tener | El usuario modifica los datos de su perfil, excepto el correo; el sistema los valida y guarda. | RN-04, RN-08 | RNF-04, RNF-08 |
| RF-10 | Cambiar contraseña | Usuario registrado | Debería tener | El usuario ingresa su contraseña actual y la nueva; el sistema verifica la actual, valida la nueva y la guarda. | RN-08, RN-54 | RNF-03, RNF-25 |
| RF-31 | Cambiar correo | Usuario registrado | Debería tener | El usuario ingresa su contraseña actual y el nuevo correo; el sistema envía un enlace de confirmación al nuevo correo y solo actualiza el correo de la cuenta cuando el usuario abre ese enlace. | RN-01, RN-05, RN-06, RN-07, RN-08 | RNF-03, RNF-04, RNF-05, RNF-06 |
| RF-34 | Cerrar sesión | Usuario registrado | Debe tener | El usuario cierra su sesión en el dispositivo actual o en todos sus dispositivos; el sistema invalida las sesiones correspondientes y lo devuelve al inicio como visitante. | RN-54, RN-55 | RNF-04, RNF-25 |
| RF-11 | Eliminar cuenta propia | Usuario registrado | Podría tener | El usuario solicita eliminar su cuenta; el sistema muestra una advertencia y, al confirmar, la elimina de forma lógica, oculta sus perros, rechaza sus solicitudes pendientes y borra su test. | RN-09, RN-10, RN-11, RN-16 | RNF-07, RNF-08, RNF-10 |

---

## 3. Favoritos

> ERS §6, RF-12 y RF-33.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-12 | Guardar perros en favoritos | Usuario registrado | Debería tener | Desde el perfil de un perro, el usuario lo marca o desmarca como favorito; el sistema actualiza su lista de favoritos, que puede consultar en su perfil. | RN-16, RN-17, RN-19 | RNF-01 |
| RF-33 | Consultar favoritos | Usuario registrado | Debería tener | El usuario abre su lista de favoritos; el sistema muestra los perros que marcó, del último agregado al primero. | RN-16, RN-17, RN-18 | RNF-01 |

---

## 4. Test y recomendaciones

> ERS §6, RF-13 a RF-15.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-13 | Realizar test | Adoptante | Debe tener | El adoptante responde 7 preguntas sobre su estilo de vida; el sistema muestra un aviso de privacidad, valida las respuestas obligatorias y las guarda para generar sus recomendaciones. | RN-39, RN-40, RN-41, RN-42, RN-52 | RNF-08, RNF-09, RNF-15, RNF-19 |
| RF-14 | Generar recomendaciones de perros | Adoptante, Servicio de IA | Debe tener | El sistema filtra y ordena por afinidad los perros compatibles con el test y envía los 20 mejores al Servicio de IA, que elige 6 y redacta para cada uno «Por qué encaja» y «Lo que debes saber»; el sistema valida la respuesta y presenta la lista, o usa la fórmula con plantillas si la IA falla. | RN-16, RN-17, RN-43, RN-44, RN-45, RN-46, RN-47, RN-48, RN-49, RN-50, RN-51, RN-52 | RNF-02, RNF-09, RNF-11, RNF-13, RNF-18, RNF-19 |
| RF-15 | Cambiar o eliminar test | Adoptante | Debería tener | El adoptante elimina su test o cambia sus respuestas, previa confirmación; el sistema borra o reemplaza las respuestas y, en el segundo caso, genera nuevas recomendaciones. | RN-40, RN-53 | RNF-08, RNF-10 |

---

## 5. Solicitudes de adopción del adoptante

> ERS §6, RF-16 a RF-18.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-16 | Solicitar adopción | Adoptante | Debe tener | Desde el perfil de un perro disponible, el adoptante solicita su adopción y confirma; el sistema genera un mensaje con su nombre, teléfono y enlace a su perfil, y registra la solicitud como pendiente. | RN-17, RN-28, RN-29, RN-30, RN-33 | RNF-04 |
| RF-17 | Consultar historial de solicitudes | Adoptante | Debe tener | El adoptante abre sus solicitudes; el sistema lista cada una con su perro, fecha y estado. | RN-33, RN-34 | RNF-01 |
| RF-18 | Cancelar solicitud pendiente | Adoptante | Debería tener | Desde su historial, el adoptante cancela una solicitud pendiente; el sistema cambia su estado a cancelada. | RN-29, RN-31 | RNF-04, RNF-14 |

---

## 6. Publicación de perros y gestión de solicitudes

> ERS §6, RF-19 a RF-23 y RF-32.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-19 | Registrar perro en adopción | Publicador | Debe tener | El publicador completa el formulario del perro; el sistema valida los datos, guarda las fotos y publica el perro como disponible. | RN-20, RN-21, RN-26 | RNF-11, RNF-13 |
| RF-20 | Editar información del perro | Publicador | Debe tener | El publicador modifica los datos de uno de sus perros; el sistema verifica que sea su publicador, valida los cambios y los guarda. | RN-22, RN-26 | RNF-04, RNF-13 |
| RF-21 | Eliminar perro | Publicador | Debe tener | El publicador elimina uno de sus perros; el sistema pide confirmación, lo marca como eliminado de forma lógica e irreversible y rechaza sus solicitudes pendientes. | RN-16, RN-22, RN-23 | RNF-04, RNF-07, RNF-10, RNF-14 |
| RF-32 | Consultar mis perros | Publicador | Debe tener | El publicador abre «Mis perros»; el sistema muestra sus perros disponibles y adoptados, de la publicación más reciente a la más antigua, con las acciones para editarlos, eliminarlos o cambiar su estado. | RN-16, RN-18, RN-24 | RNF-01, RNF-04 |
| RF-22 | Aceptar o rechazar solicitudes de adopción | Publicador | Debe tener | El publicador revisa las solicitudes de sus perros y acepta o rechaza una; al aceptar, el sistema marca el perro como adoptado y rechaza las demás solicitudes pendientes de ese perro. | RN-30, RN-31, RN-32, RN-33 | RNF-04, RNF-14 |
| RF-23 | Cambiar estado de adopción del perro | Publicador | Debería tener | El publicador marca manualmente un perro como adoptado o lo devuelve a disponible; al marcarlo adoptado, el sistema rechaza sus solicitudes pendientes. | RN-22, RN-24, RN-25 | RNF-04, RNF-14 |

---

## 7. Verificación de rescatistas y refugios

> ERS §6, RF-24 y RF-25.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-24 | Solicitar verificación como rescatista | Persona | Debe tener | La persona solicita desde su perfil ser verificada como rescatista; el sistema valida que no tenga otra solicitud pendiente ni un rechazo reciente y registra la solicitud como pendiente. | RN-36, RN-37 | RNF-04 |
| RF-25 | Resolver solicitudes de verificación | Administrador | Debe tener | El administrador revisa las solicitudes pendientes y las aprueba o rechaza; al aprobar, el sistema crea la cuenta del refugio o verifica al rescatista, y en todos los casos envía un correo con el resultado. | RN-13, RN-35, RN-37 | RNF-04, RNF-06, RNF-11 |

---

## 8. Administración de usuarios, perros y catálogos

> ERS §6, RF-26 a RF-30.

| ID | Requisito | Actor(es) | Prioridad | Descripción | RN | RNF |
|---|---|---|---|---|---|---|
| RF-26 | Crear usuarios | Administrador | Debería tener | El administrador ingresa los datos de una persona, rescatista, refugio o administrador; el sistema crea la cuenta (verificada si es rescatista o refugio) y envía un enlace por correo para definir la contraseña. | RN-01, RN-12, RN-13, RN-38 | RNF-04, RNF-06 |
| RF-27 | Editar datos de usuarios | Administrador | Podría tener | El administrador busca un usuario y modifica sus datos, excepto el correo, la contraseña y el tipo de cuenta; el sistema los valida y guarda. | RN-04, RN-08 | RNF-03, RNF-04 |
| RF-28 | Eliminar y restaurar usuarios | Administrador | Debería tener | El administrador elimina un usuario de forma lógica o restaura uno eliminado; el sistema oculta o vuelve a mostrar sus perros y, al eliminar, rechaza sus solicitudes pendientes y borra su test. | RN-09, RN-10, RN-11, RN-37 | RNF-04, RNF-07, RNF-08, RNF-10, RNF-14 |
| RF-29 | Gestionar perros | Administrador | Debería tener | El administrador consulta cualquier perro, incluidos los adoptados y eliminados, y elimina el que considere; el sistema lo marca como eliminado sin avisar al publicador. | RN-16, RN-18, RN-22, RN-23 | RNF-04, RNF-07, RNF-10, RNF-14 |
| RF-30 | Gestionar catálogos de raza y color | Administrador | Debería tener | El administrador agrega, renombra, activa, desactiva o elimina valores de raza y color; el sistema valida que no se repitan y que un valor en uso no se elimine, y solo ofrece los activos al publicar. | RN-26, RN-27 | RNF-04, RNF-14 |
