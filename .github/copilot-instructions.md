# Instrucciones para GitHub Copilot - repositorio de contrato OpenAPI

Este repositorio contiene un contrato OpenAPI (contract-first) en la carpeta `api/`. Todo contrato se valida automaticamente en CI con GitHub Actions usando Spectral: Capa 1 en `.github/rules/01-estructura.yml` y Capa 2 en `.github/rules/02-funcional-bian.yml`. Cada regla tiene un codigo `GOB-*`.

Copilot es asistente, no autor autonomo: sugiere completado en el IDE y puede ayudar a redactar observaciones semanticas en el PR, pero el developer decide que aplicar. No agregues cambios por criterio propio ni inventes excepciones de gobierno.

## Estructura obligatoria del contrato

- OpenAPI `3.0.x` o `3.1.x`, en YAML, un solo archivo dentro de `api/`.
- `info` con `title`, `description` (propósito de la API), `version` y `contact`.
- `info.x-domains` (propuesta, GOB-015): lista de dominios de negocio que expone la API (`Clientes`, `Cuentas`, `Tarjetas`, `Prestamos`, `Lealtad`).
- Extensiones a nivel raíz (obligatorias):
  - `x-org-api-type`: `"UX"` (Experiencia), `"BS"`/`"BUSINESS"` (Negocio), `"CR"` (Core) o `"PR"`/`"PARTNER"` (Partner).
  - `x-org-api-id`: identificador en kebab-case, concordante con el nombre del repositorio.
  - `x-bian-service-domains`: lista de Service Domains BIAN trazados o alineados.
- `servers[0].url` con la taxonomia `/channel-{api-id}/v{n}`. La version de la API vive aqui, **nunca** en los paths.
- `tags` declarados a nivel raíz, cada uno con `description`.
- `components.securitySchemes` y `security` global declarados para que autenticación/autorización sean visibles por herramientas OpenAPI.

## Paths y operaciones

- Paths en kebab-case, solo recursos (ej. `/reward-agreements`), sin versión.
- Toda operación debe tener:
  - `tags`, `operationId` en camelCase iniciando con verbo (`getRewardAgreements`, `registerRewardRedemption`).
  - `summary`: una oración corta terminada en punto.
  - `description` con TODAS estas secciones markdown, usando `No aplica.` cuando corresponda:
    ```
    ### Seguridad
    ### Acerca de la funcionalidad expuesta
    ### Precondiciones para el consumo de esta versión de la API
    ### Usos válidos de Query Parameters
    ### Variantes válidas del Payload (Cuerpo del mensaje)
    ### Lista de valores usadas en esta versión de la API
    ```
  - Si la API es UX: `x-business-apis` listando las APIs de Negocio que invoca.
  - Trazabilidad BIAN:
    - `x-bian-service-domain`: Service Domain principal de la operación.
    - `x-bian-alignment`: `direct`, `adapted`, `traceability` o `exempted`.
    - Para `BS`/`CR`, preferir `direct`; para `UX`, usar `traceability` salvo excepción aprobada.
  - Headers de autenticación como `$ref` a `components/parameters` (`Authorization`, `x-auth-token`).
  - Respuestas de error estándar **400, 401, 403 y 404** (y **409** si la operación valida reglas de negocio), todas con schema `ApiException`.

## Components

- Nombres de schemas en PascalCase (`RewardAgreementDetail`, `ApiException`).
- Toda propiedad con `description` y `example`.
- Arrays con `maxItems`.
- Parámetros de header reutilizables en `components/parameters` con `required`, `style: simple`, `explode: false`, `schema` y `example`.
- Security schemes mínimos:
  - `bearerAuth`: `type: http`, `scheme: bearer`, `bearerFormat: JWT`.
  - `xAuthToken`: `type: apiKey`, `in: header`, `name: x-auth-token`.

## Reglas de contenido

- Una API de Experiencia (UX) solo ensambla, filtra o formatea datos de APIs de Negocio: **no** generes logica de negocio, calculos ni condiciones propias en la documentacion de la API.
- No mezcles informacion de dominios de negocio no declarados en `info.x-domains`.
- BIAN no debe forzarse como diseno de payload del front en UX; usalo como trazabilidad a capacidades bancarias. En BS/CR/PR, usa BIAN como alineamiento semantico mas estricto.
- En ejemplos usa siempre datos ficticios: nunca nombres, documentos, tokens o identificadores reales.
