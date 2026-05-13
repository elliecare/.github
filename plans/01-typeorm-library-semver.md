# Plan de ejecución — typeorm-library → SemVer

**Item del roadmap:** #1
**Status:** 📋 Plan listo, ejecución no iniciada
**Owner sugerido:** Backend team lead
**Duración estimada:** 2-3 semanas calendario
**Última actualización:** 2026-05-12

---

## 🎯 Objetivo

Migrar `typeorm-library` de distribución por GitHub branch (`#develop`) a **versionado SemVer** publicado en **GitHub Packages**, eliminando el riesgo de que un push a develop rompa 4 servicios productivos en su próximo cold start.

**Estado final esperado:**
```json
// ANTES (todos los consumers)
"typeorm-library": "github:elliecare/typeorm-library#develop"

// DESPUÉS
"typeorm-library": "^4.20.0"
```

---

## 📊 Resumen ejecutivo

| Métrica | Valor |
|---|---|
| Consumers afectados | 4 (`ec-be`, `lambdas-sensors`, `aws-lambdas`, `lambdas-ec-reporting`) |
| Fases | 6 |
| Tiempo activo de trabajo | ~5-7 días persona |
| Tiempo calendario | 2-3 semanas (incluyendo validaciones) |
| Downtime productivo | 0 (cada migración es por consumer + deploy normal) |
| Rollback path | Sí — cada fase es independiente |

---

## ✅ Pre-requisitos

Antes de arrancar:

- [ ] PAT (Personal Access Token) de GitHub con scope `write:packages` configurado en CI
- [ ] Confirmar versión inicial (sugerencia: continuar desde `4.20.0`)
- [ ] Acceso de admin al repo `elliecare/typeorm-library`
- [ ] Coordinación con owners de los 4 consumers
- [ ] Ambiente UAT funcional para cada consumer (donde validar)
- [ ] Slack channel `#deploy-coordination` (o equivalente) para comunicación

---

## Fase 0 — Auditoría inicial

**Duración:** 1 día
**Trabajo:** Read-only — verificar estado real antes de tocar nada

### Tareas

1. **Verificar consumers reales:**
   ```bash
   gh search code "typeorm-library" --owner elliecare \
     --json repository --jq '.[].repository.nameWithOwner' | sort -u
   ```
   Esperado: 4 repos confirmados. **Si aparecen más → agregar al plan.**

2. **Inspeccionar el estado actual de `develop`:**
   ```bash
   git clone git@github.com:elliecare/typeorm-library.git
   cd typeorm-library
   git checkout develop

   # Verificar que dist/ está sincronizado con src/
   npm install
   npm run build
   git diff dist/

   # Si hay diffs → el dist/ committeado está desincronizado
   # Antes de publicar, dejar dist/ en sync
   ```

3. **Listar las entities exportadas:**
   ```bash
   grep -oP 'export \{ \K[A-Z][A-Za-z_]+' index.ts | sort -u
   ```
   Documentar la lista — será el contrato de la `v4.20.0`.

4. **Confirmar versión inicial:**
   - El `package.json` actual dice `4.19.0`
   - Próxima versión: **`4.20.0`** (minor bump — primer release SemVer real)
   - Alternativa: empezar en `1.0.0` para señalar "nueva era" (menos recomendado, confunde a quienes leen el package.json hoy)

### Validation gate

- [ ] 4 consumers confirmados (no más)
- [ ] `dist/` en sync con `src/` después de `npm run build`
- [ ] Lista de entities exportadas documentada
- [ ] Versión inicial decidida y comunicada al equipo

---

## Fase 1 — Publicar v4.20.0 como primera release SemVer

**Duración:** 1-2 días
**Trabajo:** Setup de publishing + primera publicación

### 1.1 Configurar GitHub Packages en el repo

Crear `.github/workflows/publish.yml`:

```yaml
name: Publish to GitHub Packages

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://npm.pkg.github.com'
          scope: '@elliecare'

      - run: npm install
      - run: npm run build
      - run: |
          # Verificar que dist/ está actualizado
          git diff --exit-code dist/ || (echo "dist/ desincronizado con src/" && exit 1)

      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 1.2 Ajustar `package.json`

```json
{
  "name": "@elliecare/typeorm-library",
  "version": "4.20.0",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  },
  // ... resto igual
}
```

⚠️ El cambio de `"name"` a `@elliecare/typeorm-library` (scoped package) es necesario para GitHub Packages.

### 1.3 Mantener compatibilidad con el pin viejo

**No remover** el package.json + `dist/` del repo todavía. Los consumers siguen pegando a `#develop` durante la migración.

