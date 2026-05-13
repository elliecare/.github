# elliecare/.github

**Repo de configuración pública de la organización Ellie Care.** Aloja la landing pública del org y los templates / política de seguridad estándar de GitHub.

Este repo es **especial de GitHub**: el archivo `profile/README.md` se renderiza automáticamente como landing page de https://github.com/elliecare.

---

## 📂 Contenido

```
elliecare/.github/
├── profile/
│   └── README.md          ← landing pública (github.com/elliecare)
├── SECURITY.md            ← política de reporte de vulnerabilidades
├── PULL_REQUEST_TEMPLATE.md
├── ISSUE_TEMPLATE/
│   ├── bug.yml
│   └── feature.yml
└── README.md              ← este archivo
```

---

## 🔒 Documentación interna

La documentación técnica detallada (manifest de servicios, roadmap, plans, convenciones) vive en un repo privado separado por razones de seguridad.

Para members de la organización:
👉 [`elliecare/internal-docs`](https://github.com/elliecare/internal-docs) (privado)

Ese repo contiene:
- `services.yaml` — manifest machine-readable de los servicios productivos
- `ROADMAP.md` — plan técnico transversal
- `AGENTS.md` — guía universal para agentes de IA
- `plans/` — plans detallados de ejecución de items del roadmap

---

## 🤖 Repos AI-Ready

Cada repo productivo sigue el estándar **AI-Ready** con:

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

## 🚀 Cómo contribuir

### Actualizar el landing público (`profile/README.md`)
Para cambios visuales del landing público de la org. Se renderiza automáticamente en github.com/elliecare cuando se mergea a `main`.

⚠️ Mantener el contenido **público-friendly** — sin info sensible de arquitectura, vulnerabilities, ni detalles de stack.

### Templates / política de seguridad
Los templates de PR/issues y la política de seguridad afectan a todos los repos de la org. PR + revisión obligatoria.

---

## 🔗 Links útiles

- **Landing de la org:** https://github.com/elliecare
- **Documentación interna** (members only): https://github.com/elliecare/internal-docs
- **Web pública:** https://ellie.care
- **Contacto seguridad:** patricio.alba@ellie.care

---

<sub>Ellie Care © · Salud digital con cuidado humano</sub>
