
---
applyTo: '**'
---

# Design Rules for GitHub Copilot (BOI-MCP, RAML, API Governance)
# ✅ Copilot MUST validate all fields against BOI-MCP
# ❌ Copilot MUST NOT generate fields, types, or contexts unless validated
# ✅ Use AddField DSL for new field suggestions
# ❌ Do NOT fabricate data types, paths, or examples
# ✅ Use inline comments like "# From BOI-MCP: concept.context.fieldName"
# ✅ If field not found, insert fallback:
# TODO: Field not validated in BOI-MCP — confirm before using

# API Design Rules

- Each endpoint must include a `description`.
- Each endpoint must define responses for: 200, 400, 404, 500.
- All query parameters and headers must be typed.
- Traits such as `secured`, `pagination`, and `errorResponse` must be applied where appropriate.
- All URI parameters (e.g., `{customer-id}`) must be explicitly defined using the `uriParameters:` block within the resource. Each parameter must include:
  - `type`
  - `description`
  - `example`

## HTTP Method Enforcement
- Use `GET` for read operations only.
- Use `POST` for creating new resources only.
- Use `PUT` for full resource replacement only.
- Use `PATCH` for partial updates only.
- Use `DELETE` for removal of resources only.

## Status Code Enforcement
- `200` must be used for successful `GET` or `PATCH`.
- `201` must be used after successful `POST`.
- `204` must be used when no content is returned.
- `400`, `401`, `403`, `404`, `409`, `422`, `429`, and `500` must be used appropriately.

## Health Check
- A `/ping` endpoint must be defined using `ping` resourceType from `commonLibrary`.

## Request/Response Type Naming
- Request body types must be named with `Request` suffix, e.g., `CreateAccountRequest`.
- Response body types must use `Response`, e.g., `CreateAccountResponse`.
- Standard error type must be named `ErrorResponse`.
- Use `Paginated{Entity}Result` for paginated list responses.

## File Naming
- RAML type files must use PascalCase.
- Example files must use kebab-case.

## Field Naming
- All field and property names must use camelCase.

## URI Naming
- Resource paths must use plural nouns.
- URI segments must use nouns, not verbs.
- URI nesting must not exceed 2 levels.
- Filters and search must use query parameters, not nested URI paths.

## Naming Consistency
- RAML types, field names, example files, and resource paths must use consistent naming.

## Suffix Rules
- Fields ending with `Id` must be used for identifiers.
- Fields ending with `At` must be used for timestamps.

## Enum Rules
- Enum values must be uppercase.
- Abbreviations must not be used unless standard.

# Performance Rules

## Caching
- All `GET` endpoints must support `ETag` or `Last-Modified`.
- All `GET` endpoints must accept `If-None-Match` or `If-Modified-Since`.
- All cacheable responses must include `Cache-Control`.

## Pagination
- Endpoints returning collections must support `limit` and `offset` or `page`.
- A `pagination` trait must be applied.

## Payload Optimization
- Large nested structures must not be returned by default.
- `fields` query parameter must be supported for sparse responses.
- `204` must be used when the response body is not required.

# Error Handling Rules

## Error Response Enforcement
- All 4xx and 5xx responses must use `ErrorResponse` from `rest-error-library`.
- `ErrorResponse` must include `code`, `message`, and optionally `details` and `traceId`.
- No stack traces or internal error names must be exposed in the response.

# Observability Rules

## Correlation ID
- All requests must accept `x-correlation-id` or `traceId`.
- This ID must be included in error responses.
- The ID must be passed to downstream services and logs.

## Logging
- Logs must include: `timestamp`, `method`, `path`, `statusCode`, `durationMs`, and `traceId`.
- Logs must not contain sensitive data or full request/response bodies unless approved.

# Folder Structure Rules

## Enforce Folders
- The project must include:
  - `dataTypes/`
  - `dataTypes/enums/`
  - `dataTypes/primitives/`
  - `dataTypes/types/`
  - `examples/`
  - `traits/`
  - `securitySchemes/`
  - `changelog.raml`

## Enforce Resource Folders
- Each resource must have its own folder inside `dataTypes/types/`.
- Each resource must have its own folder inside `examples/`.

## RAML Type Location
- All RAML type files must be placed inside `dataTypes/types/{resource}/`.
- Request body types must have dedicated files.
- Response body types must have dedicated files.
- Error response types must be defined.

## Enforce Example Placement
- All example files must be placed inside `examples/{resource}/`.

# Type Definition Rules

## External Type Enforcement
- All schemas must be defined using external RAML types via the `types:` section.
- Inline type definitions must not be used inside resource or method definitions.

- Every field in a type must include a `description`.
- The `description` value must exactly match the `Common Data Requirement Definition` from the CDR, quoted as-is.
- Every type must include at least one valid `example`.
- Fields must explicitly declare `required: true` or `required: false`.

## RAML Structural Compliance

