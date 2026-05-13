# Security Policy — Ellie Care

## Reportar una vulnerabilidad

**No abras un issue público.** Mandá un email a **security@elliecare.app** con:

- Descripción del problema
- Pasos para reproducir
- Impacto potencial
- Sugerencia de mitigación (si aplica)

Recibirás confirmación dentro de 48 horas hábiles.

## Datos protegidos

Ellie Care opera con **PHI (Protected Health Information)** y datos personales sensibles bajo:

- HIPAA (Estados Unidos — BAA con Google Cloud)
- LGPD (Brasil)
- Ley 25.326 (Argentina — Protección de Datos Personales)
- GDPR (Europa)

**Nunca** incluyas datos reales de pacientes en:
- Issues, PRs o comentarios públicos
- Logs accesibles externamente
- Prompts a LLMs externos

## Versiones soportadas

Solo el branch `main` (o `production` cuando aplique) de cada repo activo en [`services.yaml`](services.yaml) recibe patches de seguridad.

Repos marcados como `status-legacy` en [`services.yaml`](services.yaml) no reciben mantenimiento activo de seguridad.
