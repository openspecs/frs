# Fullstack Requirement Specification (FRS)

**Version 1.1.0** | February 2026

|  |  |
| --- | --- |
| **Status** | Working Draft |
| **Editors** | Love Lundquist @ Decerno AB |
| **License** | CC0 1.0 Universal (Public Domain) |
| **Based On** | Gherkin BDD, YAML frontmatter, OpenAPI Specification |

## Abstract

The Fullstack Requirement Specification (FRS) is a lightweight, human-readable, and machine-parseable format for documenting software requirements. FRS combines YAML frontmatter for metadata, numbered flow steps for implementation priority, and optional technical specifications. The standard is designed to be executable by both human developers and AI agents, enabling seamless translation from requirements to implementation, testing, and documentation.

## 1. Introduction

### 1.1 Purpose

Modern software development faces challenges in translating requirements across stakeholders—product managers, developers, UX designers, QA engineers, and AI agents. Existing specifications (Gherkin, OpenAPI, user stories) address specific domains but lack a unified format covering user context, business outcomes, technical implementation, and priority flow. FRS addresses this gap by providing a single, extensible format for full-stack requirements.

### 1.2 Design Goals

* **Human readability**: Plain text format readable without specialized tools
* **Machine parseability**: Structured YAML and flow syntax for automated tooling and validation
* **Implementation priority**: Numbered steps create vertical happy path
* **Alternative flows**: Indented dash syntax for error handling and edge cases
* **Verifiable intent**: Inline validation criteria enable closed-loop testing without implementation knowledge
* **Extensibility**: Optional fields for technical specifications as needed
* **Version control**: Git-friendly plain text format

### 1.3 Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 2. File Format Specification

### 2.1 File Extension

FRS files SHOULD use the `.md` extension. Files are encoded in UTF-8 without BOM.

### 2.2 File Naming Convention

File names SHOULD follow the pattern: `{module}-{action}.md`

**Examples:** `credit-validate-ltv.md`, `auth-login.md`, `report-generate.md`

### 2.3 Document Structure

An FRS document consists of two sections:

1. **YAML frontmatter** (REQUIRED) - metadata enclosed in triple dashes (`---`)
2. **Flow specification** (REQUIRED) - numbered steps with optional alternatives, technical details, and validation criteria

## 3. YAML Frontmatter

### 3.1 Required Fields

| Field | Description | Example |
| --- | --- | --- |
| `id` | Unique identifier for the requirement | `AUTH-001`, `CREDIT-LTV-001` |
| `user` | Role or actor performing the action | `Analyst`, `Manager`, `System` |
| `context` | Location or state where the user begins | `Login page`, `Dashboard` |
| `trigger` | Event or action that initiates the flow | `Clicks Submit`, `Opens report` |
| `user_outcome` | What the user achieves | `Access dashboard in 5 seconds` |

### 3.2 Optional Fields

| Field | Description | Example |
| --- | --- | --- |
| `business_outcome` | Measurable business value | `Reduce default rate by 15%` |
| `priority` | Importance level | `critical` | `high` | `medium` | `low` |
| `status` | Current state | `draft` | `approved` | `implemented` |
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
| --- | --- | --- |
| `API:` | Endpoint signatures | `POST /api/auth {email, password} → {token}` |
| `Performance:` | Speed requirements | `<500ms response` |
| `Security:` | Auth requirements | `Rate limit 5 attempts/minute` |
| `Data:` | Storage requirements | `Retention: 90 days` |
| `Rule:` | Business constraints | `LTV must be < 80%` |

### 4.4 Validate Section

After the Flow and optional technical sections, documents MAY include a `Validate:` section. The Validate section defines machine-executable verification criteria that form a closed loop with the Flow specification. Together, the Flow and Validate sections enable any implementation—human or AI-generated—to be verified against the original intent without knowledge of the underlying code.

The Validate section supports four subsections:

| Subsection | Purpose | Derived From |
| --- | --- | --- |
| `happy_path` | Golden test cases for the primary success scenario | Numbered Flow steps |
| `boundaries` | Edge case test cases at threshold values | Alternative paths (dash lines) |
| `invariants` | Conditions that must always hold true regardless of input | Business rules and domain constraints |
| `contracts` | Mathematical or logical relationships between inputs and outputs | Domain logic |

