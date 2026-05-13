# Roadmap — Ellie Care

Plan técnico transversal de la org. Items priorizados por **riesgo si no se hacen** + **valor cuando se hagan**.

**Última actualización:** 2026-05-12
**Status del Sprint AI-Ready:** ✅ Completo (22/22 repos productivos estandarizados)

---

## Resumen ejecutivo

| Item | Prioridad | Riesgo si no se hace | Esfuerzo |
|---|---|---|---|
| 1. typeorm-library → SemVer | 🔴 CRÍTICA | Alto — un push rompe 4 servicios | Medio |
| 2. Jenkins → GitHub Actions | 🟡 ALTA | Medio — Jenkins funcional pero opaco | Alto |
| 3. PostgreSQL 12 → 16 + pgvector | 🔴 CRÍTICA | Alto — PG12 EOL desde nov 2024 | Medio |
| 4. RDS Single-AZ → Multi-AZ | 🟡 ALTA | Medio — falla de zona = sistema caído | Bajo |
| 5. HITL queue en ec-bo | 🟢 MEDIA | Bajo — sin agentes clínicos productivos | Medio |
| 6. MCP Server expansion | 🟢 MEDIA | Bajo — capacidad incremental | Medio |
| 7. Archivar repos legacy | 🟢 BAJA | Bajo — ruido visual nomás | Bajo |

---

## 1. typeorm-library → SemVer

**Status:** 🔴 Abierto · **Prioridad:** CRÍTICA · **Owner:** Backend team

### Contexto

`typeorm-library` define las entidades TypeORM compartidas del stack AWS. Hoy los 4 consumers la consumen así:

```json
"typeorm-library": "github:elliecare/typeorm-library#develop"
```

Esto significa que cada push a `develop` afecta inmediatamente a:
- `ec-be`
- `lambdas-sensors`
- `aws-lambdas`
- `lambdas-ec-reporting`

No hay pin de versión, no hay rollback por versión, no hay changelog formal.

### Plan

1. Tagear el estado actual de `develop` como `v1.0.0`
2. Publicar a un npm registry privado (GitHub Packages o Verdaccio)
3. Migrar consumers uno por uno, de menor a mayor riesgo:
   - `lambdas-ec-reporting` (menor riesgo)
   - `lambdas-sensors`
   - `aws-lambdas`
   - `ec-be` (mayor riesgo)
4. Después de migración, `develop` deja de ser load-bearing
5. Cambios futuros siguen SemVer estándar (major = breaking, minor = feature, patch = fix)

### Riesgo

- Durante la migración, distintos consumers pueden tener distintas versiones → coordinar deploys
- Schema drift entre versiones → cada bump requiere checklist

### Dependencias

Ninguna externa. Es trabajo interno del backend team.

---

## 2. Jenkins → GitHub Actions

**Status:** 🟡 Abierto · **Prioridad:** ALTA · **Owner:** Platform team

### Contexto

Hoy Jenkins gestiona todos los deploys productivos (`deploys-jenkins`). Problemas:
- Knowledge concentrado en pocos people
- Difícil reproducir un deploy localmente
- Mezcla de tools (4 sistemas de CI/CD: Jenkins + GHA + Serverless + Firebase CLI)
- Server de Jenkins requiere mantenimiento

### Plan

1. Crear reusable workflows en `elliecare/.github/.github/workflows/`:
   - `deploy-ecs.yml` (para ec-be)
   - `deploy-lambda.yml` (para lambdas-*)
   - `deploy-firebase.yml` (para ec-firebase-functions + device-manager)
   - `deploy-static.yml` (para frontends)
2. Migrar el primer repo de bajo riesgo: `lambdas-ec-reporting`
3. Correr Jenkins + GHA en paralelo 2 semanas (GHA solo valida, no despliega)
4. Validar consistencia → cortar Jenkins para ese repo
5. Repetir por orden de riesgo: lambdas → frontends → `ec-be` (último)
6. Archivar `deploys-jenkins` cuando todo esté migrado

### Riesgo

- Cortar Jenkins prematuramente sin GHA testeada = no se puede deployar
- Credenciales/secrets del Jenkins server hay que migrar a GitHub Secrets
- Comportamientos implícitos del Jenkins server (env vars, ordering) no documentados

### Dependencias

