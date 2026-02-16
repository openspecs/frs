# FullStack Requirement Standard (FRS)

**Version 1.0.3** | January 2026

| | |
|---|---|
| **Status** | Working Draft |
| **Editors** | Love Lundquist @ Decerno AB |
| **License** | CC0 1.0 Universal (Public Domain) |
| **Based On** | Gherkin BDD, YAML frontmatter, OpenAPI Specification |

## Abstract

The FullStack Requirement Standard (FRS) is a lightweight, human-readable, and machine-parseable format for documenting software requirements. FRS combines YAML frontmatter for metadata, numbered flow steps for implementation priority, and optional technical specifications. The standard is designed to be executable by both human developers and AI agents, enabling seamless translation from requirements to implementation, testing, and documentation.

## 1. Introduction

### 1.1 Purpose

Modern software development faces challenges in translating requirements across stakeholders—product managers, developers, UX designers, QA engineers, and AI agents. Existing standards (Gherkin, OpenAPI, user stories) address specific domains but lack a unified format covering user context, business outcomes, technical implementation, and priority flow. FRS addresses this gap by providing a single, extensible format for full-stack requirements.

### 1.2 Design Goals

- **Human readability**: Plain text format readable without specialized tools
- **Machine parseability**: Structured YAML and flow syntax for automated tooling
- **Implementation priority**: Numbered steps create vertical happy path
- **Alternative flows**: Indented dash syntax for error handling and edge cases
- **Extensibility**: Optional fields for technical specifications as needed
- **Version control**: Git-friendly plain text format

### 1.3 Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 2. File Format Specification

### 2.1 File Extension

FRS files MUST use the `.frs` extension. Files are encoded in UTF-8 without BOM.

### 2.2 File Naming Convention

File names SHOULD follow the pattern: `{module}-{action}.frs`

**Examples:** `credit-validate-ltv.frs`, `auth-login.frs`, `report-generate.frs`

### 2.3 Document Structure

An FRS document consists of two sections:

1. **YAML frontmatter** (REQUIRED) - metadata enclosed in triple dashes (`---`)
2. **Flow specification** (REQUIRED) - numbered steps with optional alternatives and technical details

## 3. YAML Frontmatter

### 3.1 Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| `id` | Unique identifier for the requirement | `AUTH-001`, `CREDIT-LTV-001` |
| `user` | Role or actor performing the action | `Analyst`, `Manager`, `System` |
| `context` | Location or state where the user begins | `Login page`, `Dashboard` |
| `trigger` | Event or action that initiates the flow | `Clicks Submit`, `Opens report` |
| `user_outcome` | What the user achieves | `Access dashboard in 5 seconds` |

### 3.2 Optional Fields

| Field | Description | Example |
|-------|-------------|---------|
| `business_outcome` | Measurable business value | `Reduce default rate by 15%` |
| `priority` | Importance level | `critical` \| `high` \| `medium` \| `low` |
| `status` | Current state | `draft` \| `approved` \| `implemented` |
| `estimate` | Time estimate | `3d`, `2w` |
| `depends_on` | Array of requirement IDs | `[AUTH-001, USER-002]` |
| `tags` | Array of categorization labels | `[security, mvp]` |

## 4. Flow Specification

### 4.1 Flow Structure

The Flow section describes the sequence of actions and outcomes. It MUST begin with the keyword `Flow:` followed by numbered steps.

Numbered steps represent the happy path—the primary success scenario. Each numbered step SHOULD represent one clear action or outcome.

### 4.2 Alternative Paths

Alternative paths (error handling, edge cases, conditional branches) are expressed using indented lines beginning with a dash (`-`).

Alternative paths MUST be indented with 3 spaces and MUST begin with a conditional phrase (e.g., "If", "When", "On error").

```
2. Check credentials against database
   - If mismatch: Show "Invalid credentials"
   - If 5+ failures: Lock account for 15 mins
```

### 4.3 Optional Technical Sections

After the Flow, documents MAY include optional technical sections:

