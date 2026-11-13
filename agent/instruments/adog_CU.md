# ADOGTAME — Casos de uso

> **Basado en:** ERS v2.1.0 (13/11/2026), §3 *Actores del sistema*, §4 *Casos de uso* y Anexo A *Diagrama de casos de uso*. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Describe **quién** usa el sistema y **qué** casos de uso tiene. El detalle de cada flujo, sus entradas, flujos alternativos y mensajes está en las fichas de requisitos funcionales (ERS §6).

El sistema contempla **9 actores** (3 abstractos) y **24 casos de uso**.

---

## 1. Actores

Los actores **abstractos** no corresponden a un tipo de cuenta: solo agrupan las acciones que heredan los demás. Un actor hereda **todos** los casos de uso de su padre.

| Actor | Tipo | Hereda de | Descripción |
|---|---|---|---|
| Visitante | Concreto | — | Persona sin cuenta. Explora los listados de perros (con filtros), de rescatistas y de refugios, y ve los perfiles sin datos de contacto. Puede registrarse como persona o solicitar el registro como refugio. |
| Usuario registrado | Abstracto | Visitante | Cuenta con correo confirmado. Inicia sesión, recupera su contraseña, guarda favoritos, ve los datos de contacto de los publicadores y administra su perfil. |
| Adoptante | Abstracto | Usuario registrado | Realiza el test, revisa sus recomendaciones, solicita adopciones y consulta o cancela sus solicitudes. |
| Publicador | Abstracto | Usuario registrado | Registra, edita y elimina sus perros, acepta o rechaza las solicitudes de adopción y cambia el estado de adopción de sus perros. |
| Persona | Concreto | Adoptante, Publicador | Cuenta creada mediante el registro público. Además, puede solicitar su verificación como rescatista. |
| Rescatista | Concreto | Persona | Persona verificada por un administrador. Conserva todas las funciones de Persona y muestra una insignia en su perfil y en el listado de rescatistas. |
| Refugio | Concreto | Publicador | Institución cuya cuenta se crea solo cuando un administrador aprueba su solicitud. No realiza el test ni solicita adopciones. Muestra una insignia y aparece en el listado de refugios. |
| Administrador | Concreto | Adoptante, Publicador | Miembro del equipo que resuelve verificaciones y gestiona usuarios, perros y catálogos. No puede solicitar ser rescatista ni eliminar su propia cuenta. |
| Servicio de IA | Sistema externo | — | Recibe el perfil del adoptante y una lista de perros candidatos, elige 6 y redacta para cada uno «Por qué encaja» y «Lo que debes saber». |

**Actores concretos y lo que pueden hacer** (por herencia):

| Actor concreto | Ejecuta casos de uso de |
|---|---|
| Visitante | Visitante |
| Persona | Visitante, Usuario registrado, Adoptante, Publicador y los propios de Persona |
| Rescatista | Los mismos que Persona |
| Refugio | Visitante, Usuario registrado y Publicador (no Adoptante) |
| Administrador | Visitante, Usuario registrado, Adoptante, Publicador y los propios de Administrador (no los de Persona) |

---

## 2. Diagrama de casos de uso (representación en texto)

Equivale al diagrama del Anexo A del ERS.