#### 4.4.1 happy_path

The `happy_path` subsection defines input/output pairs that validate the numbered Flow steps. There SHOULD be at least one test case per numbered Flow step that produces a verifiable output.

Each test case consists of:
- `input`: A set of key-value pairs representing the input state
- `expect`: A set of key-value pairs representing the expected output

```
Validate:
  happy_path:
    - input: {email: "user@example.com", password: "valid123"}
    - expect: {status: 200, body: {token: "non-empty", user: "non-empty"}}
```

#### 4.4.2 boundaries

The `boundaries` subsection defines test cases at the edges of acceptable input ranges. These SHOULD be derived directly from the alternative paths (dash lines) in the Flow section. Boundary tests verify that threshold logic and error handling behave correctly at critical values.

```
Validate:
  boundaries:
    - input: {attempts: 5}
    - expect: {locked: true, message: "Account locked for 15 mins"}
    - input: {attempts: 4}
    - expect: {locked: false}
```

#### 4.4.3 invariants

The `invariants` subsection defines conditions that MUST hold true for all possible inputs. Invariants are expressed as natural language statements that can be translated into property-based tests or runtime assertions. An invariant violation at any point indicates a system failure.

Invariants MAY span multiple FRS files when prefixed with the dependent requirement ID.

```
Validate:
  invariants:
    - "Session token must never be returned on failed authentication"
    - "Locked accounts must reject all login attempts"
    - "CREDIT-LTV-001: risk_level must never be null"
```

#### 4.4.4 contracts

The `contracts` subsection defines mathematical or logical relationships between inputs and outputs. Contracts express the domain logic that any correct implementation must satisfy, regardless of how it is implemented. Tolerance values SHOULD be specified where floating-point arithmetic is involved.

```
Validate:
  contracts:
    - "Output token expiry must equal current_time + 8 hours ± 5 seconds"
```

#### 4.4.5 Relationship Between Flow and Validate

The Validate section does not replace the Flow—it completes it. The Flow defines *what should happen*, and Validate defines *how to prove it happened correctly*. Together they form a single specification of intent that is both human-readable and machine-verifiable.

| Flow Element | Validates With |
| --- | --- |
| Numbered steps (happy path) | `happy_path` test cases |
| Dash alternatives (edge cases) | `boundaries` test cases |
| `Rule:` sections | `invariants` and `contracts` |
| `Performance:` sections | `happy_path` with timing assertions |

## 5. Complete Example

```
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
Performance: <500ms response

Validate:
  happy_path:
    - input: {email: "analyst@company.com", password: "valid123"}
    - expect: {status: 200, token: "non-empty", redirect: "/dashboard"}
  boundaries:
    - input: {email: "", password: "valid123"}
    - expect: {status: 400, error: "Required field"}
    - input: {email: "user@company.com", password: "wrong", attempts: 5}
    - expect: {status: 429, locked: true}
    - input: {email: "user@company.com", password: "wrong", attempts: 4}
    - expect: {status: 401, locked: false}
  invariants:
    - "Session token must never be returned on failed authentication"
    - "Locked accounts must reject all login attempts regardless of correct password"
    - "Error messages must not reveal whether email exists in system"
  contracts:
    - "Output token expiry must equal current_time + 8 hours ± 5 seconds"
    - "Response time must be < 500ms for all authentication attempts"
```

## 6. Implementation Guidelines for Humans

### 6.1 Reading FRS Documents

1. Read the YAML frontmatter to understand who, what, where, when, and why
2. Scan numbered Flow steps to see the happy path (read vertically 1-2-3-4-5)
3. Review indented alternatives for error handling and edge cases
4. Reference optional technical sections for implementation constraints
5. Review the Validate section to understand acceptance criteria and edge case expectations

### 6.2 Implementation Priority

Numbered steps define implementation priority. Developers SHOULD implement steps sequentially, ensuring each step works before proceeding to the next.

Alternative paths MAY be implemented after the happy path is functional, or in parallel if resources permit.

### 6.3 Testing Strategy

