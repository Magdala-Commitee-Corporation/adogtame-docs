# ADOGTAME — Requisitos no funcionales

> **Basado en:** ERS v2.1.0 (13/11/2026), §7 *Requisitos no funcionales*. Si este documento y el ERS difieren, prevalece el ERS y este debe actualizarse.
> Resume las **25 restricciones y atributos de calidad** del sistema, agrupados por tipo. La ficha de cada RNF en el ERS §7 es la especificación detallada. Las funcionalidades están en `adog_RF.md`, las reglas de negocio en `adog_RN.md` y los casos de uso en `adog_CU.md`.

## Resumen

| Tipo | Cantidad | Requisitos |
|---|---|---|
| Rendimiento | 2 | RNF-01, RNF-02 |
| Seguridad | 10 | RNF-03, RNF-04, RNF-05, RNF-06, RNF-07, RNF-19, RNF-20, RNF-21, RNF-22, RNF-25 |
| Privacidad | 2 | RNF-08, RNF-09 |
| Fiabilidad | 2 | RNF-10, RNF-11 |
| Disponibilidad | 2 | RNF-12, RNF-23 |
| Mantenibilidad | 1 | RNF-13 |
| Usabilidad | 2 | RNF-14, RNF-15 |
| Portabilidad | 2 | RNF-16, RNF-17 |
| Costo | 2 | RNF-18, RNF-24 |

- **Prioridad (escala MoSCoW):** 16 «Debe tener», 8 «Debería tener» y 1 «Podría tener».
- **Columna *RF que lo citan*:** son los requisitos funcionales cuyas fichas (ERS §6) listan el RNF. Los RNF sin RF asociado (RNF-12, RNF-16, RNF-17, RNF-20, RNF-21, RNF-22, RNF-23, RNF-24) son **transversales**: aplican a todo el sistema.

---

## 1. Rendimiento

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-01 | Tiempo de carga de listados y perfiles | Debe tener | Con las funciones serverless activas, la consulta y presentación de listados, búsquedas con filtros y perfiles no debe superar los 3 segundos. La primera petición tras un periodo de inactividad (cold start, aceptado por diseño) no debe superar los 8 segundos, y mientras tanto la interfaz muestra un indicador de carga. | RF-01, RF-02, RF-03, RF-12, RF-17, RF-32, RF-33 |
| RNF-02 | Tiempo de respuesta de las recomendaciones | Debería tener | El Servicio de IA debe responder en un máximo de 10 segundos con las funciones activas, o de 15 segundos si se produce un cold start; pasado ese tiempo, el sistema muestra las recomendaciones calculadas con la fórmula de afinidad y textos de plantilla. | RF-14 |

---

## 2. Seguridad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-03 | Almacenamiento seguro de contraseñas | Debe tener | Las contraseñas se almacenan únicamente como hash con sal (por ejemplo, bcrypt) y nunca se muestran ni se guardan en texto plano. | RF-05, RF-07, RF-08, RF-10, RF-27, RF-31 |
| RNF-04 | Autorización en el servidor | Debe tener | Toda acción se autoriza en el servidor según el tipo de cuenta y, cuando aplica, la propiedad del recurso (perro, solicitud o perfil); ocultar opciones en la interfaz no sustituye esta validación. | RF-04, RF-07, RF-09, RF-16, RF-18, RF-20, RF-21, RF-22, RF-23, RF-24, RF-25, RF-26, RF-27, RF-28, RF-29, RF-30, RF-31, RF-32, RF-34 |
| RNF-05 | Comunicación cifrada | Debe tener | El 100 % de la comunicación entre el cliente, el servidor y los servicios externos se realiza mediante HTTPS. | RF-05, RF-06, RF-07, RF-31 |
| RNF-06 | Seguridad de los enlaces enviados por correo | Debe tener | Los enlaces de recuperación de contraseña, creación de contraseña y confirmación de cambio de correo son de un solo uso y caducan a los 10 minutos. El enlace de confirmación de registro es de un solo uso y caduca a las 24 horas. | RF-08, RF-25, RF-26, RF-31 |
| RNF-07 | Respuesta ante recursos eliminados | Debería tener | El acceso a un perro eliminado, a un perro de un usuario eliminado o al perfil de un usuario eliminado responde con el mismo código 404 y contenido que un recurso inexistente. | RF-02, RF-11, RF-21, RF-28, RF-29 |
| RNF-19 | Protección contra la manipulación de la IA | Debe tener | El comentario del adoptante y las descripciones de los perros se envían al Servicio de IA como datos, nunca como instrucciones. El sistema solo acepta respuestas con exactamente 6 perros (o todos los candidatos, si eran 6 o menos), todos pertenecientes a los candidatos enviados, sin repetidos y con textos dentro del límite (200 caracteres para «Por qué encaja» y 160 para «Lo que debes saber»); en caso contrario usa el respaldo. | RF-13, RF-14 |
| RNF-20 | Protección contra inyección SQL | Debe tener | Todas las consultas a la base de datos usan parámetros enlazados o un ORM; ningún dato ingresado por el usuario se concatena en una consulta. | Transversal |
| RNF-21 | Protección contra XSS | Debe tener | Todo texto ingresado por el usuario (nombres, descripciones, comentarios, mensajes) se valida y sanitiza en el servidor y se escapa al mostrarse; el sistema no admite HTML en ningún campo. La aplicación aplica una política de seguridad de contenido (CSP) que solo permite scripts y estilos del propio dominio, sin scripts en línea. | Transversal |
| RNF-22 | Validación de archivos y enlaces | Debe tener | Las imágenes se validan por su contenido real (no solo por la extensión), se guardan con un nombre generado por el sistema, sin metadatos, y se sirven desde el almacenamiento sin permisos de ejecución. Las direcciones web ingresadas por los usuarios solo admiten los protocolos http y https. | Transversal |
| RNF-25 | Gestión de sesiones | Debe tener | El sistema debe cerrar la sesión tras 30 minutos sin actividad y permitir cerrar sesión en todos los dispositivos. | RF-08, RF-10, RF-34 |