```
SISTEMA: Adogtame

LEYENDA
  [Actor]            actor concreto
  (Actor)            actor abstracto (solo agrupa casos de uso)
  <<sistema>> Actor  actor externo
  A --|> B           A hereda de B (A también ejecuta los casos de uso de B)
  A --- CU           A es el actor principal del caso de uso CU
  CU1 ..<<extend>>..> CU2    CU1 extiende a CU2 (comportamiento opcional o condicional)
  CU1 ..<<include>>..> CU2   CU1 incluye siempre a CU2

HERENCIA DE ACTORES
  (Usuario registrado) --|> [Visitante]
  (Adoptante)          --|> (Usuario registrado)
  (Publicador)         --|> (Usuario registrado)
  [Persona]            --|> (Adoptante)
  [Persona]            --|> (Publicador)
  [Rescatista]         --|> [Persona]
  [Refugio]            --|> (Publicador)
  [Administrador]      --|> (Adoptante)
  [Administrador]      --|> (Publicador)
  <<sistema>> Servicio de IA            (sin herencia)

ACTORES Y CASOS DE USO
  [Visitante]
    --- Explorar y filtrar perros
    --- Ver perfil del perro
    --- Ver listados y perfiles públicos
    --- Registrarse como persona
    --- Solicitar registro como refugio

  (Usuario registrado)
    --- Iniciar sesión
    --- Guardar perros en favoritos
    --- Ver datos de contacto de usuarios
    --- Administrar perfil
    (Recuperar contraseña no tiene actor propio: extiende Iniciar sesión)

  (Adoptante)
    --- Realizar test
    --- Revisar recomendaciones de perros
    --- Solicitar adopción
    --- Revisar mis solicitudes de adopción

  [Persona]
    --- Solicitar verificación como rescatista

  (Publicador)
    --- Registrar perro en adopción
    --- Gestionar información del perro
    --- Gestionar solicitudes de adopción
    --- Cambiar estado de adopción del perro

  [Administrador]
    --- Gestionar verificaciones
    --- Gestionar usuarios
    --- Gestionar perros
    --- Gestionar catálogos de formularios

  <<sistema>> Servicio de IA
    --- Seleccionar y explicar recomendaciones

RELACIONES ENTRE CASOS DE USO
  Recuperar contraseña                   ..<<extend>>..>  Iniciar sesión
  Solicitar adopción                     ..<<extend>>..>  Ver perfil del perro
  Guardar perros en favoritos            ..<<extend>>..>  Ver perfil del perro
  Cambiar estado de adopción del perro   ..<<extend>>..>  Gestionar solicitudes de adopción
  Revisar recomendaciones de perros      ..<<include>>..> Seleccionar y explicar recomendaciones
```

Condiciones de las relaciones:

- **Recuperar contraseña → Iniciar sesión:** se ofrece cuando el usuario olvidó su contraseña.
- **Revisar recomendaciones de perros → Seleccionar y explicar recomendaciones:** toda consulta de recomendaciones solicita al Servicio de IA que elija los perros y redacte sus explicaciones.
- **Solicitar adopción → Ver perfil del perro:** solo para adoptantes y perros disponibles que no publicaron.
- **Guardar perros en favoritos → Ver perfil del perro:** solo con sesión iniciada.
- **Cambiar estado de adopción del perro → Gestionar solicitudes de adopción:** ocurre al aceptar una solicitud; el publicador también puede ejecutarlo de forma manual o devolver el perro a disponible.

---

## 3. Fichas de casos de uso

Los RF y RNF listados son los que el ERS asocia a cada caso de uso en las fichas de sus requisitos funcionales. Los RNF de un caso de uso son la unión de los RNF de sus RF. Los RNF **transversales**, sin un RF asociado (RNF-12, RNF-16, RNF-17, RNF-20, RNF-21, RNF-22, RNF-23 y RNF-24), aplican a todo el sistema y no se repiten en cada ficha.

### 3.1 Casos de uso del Visitante

#### Explorar y filtrar perros

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Visitante | Consulta el listado de perros disponibles, mostrados en tarjetas (foto principal, nombre, edad, tamaño y género), y lo acota con filtros opcionales: raza, edad, tamaño, género, pelaje, color, esterilizado, fecha de publicación y nombre. | **RF:** RF-01<br>**RNF:** RNF-01 |

#### Ver perfil del perro

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Visitante | Abre el perfil de un perro desde el listado o por su enlace: fotos, datos físicos, información médica destacada (esterilizado, vacunas al día, necesidades especiales) y publicador con enlace a su perfil. Las acciones de guardar en favoritos y solicitar adopción se ofrecen según el usuario (casos de uso que lo extienden). Un perro adoptado se muestra con la etiqueta «Adoptado» y sin opción de adopción; uno eliminado responde 404. | **RF:** RF-02, RF-04, RF-12, RF-16<br>**RNF:** RNF-01, RNF-04, RNF-07, RNF-08 |

#### Ver listados y perfiles públicos

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Visitante | Consulta el listado de rescatistas o de refugios verificados (nombre, foto e insignia) y el perfil público de cualquier usuario, al que también se llega por enlace, con sus perros publicados disponibles y adoptados. No existe un buscador general de usuarios. | **RF:** RF-03, RF-04<br>**RNF:** RNF-01, RNF-04, RNF-08 |

#### Registrarse como persona

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Visitante | Completa el formulario de registro. El sistema valida los datos y la unicidad del correo, crea la cuenta como persona y envía un enlace de confirmación (un solo uso, vigencia de 24 horas). Sin confirmar el correo no se puede iniciar sesión. | **RF:** RF-05<br>**RNF:** RNF-03, RNF-05, RNF-08 |

