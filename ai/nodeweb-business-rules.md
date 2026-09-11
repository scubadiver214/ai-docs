# NodeWeb Business Rules

## Scope
These rules define behavior for the NodeWeb template-processing runtime and its API proxy integration.

## Runtime Rules
1. NodeWeb is a Node.js ESM service.
2. The runtime exposes only the documented template-processing routes.
3. Health is database-backed: a successful SELECT 1 probe is required for healthy status.
4. Health failure responses include raw database error detail in application/problem+json payloads.
5. Template execution is read-only, environment-scoped, and depends on loading template body JSON by template id.
6. Unknown placeholders are preserved exactly as provided.
7. Placeholder execution is asynchronous and sequential within each processed string.
8. Whole-token object/array placeholder values remain native JSON values.
9. Embedded object/array placeholder values are JSON-stringified.
10. Non-string primitive values remain unchanged by the resolver.

## Function Registry Rules
1. Function modules are auto-discovered from the local function directory.
2. Registry metadata and executable delegates are cached after first successful load.
3. Each function module must provide id, name, description, example, and async execute.
4. Function identifiers are GUID strings.
5. fetchApiData argument handling preserves compatibility with comma-split string arguments.

## Built-In Function Rules
1. newGuid returns a new GUID string on every invocation.
2. apiAllUsers fetches upstream API data from the environment-scoped MenuManager users list route and preserves paged response shape.
3. apiUserCount fetches upstream API data from the environment-scoped MenuManager users count route.
4. fetchApiData only allows relative `/api/...` routes with optional query parameters.
5. fetchApiData allows caller-specified relative `/api/...` routes and is intentionally not constrained to a static endpoint list in the current trusted authoring model.
6. API-backed functions send the shared NodeWeb internal API key header.
7. If template authoring scope broadens beyond trusted/admin roles, fetchApiData must move to an explicit route allowlist.

## API Proxy Rules
1. API proxy endpoints are versioned under api/v{version:apiVersion}.
2. `GET /api/v1/template-functions` requires global `TemplateFunctionsRead` policy authorization.
3. Environment-scoped template execute proxy endpoints require global `TemplatesEdit` policy authorization.
4. Proxy endpoints preserve successful and failure response semantics from NodeWeb.
5. NodeWeb service base URL is externally configured and validated at startup.
6. Template execute proxy route includes environment in the route path.
7. API exposes execute route `POST /api/v1/templates/{environment}/execute/{id}` delegating to NodeWeb processing.
8. API also exposes global template CRUD routes backed by MenuManager database persistence; these are not NodeWeb runtime endpoints.

## Persistence Rules
1. Templates are stored with guid id, unique name, and non-null body.
2. body stores raw JSON template text and must not be truncated.
3. Template lookup for processing uses id and retrieves body directly.

## Observability Rules
1. Telemetry initializes at process startup before the HTTP listener is started.
2. The runtime supports OTLP trace and metric export via environment configuration.
3. Telemetry shutdown is attempted during SIGTERM handling.

## Non-Goals
1. NodeWeb does not provide template CRUD endpoints in this implementation slice.
2. NodeWeb does not own Menu Manager data mutation workflows.
3. NodeWeb does not replace existing API authorization patterns.
