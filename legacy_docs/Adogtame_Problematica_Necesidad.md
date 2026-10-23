# ADOGTAME — Problemática, necesidad y oportunidad

> Documento elaborado a partir de la sección «1. Problemática» de `Adogtame_Analisis_Final.md` y de las fuentes que allí se citan (ver [Referencias](#referencias)). Describe el problema real, a quién afecta y por qué debe resolverse. No incluye decisiones de solución, tecnología ni diseño.

---

## Índice

1. [Problemática](#1-problemática)
2. [Necesidad](#2-necesidad)
3. [Oportunidad](#3-oportunidad)
4. [Consecuencias de no resolver el problema](#4-consecuencias-de-no-resolver-el-problema)
5. [Beneficiarios directos](#5-beneficiarios-directos)
6. [Funcionalidades esenciales derivadas de la necesidad](#6-funcionalidades-esenciales-derivadas-de-la-necesidad)
7. [Referencias](#referencias)

---

## 1. Problemática

### 1.1 Un problema creciente en Quito

La presencia de perros en situación de calle en Quito ha aumentado de forma sostenida. Un estudio de la Universidad San Francisco de Quito (2018) estimó un perro por cada 22 habitantes y cerca de 600 000 animales de compañía viviendo en las calles (La Hora, 2021). En 2023, la Unidad de Bienestar Animal (UBA) midió en 53 parroquias un perro callejero por cada 19 habitantes, con un crecimiento cercano al 500 % respecto a mediciones previas; alrededor de dos tercios de esos perros se desplazan en jauría (Gavilanes, 2024). Ese mismo año, el Municipio estimó cerca de 1,3 millones de animales callejeros, entre perros y gatos, en sus 65 parroquias, y los barrios señalaron a los animales en la calle como su principal problema ambiental (Machado, 2024).

El abandono se agravó con la pandemia: los casos registrados por el Municipio pasaron de 1 a 10 diarios en abril de 2020, y el abandono creció alrededor de un 90 % (La Hora, 2021). Para 2021 se reportaban unos 100 000 perros y gatos desamparados (Rubio, 2022).

> **Sobre las cifras:** provienen de estudios con metodologías distintas (solo perros o perros y gatos; zona urbana o todo el distrito), por lo que no son estrictamente comparables. Lo que todas las fuentes confirman es la **tendencia creciente**.

### 1.2 Un problema de personas, no solo de animales

El abandono no se explica por una sola causa. Las fuentes señalan varios factores que se refuerzan entre sí:

- **Económicos:** la pérdida de empleo durante la pandemia fue un motivo frecuente de abandono (La Hora, 2021), y la crisis posterior agravó la situación (Rubio, 2022).
- **Tenencia irresponsable:** la UBA estima que cerca del 70 % de los perros que deambulan por la calle sí tiene dueño; son dejados fuera durante el día o forman parte de la comunidad (Herrera, 2024).
- **Falta de conciencia ciudadana**, junto con refugios hacinados y sin apoyo económico (Herrera, 2024).
- **Regulación reciente:** la Ordenanza 019 de Bienestar Animal se promulgó en enero de 2021, frente a un problema de larga data (La Hora, 2021).

Resolver el abandono de raíz depende de la esterilización, la regulación y la educación. Eso excede lo que puede hacer una iniciativa de adopción. Sin embargo, hay una parte del problema que sí puede abordarse: **cuando un perro ya está en manos de alguien que quiere darlo en adopción, a esa persona le resulta muy difícil encontrarle un hogar.**

### 1.3 El problema central: la dificultad para concretar una adopción

**Problema central:** _dificultad en el proceso de adopción de los perros en Quito._

Quienes tienen perros a su cargo (rescatistas independientes, familias con camadas inesperadas, refugios) y quienes desean adoptar no cuentan hoy con un punto de encuentro eficaz. Los medios disponibles tienen limitaciones claras:

| Medio actual | Limitación |
|---|---|
| **Ferias de adopción presenciales** | Son difíciles de organizar: algunas organizaciones logran solo unas pocas al año o un máximo de tres por semestre, y su alcance depende del sector donde se realicen (La Hora, 2018). |
| **Redes sociales** | Es un medio abierto y sin filtros, donde es difícil destacar entre el volumen de publicaciones y encontrar personas realmente comprometidas (La Hora, 2021; entrevista a Almeida). |
| **Gestión propia de cada organización** | Cada una maneja sus requisitos, procesos y eventos. El trámite dura entre una y dos semanas y suele pedir copia de cédula, planilla de servicios básicos, solicitud y, según el caso, contrato (La Hora, 2018). |

A esto se suma que **la información está dispersa**. Una persona interesada no tiene un lugar donde ver perros de distintos rescatistas y refugios, compararlos por las características que más pesan en su decisión (edad, salud, esterilización) y contactar directamente a quien los tiene a cargo. Tampoco tiene forma sencilla de saber si quien publica es una fuente confiable.

Las características del propio animal agravan el panorama. La edad y el estado de salud influyen directamente en la posibilidad de adopción: los perros adultos o con enfermedades crónicas pueden pasar años sin hogar (La Hora, 2018). La demanda también varía según la temporada (La Hora, 2018).

### 1.4 Causas identificadas

| Causa | Respaldo |
|---|---|
| **Falta de visibilidad** de los perros que están en adopción | La difusión se limita a ferias ocasionales y a redes sociales saturadas (La Hora, 2018; 2021). |
| **Ausencia de una red de apoyo común** entre quienes ponen en adopción y quienes quieren adoptar | No existe un espacio que reúna a rescatistas, refugios y adoptantes (entrevista a Almeida). |
| **Información descentralizada** sobre perros, requisitos y eventos | Cada organización gestiona lo suyo por separado (La Hora, 2018). |
| **Falta de apoyo para sostener a los perros mientras esperan hogar** | Los refugios carecen de apoyo económico (Herrera, 2024) y dependen de la autogestión (Rubio, 2022). |

### 1.5 Situación de quienes tienen perros a su cargo

- **Refugios:** están saturados y dependen de rifas, ventas, peluquería canina y donaciones. Con la pandemia el apoyo disminuyó mientras subían los gastos de alimentación, veterinaria y personal; algunos centros tuvieron que restringir el ingreso de nuevos animales (Rubio, 2022; Herrera, 2024).
- **Rescatistas independientes y familias:** mantienen varios perros con recursos propios mientras logran darlos en adopción. La rescatista entrevistada, con 18 años de experiencia, cuidaba 8 perros y 13 gatos con sus propios medios. Señala que lo más difícil es la carga económica y conocer al adoptante y el espacio donde vivirá el perro.
- **El refugio es un lugar de paso, no un destino.** Académicos del proyecto Poliperros (EPN) advierten que la ciudadanía tiende a verlo como un lugar permanente, cuando debería ser una estación temporal hacia un hogar definitivo (Herrera, 2024).

---

## 2. Necesidad

La necesidad que se identifica es **facilitar que los perros en adopción de Quito encuentren un hogar definitivo, reduciendo el tiempo y el esfuerzo que hoy exige conectar a quien da con quien adopta.**

Esa necesidad se expresa de forma distinta en cada grupo afectado:

| Grupo | Necesidad concreta | Respaldo |
|---|---|---|
| **Rescatistas independientes y familias** | Un medio formal y dirigido para dar perros en adopción, en lugar de depender de publicaciones sueltas en redes sociales. | Entrevista a Almeida; La Hora (2021) |
| **Refugios y perreras** | Visibilidad permanente, que no dependa de las pocas ferias de adopción que logran organizar. | Herrera (2024); Rubio (2022); La Hora (2018) |
| **Personas interesadas en adoptar** | Información reunida en un solo lugar, comparable según edad, salud y esterilización, y proveniente de fuentes en las que se pueda confiar. | La Hora (2018); entrevista a Almeida |

La entrevista aporta además criterios de confianza que quienes dan en adopción consideran esenciales: la esterilización es el dato más determinante para decidir una adopción (después vienen el comportamiento y la apariencia); conviene conocer al adoptante y su entorno; y la comunicación entre las partes debe ser directa.

---

## 3. Oportunidad

La situación descrita abre una oportunidad concreta:

1. **Existe una brecha clara.** Ni las ferias ni las redes sociales ofrecen un punto único donde se puedan ver, comparar y contactar perros de rescatistas y refugios de distintas procedencias. Quien cubra esa brecha atiende un vacío que las fuentes confirman.
2. **Hay oferta y demanda que hoy no se encuentran.** Rescatistas, familias y refugios buscan dar perros en adopción; hay personas dispuestas a adoptar, y la adopción es la alternativa que las propias organizaciones promueven frente al abandono (Gavilanes, 2024; Herrera, 2024). La dificultad está en la conexión, no en la falta de interés.
3. **La confianza puede construirse.** Hoy es difícil distinguir a quien publica con compromiso real de una publicación anónima. Hacer visible quién es quién, y mostrar la información que más pesa en la decisión, reduce la incertidumbre de ambas partes.
4. **El impacto es acotado y alcanzable.** No se pretende resolver el problema estructural del abandono, que depende de esterilización, regulación y educación. Se busca **reducir la fricción del proceso de adopción**: dar visibilidad permanente a los perros, centralizar la información y generar confianza entre quienes dan y quienes adoptan.

**Objetivos que se desprenden de la necesidad y la oportunidad:**

1. Establecer un canal de comunicación común entre quienes ponen en adopción y quienes desean adoptar un perro en Quito.
2. Mejorar la visibilidad de los perros en proceso de adopción.

**Fuera del alcance de este planteamiento:**
- El sostenimiento económico de los perros mientras esperan hogar (causa 4 de la tabla 1.4). Es un problema real, pero queda para una etapa posterior.
- El seguimiento posterior a la adopción.
- Cualquier transacción económica.

---

## 4. Consecuencias de no resolver el problema

| Consecuencia | Evidencia |
|---|---|
| **Permanencia prolongada de perros** en refugios y hogares temporales; los adultos o enfermos pueden esperar años. | La Hora (2018) |
| **Saturación de refugios** y restricción para recibir nuevos animales, lo que deja más perros en la calle. | Herrera (2024); Rubio (2022) |
| **Desgaste económico y personal de rescatistas independientes**, que sostienen a los perros con recursos propios hasta que alguien los adopte. | Rubio (2022); entrevista a Almeida |
| **Procesos de adopción lentos y poco confiables**, que desalientan a quien quiere adoptar y favorecen que adopte quien no está comprometido. | La Hora (2018); entrevista a Almeida |
| **Menor motivación para adoptar**, al no entenderse el sentido de la adopción y percibirse el refugio como destino final. | Herrera (2024) |
| **Deterioro de la convivencia y del entorno barrial**, con animales en la calle como principal problema ambiental percibido. | Machado (2024) |

La tendencia de crecimiento documentada desde 2018 indica que, sin cambios, la presión sobre refugios y rescatistas seguirá aumentando.

---

## 5. Beneficiarios directos

| Beneficiario | Cómo se beneficia |
|---|---|
| **Rescatistas independientes y familias que acogen perros** | Dan a sus perros una vía formal y visible hacia un hogar definitivo. Reducen el tiempo y el costo de mantenerlos y el desgaste de buscar adoptantes sin filtro. |
| **Refugios y perreras** | Ganan visibilidad permanente sin depender de ferias ocasionales. Al colocar más perros liberan espacio y recursos para acoger a otros. |
| **Personas interesadas en adoptar** | Encuentran en un solo lugar información comparable y de fuentes confiables, y pueden contactar directamente a quien tiene al perro. Deciden con mejor información. |
| **Los perros en adopción** | Reducen su tiempo de espera y su permanencia en refugios o en la calle. Esto aplica en especial a los de mayor edad o con necesidades especiales, que son los de menor probabilidad de adopción. |
| **La ciudadanía de Quito** | Una mayor adopción contribuye a disminuir la presión de los animales sin hogar sobre los barrios, aunque no sustituye las medidas estructurales (esterilización, regulación, educación). |

---

## 6. Funcionalidades esenciales derivadas de la necesidad

Los requisitos y artefactos remiten a `Adogtame_Analisis_Final.md` (FR = requisito funcional, CR = requisito del cliente, CU = caso de uso).

| Necesidad detectada | Requisito derivado | Artefacto que lo representa |
|---|---|---|
| Rescatistas, familias y refugios necesitan un medio formal y dirigido para dar perros en adopción, en lugar de publicaciones sueltas en redes sociales. | **Publicar un perro en adopción:** quien tiene un perro a cargo registra sus datos (nombre, fotos, raza, edad, tamaño, ubicación, vacunas, esterilización, necesidades especiales) y puede editarlo o retirarlo cuando ya no esté disponible. (FR-05, FR-06, FR-12; CR-01) | CU-07 «Poner perro en adopción», CU-08 «Editar información de perro», CU-14 «Eliminar perro»; entidad de datos **Perros**. |
| Las personas interesadas necesitan información reunida en un solo lugar, comparable según edad, salud y esterilización, y los perros necesitan visibilidad permanente que no dependa de las ferias. | **Explorar, buscar y filtrar perros:** cualquier visitante ve los perros disponibles, los filtra por sus características y consulta el perfil completo de cada uno, con los datos médicos destacados. (FR-09, FR-10, FR-13; CR-01) | CU-03 «Ver perros en adopción», CU-04 «Buscar y filtrar perros», CU-05 «Ver información de perro». |
| Es difícil saber si quien publica es una fuente confiable y distinguir a quien tiene compromiso real con el bienestar animal de una publicación anónima. | **Verificar a rescatistas y refugios:** quienes quieran ser reconocidos como rescatista o refugio solicitan una revisión; un administrador la aprueba, la rechaza o la revoca, y las cuentas verificadas se distinguen en su perfil y en un directorio público. (FR-02, FR-03, FR-11, FR-14; CR-02) | CU-25 «Solicitar verificación como rescatista», CU-15 «Revisar solicitudes de verificación», CU-16 «Resolver solicitud de verificación», CU-27 «Ver perfiles públicos y directorios»; actor **Administrador**. |
| Quien da en adopción necesita conocer al adoptante y comunicarse directamente con él; hoy el trámite es lento y disperso entre organizaciones. | **Solicitar y gestionar adopciones con contacto directo:** el interesado solicita adoptar un perro disponible; el publicador recibe aviso, acepta o rechaza, y ambos pueden contactarse mediante datos de contacto visibles para usuarios registrados. (FR-07, FR-11, FR-04) | CU-09 «Solicitar adopción», CU-10 «Confirmar adopción», CU-11 «Recibir notificación de solicitud», CU-13 «Revisar notificaciones». |

**Por qué estas cuatro son esenciales:** sin la primera no hay perros que mostrar; sin la segunda nadie los encuentra; sin la tercera no hay confianza entre las partes; y sin la cuarta no se concreta la adopción. Juntas cubren el objetivo de un canal común y el de mayor visibilidad (sección 3).

---

## Referencias

1. Gavilanes, P. (2024, 15 de abril). *Quito tiene un perro callejero por cada 19 habitantes*. El Comercio. <https://www.elcomercio.com/tendencias/perros-callejeros-quito-esterilizacion-adopcion.html>
2. La Hora. (2018, 11 de abril). *Adopción de mascotas en Quito depende de tres factores*. <https://www.lahora.com.ec/archivo/Adopcion-de-mascotas-en-Quito-depende-de-tres-factores-20180411-0180.html>
3. La Hora. (2021, 2 de mayo). *El abandono de perros en Quito se incrementó en 90%*. <https://www.lahora.com.ec/archivo/El-abandono-de-perros-en-Quito-se-incremento-en-90-20210502-0009.html>
4. Rubio, E. (2022, 6 de noviembre). *El abandono de mascotas agudiza la crisis de los refugios en Quito*. Primicias. <https://www.primicias.ec/noticias/sociedad/abandono-animales-agudiza-crisis-refugios-quito/>
5. Herrera, C. (2024, 7 de agosto). *Refugios de animales en Quito no solventan situación de los perros callejeros*. El Telégrafo. <https://www.eltelegrafo.com.ec/noticias/quito/220/refugios-de-animales-en-quito-no-solventan-situacion-de-los-perros-callejeros>
6. Machado, J. (2024, 25 de enero). *Animales callejeros, el principal problema ambiental con el que lidian los barrios*. Primicias. <https://www.primicias.ec/noticias/quito/animales-perros-gatos-hogares-inec/>
7. Entrevista a Dayra Almeida, rescatista independiente (Quito). <https://www.youtube.com/watch?v=yuA9b32NnHM>
