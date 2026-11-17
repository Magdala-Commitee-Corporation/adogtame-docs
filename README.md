# Adogtame V2 - Repositorio de documentación

Este repositorio es la **fuente de verdad de la especificación** de Adogtame V2, un sistema web de adopción de perros. Contiene el ERS, los documentos de análisis y diseño (`adog_*.md`) y los diagramas. El código vive en otros repositorios; este solo cambia por PR.

## Documentos y precedencia

| Documento | Contenido |
|---|---|
| ERS (`Adog_ERS.docx`) | Requisitos de software. **Prevalece sobre todo lo demás.** |
| `adog_RF.md`, `adog_RNF.md`, `adog_RN.md`, `adog_CU.md` | Requisitos funcionales, no funcionales, reglas de negocio y casos de uso. |
| `adog_ARCH.md` | Arquitectura. Prevalece sobre `adog_MS.md` en decisiones de arquitectura. |
| `adog_MER.md` | Modelo de datos: cada microservicio posee su propia base de datos (Dogs en PostgreSQL/Aurora Serverless; Auth, Requests y Recommendation en DynamoDB), sin claves foráneas ni JOIN entre servicios. |
| `adog_MS.md` | Detalle de los 4 microservicios (`Auth`, `Dogs`, `Requests`, `Recommendation`), llamadas internas y flujos F1 a F8. |
| `adog_CLS.md` | Modelo conceptual de funciones por paquete. |
| `adog_REC.md` | Módulo de recomendación (fórmula, IA, plantillas). |

Regla general: si un documento derivado difiere del ERS, prevalece el ERS y el derivado debe actualizarse. Un cambio en un documento casi siempre obliga a revisar los que dependen de él (ver la cabecera «Derivado de» de cada archivo).

## Convenciones

- Documentación en español. Microservicios y rutas con nombre en inglés.
- Identificadores estables: RF-nn, RNF-nn, RN-nn, HU-nn (= RF-nn), HT-nn, F1 a F8.
- Diagramas en Mermaid dentro del Markdown, o `.drawio.svg` para que GitHub los muestre y sigan siendo editables.

## Flujo de un cambio de especificación

1. Debe existir una incidencia en Linear (equipo **ADO**, plantilla «Cambio de Especificación»).
2. Rama con el nombre que propone Linear: `ado-<n>-<titulo>`.
3. PR con `Fixes ADO-<n>` en la descripción. Lo revisa el otro integrante.
4. Al fusionar, se avisa con un comentario en las tareas de Linear afectadas. El historial de Git y la incidencia son el registro del cambio; no se mantiene un CHANGELOG aparte.