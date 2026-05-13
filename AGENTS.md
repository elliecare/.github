# AGENTS.md — Ellie Care Organization

Esta es la guía universal para cualquier agente de IA (Claude Code, Cursor, Copilot, Aider, etc.) que trabaje en repos de `elliecare`.

## Cómo navegar la org

1. **Empezá por [`services.yaml`](services.yaml)** — index machine-readable de los 22 servicios productivos
2. **Cada repo productivo tiene su propio `AGENTS.md`** con contexto específico
3. **Cada repo productivo tiene `docs/_for_agents/`** con 4 archivos: `repo-map.md`, `glossary.md`, `safe-zones.md`, `common-tasks.md`

## Convenciones del ecosistema

### Clouds y responsabilidades

- **GCP**: AI, telemetría, FHIR clínico, agentes (Gemini 2.5 + ADK + MCP)
- **AWS**: Backend transaccional, contact center, lambdas de integración
- **Firebase (sub-GCP)**: Auth (Firebase Auth), orquestación de dispositivos (Functions)

### Naming

| Prefijo | Significado |
|---|---|
| `ai-` / `ellie-ai-` | AI Platform (GCP) |
| `ec-` | **Ellie Care** — productos, apps, portales (ej: `ec-be` = Ellie Care Back-End) |
| `lambdas-` | AWS Lambda services (Serverless Framework) |
| `sw-` | Smartwatch native apps |
| `automation-` / `db-` | Ops y automation |

### Repos legacy

Los repos legacy aparecen marcados con topic `status-legacy` y listados en [`services.yaml`](services.yaml) bajo la clave `legacy:`. **No modificar repos legacy sin confirmación humana** — no son código activo.

## Reglas universales para agentes

1. **PHI (datos de pacientes) NUNCA en LLMs externos sin redaction**
   - El layer de PHI redaction vive en `ellie-ai-platform/agents/shared/phi_redaction.py`
   - En ejemplos de código y prompts, usar `patient_id = 'test-user'`

2. **Safe zones**
   - Cada repo declara qué archivos requieren aprobación humana en `docs/_for_agents/safe-zones.md`
   - Críticos transversales:
     - `ellie-ai-platform/ingestion/cdc-functions/events.ts` (fall detection <2s)
     - `ellie-ai-platform/processing/dataflow/` (escrituras a FHIR)
     - `ellie-ai-platform/security/` (KMS, VPC-SC, DLP)
     - `ec-be/migrations/` (migraciones de DB)
     - `*-lambdas*/serverless.yml` IAM statements

3. **Dependencia más crítica: `typeorm-library`**
   - Hoy pinada a `github:elliecare/typeorm-library#develop`
   - Un cambio rompe 4 repos en producción silenciosamente
   - Migrar a SemVer es el fix más urgente del stack AWS

4. **Compliance**
   - HIPAA (BAA con Google Cloud)
   - LGPD (Brasil)
   - Ley 25.326 (Argentina)
   - GDPR

## Discoverabilidad

Filtros útiles en la UI de GitHub:
- [`cloud-aws`](https://github.com/orgs/elliecare/repositories?q=topic%3Acloud-aws)
- [`cloud-gcp`](https://github.com/orgs/elliecare/repositories?q=topic%3Acloud-gcp)
- [`tier-backend`](https://github.com/orgs/elliecare/repositories?q=topic%3Atier-backend)
- [`tier-frontend`](https://github.com/orgs/elliecare/repositories?q=topic%3Atier-frontend)
- [`status-production`](https://github.com/orgs/elliecare/repositories?q=topic%3Astatus-production)
- [`status-legacy`](https://github.com/orgs/elliecare/repositories?q=topic%3Astatus-legacy)

## Reglas de PR para agentes

- Diff budget recomendado: ≤ 500 líneas por PR
- Tocar safe-zones requiere aprobación humana explícita
- Cambios de IAM, migraciones, secretos: humano obligatorio
- Prod deploys: humano obligatorio