| Section | Purpose | Example |
|---------|---------|---------|
| `API:` | Endpoint signatures | `POST /api/auth {email, password} → {token}` |
| `Performance:` | Speed requirements | `<500ms response` |
| `Security:` | Auth requirements | `Rate limit 5 attempts/minute` |
| `Data:` | Storage requirements | `Retention: 90 days` |
| `Rule:` | Business constraints | `LTV must be < 80%` |

## 5. Complete Example

```yaml
---
id: AUTH-001
user: Analyst
context: Login page
trigger: Submits credentials
user_outcome: Access dashboard in under 5 seconds
business_outcome: Reduce password reset tickets by 40%
---

Flow:
1. Validate email and password presence
   - If empty: Show "Required field" error
2. Check credentials against database
   - If mismatch: Show "Invalid credentials"
   - If 5+ failures: Lock account for 15 mins
3. Create session token (JWT, expires 8h)
4. Redirect to /dashboard

API: POST /auth/login {email, password} → {token, user}
Security: Rate limit 5 attempts/minute per IP
```

## 6. Implementation Guidelines for Humans

### 6.1 Reading FRS Documents

1. Read the YAML frontmatter to understand who, what, where, when, and why
2. Scan numbered Flow steps to see the happy path (read vertically 1-2-3-4-5)
3. Review indented alternatives for error handling and edge cases
4. Reference optional technical sections for implementation constraints

### 6.2 Implementation Priority

Numbered steps define implementation priority. Developers SHOULD implement steps sequentially, ensuring each step works before proceeding to the next.

Alternative paths MAY be implemented after the happy path is functional, or in parallel if resources permit.

### 6.3 Testing Strategy

- Each numbered step corresponds to one happy path test case
- Each dash alternative corresponds to one edge case or error handling test
- User and business outcomes define acceptance criteria

## 7. Implementation Guidelines for AI Agents

### 7.1 Parsing FRS Documents

AI agents MUST parse FRS documents as follows:

1. Extract YAML frontmatter (between `---` delimiters) and parse as YAML
2. Identify `Flow:` keyword and parse numbered steps (regex: `^\d+\.`)
3. Parse alternative paths (lines starting with spaces + dash: `^\s+-`)
4. Extract optional technical sections (`API:`, `Performance:`, etc.)

### 7.2 Code Generation Strategy

When generating code from FRS documents:

- Generate functions/methods for each numbered Flow step
- Implement alternative paths as conditional branches (if/else, try/catch)
- Use `API:` section to generate endpoint signatures and data contracts
- Apply `Performance:` constraints as implementation requirements
- Implement `Security:` and `Rule:` sections as validation logic

### 7.3 Test Generation

AI agents SHOULD generate tests as follows:

- One happy path test per numbered Flow step
- One edge case test per alternative path (indented dash lines)
- Acceptance tests validating `user_outcome` and `business_outcome`
- Performance tests based on `Performance:` section

### 7.4 Context Understanding

The `context` field provides starting state information. AI agents SHOULD use this to understand prerequisites, UI state, or system configuration before the trigger event occurs. Context is descriptive, not prescriptive—it indicates where the user begins, not technical implementation details.

## 8. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.3 | January 2026 | Moved `business_outcome` field to optional field |
| 1.0.2 | December 2025 | Added `context` field, formalized flow syntax with numbered steps, introduced dash-indented alternative paths |
| 1.0.1 | — | Added `user_outcome` and `business_outcome` required fields |
| 1.0.0 | — | Initial release with core specification |

## 9. References

- [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs to Indicate Requirement Levels
- [Gherkin Syntax Reference](https://cucumber.io/docs/gherkin/) - Cucumber BDD specification language
- [OpenAPI Specification v3.x](https://spec.openapis.org/oas/latest.html) - REST API description standard
- [YAML Version 1.2](https://yaml.org/spec/1.2.2/)

---

*© Decerno AB - Released under CC0 1.0 Universal (Public Domain)*
