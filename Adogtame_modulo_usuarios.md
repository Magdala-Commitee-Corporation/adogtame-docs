# Revisión del módulo de roles: esquema por perfil (sin RBAC)

> Documento complementario a `Adogtame_modulo_usuarios.md` (v0.7) y `Adogtame_Analisis_Final.md`.
> Reemplaza **DEC-01** y modifica las partes del modelo de roles que dependían de las tablas RBAC. El catálogo de acciones y las decisiones DEC-02 a DEC-15 se mantienen.
> Sus cambios se integran en `Revision_Final_v2.md`.
>
> **Alcance de esta fase.** Este documento es de análisis: define **qué** reglas de autorización debe cumplir el sistema, no **cómo** se implementan. El código de Express y el SQL que aparecen aquí son **referencias ilustrativas** de la lógica; no fijan decisiones de tecnología, estructura de código ni librerías.

---

## 1. Motivo del cambio

Con las decisiones tomadas, el RBAC con catálogo cerrado guarda en tablas información que nunca cambia fuera de una migración:

- **Los roles casi no se combinan.** REFUGIO y ADMIN son exclusivos (DEC-03, DEC-08). La única combinación posible es PERSONA + RESCATISTA, que equivale a un perfil más una verificación.
- **No hay pantalla para editar permisos** (DEC-01). Las tablas `rol`, `permiso`, `rol_permiso` y `cuenta_rol` solo cambiarían por semilla.
- **El perfil y el rol base son redundantes.** Esto obligaba a mantener invariantes entre `tipo_perfil` y `cuenta_rol`, y a sincronizarlos en el registro, la conversión de refugio (DEC-11) y el retiro de administradores.
- **RESCATISTA no tiene permisos propios.** En la semilla no tenía filas en `rol_permiso`; su efecto era solo el distintivo y el directorio, que ya se derivan de `solicitud_verificacion`.

## 2. Nueva decisión

**DEC-01 (revisada):** Se elimina el RBAC en la base de datos. Los permisos efectivos de una cuenta se derivan de tres datos que ya existen:

1. su **perfil** (`cuenta.tipo_perfil`),
2. su **verificación vigente** (`solicitud_verificacion` aprobada), que determina si una persona es rescatista o si un refugio está verificado,
3. su **estado** (`cuenta.estado`).

La matriz de permisos por perfil (sección 5) es una **especificación**. Sea cual sea la implementación, debe cumplir dos reglas:

- **R1.** La autorización se resuelve en el servidor, por acción y, cuando aplica, por propiedad del recurso (NFR-09).
- **R2.** La matriz se define en **un solo lugar**. Las reglas de negocio preguntan si una cuenta puede hacer una acción, no qué perfil tiene.

### Comparación con la decisión anterior

| | RBAC con catálogo cerrado (anterior) | Perfil + matriz de permisos (nuevo) |
|---|---|---|
| Tablas de autorización | `rol`, `permiso`, `rol_permiso`, `cuenta_rol` | Ninguna |
| Dónde vive la matriz | Semilla de la base de datos | Un único punto de la lógica de la aplicación |
| Invariantes perfil–rol | Validación en aplicación o trigger | No existen |
| Rescatista | Rol asignado al aprobar y retirado al revocar | Persona con verificación de rescatista aprobada |
| Agregar una acción | Migración + asignación a roles | Actualizar la matriz |
| Agregar un rol nuevo (p. ej., MODERADOR) | Insertar rol y permisos | Requiere un perfil nuevo o reintroducir roles |

---

## 3. Roles

Los roles se mantienen como **concepto funcional** (actores, casos de uso, matriz), pero no se almacenan como roles.

| Rol | Cómo se representa | Cómo se obtiene | Cómo se pierde |
|---|---|---|---|
| **Visitante** | Petición sin sesión. | — | Al iniciar sesión. |
| **Persona** | `tipo_perfil = 'persona'` | Registro público (FR-01) o conversión de refugio (DEC-04, DEC-11). | Suspensión o desactivación (pierde el acceso, no el perfil). |
| **Rescatista** | Persona con una `solicitud_verificacion` de tipo `rescatista` en estado `aprobada`. | Un administrador aprueba su solicitud (FR-03). | Un administrador revoca la verificación (`revocada`). |
| **Refugio** | `tipo_perfil = 'refugio'` | Registro público (FR-02). | Rechazo o revocación de la verificación: se convierte en Persona (DEC-11). |
| **Administrador** | `tipo_perfil = 'administrador'` | Lo crea otro administrador o el despliegue inicial. Nunca por registro público. | Otro administrador desactiva su cuenta (CU-22). |

**Verificación.** Rescatista y refugio verificado se derivan de la misma fuente: una `solicitud_verificacion` aprobada. Así hay una sola fuente de verdad y se conserva el historial.