#### Solicitar registro como refugio

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Visitante | Completa el formulario de la institución. El sistema guarda una solicitud pendiente **sin crear la cuenta** ni pedir contraseña; el refugio recibe el resultado por correo cuando un administrador la resuelve (ver *Gestionar verificaciones*). | **RF:** RF-06<br>**RNF:** RNF-05, RNF-08 |

### 3.2 Casos de uso del Usuario registrado

#### Iniciar sesión

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Usuario registrado | Ingresa correo y contraseña. El sistema valida las credenciales, que el correo esté confirmado y que la cuenta no esté eliminada, y da acceso según el tipo de cuenta. Sus mensajes no revelan si un correo está registrado. Es extendido por *Recuperar contraseña*. | **RF:** RF-07, RF-08<br>**RNF:** RNF-03, RNF-04, RNF-05, RNF-06, RNF-25 |

#### Recuperar contraseña

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Usuario registrado | Extiende *Iniciar sesión* cuando el usuario olvidó su contraseña. Ingresa su correo; el sistema envía (con el mismo mensaje exista o no el correo) un enlace de un solo uso válido por 10 minutos que permite definir una nueva contraseña. Al guardarla, cierra todas las sesiones abiertas de la cuenta. | **RF:** RF-08<br>**RNF:** RNF-03, RNF-06, RNF-25 |

#### Guardar perros en favoritos

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Usuario registrado | Extiende *Ver perfil del perro* (solo con sesión iniciada): marca o desmarca un perro como favorito. Además consulta su lista de favoritos, del último agregado al primero, que incluye los perros adoptados y excluye los eliminados. Disponible para todo usuario registrado, incluidos refugios y administradores. | **RF:** RF-12, RF-33<br>**RNF:** RNF-01 |

#### Ver datos de contacto de usuarios

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Usuario registrado | Con sesión iniciada, ve además el teléfono y el correo del publicador o del usuario en el perfil de un perro o de un usuario. Al visitante se le ocultan y se le invita a iniciar sesión. | **RF:** RF-04<br>**RNF:** RNF-04, RNF-08 |

#### Administrar perfil

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Usuario registrado | Gestiona su propia cuenta: actualizar datos del perfil (no el correo ni el tipo de cuenta), cambiar contraseña (cierra las demás sesiones y mantiene la actual), cambiar correo (se aplica solo al confirmar el enlace enviado al nuevo correo), cerrar sesión (en el dispositivo actual o en todos) y eliminar su cuenta de forma lógica tras una advertencia (un administrador no puede). La sesión también se cierra sola tras 30 minutos sin actividad. | **RF:** RF-09, RF-10, RF-11, RF-31, RF-34<br>**RNF:** RNF-03, RNF-04, RNF-05, RNF-06, RNF-07, RNF-08, RNF-10, RNF-25 |

### 3.3 Casos de uso del Adoptante

#### Realizar test

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Adoptante | Tras un aviso de privacidad, responde 7 preguntas sobre su estilo de vida (6 obligatorias de opción única y un comentario opcional de hasta 150 caracteres); el sistema valida y guarda las respuestas. También puede cambiarlas o eliminar su test, previa confirmación. Los refugios no realizan el test. | **RF:** RF-13, RF-15<br>**RNF:** RNF-08, RNF-09, RNF-10, RNF-15, RNF-19 |

#### Revisar recomendaciones de perros

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Adoptante | Con su test realizado, abre sus recomendaciones. El sistema excluye perros no disponibles, propios o incompatibles, ordena los restantes por afinidad y presenta 6 perros con «Por qué encaja» y «Lo que debes saber». Incluye siempre *Seleccionar y explicar recomendaciones*; si la IA falla, tarda o responde de forma inválida, usa la fórmula de afinidad con textos de plantilla. | **RF:** RF-14<br>**RNF:** RNF-02, RNF-09, RNF-11, RNF-13, RNF-18, RNF-19 |

#### Solicitar adopción

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Adoptante | Extiende *Ver perfil del perro*: desde el perfil de un perro disponible que no publicó, solicita su adopción. El sistema genera el mensaje (nombre, teléfono vigente y enlace al perfil del adoptante) y registra la solicitud como pendiente; solo se admite una pendiente por perro. No hay notificaciones: el publicador la ve en su panel. | **RF:** RF-16<br>**RNF:** RNF-04 |

#### Revisar mis solicitudes de adopción

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Adoptante | Consulta el historial de sus solicitudes con perro, fecha y estado (pendiente, aceptada, rechazada o cancelada) y cancela las que estén pendientes. | **RF:** RF-17, RF-18<br>**RNF:** RNF-01, RNF-04, RNF-14 |