- All RAML `type:` definitions must use a top-level root object named after the canonical Concept.
- Nested inside the Concept, each Context must be its own object.
- All fields (Data Requirements) must reside within their defined Context block.
- This structure must be consistent across all `type:` files and example payloads.

### Example:
For Concept = Customer, Context = Consent, and field = consentTypeCode:

```yaml
customer:
  consent:
    consentTypeCode: "AUDIO"
```

- Example files must mirror the same structure as defined in the corresponding RAML type.
- This structural compliance is required to enforce canonical field mapping and CDR traceability.

- Each field must map to a canonical path defined in BOI-MCP.
- The field name must be derived from the canonical `{concept}.{context}.{fieldName}` structure.
- The RAML structure must reflect the concept and context hierarchy as nested objects.  
  Fields must be declared as:  
  ```yaml
  {Concept}:
    {Context}:
      {Data Requirement}: value
  ```  
  Example for Concept = Customer, Context = Consent, Field = consentTypeCode:  
  ```yaml
  customer:
    consent:
      consentTypeCode: "AUDIO"
  ```  
  This structure must be followed in all RAML types and examples derived from BOI-MCP.
- Do not include inline comments like `# From BOI-MCP:` inside RAML files, as comments are not reliably retained and may interfere with tooling or transformations.
- Instead, ensure traceability is enforced via consistent naming, folder structure, and automated CDR validation tooling.

### RAML Field Example – Good vs. Bad

```yaml
# ❌ BAD: Field added without BOI-MCP validation
# paymentCode:
#   type: string

# ✅ GOOD: Field validated and annotated
# From BOI-MCP: loan.application.paymentCode
paymentCode:
  type: string
  required: true
  description: Internal code for type of payment
  example: "BANK_TRANSFER"

# ❓ UNVALIDATED: Use TODO fallback
# TODO: Field not validated in BOI-MCP — confirm before using
# manualReviewFlag:
#   type: boolean

- RAML types must always start with `#%RAML 1.0 DataType`
- Include `type: object` and a `properties:` block at the correct nesting level
- Flat root-level fields (e.g. without concept/context) are invalid unless explicitly specified
- These rules apply to both type definitions and examples
```

- If a resource path includes URI parameters (e.g., `/customers/{customer-id}`), the RAML must declare a `uriParameters:` block at the resource level. The block must include complete metadata as in the following example:

  ```yaml
  /customers/{customer-id}:
    uriParameters:
      customer-id:
        type: string
        description: Unique identifier for the customer
        example: "CUST123"
  ```

## Type Conventions
- All UUID fields must use `type: string` and be named with the `Id` suffix.
- All date or datetime fields must use ISO 8601 format.
- For datetime values, use `type: datetime` and format `YYYY-MM-DDTHH:mm:ssZ`.
- Use enums instead of booleans when future values are possible.

## Example File Rules
- Each example file must be located in `examples/{resource}/`.
- The file name must clearly describe its usage and match the type it represents.
- Example files must follow kebab-case.

## Type Reuse Restrictions
- Do not reuse types across unrelated resources.
- Shared types must be placed in `dataTypes/types/common/`.

# Security Best Practices

## Endpoint Protection
- All endpoints must be secured using a supported authentication scheme.
- Public endpoints must be explicitly marked as unauthenticated.
- HTTPS must be enforced for all endpoints.
- Credentials must only be passed via `Authorization`, `x-client-id`, or `client_id` headers.
- Credentials must not be passed in query parameters or request bodies.
- Sensitive fields must be encrypted or tokenized.

# Rate Limiting Standards

## Gateway Protection
- All APIs must implement rate limiting at the gateway.
- `429 Too Many Requests` must be returned when rate limits are exceeded.
- A `Retry-After` header must be included where supported.
- Rate limit headers may be included.
- Policy internals must not be exposed.

# Versioning and Governance

## Versioning Enforcement
- The `api.raml` file must include a `version:` field using Semantic Versioning.
- The base URI must include the version prefix.

## Version Increment Rules
- MAJOR version increments must be used for breaking changes.
- MINOR version increments must be used for backward-compatible enhancements.
- PATCH version increments must be used for documentation or non-contract changes.

## Changelog Rules
- All changes must be documented in `changelog.raml`.
- Changelog entries must be grouped by version and date.
- Each version entry must include a bullet list of changes.
- Entries must be in reverse chronological order.
- Each version must use ISO format date (`YYYY-MM-DD`).

## Changelog Format
```yaml
#%RAML 1.0 DocumentationItem
title: CHANGELOG
content: |
  * Version 1
    * Version 1.3.0 (2025-05-12)
      * Added POST /accounts/close endpoint
    * Version 1.2.2 (2025-05-10)
      * Updated description of CustomerType
      * Fixed example in create-customer.raml
