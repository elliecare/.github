# elliecare/.github

**Repo central de la organización Ellie Care.** Aloja la landing pública del org, el manifest de servicios, las guías para agentes de IA, templates de PR/issues, política de seguridad y el roadmap técnico.

Este repo es **especial de GitHub**: el archivo `profile/README.md` se renderiza automáticamente como landing page de https://github.com/elliecare.

---

## 📂 Contenido

```
elliecare/.github/
├── profile/
│   └── README.md          ← landing pública (github.com/elliecare)
├── services.yaml          ← manifest machine-readable de los 22 servicios productivos
├── AGENTS.md              ← guía universal para agentes de IA (Claude, Cursor, Copilot...)
├── ROADMAP.md             ← plan técnico transversal de la org
├── SECURITY.md            ← política de reporte de vulnerabilidades
├── PULL_REQUEST_TEMPLATE.md
├── ISSUE_TEMPLATE/
│   ├── bug.yml
│   └── feature.yml
└── README.md              ← este archivo
```

---

## 🎯 Qué encontrás acá

### 🏠 [`profile/README.md`](profile/README.md)
La landing visual de la organización. Diagrama de arquitectura por cloud, lista de productos activos categorizados, y sección legacy colapsable. Es lo primero que ve cualquiera que entra a github.com/elliecare.

### 📋 [`services.yaml`](services.yaml)
Manifest **machine-readable** de los 22 servicios productivos de Ellie Care. Cada servicio tiene metadata: cloud, tier, stack, status, dependencies, branches, AI-ready level. Pensado para que herramientas + agentes puedan descubrir el ecosistema sin leer 22 READMEs.

### 🤖 [`AGENTS.md`](AGENTS.md)
Guía universal para cualquier agente de IA (Claude Code, Cursor, Copilot, Aider, etc.). Define:
- Convenciones de naming (`ec-` = Ellie Care, `lambdas-`, `sw-`, etc.)
- Dominios y mapeo de repos
- Reglas universales (PHI, safe-zones, compliance, diff budget)
- Filtros nativos de GitHub para discovery

### 🗺️ [`ROADMAP.md`](ROADMAP.md)
Plan técnico transversal con 7 items priorizados. Cada uno tiene contexto, plan, riesgos, dependencias y owner. Documenta lo que está pendiente del stack (typeorm-library SemVer, Jenkins → GHA, PostgreSQL upgrade, etc.).

### 🔒 [`SECURITY.md`](SECURITY.md)
Política de reporte de vulnerabilidades. Email de contacto, compliance frameworks (HIPAA, LGPD, Ley 25.326, GDPR), versiones soportadas.

### 📝 [`PULL_REQUEST_TEMPLATE.md`](PULL_REQUEST_TEMPLATE.md)
Template default que aparece en cada PR de cualquier repo de la org. Incluye checklist de tests, secrets, PHI, safe-zones.

### 🐛 [`ISSUE_TEMPLATE/`](ISSUE_TEMPLATE/)
Templates para issues (bug + feature) — aparecen automáticamente cuando alguien abre un issue en cualquier repo de la org.

---

## 🌐 Descubrir la org

### Por la UI de GitHub

Filtros nativos vía topics:

- [`cloud-aws`](https://github.com/orgs/elliecare/repositories?q=topic%3Acloud-aws) — repos en AWS
- [`cloud-gcp`](https://github.com/orgs/elliecare/repositories?q=topic%3Acloud-gcp) — repos en GCP
- [`domain-iot`](https://github.com/orgs/elliecare/repositories?q=topic%3Adomain-iot) — IoT Ecosystem
- [`domain-frontends`](https://github.com/orgs/elliecare/repositories?q=topic%3Adomain-frontends) — Frontends
- [`status-production`](https://github.com/orgs/elliecare/repositories?q=topic%3Astatus-production) — solo productivos
- [`status-legacy`](https://github.com/orgs/elliecare/repositories?q=topic%3Astatus-legacy) — legacy

### Programáticamente

```bash
# Listar todos los servicios productivos del manifest
curl -s https://raw.githubusercontent.com/elliecare/.github/main/services.yaml | \
  yq '.services[] | select(.status == "production") | .name'

# Servicios por dominio
yq '.domains.iot-ecosystem' services.yaml
```

---

## 📦 Repos productivos (22)

Resumen por dominio. Detalle en [`services.yaml`](services.yaml).

| Dominio | Repos |
|---|---|
| 🤖 AI Platform | `ellie-ai-platform` |
| ⚙️ Core Backend | `ec-be`, `typeorm-library` |
| 📞 Contact Center | `aws-lambdas` |
| λ AWS Lambdas | `lambdas-sensors`, `lambdas-ec-reporting`, `lambdas-ec-be`, `lambdas-user-portal`, `lambdas-backup-database`, `automation-lambdas`, `db-lambdas`, `updateSensibility` |
| 🖥️ Frontends | `ec-portal`, `ec-bo`, `ec-landing`, `ec-checkout` |
| 📡 IoT Ecosystem | `sw-android`, `device-manager`, `ec-firebase-functions` |
| 🔧 Infra & Tooling | `ec-infra-2.0`, `ec-metrics-catalog`, `deploys-jenkins`, `firmas-institucionales` |

---

## 🤖 AI-Ready

Todos los repos productivos siguen el estándar **AI-Ready**. Cada uno tiene:

```
<repo>/
├── AGENTS.md                ← instrucciones universales para agentes
├── CLAUDE.md                ← guía específica para Claude Code
└── docs/_for_agents/
    ├── repo-map.md          ← mapa del repo
    ├── glossary.md          ← términos del dominio
    ├── safe-zones.md        ← qué NO modificar sin aprobación humana
    └── common-tasks.md      ← recetas para tareas frecuentes
```

Cualquier agente puede entrar a un repo y obtener contexto completo en minutos.

---

## 🚀 Cómo contribuir a este repo

### Actualizar el landing (`profile/README.md`)
Cualquier cambio visual del landing de la org. Se rendea automáticamente en github.com/elliecare cuando se mergea a `main`.

### Actualizar el manifest (`services.yaml`)
Cuando:
- Se agrega un servicio nuevo
- Cambia el status de un servicio (production → legacy)
- Cambia el stack de un servicio
- Se confirma un archivado de repo legacy

### Actualizar el roadmap (`ROADMAP.md`)
Cuando:
- Un item avanza (cambia `Status`)
- Se agrega un item nuevo
- Cambian las prioridades

PR + discusión, no commitear directo a `main`.

### Templates / política de seguridad
Cambios afectan a todos los repos de la org. PR + revisión obligatoria.

---

## 📚 Convenciones cross-org

Documentadas en distintos lugares:

| Convención | Documento |
|---|---|
| Git flow (creación/promoción de ramas) | [`ec-metrics-catalog/docs/git_flow.md`](https://github.com/elliecare/ec-metrics-catalog/blob/main/docs/git_flow.md) |
| Naming de repos | [`AGENTS.md`](AGENTS.md) |
| Topics de GitHub | [`AGENTS.md`](AGENTS.md) |
| Reporte de vulnerabilidades | [`SECURITY.md`](SECURITY.md) |
| Definiciones de métricas | [`ec-metrics-catalog`](https://github.com/elliecare/ec-metrics-catalog) |

---

## 🔗 Links útiles

- **Landing de la org:** https://github.com/elliecare
- **Service manifest:** [`services.yaml`](services.yaml)
- **Roadmap:** [`ROADMAP.md`](ROADMAP.md)
- **Security:** [`SECURITY.md`](SECURITY.md)
- **Contacto seguridad:** patricio.alba@ellie.care

---

<sub>Ellie Care © · Salud digital con cuidado humano</sub>