- Acceso a configurar GitHub Actions a nivel org
- Coordinación con el equipo de ops para credenciales

---

## 3. PostgreSQL 12 → 16 + pgvector

**Status:** 🔴 Abierto · **Prioridad:** CRÍTICA · **Owner:** Platform team + DBA

### Contexto

PostgreSQL 12 está **EOL desde noviembre 2024**. Sin patches de seguridad. AWS RDS irá deprecando soporte gradualmente.

`pgvector` es una extensión de PG que habilita búsqueda semántica (vector similarity). Útil para:
- Búsqueda inteligente de pacientes
- Análisis de patrones clínicos
- Habilitar features de IA sobre la DB existente

### Plan

1. Crear un Blue/Green deployment de RDS:
   - "Green" = nueva instancia PG16 con la data replicada en tiempo real
   - "Blue" = la instancia PG12 actual
2. Habilitar pgvector en la Green
3. Tests exhaustivos contra la Green (subset de tráfico read-only)
4. Switchover: AWS swap automático Blue ↔ Green (downtime ~30s)
5. Verificar todos los servicios contra la nueva primary
6. Eliminar la Blue después de 1 semana de validación

### Riesgo

- Cambios de comportamiento entre PG12 y PG16 (deprecated functions, performance regressions)
- Extensiones no soportadas en PG16
- Connection strings: cambiar si la app las cachea
- Queries con sintaxis vieja pueden romper

### Dependencias

- Ventana de downtime de ~30s autorizada (medianoche AR)
- Tests de carga en uat antes de prod

---

## 4. RDS Single-AZ → Multi-AZ

**Status:** 🟡 Abierto · **Prioridad:** ALTA · **Owner:** Platform team

### Contexto

Hoy la RDS productiva corre en **Single-AZ**. Si AWS tiene un incidente en la AZ donde vive nuestra DB → sistema caído hasta que AWS recupere o nosotros hagamos restore manual.

Multi-AZ:
- Replica sincrónica en otra zona
- Failover automático en ~60-120s
- Costo: ~2x (la replica es real, no on-demand)

### Plan

Combinar con el upgrade a PG16:

1. En el Blue/Green deployment del item #3, activar `multi_az = true` desde el inicio
2. La Green ya nace Multi-AZ
3. Switchover → automáticamente la prod queda Multi-AZ

Si se hace separado del upgrade:

1. Modificar `aws_db_instance.multi_az = true` en Terraform
2. `terraform apply` → AWS hace la promoción in-place (downtime de ~60s)

### Riesgo

- Costo se duplica para RDS
- Failover automático puede causar 1-2 min de degradación
- Algunas queries muy escritura-intensivas pueden tener latencia un poco mayor

### Dependencias

- Aprobación del costo extra
- Idealmente sincronizado con #3 para evitar dos downtimes

---

## 5. HITL queue en ec-bo

**Status:** 🟢 Abierto · **Prioridad:** MEDIA · **Owner:** Product + Backend + Frontend

### Contexto

HITL = Human In The Loop. Para decisiones clínicas asistidas por IA (sugerencias del agente clínico, alertas detectadas por modelos), debe haber siempre un humano que apruebe antes de que llegue al paciente.

Hoy no existe esta cola → todavía no podemos lanzar agentes clínicos a producción.

### Plan

1. Backend (`ec-be`):
   - Nueva tabla `hitl_approvals` con estados: pending / approved / rejected / expired
   - Endpoints: POST (crear), PATCH (aprobar/rechazar), GET (queue)
   - Webhook al frontend cuando hay aprobaciones pendientes
2. Frontend (`ec-bo`):
   - Nueva sección "Aprobaciones pendientes"
   - UI para ver el contexto, payload, agente que originó la sugerencia
   - Botones Aprobar / Rechazar / Más contexto
3. `ellie-ai-platform`:
   - Cliente que encola en `hitl_approvals` antes de aplicar sugerencias clínicas
   - Espera el resultado antes de proceder
4. Audit log de cada aprobación (compliance)

### Riesgo

- Si la queue está saturada, los agentes se traban
- UX del operador debe ser muy clara — confundir aprobar/rechazar tiene impacto clínico
- Compliance: cada aprobación es un evento auditable, retención 7 años

### Dependencias