---

## 3. Privacidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-08 | Minimización de datos personales | Debería tener | Solo se solicitan los datos personales necesarios para cada funcionalidad; los registros del sistema (logs) no contienen contraseñas, tokens ni teléfonos o correos completos; el tratamiento se alinea con la Ley Orgánica de Protección de Datos Personales del Ecuador. | RF-04, RF-05, RF-06, RF-09, RF-11, RF-13, RF-15, RF-28 |
| RNF-09 | Datos enviados al Servicio de IA | Debe tener | Al Servicio de IA solo se envían las respuestas del test (incluido el comentario opcional) y los atributos y la descripción recortada de los perros candidatos; nunca el nombre, correo, teléfono o identificador del usuario ni la ubicación del perro. Antes del test se advierte que no se escriban datos personales en el comentario. | RF-13, RF-14 |

---

## 4. Fiabilidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-10 | Borrado lógico de perros y usuarios | Debe tener | Ningún perro ni usuario se elimina físicamente: se marcan como eliminados y se conservan sus registros (0 borrados físicos), salvo el test, que se borra definitivamente. | RF-11, RF-15, RF-21, RF-28, RF-29 |
| RNF-11 | Tolerancia a fallos de servicios externos | Debería tener | Si el Servicio de IA falla, el sistema presenta igualmente las recomendaciones con la fórmula de afinidad y textos de plantilla. Si fallan el envío de correos o el almacenamiento de imágenes, muestra un mensaje comprensible. En ningún caso se pierden ni quedan incompletos los datos ya registrados. | RF-14, RF-19, RF-25 |

---

## 5. Disponibilidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-12 | Disponibilidad mensual | Debería tener | El sistema está disponible al menos el 99 % del tiempo en cada mes calendario, excluyendo mantenimientos programados. | Transversal |
| RNF-23 | Limitación de tráfico y protección perimetral | Debería tener | API Gateway limita las peticiones (throttling) a 50 por segundo con ráfagas de hasta 100, y AWS WAF bloquea las direcciones IP que superen 500 peticiones en 5 minutos y aplica reglas administradas contra ataques comunes; las peticiones rechazadas reciben el código 429. | Transversal |

---

## 6. Mantenibilidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-13 | Proveedores externos sustituibles | Debería tener | El almacenamiento de imágenes, el Servicio de IA y el envío de correos se acceden mediante una capa de abstracción, de modo que el proveedor pueda cambiarse sin modificar la lógica de negocio. | RF-14, RF-19, RF-20 |

---

## 7. Usabilidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-14 | Interfaz clara y en español | Debe tener | La interfaz está en español, con diseño minimalista de tonos claros y botones que jerarquizan las acciones por tamaño y color; toda acción irreversible pide confirmación. | RF-18, RF-21, RF-22, RF-23, RF-28, RF-29, RF-30 |
| RNF-15 | Duración del test | Podría tener | El test puede completarse en un máximo de 2 minutos (objetivo de 1 minuto). | RF-13 |

---

## 8. Portabilidad

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-16 | Compatibilidad con navegadores | Debe tener | El sistema funciona en las 2 últimas versiones de Chrome, Firefox y Safari en computadoras de escritorio o portátiles. | Transversal |
| RNF-17 | Diseño adaptable (responsive) | Debería tener | La interfaz se adapta a anchos de pantalla desde 360 px (teléfonos) hasta 1920 px (computadoras de escritorio), sin desbordamiento horizontal ni elementos superpuestos; por encima de 1920 px el contenido se centra con un ancho máximo. No se desarrolla una aplicación móvil nativa. | Transversal |

---

## 9. Costo

| ID | Requisito | Prioridad | Descripción | RF que lo citan |
|---|---|---|---|---|
| RNF-18 | Costo de operación | Debe tener | El costo mensual del Servicio de IA no supera los USD 5, y la infraestructura se despliega en servicios serverless de AWS que se cobran por uso. | RF-14 |
| RNF-24 | Alertas de presupuesto | Debe tener | AWS Budgets envía alertas por correo al equipo cuando el gasto mensual alcanza el 50 %, 80 % y 100 % del presupuesto definido para AWS. | Transversal |