### 1.4 Publicar v4.20.0

```bash
# En el repo typeorm-library, branch main:
git checkout main
git pull
npm run build

# Verificar dist/ en sync
git diff dist/

# Tag la versión
git tag -a v4.20.0 -m "First SemVer release - aligns with #develop state"
git push origin v4.20.0

# El workflow se dispara automáticamente y publica a GitHub Packages
```

### 1.5 Verificar que se publicó

```bash
# En cualquier máquina, intentar instalar
mkdir /tmp/test-install && cd /tmp/test-install
npm init -y
npm config set @elliecare:registry https://npm.pkg.github.com
npm config set //npm.pkg.github.com/:_authToken <YOUR_PAT_WITH_read:packages>
npm install @elliecare/typeorm-library@4.20.0

# Verificar que se instaló correctamente
ls node_modules/@elliecare/typeorm-library/dist/
node -e "console.log(Object.keys(require('@elliecare/typeorm-library')))"
```

### Validation gate

- [ ] `v4.20.0` publicada en GitHub Packages
- [ ] Workflow de publish funcionando
- [ ] Test de instalación exitoso desde otro environment
- [ ] `dist/` y `src/` siguen en sync en `main`
- [ ] CI configurado para auto-publicar en próximos tags

---

## Fase 2 — Migrar `lambdas-ec-reporting` (primer consumer)

**Duración:** 2-3 días
**Por qué primero:** Menor blast radius productivo. Si algo sale mal, no afecta a `ec-be` (el core).

### 2.1 Crear feature branch

```bash
cd lambdas-ec-reporting
git checkout develop
git pull
git checkout -b feat/migrate-to-semver-typeorm-library
```

### 2.2 Configurar auth para GitHub Packages

Crear `.npmrc` en el root del repo:
```
@elliecare:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${NPM_AUTH_TOKEN}
```

En `.gitignore` ya debería estar pero verificar que `.npmrc` con token no se commitee.

En CI (Jenkins por ahora, GHA en el futuro): exponer `NPM_AUTH_TOKEN` como secret.

### 2.3 Actualizar dependencia

```bash
# Reemplazar el pin viejo
npm uninstall typeorm-library
npm install @elliecare/typeorm-library@^4.20.0
```

Esto modifica el `package.json`:
```diff
- "typeorm-library": "github:elliecare/typeorm-library#develop"
+ "@elliecare/typeorm-library": "^4.20.0"
```

### 2.4 Actualizar imports

Buscar todos los imports:
```bash
grep -r "from \"typeorm-library\"" --include="*.ts" .
grep -r "require(\"typeorm-library\")" --include="*.ts" .
```

Reemplazar:
```diff
- import { Battery, Entities } from "typeorm-library";
+ import { Battery, Entities } from "@elliecare/typeorm-library";
```

⚠️ Cuidado: hacer el reemplazo automático con `sed` puede romper si hay matches parciales. Revisar uno por uno o usar `replace_all` con confirmación.

### 2.5 Tests + build local

```bash
npm install
npm run build
npm test  # si hay tests
```

Si hay errores de tipos → diferencias entre el `dist/` viejo (de `#develop`) y la versión publicada (`v4.20.0`). En teoría son idénticos, pero verificar.

### 2.6 Deploy a dev

```bash
git add .
git commit -m "feat: migrate to @elliecare/typeorm-library@^4.20.0"
git push -u origin feat/migrate-to-semver-typeorm-library

# Abrir PR a develop
gh pr create --base develop --title "Migrate to @elliecare/typeorm-library SemVer"
```

Mergear a `develop` → deploy automático a dev.

### 2.7 Validación en dev

- [ ] Lambda invoca exitosamente
- [ ] Query a la DB devuelve la misma data que antes
- [ ] Logs sin errores nuevos
- [ ] Smoke test del endpoint principal

### 2.8 Promover a uat → prod