**Estado de cuenta.** Sin cambios: `activa`, `suspendida` o `desactivada`. Una cuenta no activa no inicia sesión, no tiene permisos y sus perros quedan ocultos (DEC-09).

---

## 4. Modelo de datos

| Cambio respecto al análisis final | Motivo |
|---|---|
| Se **eliminan** `rol`, `permiso`, `rol_permiso`, `cuenta_rol` y el índice `ix_cuenta_rol_rol`. | La matriz deja de vivir en la base de datos. |
| Se **elimina** la semilla de roles y permisos. | Se reemplaza por la matriz de la sección 5. |
| Se **elimina** la regla de invariantes perfil–rol. | Ya no hay redundancia que sincronizar. |
| Se **simplifica** el script del primer administrador. | Basta con insertar la cuenta con perfil `administrador`. |
| Se **mantienen** `cuenta.tipo_perfil`, la FK compuesta de los subtipos, `cuenta.estado` y `solicitud_verificacion` con sus índices. | Siguen siendo la base de la autorización. |
| No se agrega `persona.es_rescatista`. | Se deriva de la solicitud aprobada (índice `ux_sv_aprobada`). |

Referencia ilustrativa de cómo se consulta la condición de rescatista:

```sql
-- Referencia: una persona es rescatista si tiene una verificación de ese tipo aprobada
SELECT EXISTS (
    SELECT 1
    FROM solicitud_verificacion
    WHERE id_cuenta = :id_cuenta
      AND tipo   = 'rescatista'
      AND estado = 'aprobada'
) AS es_rescatista;
```

---

## 5. Matriz de permisos por perfil

Mismos códigos que la semilla del análisis final, con formato `recurso:acción[:alcance]`. Las acciones del visitante siguen siendo públicas y no tienen permiso. Rescatista hereda todo lo de Persona y, por ahora, no tiene permisos propios (DEC-05).

| Permiso | Persona | Rescatista | Refugio | Administrador |
|---|:-:|:-:|:-:|:-:|
| `cuenta:editar:propio` | ✅ | ✅ | ✅ | ✅ |
| `cuenta:desactivar:propio` | ✅ | ✅ | ✅ | ✅ |
| `verificacion:solicitar:propio` | ✅ ¹ | ❌ | ❌ | ❌ |
| `verificacion:ver:propio` | ✅ | ✅ | ✅ | ❌ |
| `verificacion:listar:cualquiera` | ❌ | ❌ | ❌ | ✅ |
| `verificacion:resolver:cualquiera` | ❌ | ❌ | ❌ | ✅ |
| `verificacion:revocar:cualquiera` | ❌ | ❌ | ❌ | ✅ |
| `perro:publicar` | ✅ | ✅ | ✅ | ❌ |
| `perro:editar:propio` | ✅ | ✅ | ✅ | ❌ |
| `perro:eliminar:propio` | ✅ | ✅ | ✅ | ❌ |
| `perro:eliminar:cualquiera` | ❌ | ❌ | ❌ | ✅ |
| `perro:ver_eliminados` | ❌ | ❌ | ❌ | ✅ |
| `favorito:gestionar:propio` | ✅ | ✅ | ❌ | ❌ |
| `adopcion:solicitar` | ✅ | ✅ | ❌ | ❌ |
| `adopcion:cancelar:propio` | ✅ | ✅ | ❌ | ❌ |
| `adopcion:ver:propio` | ✅ | ✅ | ❌ | ❌ |
| `adopcion:ver_recibidas:propio` | ✅ | ✅ | ✅ | ❌ |
| `adopcion:resolver:propio` | ✅ | ✅ | ✅ | ❌ |
| `notificacion:gestionar:propio` | ✅ | ✅ | ✅ | ❌ |
| `usuario:listar` | ❌ | ❌ | ❌ | ✅ |
| `usuario:ver_detalle` | ❌ | ❌ | ❌ | ✅ |
| `usuario:suspender` | ❌ | ❌ | ❌ | ✅ |
| `admin:crear` | ❌ | ❌ | ❌ | ✅ |
| `admin:revocar` | ❌ | ❌ | ❌ | ✅ |
| `catalogo:gestionar` | ❌ | ❌ | ❌ | ✅ |

¹ Solo sin solicitud pendiente ni verificación vigente (CU-25). Al aprobarse, la persona pasa a la columna Rescatista y deja de tenerlo.

**Reglas de resolución:**

- Una cuenta no activa no tiene ningún permiso.
- Un permiso con alcance `propio` exige además que la cuenta sea dueña del recurso (publicador del perro, solicitante, destinatario de la notificación, etc.).
- Un permiso con alcance `cualquiera` no exige propiedad.

---

## 6. Referencia de lógica en Express (ilustrativa)

> **No es una decisión técnica.** El fragmento muestra, con la tecnología de la implementación original (TR-02), cómo se aplicarían las reglas R1 y R2 y las de resolución de la sección 5. La estructura de archivos, los nombres, el manejo de sesión y la forma de cargar los datos de la cuenta se definirán en la fase de diseño.