- Diseño UX/UI consensuado con el equipo clínico
- Coordinación con `ellie-ai-platform` para integración

---

## 6. MCP Server expansion

**Status:** 🟢 Abierto · **Prioridad:** MEDIA · **Owner:** AI Platform team

### Contexto

`ellie-ai-platform` ya tiene MCP Servers funcionando:
- `bigquery_server.py` — sensor analytics
- `firestore_server.py` — real-time state
- `healthcare_api_server.py` — FHIR R4

Para que más agentes (clínico, BI, soporte, wellness) puedan operar, hay que expandir las tools disponibles.

### Plan

1. Identificar las tools faltantes:
   - Buscar paciente por nombre / ID
   - Historial de alertas
   - Estado de dispositivo en tiempo real
   - Crear nota clínica (con HITL del item #5)
   - Recalcular thresholds (vía `automation-lambdas`)
   - Triggerar export (vía `db-lambdas`)
2. Implementarlas con PHI redaction obligatoria
3. Documentar el contrato JSON-RPC 2.0 de cada tool
4. Testear con golden datasets (eval pipeline)
5. Habilitar gradualmente por agente

### Riesgo

- PHI leak si la tool no redacta correctamente
- Costo de inferencia sube con más tools (más context window)
- Schema drift entre tools y consumers

### Dependencias

- HITL queue del item #5 para tools que escriben
- typeorm-library SemVer (item #1) si las tools usan TypeORM entities

---

## 7. Archivar repos legacy

**Status:** 🟢 Abierto · **Prioridad:** BAJA · **Owner:** Org admin

### Contexto

Hay 13 repos `status-legacy` en la org. Algunos no se tocan hace años, otros reciben hotfixes ocasionales.

Hoy solo están taggeados — están visibles en la lista de repos, generan ruido. Archivar los que confirmemos definitivamente discontinuados los pone read-only y los baja de la vista principal.

### Plan

1. Validar con cada owner / equipo qué repos:
   - **Pueden archivarse** (no se tocan, sin hotfixes esperados):
     - `ec-web`
     - `ec-portal-old`
     - `facewatch-digital`
     - `sw-fallcare`
     - `health-measurements`
     - `test-pipeline`
     - `test-webhook`
   - **Quedan en `status-legacy`** (posibles hotfixes):
     - `ec-fe-hospital` / `ec-be-hospital` (línea de hospital)
     - `ec-mobile` (mobile)
     - `ec-sw` / `sw-watchos` (Apple Watch)
2. Archivar los confirmados
3. Actualizar `services.yaml` para que refleje el cambio
4. Actualizar `profile/README.md`

### Riesgo

- Mínimo: si necesitás algo del repo archived, se puede desarchivar
- Cuidado de no archivar repos que todavía tienen dependencias activas

### Dependencias

- Aprobación de cada owner

---

## Calendario tentativo

```
2026 Q2 (mayo-junio):
  ✅ Sprint AI-Ready (completado)
  🔴 Item #1 — typeorm-library SemVer
  🟡 Item #4 — RDS Multi-AZ (preparación para #3)

2026 Q3 (julio-septiembre):
  🔴 Item #3 — PostgreSQL 12 → 16 + pgvector (combinado con #4)
  🟢 Item #7 — Archivar repos legacy

2026 Q4 (octubre-diciembre):
  🟡 Item #2 — Jenkins → GitHub Actions (gradual, repo por repo)
  🟢 Item #5 — HITL queue en ec-bo
  🟢 Item #6 — MCP Server expansion

2027 Q1+:
  🤖 Agentes clínicos en producción (depende de #5 + #6)
```

Las prioridades 🔴 son fixes urgentes de deuda técnica/seguridad. Las 🟡 son mejoras importantes de infraestructura. Las 🟢 son habilitadores de capacidades futuras.

---

## Cómo usar este roadmap

- **Nuevos PRs:** verificar si el cambio toca alguno de estos items — coordinar con el owner
- **Decisiones técnicas:** referenciar el item correspondiente en el ADR
- **Updates:** actualizar `Status` y `Last updated` cuando un item avanza
- **Discusión:** abrir issue en `elliecare/.github` taggeado `roadmap`

---

<sub>Este roadmap es un documento vivo. Cambios requieren PR + discusión.</sub>