```bash
# Mergear develop a uat
git checkout uat
git pull
git merge develop
git push
# → deploy automático a uat

# Validar 24-48hs en uat
# Después mergear a main para deploy a prod

git checkout main
git pull
git merge uat
git push
# → deploy a prod (con aprobación si hay)
```

### Validation gate

- [ ] `lambdas-ec-reporting` corriendo en prod con `@elliecare/typeorm-library@^4.20.0`
- [ ] 48hs sin errores nuevos en logs
- [ ] Sin regresiones detectadas en reports generados

---

## Fase 3 — Migrar `lambdas-sensors`

**Duración:** 2 días
**Por qué segundo:** Mismo patrón que reporting, mayor data volume pero solo read.

### Particularidad

`lambdas-sensors` es **npm workspaces** con 5 packages:
- `getBatteryLevels`, `getHeartRateValues`, `getLocationsValues`, `getOffBodyValues`, `getStepsValues`

Cada package tiene su propio `package.json`. La dependencia puede estar:
- En el `package.json` root (compartida vía workspaces)
- En cada package individual

### 3.1 Verificar dónde está la dependencia

```bash
cd lambdas-sensors
grep -l "typeorm-library" packages/*/package.json package.json
```

### 3.2 Actualizar según corresponda

Si está en root:
```bash
npm uninstall typeorm-library
npm install @elliecare/typeorm-library@^4.20.0
```

Si está en cada package:
```bash
for pkg in packages/*/; do
  cd $pkg
  npm uninstall typeorm-library
  npm install @elliecare/typeorm-library@^4.20.0
  cd ../..
done
```

### 3.3 Actualizar imports en los 5 packages

```bash
grep -r "from \"typeorm-library\"" packages/
# Reemplazar uno por uno
```

### 3.4 Tests por package

```bash
# Por package (no hay tests globales)
for pkg in getBatteryLevels getHeartRateValues getLocationsValues getOffBodyValues getStepsValues; do
  cd packages/$pkg
  npm run build
  cd ../..
done
```

### 3.5 Validación especial: `OffBody`

⚠️ **Atención:** `getOffBodyValues` tiene la lógica de previous-day baseline (documentado en safe-zones). Verificar que sigue funcionando:

```bash
# Invocar el Lambda en uat con un device real
aws lambda invoke \
  --function-name getOffBodyValues-uat \
  --payload '{"body":"{\"device_id\":\"<test-device>\",\"from\":\"2026-04-01T00:00:00Z\",\"to\":\"2026-04-02T00:00:00Z\"}"}' \
  /tmp/response.json

# Verificar que el primer registro del array es del día anterior (baseline)
cat /tmp/response.json | jq '.input[0].created_on_device'
```

### 3.6 Deploy gradual

```bash
# Mismo patrón: dev → uat → prod
# Pero deployar package por package por si algo falla
```

### Validation gate

- [ ] Los 5 Lambdas corriendo en prod con la nueva dependencia
- [ ] `getOffBodyValues` sigue devolviendo el baseline del día anterior
- [ ] 48hs sin errores

---

## Fase 4 — Migrar `aws-lambdas`

**Duración:** 2 días
**Por qué tercero:** Contact Center es crítico pero las entidades que usa de typeorm-library son acotadas.

### 4.1 Identificar uso de typeorm-library

```bash
cd aws-lambdas
grep -r "typeorm-library" --include="*.ts" .
```

`aws-lambdas` posiblemente solo use algunas entities específicas (Connect-related). Verificar el alcance.

### 4.2 Aplicar el patrón estándar

Igual que Fases 2 y 3:
1. `.npmrc` con auth
2. `npm uninstall typeorm-library && npm install @elliecare/typeorm-library@^4.20.0`
3. Update imports
4. Tests + build
5. Deploy dev → uat → prod

### 4.3 Validación especial: Cases integration

`aws-lambdas` integra con AWS Connect Cases. Verificar que después del switch:
- [ ] Búsqueda de cases sigue funcionando
- [ ] Creación de case funciona
- [ ] `caseStatusChanged` trigger sigue disparando el webhook a `ec-be`

### Validation gate

- [ ] `aws-lambdas` en prod con la nueva dependencia
- [ ] Contact Center funcional
- [ ] 48hs sin issues reportados por operadores

