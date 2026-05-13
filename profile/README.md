<div align="center">

# 🩺 Ellie Care

**Plataforma de telemedicina, IoT clínico y cuidado activo.**

AI · IoT · Contact Center · Clinical Decision Support

</div>

---

## 🗺️ Arquitectura por cloud

```
┌─────────────────────────────┐  ┌──────────────────────────────────┐
│           🟠 GCP            │  │             ☁️  AWS              │
├─────────────────────────────┤  ├──────────────────────────────────┤
│ ellie-ai-platform           │  │ ec-be (NestJS + ECS Fargate)     │
│   ADK + Gemini 2.5          │  │ aws-lambdas (Contact Center)     │
│   MCP Servers + A2A         │  │ lambdas-sensors                  │
│   Healthcare API (FHIR R4)  │  │ lambdas-ec-reporting             │
│   BigQuery + Pub/Sub        │  │ lambdas-ec-be                    │
│                             │  │ lambdas-user-portal              │
│ ec-firebase-functions       │  │ lambdas-backup-database          │
│   Watch middleware          │  │ automation-lambdas               │
│   (GCP Firebase Functions)  │  │ db-lambdas                       │
└─────────────────────────────┘  │ updateSensibility                │
                                  └──────────────────────────────────┘

┌─────────────────────────────┐  ┌──────────────────────────────────┐
│        🖥️  Frontends        │  │       📡 IoT Ecosystem           │
├─────────────────────────────┤  ├──────────────────────────────────┤
│ ec-portal   (paciente PWA)  │  │ sw-android                       │
│ ec-bo       (back office)   │  │   Samsung Galaxy Watch (WearOS)  │
│ ec-landing  (marketing)     │  │ device-manager                   │
│ ec-checkout (pagos)         │  │   Front de admin de relojes      │
└─────────────────────────────┘  │ ec-firebase-functions            │
                                  │   Watch middleware (GCP Firebase)│
                                  └──────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│                       📚 Shared, Infra & Tooling                     │
├──────────────────────────────────────────────────────────────────────┤
│ typeorm-library · ec-infra-2.0 · ec-metrics-catalog                 │
│ deploys-jenkins · firmas-institucionales                            │
└──────────────────────────────────────────────────────────────────────┘
```

> **Nota:** el prefijo `ec-` significa **Ellie Care**. Ejemplo: `ec-be` = Ellie Care Back-End.

---

## 📦 Productos activos

### 🤖 AI Platform (GCP)
| Repo | Descripción |
|---|---|
| [ellie-ai-platform](https://github.com/elliecare/ellie-ai-platform) | Plataforma AI con ADK + Gemini 2.5, MCP Servers, FHIR, BigQuery |

### ⚙️ Core Backend (AWS)
| Repo | Descripción |
|---|---|
| [ec-be](https://github.com/elliecare/ec-be) | Ellie Care Back-End — NestJS + TypeORM + ECS Fargate |
| [typeorm-library](https://github.com/elliecare/typeorm-library) | Entidades TypeORM compartidas |

### 📞 Contact Center (AWS)
| Repo | Descripción |
|---|---|
| [aws-lambdas](https://github.com/elliecare/aws-lambdas) | Contact Center sobre AWS Connect + Connect Cases (Serverless Framework v3) |

### λ AWS Lambdas — Servicios y Ops
| Repo | Descripción |
|---|---|
| [lambdas-sensors](https://github.com/elliecare/lambdas-sensors) | Queries de sensores (battery, HR, location, off-body, steps) |
| [lambdas-ec-reporting](https://github.com/elliecare/lambdas-ec-reporting) | Generación de reportes |
| [lambdas-ec-be](https://github.com/elliecare/lambdas-ec-be) | Integraciones con ec-be |
| [lambdas-user-portal](https://github.com/elliecare/lambdas-user-portal) | Backend de ec-portal |
| [lambdas-backup-database](https://github.com/elliecare/lambdas-backup-database) | Backups automáticos |
| [automation-lambdas](https://github.com/elliecare/automation-lambdas) | Automatización ops |
| [db-lambdas](https://github.com/elliecare/db-lambdas) | Operaciones de DB |
| [updateSensibility](https://github.com/elliecare/updateSensibility) | Lambda standalone |

### 🖥️ Frontends
| Repo | Stack | Descripción |
|---|---|---|
| [ec-portal](https://github.com/elliecare/ec-portal) | React 18 + Tailwind v4 + PWA | Ellie Care Portal del paciente |
| [ec-bo](https://github.com/elliecare/ec-bo) | React 18 + MUI v5 + Redux | Ellie Care Back Office |
| [ec-landing](https://github.com/elliecare/ec-landing) | React 18 + Vite + Tailwind | Ellie Care Landing pública |
| [ec-checkout](https://github.com/elliecare/ec-checkout) | TypeScript | Ellie Care Checkout (pagos) |

### 📡 IoT Ecosystem
Código relacionado a dispositivos que emiten telemetría — wearables, middleware y herramientas de administración.

| Repo | Plataforma | Descripción |
|---|---|---|
| [sw-android](https://github.com/elliecare/sw-android) | Samsung Galaxy Watch (WearOS) | App del smartwatch |
| [device-manager](https://github.com/elliecare/device-manager) | TypeScript | Front de administración de relojes — ver logs, setear configuración |
| [ec-firebase-functions](https://github.com/elliecare/ec-firebase-functions) | GCP Firebase Functions | Middleware del reloj — orquesta eventos y comandos |

### 🔧 Infra & Tooling
| Repo | Descripción |
|---|---|
| [ec-infra-2.0](https://github.com/elliecare/ec-infra-2.0) | Terraform — infra activa |
| [ec-metrics-catalog](https://github.com/elliecare/ec-metrics-catalog) | Catálogo de métricas/observabilidad |
| [deploys-jenkins](https://github.com/elliecare/deploys-jenkins) | Pipelines Jenkins |
| [firmas-institucionales](https://github.com/elliecare/firmas-institucionales) | Templates de firma de email |

---

## 🤖 AI-Ready

Todos los repos productivos siguen el estándar **AI-Ready**:

- `AGENTS.md` — instrucciones universales (Claude, Cursor, Copilot, Aider)
- `CLAUDE.md` — instrucciones específicas para Claude Code
- `docs/_for_agents/` — repo-map, glossary, safe-zones, common-tasks

→ Ver guía completa en [AGENTS.md](../AGENTS.md) de este repo
→ Manifest machine-readable: [services.yaml](../services.yaml)

---

## 🗄️ Legacy

<details>
<summary>Productos discontinuados y experimentos (15 repos)</summary>

**Línea cerrada — Hospital monitoring**
- ec-be-hospital · ec-fe-hospital

**Línea cerrada — Mobile app**
- ec-mobile

**Línea cerrada — Apple Watch**
- sw-watchos · ec-sw

**Línea cerrada — Web marketing antigua**
- ec-web · ec-portal-old · ec-fe (archived)

**Wearable experiments**
- facewatch-digital · sw-fallcare · health-measurements

**Jenkins testing históricos**
- test-pipeline · test-webhook

**Histórico (archived)**
- formulario-mater-dei · Ellie-Care---Epidata

</details>

---

## 🚀 Empezando

1. Explorá los repos productivos arriba
2. Cada repo tiene su propio `README.md` y `AGENTS.md`
3. El manifest `services.yaml` describe todas las dependencias

---

<sub>Ellie Care © · Salud digital con cuidado humano</sub>
