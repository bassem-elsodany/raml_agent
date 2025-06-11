# Digital Banking RAML API Design with AI Agent and BOI-MCP Integration

This project demonstrates how to use an AI agent (GitHub Copilot) to automate RAML 1.0 API design activities, strictly governed by BOI-MCP (Bank of Ireland Model Context Protocol) canonical field validation and governance rules.

## Key Features
- **AI-driven RAML generation**: Use natural language prompts to generate RAML endpoints, types, and examples.
- **Canonical field validation**: All fields are validated against the BOI-MCP CDR (Common Data Requirements) Excel using the MCP toolset.
- **Governance enforcement**: The agent enforces naming, structure, and folder conventions as per the governance instructions.
- **Non-canonical field handling**: If a field is not found in the CDR, the agent suggests canonical alternatives or prompts for CDR insertion, ensuring traceability and compliance.

## How to Use the AI Agent for RAML Design
1. **Describe your API resource or endpoint in plain English.**
   - Example: "Create a POST /contacts endpoint. Use only CDR fields from Customer > Contact: emailAddress (required), smsNumber (optional)."
2. **The agent validates all fields against the CDR using the MCP tool.**
   - If a field is not canonical, the agent will:
     - Suggest alternatives from the CDR.
     - Ask if you want to insert the new field into the CDR (with definition and type).
3. **The agent generates RAML types, examples, and endpoint definitions.**
   - All types use canonical Concept > Context > Field nesting.
   - Descriptions are copied verbatim from the CDR definition column.
   - Example files are created in the correct folder structure.
4. **Error handling and governance are enforced.**
   - All error responses use a standard ErrorResponse type.
   - Naming, folder, and structural rules are automatically applied.

## Example Workflow
1. **User prompt:**
   ```
   Add mobileNumber under Customer > Contact in CreateContactRequest.
   If it’s not in BOI-MCP, suggest alternatives or ask to insert it before proceeding.
   ```
2. **Agent response:**
   - "mobileNumber is not in the CDR. Closest alternatives: smsNumber, homePhoneNumber, workPhoneNumber. Would you like to use one of these or insert mobileNumber into the CDR?"
3. **User reply:**
   - "Use smsNumber."
4. **Agent action:**
   - Adds smsNumber to the RAML type, using the CDR definition for the description.

## Real Integration with MCP Tool
- The agent uses the MCP toolset to:
  - Query the CDR Excel for canonical field validation.
  - Insert new fields into the CDR (with user approval).
  - Validate and confirm all changes before generating RAML.
- This ensures all API designs are traceable, governed, and ready for real-world integration.

## Folder Structure
- `dataTypes/types/{resource}/` — RAML type files (request/response)
- `examples/{resource}/` — Example files for each resource
- `traits/`, `securitySchemes/`, `changelog.raml` — Governance and API support files

## Governance
- See `.github/instructions/copilot-instructions.md` for full governance rules and agent behavior.

## Getting Started
- Open this workspace in VS Code with GitHub Copilot enabled.
- Use natural language prompts to drive RAML design.
- The agent will guide you through canonical validation and governance at every step.

---

## Guided Demo Prompts

This section demonstrates a realistic Copilot-powered RAML design flow using BOI-MCP governance — all within a single context (`Customer > Contact`), where the concept (Customer) serves as the root resource.

### ✅ Step 1: Create a Canonical Endpoint

**Prompt:**
```
Create a POST customers/{customer-id}/contacts endpoint using Customer > Contact fields from BOI-MCP.
Use existing CDR fields only: emailAddress, phoneNumber.
Name the request type CreateContactRequest.
Add response and example file.
```

### ✅ Step 2: Add a Missing Field and Trigger Fallback Logic

**Prompt:**
```
Now add a field mobileNumber to CreateContactRequest under Customer > Contact.
If not in CDR, propose alternatives.
If no match, insert into CDR before updating the RAML.
```

### 🔁 Expected Agent Behavior

- In Step 1: Agent generates a valid `CreateContactRequest` with canonical nesting and BOI-MCP-validated field descriptions.
- In Step 2:
  - Agent checks CDR for `mobileNumber` under Customer > Contact.
  - If missing, proposes similar fields like smsNumber or phoneNumber.
  - If user confirms, inserts mobileNumber into the CDR using:
    - Concept: Customer
    - Context: Contact
    - Data Requirement: mobileNumber
    - Long Name: Customer:Contact:mobileNumber
    - Definition: "Customer's mobile phone number in international format"
  - Then regenerates the RAML with the new field included properly.

Use these two minimal prompts to test canonical validation, CDR fallback, structural compliance, and agent reasoning in one clear scenario.

### 🧩 Step 3: Rename an Existing Field

**Prompt:**
```
Replace phoneNumber in CreateContactRequest with preferredCanonicalPhoneNumber.
Use the definition from the CDR.
```

- Agent validates the replacement field exists under Customer > Contact.
- If found, it replaces the field in the RAML and updates the description from the CDR.

### 🛠️ Step 4: Partially Update the Resource (PATCH)

**Prompt:**
```
Create a PATCH /contacts/{contactId} endpoint.
Allow updating only smsNumber and emailAddress under Customer > Contact.
Use canonical field validation.
```

- Agent creates a PATCH endpoint with a partial request body using optional fields.
- Ensures nesting and descriptions are retained per BOI-MCP rules.

### 🧪 Step 5: Submit and Validate Full Contact

**Prompt:**
```
Create a POST /contacts/validate endpoint to validate all Customer > Contact fields.
Reuse existing types and add a validationResult field to the response.
```

- Agent reuses canonical types and includes a validation response using ErrorResponse and validationResult.