---

## Fase 5 — Migrar `ec-be` (el más crítico)

**Duración:** 3-5 días
**Por qué último:** Es el monolito principal — usa la mayoría de las entities.

### 5.1 Coordinación

Antes de empezar:
- [ ] Comunicar a todo el equipo backend
- [ ] Programar la ventana de deploy a prod (idealmente fuera de horario peak)
- [ ] Tener rollback plan listo (revert + redeploy de la versión anterior)

### 5.2 Branch + cambios

```bash
cd ec-be
git checkout develop
git pull
git checkout -b feat/migrate-to-semver-typeorm-library
```

Mismo patrón:
1. `.npmrc` con auth
2. Update `package.json`
3. Update imports — pero acá serán **muchos archivos** (todo el backend usa entities)
4. Tests completos

### 5.3 Tests críticos antes de mergear

```bash
# Unit tests
npm test

# Integration tests
npm run test:integration

# E2E tests si existen
npm run test:e2e

# Type check
npm run typecheck

# Lint
npm run lint
```

⚠️ Si hay tests que mockean `typeorm-library`, actualizarlos al nuevo path.

### 5.4 Deploy gradual — develop → uat → prod

**Pausa de 48hs entre uat y prod** para asegurar que no aparecen issues sutiles.

```bash
# 1. Merge a develop → deploy a dev
# 2. Merge a uat → deploy a uat
# 3. Validar 48hs en uat
#    - Logs sin errores
#    - Métricas operacionales normales
#    - Smoke tests pasan
# 4. Merge a main → deploy a prod (con humano aprobando)
```

### 5.5 Plan de rollback

Si algo falla en prod:

```bash
# Opción A: Revert del PR + deploy
git revert <merge-commit>
git push
# Deploy automático

# Opción B: Re-deployar la imagen anterior
# Desde la consola AWS ECS → revert task definition

# Opción C: Force pin a la version anterior
# Si el problema es solo de entity drift, pinear a una version anterior de @elliecare/typeorm-library
```

### Validation gate

- [ ] `ec-be` en prod con `@elliecare/typeorm-library@^4.20.0`
- [ ] 1 semana de operación normal sin issues nuevos
- [ ] Performance similar o mejor

---

## Fase 6 — Cutover y limpieza

**Duración:** 1 día
**Trabajo:** Decommissioning del flujo viejo + docs

### 6.1 Confirmar que `#develop` ya no es load-bearing

Buscar en todos los repos cualquier referencia restante a `github:elliecare/typeorm-library`:

```bash
gh search code "github:elliecare/typeorm-library" --owner elliecare
```

Resultado esperado: 0 matches en productivos.

### 6.2 Cambiar el workflow del repo `typeorm-library`

A partir de ahora:
- Branch `main` es la fuente de verdad
- `develop` se puede archivar o seguir usando como pre-release (alpha)
- Cada PR a `main` que mergea genera un tag → CI publica una nueva versión

Documentar el nuevo workflow en `typeorm-library/CONTRIBUTING.md`:

```markdown
# Contributing — typeorm-library

## Workflow
1. Branch off de `main`
2. Cambios + `npm run build` + commit
3. Bumpear versión en package.json (`npm version patch|minor|major`)
4. PR a `main`
5. Después del merge, tagear: `git tag v<version> && git push --tags`
6. CI publica automáticamente a GitHub Packages

## SemVer
- `patch` (4.20.0 → 4.20.1): bugfix, agregar columna nullable
- `minor` (4.20.0 → 4.21.0): nueva entity, nueva columna
- `major` (4.20.0 → 5.0.0): renombrar entity, cambio breaking
```

### 6.3 Actualizar documentación AI-Ready

Actualizar los archivos del repo `typeorm-library`:

```bash
cd typeorm-library
# En AGENTS.md, CLAUDE.md, docs/_for_agents/:
# Reemplazar "pinned to #develop" con "published as @elliecare/typeorm-library"
```

### 6.4 Actualizar el manifest

En `elliecare/.github/services.yaml`:

```diff
  - name: typeorm-library
    repo: elliecare/typeorm-library
    cloud: multi
    tier: library
-   distribution: github-branch
+   distribution: github-packages
+   semver: true
    consumed_by: [ec-be, lambdas-sensors, aws-lambdas, lambdas-ec-reporting]
    status: production
-   ai_ready: pending
+   ai_ready: full
```