* Each numbered step corresponds to one happy path test case
* Each dash alternative corresponds to one edge case or error handling test
* User and business outcomes define acceptance criteria
* `Validate: happy_path` cases define minimum passing test suite
* `Validate: boundaries` cases define edge case coverage
* `Validate: invariants` define property-based tests or runtime assertions that must always hold
* `Validate: contracts` define mathematical verification of domain logic

## 7. Implementation Guidelines for AI Agents

### 7.1 Parsing FRS Documents

AI agents MUST parse FRS documents as follows:

1. Extract YAML frontmatter (between `---` delimiters) and parse as YAML
2. Identify `Flow:` keyword and parse numbered steps (regex: `^\d+\.`)
3. Parse alternative paths (lines starting with spaces + dash: `^\s+-`)
4. Extract optional technical sections (`API:`, `Performance:`, etc.)
5. Parse `Validate:` section and its subsections (`happy_path`, `boundaries`, `invariants`, `contracts`)

### 7.2 Code Generation Strategy

When generating code from FRS documents:

* Generate functions/methods for each numbered Flow step
* Implement alternative paths as conditional branches (if/else, try/catch)
* Use `API:` section to generate endpoint signatures and data contracts
* Apply `Performance:` constraints as implementation requirements
* Implement `Security:` and `Rule:` sections as validation logic

### 7.3 Test Generation

AI agents SHOULD generate tests as follows:

* One happy path test per numbered Flow step
* One edge case test per alternative path (indented dash lines)
* Acceptance tests validating `user_outcome`
* Performance tests based on `Performance:` section
* Direct test cases from `Validate: happy_path` and `Validate: boundaries` as executable test fixtures
* Property-based tests from `Validate: invariants` using randomized input generation
* Assertion functions from `Validate: contracts` applied to all test outputs

### 7.4 Validation Loop

AI agents SHOULD use the Validate section to verify generated implementations:

1. Generate implementation from Flow specification
2. Execute `happy_path` test cases against the implementation
3. Execute `boundaries` test cases against the implementation
4. Verify all `invariants` hold across a representative input set
5. Verify all `contracts` produce correct input-output relationships
6. If any validation fails, regenerate or fix the implementation and repeat

This creates a closed loop where the FRS document serves as both the specification and the acceptance gate, enabling autonomous implementation cycles without human review of generated code.

### 7.5 Context Understanding

The `context` field provides starting state information. AI agents SHOULD use this to understand prerequisites, UI state, or system configuration before the trigger event occurs. Context is descriptive, not prescriptive—it indicates where the user begins, not technical implementation details.

## 8. Version History

| Version | Date | Changes |
| --- | --- | --- |
| 1.1.0 | February 2026 | Added `Validate:` section (4.4) with `happy_path`, `boundaries`, `invariants`, and `contracts` subsections. Added validation loop for AI agents (7.4). Updated design goals, document structure, testing strategy, and complete example. |
| 1.0.5 | February 2026 | Renamed and updated wording from Fullstack Requirement Standard to Fullstack Requirement Specification |
| 1.0.4 | February 2026 | Removed `business_outcome` field from acceptance testing, relaxing the requirement for file format as SHOULD and to \*.md, change to 1.2 Design Goals -> Machine parseability |
| 1.0.3 | January 2026 | Moved `business_outcome` field to optional field |
| 1.0.2 | December 2025 | Added `context` field, formalized flow syntax with numbered steps, introduced dash-indented alternative paths |
| 1.0.1 | — | Added `user_outcome` and `business_outcome` required fields |
| 1.0.0 | — | Initial release with core specification |

## 9. References

* [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) - Key words for use in RFCs to Indicate Requirement Levels
* [Gherkin Syntax Reference](https://cucumber.io/docs/gherkin/) - Cucumber BDD specification language
* [OpenAPI Specification v3.x](https://spec.openapis.org/oas/latest.html) - REST API description standard
* [YAML Version 1.2](https://yaml.org/spec/1.2.2/)
* [Design by Contract](https://en.wikipedia.org/wiki/Design_by_contract) - Bertrand Meyer's contract-based design methodology
* [Property-based Testing](https://hypothesis.readthedocs.io/) - Generative testing approach for invariant verification

---

*© Decerno AB - Released under CC0 1.0 Universal (Public Domain)*
