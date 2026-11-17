# ADOGTAME — Documento maestro de la especificación v2

> **Punto de entrada único** del análisis de Adogtame. Resume el sistema y **delega** cada especificación en un documento concreto. Ninguna especificación se repite aquí: si este documento y un documento delegado difieren, prevalece el delegado y este debe actualizarse.

---

## 1. Contexto del proyecto

- **Origen:** proyecto de 4.º semestre de Ingeniería de Software (Grupo 1, ESPE). El proyecto original constaba con su propio análisis y diseño de un sistema construido con React/Next.js, Express y MongoDB.
- **Objetivo actual:** retomar el sistema en la materia de **Construcción y Evolución de Software**, aplicando ingeniería inversa sobre lo que ya estaba construido, especificando con más rigor todos los artefactos del SDLC (requisitos, casos de uso, diseño, arquitectura, pruebas y despliegue) en una **versión 2 (v2)**.
- **Qué es Adogtame:** aplicación web responsive desplegada en AWS serverless que actúa como canal central para la adopción de perros en el Distrito Metropolitano de Quito. Reúne a quienes dan en adopción (personas, rescatistas verificados, refugios) con quienes quieren adoptar, con visibilidad permanente, información filtrable, verificación de publicadores y facilidad para recomendar perros en adopción que se encuentren más alineados al estilo de vida de adoptantes interesados mediante el apoyo de la IA. No maneja pagos, ni seguimiento directo durante la adopción.

---

Los instrumentos se refieren a documentación escrita en texto natural, para que sea comprendida por agentes. Cada instrumento puede contar con material de apoyo, en cuyo caso los referenciará de manera correspondiente.

## 2. Instrumentos de Análisis

### 2.1. Casos de Uso

**Archivo:** `adog_CU.md`

Define los 9 actores del sistema (con su tipo, herencia y descripción), una representación en texto del diagrama de casos de uso y una ficha por cada uno de los 24 casos de uso, con su actor, una descripción y los RF y RNF relacionados.

### 2.2. Reglas de Negocio

**Archivo:** `adog_RN.md`

Reúne las 55 reglas de negocio (RN-01 a RN-55) agrupadas en seis módulos (cuentas y acceso, perfiles y favoritos, perros y catálogos, adopción, verificación y recomendación), cada una con su regla y la excepción o mensaje asociado. La especificación detallada está en el ERS §5. Adicionalmente se agregan 2 criterios, cuya explicación está en la sección 7 del documento.

### 2.3. Requisitos Funcionales

**Archivo:** `adog_RF.md`

Resume los 34 requisitos funcionales (RF-01 a RF-34) en ocho grupos, con su actor, prioridad MoSCoW, una descripción general y las RN y RNF que cita cada ficha. La especificación detallada (entradas, flujos y postcondiciones) está en el ERS §6.

### 2.4. Módulo de Recomendación

**Archivo:** `adog_REC.md`

Especifica de forma autocontenida cómo se implementa el módulo de recomendación: el test de 7 preguntas, los atributos del perro, la preselección de 20 candidatos por fórmula de afinidad, el Servicio de IA (petición, prompt, respuesta y validación), el respaldo con plantillas, la interfaz y los mensajes, los datos almacenados y las reglas de negocio. Incluye además el fundamento y los límites de la evidencia que lo respalda, sus limitaciones y las decisiones abiertas.

### 2.5. Requisitos No Funcionales

**Archivo:** `adog_RNF.md`

Resume los 25 requisitos no funcionales (RNF-01 a RNF-25) agrupados por tipo (rendimiento, seguridad, privacidad, fiabilidad, disponibilidad, mantenibilidad, usabilidad, portabilidad y costo), con su prioridad MoSCoW, su descripción y los RF que los citan. La especificación detallada está en el ERS §7.


## 3. Instrumentos de Diseño

### 3.1. Modelo Entidad - Relación

**Archivo:** `adog_MER.md`

Describe los datos que conserva el sistema: el diagrama de las 15 entidades con sus atributos y relaciones, la trazabilidad de cada entidad con los requisitos y reglas, las convenciones del modelo y un diccionario de datos con el tipo, la restricción y la descripción de cada campo.

### 3.2. Arquitectura

**Archivo:** `adog_ARCH.md`

Describe cómo se organiza el sistema: el estilo de tres capas desplegado en AWS serverless con lógica funcional, el diagrama de componentes (presentación, entrega y perímetro, negocio por servicios en funciones Lambda, adaptadores, datos y servicios externos), la responsabilidad y los requisitos de cada componente, las decisiones de diseño trazadas con los RNF y los controles de seguridad por capa.

### 3.3. Clases

**Archivo:** `adog_CLS.md`

Presenta el modelo conceptual preliminar del servidor en tres diagramas de clases (cuentas y verificación; perros y adopción; recomendación e integración), con atributos, operaciones, herencias, interfaces de los adaptadores y su trazabilidad con las entidades del MER junto los requisitos. Como el sistema se implementa con programación funcional y no con clases reales, especifica además las funciones principales (entradas, salida y reglas), los módulos de apoyo que se definirán al construir y las observaciones para la construcción.