### 6.5 Actualizar el ROADMAP

En `elliecare/.github/ROADMAP.md`:

```diff
- ## 1. typeorm-library → SemVer
- **Status:** 🔴 Abierto · **Prioridad:** CRÍTICA
+ ## 1. typeorm-library → SemVer
+ **Status:** ✅ Completado (2026-XX-XX) · **Prioridad:** CRÍTICA
```

### 6.6 Comunicación

```
📢 typeorm-library migración a SemVer completada

A partir de hoy:
- ✅ Todos los consumers usan @elliecare/typeorm-library@^4.20.0
- ✅ Cambios a typeorm-library siguen SemVer
- ✅ Para usar una nueva version, bumpear en el consumer y deployar
- ✅ Pushes a #develop ya no afectan producción

Próxima version: v4.21.0 saldrá cuando mergemos el PR #XXX
```

### Validation gate

- [ ] 0 referencias a `github:elliecare/typeorm-library` en repos productivos
- [ ] `typeorm-library/main` es la fuente de verdad
- [ ] Docs actualizados
- [ ] Manifest actualizado
- [ ] ROADMAP actualizado
- [ ] Comunicación enviada al equipo

---

## ⚠️ Riesgos identificados y mitigación

| Riesgo | Impacto | Mitigación |
|---|---|---|
| `dist/` publicado desincronizado con `src/` | Consumers fallan en runtime | Workflow CI verifica `git diff --exit-code dist/` antes de publicar |
| Auth de GitHub Packages mal configurada en CI | Builds fallan | Test en feature branch antes de mergear |
| Entity con shape diferente entre `#develop` y `v4.20.0` | Schema mismatch en runtime | Auditoría en Fase 0 + tag desde el estado exacto |
| Algún repo consumer no identificado | Olvido en la migración | Audit del Fase 0 + post-cutover scan |
| Rollback necesario en `ec-be` | Downtime productivo | Plan de rollback documentado en Fase 5 |
| Versión equivocada se publica | Consumers pueden bajar accidentalmente | SemVer caret (`^4.20.0`) protege de major bumps |
| Cambio breaking entre fases | Consumers a distintas versiones se desincronizan | Tagear todas las migrations a la misma version |

---

## 📋 Checklist consolidado

### Pre-flight
- [ ] PAT con `write:packages` configurado
- [ ] Auditoría de consumers completada
- [ ] Versión inicial decidida (recomendado: `4.20.0`)
- [ ] Aprobación del equipo

### Fase 1 — Publish
- [ ] Workflow `publish.yml` creado en `typeorm-library`
- [ ] `package.json` actualizado a `@elliecare/typeorm-library`
- [ ] `v4.20.0` publicada
- [ ] Test de instalación desde otra máquina exitoso

### Fase 2 — lambdas-ec-reporting
- [ ] PR creado y mergeado
- [ ] Deploy a dev, uat, prod
- [ ] 48hs sin issues

### Fase 3 — lambdas-sensors
- [ ] 5 packages actualizados
- [ ] OffBody baseline validado
- [ ] Deploy completo

### Fase 4 — aws-lambdas
- [ ] Migración completada
- [ ] Cases + Contact Center validados
- [ ] Deploy completo

### Fase 5 — ec-be
- [ ] Tests pasando
- [ ] Deploy gradual (dev → uat → prod)
- [ ] 1 semana de operación estable

### Fase 6 — Cutover
- [ ] 0 referencias a `#develop` en productivos
- [ ] Workflow del repo actualizado
- [ ] Docs AI-Ready actualizados
- [ ] Manifest + ROADMAP actualizados
- [ ] Comunicación enviada

---

## 🎬 Después de completar

Esta migración desbloquea:

1. **Roadmap item #6 (MCP Server expansion)** — los MCP servers que usen entities pueden pinear a una version segura
2. **Cambios cross-stack más seguros** — agregar una columna ya no requiere un dance frágil
3. **Posibilidad de SemVer en otros internal libraries** — patrón aplicable a otros repos

---

<sub>Plan vivo. Actualizar después de cada fase con learnings reales.</sub>