```

# BOI-MCP Governance

## Canonical Field Derivation (via BOI-MCP)

...

## Non-Canonical Field Proposal Rule
- If a field is proposed by the agent that is **not found in the BOI-MCP/CDR**, the agent must:
  1. Stop and notify the user that the field is not canonical.
  2. Propose valid alternatives (if available) using a single call to `BOI_filter_rows_by_columns` with both Concept and Context as filters.
  3. Ask the user whether to:
     - Replace the field with a canonical one, or
     - Proceed to insert it into the CDR (requires justification).
- The agent must not auto-insert or fabricate fields without user approval.
- If the user approves adding the field, the agent must:
  - Insert it into the CDR Excel file under the correct `{concept}.{context}` section using `BOI_write_workbook_data` (Workbook Tool).
  - Populate the `Common Data Requirement`, `Definition`, and `Type` based on user input.
  - Validate the row using `BOI_get_worksheet` (Worksheet Tool) and/or `BOI_read_workbook_data` (Workbook Tool) as needed.
  - Suggest the next available row or coordinate to avoid overwrites using `BOI_get_workbook_info` (Workbook Tool) and/or `BOI_get_worksheet` (Worksheet Tool).
  - Confirm the field was successfully added before proceeding with RAML generation by checking the result of `BOI_write_workbook_data` and re-validating with `BOI_get_worksheet` or `BOI_read_workbook_data`.

### Agent Behavior

- Prompt the user to confirm the **Concept** and **Context**.
  - These must match exactly with the `Concept` and `Context` columns in the BOI-MCP CDR Excel.
  - Do not proceed if either is not confirmed.
- Validate if the field (Data Requirement) exists in the CDR by checking against:
  - `Concept`
  - `Context`
  - `Data Requirement`
- Use a single call to `BOI_filter_rows_by_columns` with both `Concept` and `Context` as keys in the same invocation.  
  Example: `BOI_filter_rows_by_columns({ Concept: "Customer", Context: "Customer" })`
- Then match the field name in the `Data Requirement` column within those results.
  - If the field is **missing**:
    - First, use `BOI_filter_rows_by_columns({ Concept, Context })` to retrieve all valid fields.
    - Then, compare the requested field name to the CDR `Data Requirement` entries using similarity ranking.
    - If a close match is found, suggest canonical alternatives and ask the user to confirm one.
    - If no close match exists:
      - Ask the user to confirm adding the new field to the CDR.
      - Upon confirmation:
        - Insert the new field using `BOI_write_workbook_data`, including:
          - `Concept`
          - `Context`
          - `Data Requirement`
          - `Common Data Requirement (CDR) (Long Name)` as `{Concept}:{Context}:{fieldName}`
          - `Common Data Requirement Definition` quoted and reused as the RAML `description`
          - `CDR UID (BDO Aligned)` — must be requested from the user when inserting a new field. Agent must prompt:  
            "Please provide a unique CDR UID (e.g., CDRXXXX) to associate with this new data requirement:"
        - Validate the row using `BOI_get_worksheet` or `BOI_read_workbook_data`
        - Confirm success before generating RAML
        - The RAML `description` for each field must always be copied verbatim from the `Common Data Requirement Definition` column in the CDR and enclosed in double quotes.

This ensures consistency, traceability, and controlled governance across all API design activities.

# Security Scheme Templates (Appendix)

## Basic Authentication
- When requested, generate:
  #%RAML 1.0 SecurityScheme
  type: Basic Authentication
  description: This security scheme uses basic authentication user ID and password to validate incoming requests.
  describedBy: 
    headers: 
      Authorization:
        type: string
        description: >
          Credentials used to authenticate the client. The username and the password are combined with a colon (username:password)
          and the resulting string is base64 encoded.
        example: Basic dXNlcm5hbWU6cGFzc3dvcmQ=

## Client ID Authentication
- When `client_id` is requested, generate:
  #%RAML 1.0 SecurityScheme
  type: x-custom
  description: >
    This security scheme ensures that the client credentials sent on each request have been approved to consume the API.
  describedBy: 
    headers:
      client_id:
        type: string
        description: The client_id required for authentication.
      client_secret:
        type: string
        description: The client_secret required for authentication.

## x-client-id Authentication
- When `x-client-id` is requested, generate:
  #%RAML 1.0 SecurityScheme
  type: x-custom
  description: >
    This security scheme ensures that the client credentials sent on each request have been approved to consume the API.
  describedBy: 
    headers:
      x-client-id:
        type: string
        description: The x-client-id required for authentication.
      x-client-secret:
        type: string
        description: The x-client-secret required for authentication.

## OAuth 2.0 Authentication
- When OAuth 2.0 is requested, generate:
  #%RAML 1.0 SecurityScheme
  type: OAuth 2.0
  description: This scheme validates requests using an OAuth 2.0 access token.
  describedBy:
    headers:
      Authorization:
        type: string
        required: true
        pattern: '^[B,b]earer .+$'
    responses:
      401:
        description: Invalid token.
      403:
        description: Access denied.
  settings:
    authorizationUri: https://www.tbd.com/1/oauth2/authorize
    accessTokenUri: https://api.tbd.com/1/oauth2/token
    authorizationGrants: [ authorization_code, implicit, urn:ietf:params:oauth:grant-type:saml2-bearer ]