```js
// Referencia: matriz de la sección 5 definida en un solo lugar (R2)
const PERMISOS = {
  persona: ['cuenta:editar:propio', 'perro:publicar', 'perro:editar:propio',
            'adopcion:solicitar', 'favorito:gestionar:propio', /* ... */],
  refugio: ['cuenta:editar:propio', 'perro:publicar', 'perro:editar:propio',
            'adopcion:resolver:propio', /* ... */],
  administrador: ['perro:eliminar:cualquiera', 'verificacion:resolver:cualquiera',
                  'usuario:suspender', 'admin:crear', /* ... */],
};
const SOLO_RESCATISTA = [];        // DEC-05: preparado para módulos futuros
const SOLO_PERSONA_NO_RESCATISTA = ['verificacion:solicitar:propio'];

function permisosDe(cuenta) {
  if (!cuenta || cuenta.estado !== 'activa') return new Set();       // cuenta no activa
  const permisos = new Set(PERMISOS[cuenta.tipo_perfil] ?? []);
  if (cuenta.tipo_perfil === 'persona') {
    if (cuenta.esRescatista) SOLO_RESCATISTA.forEach(p => permisos.add(p));
    else SOLO_PERSONA_NO_RESCATISTA.forEach(p => permisos.add(p));
  }
  return permisos;
}

// Referencia: ¿puede esta cuenta hacer esta acción sobre este recurso?
function puede(cuenta, recursoAccion, recurso) {
  const permisos = permisosDe(cuenta);
  if (permisos.has(`${recursoAccion}:cualquiera`)) return true;
  if (permisos.has(`${recursoAccion}:propio`)) return recurso?.idDueno === cuenta.id_cuenta;
  return permisos.has(recursoAccion);
}

// Referencia: la ruta pregunta por la acción, nunca por el perfil (R1, R2)
router.delete('/perros/:id', autenticar, async (req, res) => {
  const perro = await buscarPerro(req.params.id);
  if (!perro) return res.status(404).end();                          // NFR-07
  if (!puede(req.cuenta, 'perro:eliminar', { idDueno: perro.id_publicador })) {
    return res.status(403).end();
  }
  // ... borrado lógico (FR-12)
});
```

Lo que la referencia busca mostrar, independientemente de cómo se implemente:

- El perfil solo se lee dentro de la función que resuelve los permisos.
- El estado de la cuenta se comprueba antes que cualquier permiso.
- La propiedad del recurso se valida en el servidor, no en la interfaz.
- Un recurso inexistente o no visible responde igual (404) antes de evaluar permisos.

---

## 7. Flujos afectados

| Flujo | Antes (RBAC) | Ahora |
|---|---|---|
| Registro (CU-01) | Crear cuenta, subtipo y asignar rol base. | Crear cuenta y subtipo. |
| Aprobar rescatista (CU-16) | Aprobar solicitud y asignar RESCATISTA. | Aprobar la solicitud. |
| Revocar rescatista (CU-17) | Revocar solicitud y retirar RESCATISTA. | Revocar la solicitud. |
| Conversión de refugio (DEC-11) | Borrar `refugio`, cambiar perfil, insertar `persona`, reemplazar rol. | Borrar `refugio`, cambiar perfil, insertar `persona`. |
| Crear administrador (CU-21) | Crear cuenta y asignar ADMIN en una transacción. | Crear la cuenta con perfil `administrador`. |
| Retirar administrador (CU-22) | Retirar ADMIN y desactivar la cuenta. | Desactivar la cuenta. |
| Último administrador activo | Contar cuentas activas con rol ADMIN. | Contar cuentas activas con perfil `administrador`. |

> _[Observación]_ **Reactivar a un exadministrador.** Antes, retirar el rol dejaba una cuenta desactivada y sin rol, y reactivarla no le devolvía privilegios. Ahora, como el perfil no cambia, **reactivar (CU-20) una cuenta de administrador le devuelve el acceso administrativo**. **Decisión:** es el comportamiento esperado; la reactivación mantiene el perfil de administrador y no muestra advertencia alguna. Queda registrado en `Revision_Final_v2.md` (Obs. 20).

---

## 8. Riesgos y evolución futura

| Escenario | Impacto |
|---|---|
| Dar permisos exclusivos al rescatista (DEC-05) | Bajo: agregar acciones a su columna de la matriz. |
| Agregar una acción nueva | Bajo: agregar una fila a la matriz. |
| Dividir al administrador (p. ej., MODERADOR) | Medio: nuevo perfil o reintroducir roles. Acotado si la autorización ya está centralizada y basada en acciones (R2). |
| Necesitar editar permisos desde una pantalla | Alto: implica volver a RBAC en la base de datos. No previsto. |
