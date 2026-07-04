# Plantilla de repositorio de contrato OpenAPI

Esta carpeta acompana la PPT y sirve para probar localmente el flujo de gobierno
del contrato de servicio.

## Que representa

- `api/loyalty-rewards-v1.yaml`: contrato OpenAPI de ejemplo con datos ficticios.
- `.github/workflows/api-contract-validation.yml`: workflow que corre en
  `pull_request` (se re-ejecuta con cada push a la rama del PR) y manualmente
  por `workflow_dispatch`.
- `.github/rules/01-estructura.yml`: Capa 1, validacion OpenAPI/Spectral y buenas
  practicas estructurales.
- `.github/rules/02-funcional-bian.yml`: Capa 2, validaciones funcionales
  codificables y trazabilidad BIAN.
- `.github/spectral.yml`: ruleset agregador para ejecutar ambas capas juntas.
- `.github/copilot-instructions.md`: guia para que Copilot sugiera alineado al
  gobierno sin aplicar cambios por criterio propio.

## Decisiones de proceso

- El onboarding no es manual: cuando gobierno de APIs aprueba la propuesta,
  Deploy Go crea automaticamente el repositorio desde la plantilla y genera
  `.github/` (workflow + reglas + instrucciones). Ese es el onboarding.
- El developer crea la rama y el PR. GitHub Actions valida en el PR, y el
  evento `pull_request` se re-ejecuta con cada push a la rama del PR: cada
  push recibe feedback sin ejecuciones duplicadas.
- No se valida en push directo: la politica corporativa limita la cantidad de
  pushes, asi que el ciclo corto de correccion ocurre antes de pushear, con
  lint local (seccion "Prueba local") y Copilot en el IDE.
- El PR puede abrirse aunque existan errores. Con branch protection, lo que se
  bloquea es el merge si los checks requeridos fallan.
- El feedback llega por Checks, logs de Actions, anotaciones en el diff y
  comentarios advisory de Copilot cuando esa fase este habilitada.
- Las tres capas forman parte de Deploy Go: las capas 1 y 2 son deterministas
  y bloqueantes; la capa 3 (Copilot advisory) es otro job del mismo workflow,
  no un proceso ajeno.
- Copilot sugiere y comenta; el developer decide que aplicar. Gobierno conserva
  la aprobacion final.

## Prueba local

Desde la raiz de esta carpeta:

```bash
npx --yes @stoplight/spectral-cli@6 lint "api/**/*.{yml,yaml}" \
  --ruleset .github/rules/01-estructura.yml \
  --fail-severity error

npx --yes @stoplight/spectral-cli@6 lint "api/**/*.{yml,yaml}" \
  --ruleset .github/rules/02-funcional-bian.yml \
  --fail-severity error
```

Para correr todo junto:

```bash
npx --yes @stoplight/spectral-cli@6 lint "api/**/*.{yml,yaml}" \
  --ruleset .github/spectral.yml \
  --fail-severity error
```