### 3.4 Casos de uso de la Persona

#### Solicitar verificación como rescatista

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Persona | Solicita desde su perfil ser verificada como rescatista. El sistema la registra como pendiente si no hay otra pendiente ni un rechazo de hace menos de 5 minutos; mientras tanto la persona puede seguir publicando y adoptando. | **RF:** RF-24<br>**RNF:** RNF-04 |

### 3.5 Casos de uso del Publicador

#### Registrar perro en adopción

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Publicador | Completa el formulario del perro (campos obligatorios y opcionales, fotos, raza y color de los catálogos activos). El sistema valida, guarda las fotos y publica el perro como disponible. | **RF:** RF-19<br>**RNF:** RNF-11, RNF-13 |

#### Gestionar información del perro

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Publicador | Consulta «Mis perros» (disponibles y adoptados, del más reciente al más antiguo) y, sobre sus propios perros, edita sus datos o los elimina de forma lógica e irreversible, previa confirmación. | **RF:** RF-20, RF-21, RF-32<br>**RNF:** RNF-01, RNF-04, RNF-07, RNF-10, RNF-13, RNF-14 |

#### Gestionar solicitudes de adopción

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Publicador | Revisa las solicitudes de sus perros (mensaje, nombre, foto y contacto del interesado) y las acepta o rechaza. Al aceptar, el perro pasa a adoptado y las demás solicitudes pendientes de ese perro se rechazan (comportamiento de *Cambiar estado de adopción del perro*, que lo extiende). Al rechazar, solo cambia esa solicitud. | **RF:** RF-22<br>**RNF:** RNF-04, RNF-14 |

#### Cambiar estado de adopción del perro

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Publicador | Extiende *Gestionar solicitudes de adopción*: ocurre al aceptar una solicitud, y el publicador también puede marcar un perro como adoptado manualmente (se rechazan sus solicitudes pendientes) o devolverlo a disponible (conserva su fecha de publicación original y sus solicitudes aceptadas). | **RF:** RF-22, RF-23<br>**RNF:** RNF-04, RNF-14 |

### 3.6 Casos de uso del Administrador

#### Gestionar verificaciones

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Administrador | Revisa las solicitudes pendientes de refugios y de rescatistas y las aprueba o rechaza. Al aprobar, crea la cuenta del refugio (con enlace para definir su contraseña) o verifica al rescatista; en todos los casos envía un correo con el resultado, sin motivo en caso de rechazo. | **RF:** RF-25<br>**RNF:** RNF-04, RNF-06, RNF-11 |

#### Gestionar usuarios

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Administrador | Crea cuentas de persona, rescatista, refugio o administrador (las de rescatista y refugio quedan verificadas y reciben un enlace por correo para definir su contraseña), edita los datos de cualquier usuario (excepto correo, contraseña y tipo de cuenta) y elimina o restaura usuarios de forma lógica. No puede eliminarse a sí mismo ni dejar el sistema sin administradores. | **RF:** RF-26, RF-27, RF-28<br>**RNF:** RNF-03, RNF-04, RNF-06, RNF-07, RNF-08, RNF-10, RNF-14 |

#### Gestionar perros

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Administrador | Consulta cualquier perro, incluidos los adoptados y eliminados, y elimina el que considere de forma lógica, sin avisar al publicador. No puede editar ni restaurar perros ajenos. | **RF:** RF-29<br>**RNF:** RNF-04, RNF-07, RNF-10, RNF-14 |

#### Gestionar catálogos de formularios

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Administrador | Agrega, renombra, activa, desactiva o elimina los valores de los catálogos de raza y color. No se repiten nombres y un valor en uso solo puede desactivarse; el formulario del perro ofrece únicamente los valores activos. | **RF:** RF-30<br>**RNF:** RNF-04, RNF-14 |

### 3.7 Casos de uso del Servicio de IA

#### Seleccionar y explicar recomendaciones

| Actor | Descripción | RF y RNF relacionados |
|---|---|---|
| Servicio de IA | Incluido siempre por *Revisar recomendaciones de perros*. Recibe el perfil del adoptante (sin datos de identificación) y hasta 20 perros candidatos; elige 6, los ordena y redacta para cada uno «Por qué encaja» y «Lo que debes saber». Su respuesta se valida y, si no es válida o no llega a tiempo, se descarta. | **RF:** RF-14<br>**RNF:** RNF-02, RNF-09, RNF-11, RNF-13, RNF-18, RNF-19 |
