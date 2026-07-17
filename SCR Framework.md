# The Core Extension: [Comment Anchors](https://marketplace.visualstudio.com/items?itemName=ExodiusStudios.comment-anchors)

This extension turns code comments into interactive, high-visibility UI elements.

## The Three Pillars of Configuration

| Component      | Options                       | Purpose                                                            |
| -------------- | ----------------------------- | ------------------------------------------------------------------ |
| **Behaviors**  | `anchor`, `region`, `link`    | Defines if the tag is a bookmark, a folder, or a portal.           |
| **Scopes**     | `workspace`, `file`, `hidden` | Controls if the tag appears globally or only in the current file.  |
| **Attributes** | `epic`, `seq`, `id`           | The metadata inside `[]` used for grouping, ordering, and linking. |

## Attributes & Organization

| Attribute  | Purpose                                                                                                    | Example Usage          |
| ---------- | ---------------------------------------------------------------------------------------------------------- | ---------------------- |
| **`epic`** | **Folder Grouping**: Creates a folder in the sidebar for a specific project or vulnerability class.        | `[epic=Auth_Bypass]`   |
| **`seq`**  | **Sorting/Priority**: Orders anchors numerically (1, 2, 3). Used for priority (1=Critical) or audit steps. | `[seq=1]`              |
| **`id`**   | **Link Target**: Gives a specific line a unique name so it can be "teleported" to from elsewhere.          | `[id=token_input]`     |

---

# The Framework

This is the methodology for how you use the tags to breakdown code, identify behavior and hunt for bugs.

## The 6 Security Layers

| Layer        | Purpose                                         | Pattern                                 | Type        |
| ------------ | ----------------------------------------------- | --------------------------------------- | ----------- |
| Data         | Track data flow from **Entry** to **Execution** | Origin vs. Destination                  | Directional |
| Logic        | Shows **Intent** vs. **Execution**              | What-Should-Happen vs. What-Does-Happen | Abstract    |
| Analysis     | Provides **Context** and **Conclusion**         | Hypothesis vs. Confirmation             | Zoom        |
| Organization | Captures **Certainty** and **Uncertainty**      | Statement vs. Question                  | Certainty   |
| Structural   | Define **Structure** and **Relationships**      | Space vs. Connection                    | Arrangement |
| Metadata     | Document **Patterns** and **Instances**         | Pattern vs. Instance                    | Declarative |

| Layer    | Tag 1 (Abstract/General) | Tag 2 (Concrete/Specific) | Dynamic       |
| -------- | ------------------------ | ------------------------- | ------------- |
| Data     | SOURCE (entry point)     | SINK (execution point)    | Flow          |
| Logic    | LOGIC (rule)             | FLOW (steps)              | Realization   |
| Analysis | AREA (surface)           | BUG (finding)             | Refinement    |
| Org      | AUDIT (statement)        | REVIEW (question)         | Dialectic     |
| Struct   | SECTION (boundary)       | LINK (connection)         | Topology      |
| Metadata | RELATION (pattern)       | INSTANCE (specific path)  | Instantiation |

## The 12 Tags

| **Layer**    | **Tag**        | **Behavior** | Scope     |                                                                   |
| ------------ | -------------- | ------------ | --------- | ----------------------------------------------------------------- |
| Data         | `SEC-SOURCE`   | `anchor`     | Workspace | **Entry**: Untrusted data input points                            |
| Data         | `SEC-SINK`     | `anchor`     | Workspace | **Danger**: Functions where data is executed/stored               |
| Logic        | `SEC-LOGIC`    | `anchor`     | Workspace | **Rules**: Intended business logic and constraints                |
| Logic        | `SEC-FLOW`     | `anchor`     | Workspace | **Path**: The step-by-step code execution flow                    |
| Analysis     | `SEC-AREA`     | `anchor`     | Workspace | **Surface**: Sensitive logic (Auth/Crypto) requiring review       |
| Analysis     | `SEC-BUG`      | `anchor`     | Workspace | **Finding**: Actual vulnerabilities or confirmed bugs             |
| Organization | `SEC-AUDIT`    | `anchor`     | Workspace | **Context**: High-level component metadata and audit scope        |
| Organization | `SEC-REVIEW`   | `anchor`     | File      | **Open Items**: Questions for a second pass                       |
| Structural   | `SEC-SECTION`  | `region`     | File      | **Breakdown**: Break a single, massive file into digestible parts |
| Structural   | `SEC-LINK`     | `link`       | Hidden    | **Relate**: Teleport between related anchors                      |
| Metadata     | `SEC-RELATION` | `anchor`     | Workspace | **Pattern**: Relationship pattern type                            |
| Metadata     | `SEC-INSTANCE` | `anchor`     | Workspace | **Occurrence**: Specific instance of relationship pattern         |

## The Behavior Rulebook

1. **Markers (`SEC-*`)** = `behavior: "anchor"` (These are the "What").
2. **Containers (`SEC-SECTION`)** = `behavior: "region"` (These are the "Where").
3. **Connectors (`SEC-LINK`)** = `behavior: "link"` (These are the "How they relate").

## The Data Layer (Flow Direction)

- **SEC-SOURCE**: Identify where data enters (Request bodies, Headers, Env variables).
- **SEC-SINK**: Identify where data is executed (SQL queries, Shell commands, DOM writes).

## The Logic Layer (Business Rules)

- **SEC-LOGIC**: Identify the intended business rules and what the code _should_ be doing (e.g., "User cannot transfer more than balance").
- **SEC-FLOW**: Map the execution steps to identify what the code *actually* does and spot where a step is missing (1. Get user -> 2. Check balance -> 3. Unauthorized Deduction).

## The Analysis Layer (Attack Surface)

- **SEC-AREA**: Identify "High-Value Logic" or "Attack Surface", it's an "Area" of interest (e.g., encryption routines, auth checks).
- **SEC-BUG**: Track the attacks and vulnerabilities you suspect when the code deviates from the Logic or Flow.

## The Organizational Layer (Information Context)

- **SEC-AUDIT**: Provides context for the entire file, module or project (e.g., "Auditing JWT implementation").
- **SEC-REVIEW**: Temporary markers for questions or logic you don't understand yet; usually scoped to the `file` level.

## The Structural Layer (Spatial Layout)

- **SEC-SECTION**: Defines boundaries of a specific audit task so that tags are organized by the component they belong to (e.g., "Audit of the Login Function").
- **SEC-LINK**: Track dependencies, common assumptions, related logic, data flow etc.

## The Metadata Layer (Pattern Documentation)

- **SEC-RELATION**: Documents the relationship pattern type connecting related anchors (e.g., "This is a Data Flow pattern").
- **SEC-INSTANCE**: Documents a specific instance or occurrence of a relationship pattern.

## Attribute Logic 

- **Epic**: Group everything by feature, relationship or vulnerability class you are auditing at that moment (e.g., `[epic=Auth]`, `[epic=XSS]`).
- **Seq**: Use this to assign Priority, Severity or Chronological (e.g.,1 = Critical, 5 = Low).

---

# Link Relationship Patterns

## Data Flow (Source → Sink)

**Scenario**: Trace how untrusted user input reaches dangerous execution points.

**Pattern**:

```python
# SEC-SOURCE[epic=SQLi, seq=1, id=user_input] 🟢 Source: Username from request
username = request.json['username']

# ... processing ...

# SEC-SINK[epic=SQLi, seq=2] 🔴 Sink: Direct SQL execution
db.execute(f"SELECT * FROM users WHERE name='{username}'")
# SEC-LINK: [epic: Data_Flow] 🔗 Connector to #user_input
```

**Use Cases**: 

- Identify complete taint flow paths
- Generate exploit PoC chains
- Validate that all user inputs are tracked

## Mirroring (Logic ↔ Logic)

**Scenario**: Ensure cryptographic pairs, state transitions, or symmetric operations remain consistent.

**Pattern**:

```python
# SEC-LOGIC[epic=Auth, id=jwt_signing] ⭐ Business Logic: JWT signing with HS256
token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
# SEC-LINK: [epic: Mirroring] 🔗 Connector to #jwt_verify

# ... 200 lines later ...

# SEC-LOGIC[epic=Auth, id=jwt_verify] ⭐ Business Logic: JWT verification with HS256
payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
# SEC-LINK: [epic: Mirroring] 🔗 Connector to #jwt_signing
```

**Use Cases**:

- Crypto key mismatches (encrypt with AES-256, decrypt with AES-128)
- State machine bugs (lock acquired but never released)
- Session management (create session but forget to destroy)

## Lifecycle (Flow → Flow)

**Scenario**: Map sequential steps in a state machine, algorithm, or process flow.

**Pattern**:

```python
# SEC-FLOW[epic=Payment, seq=1, id=step1] 🪜 Step: Set order to pending
order.status = 'pending'
send_to_payment_gateway(order)

# SEC-FLOW[epic=Payment, seq=2, id=step2] 🪜 Step: Fetch payment status
payment_result = gateway.get_result()
if payment_result.success:
    # SEC-FLOW[epic=Payment, seq=3, id=step3] 🪜 Step: Set order to paid
    order.status = 'paid'
    trigger_shipment(order)
```

**Use Cases**:

- Identify missing authorization checks in multi-step workflows
- Spot TOCTOU (Time-Of-Check-Time-Of-Use) bugs
- Verify state transitions follow business rules 

## Parent (Section → \*)

**Scenario**: Organize audit workspace by grouping related tags under sections, even when code is split across files.

**Pattern**:

```python
# auth.py
# SEC-SECTION: 📁 Authentication Module
# SEC-AUDIT[epic=Auth, seq=1] 📋 Context: All login/logout logic

# SEC-SOURCE[epic=Auth, seq=1, id=login_creds] 🟢 Source: User credentials
credentials = request.json

# ... more code ...

# !SEC-SECTION
```

**Use Cases**:

- Organize large audits by feature (all Auth code in one section)
- Cross-file context (frontend validation + backend validation in same section)
- Sprint planning (section = "Sprint 3 Review")

## Reference (Area → Standard)

**Scenario**: Link SEC-AREA tags to specific OWASP categories.

**Pattern**:

```python
# SEC-AREA[epic=OWASP_A01, id=auth_logic] ⚠️ Attack Surface: Broken Access Control check
def check_permissions(user):
    # ...
```

**Use cases**: 

- Generate compliance reports ("We reviewed 5 A01 areas, 3 A03 areas...")
- Prioritize audit work by OWASP severity rankings
- Map findings to regulatory requirements (PCI-DSS, SOC2, HIPAA)

## Validation (Logic → Flow)

**Scenario**: Link a business rule to the code that enforces it.

**Pattern**:

```python
# SEC-LOGIC[epic=Auth, id=rule_admin_only] ⭐ Business Logic: Only admins can delete users

# ... 100 lines later ...

# SEC-FLOW[epic=Auth, seq=1] 🪜 Step: Admin permission check
if current_user.role != 'admin':
    raise Forbidden
# SEC-LINK: [epic: Validation] 🔗 Connector to #rule_admin_only
```

**Use cases**: 

- Verify that every `SEC-LOGIC` has a corresponding enforcement point.
- Identify business rules that are documented but not enforced.
- Detect logic bypasses where validation is missing.

## Exploit Chain (Bug → Multiple Tags)

**Scenario**: SEC-BUG links to all tags involved in the vulnerability.

**Pattern**:

```python
# SEC-SOURCE[epic=RCE, seq=1, id=user_input] 🟢 Source: Command from request
cmd = request.json['command']

# SEC-FLOW[epic=RCE, seq=1, id=no_validation] 🪜 Step: MISSING validation

# SEC-SINK[epic=RCE, seq=2, id=exec_cmd] 🔴 Sink: OS command execution
os.system(cmd)

# SEC-BUG[epic=RCE, seq=1, id=rce_finding] ☠️ Finding: Command injection - no sanitization
# SEC-LINK: [epic: Exploit Chain] 🔗 Connector to #user_input
# SEC-LINK: [epic: Exploit Chain] 🔗 Connector to #no_validation
# SEC-LINK: [epic: Exploit Chain] 🔗 Connector to #exec_cmd
```

**Use cases**: 

- Generate exploit PoC by following the chain
- Demonstrate impact to stakeholders with end-to-end attack paths
- Prioritize fixes based on exploitability

## Remediation Pair (Bug → Fix Suggestion)

**Scenario**: Link vulnerability to proposed fix location.

**Pattern**:

```python
# SEC-BUG[epic=SQLi, seq=1, id=sqli_vuln] ☠️ Finding: SQL Injection in login function

# SEC-REVIEW[epic=SQLi] 🔍 Note: Use parameterized query - db.execute("SELECT * WHERE user=?", (username,))
# SEC-LINK: [epic: Remediation] 🔗 Connector to #sqli_vuln
```

**Use cases**: 

- Track which vulnerabilities have proposed fixes
- Accelerate remediation by providing code-level guidance
- Measure remediation coverage in sprint retrospectives

## Assumption Dependency (Cross-File Logic)

**Scenario**: Code in File A assumes behavior in File B.

**Pattern**:

```python
# auth.py
# SEC-LOGIC[epic=Auth, id=token_signed_with_secret] ⭐ Business Logic: JWT tokens signed with SERVER_SECRET

# config.py
# SEC-AREA[epic=Config, id=secret_config] ⚠️ Attack Surface: SERVER_SECRET config - JWT dependency
SERVER_SECRET = os.getenv('SECRET_KEY')
# SEC-LINK: [epic: Assumption] 🔗 Connector to #token_signed_with_secret
```

**Use cases**: 

- Detect breaking changes when refactoring
- Document implicit contracts between modules
- Identify cascading failures when configuration changes

## Alternative Path (Flow Branching)

**Scenario**: Document if/else branches in execution flow.

**Pattern**:

```python
# SEC-FLOW[epic=Auth, seq=1, id=auth_flow] 🪜 Step: Token existence check
if token:
    # SEC-FLOW[epic=Auth, seq=2, id=valid_path] 🪜 Step: Happy path - validate token
    validate_token(token)
    # SEC-LINK: [epic: AltPath] 🔗 Connector to #auth_flow
else:
    # SEC-FLOW[epic=Auth, seq=3, id=invalid_path] 🪜 Step: Error path - reject request
    return 401
    # SEC-LINK: [epic: AltPath] 🔗 Connector to #auth_flow
```

**Use cases**: 

- Ensure all paths are audited (no "forgot about the else" bugs)
- Identify missing error handling in edge cases
- Verify authorization checks exist in both happy and error paths

## Composition (INSTANCE → Sub-Instances)

**Scenario**: A complex instance is composed of multiple sub-instances (phases).

**Pattern**:

```python
# SEC-RELATION[epic=Exploit_Chain] 🔀 Pattern: Multi-step attack chain

# SEC-INSTANCE[epic=Exploit_Chain, id=full_exploit, seq=1] 🛤️ Instance: Complete exploit

# Phase 1: Input
# SEC-INSTANCE[epic=Data_Flow, id=phase1, seq=1] 🛤️ Instance: Input injection
# SEC-LINK: 🔗 Connector to #full_exploit
# SEC-SOURCE[epic=RCE, id=cmd] 🟢 Source: Command parameter
cmd = request.args.get('command')

# Phase 2: Bypass
# SEC-INSTANCE[epic=Validation, id=phase2, seq=2] 🛤️ Instance: Validation bypass
# SEC-LINK: 🔗 Connector to #full_exploit
# SEC-FLOW[epic=RCE, seq=1] 🪜 Step: Weak filter (only checks 'rm')
if 'rm' not in cmd: pass

# Phase 3: Execute
# SEC-INSTANCE[epic=Data_Flow, id=phase3, seq=3] 🛤️ Instance: Execution
# SEC-LINK: 🔗 Connector to #full_exploit
# SEC-SINK[epic=RCE, seq=2] 🔴 Sink: Shell execution
os.system(cmd)
```

**Use Cases**:

- Break complex exploits into manageable phases
- Document hierarchical instance structure
- Show how sub-instances combine into complete attack chains

## Intersection (RELATION + RELATION → Combined Pattern)

**Scenario**: Two or more relationship patterns occur simultaneously, creating a compound pattern.

**Pattern**:

```python
# SEC-RELATION[epic=Data_Flow, seq=1, id=df] 🔀 Pattern: Data flow
# SEC-RELATION[epic=Validation, seq=2, id=val] 🔀 Pattern: Validation

# Combined pattern
# SEC-RELATION[epic=Secure_Data_Flow, seq=3] 🔀 Pattern: Data Flow + Validation (intersection)
# SEC-LINK: 🔗 Connector to #df
# SEC-LINK: 🔗 Connector to #val

# SEC-INSTANCE[epic=Secure_Data_Flow, id=safe_login] 🛤️ Instance: Validated login flow

# Data flow component
# SEC-SOURCE[epic=SQLi, seq=1, id=input] 🟢 Source: User input
username = request.json['username']

# Validation component
# SEC-LOGIC[epic=Validation, id=rule] ⭐ Business Logic: Sanitize input
# SEC-FLOW[epic=Validation, seq=1] 🪜 Step: Apply sanitization
username = sanitize(username)
# SEC-LINK: 🔗 Connector to #rule

# Data flow continues
# SEC-SINK[epic=SQLi, seq=2] 🔴 Sink: Safe query
db.execute("SELECT * FROM users WHERE name=?", (username,))
```

**Use Cases**:

- Document compound security patterns (secure data flow = data flow + validation).
- Distinguish pattern variants (validated vs. unvalidated data flow).
- Explain exploit chains (vulnerability = pattern combination with missing component).

---

[Global Settings](https://github.com/vizkov/oss/blob/main/Global%20Settings.md)

---

[Usage Guide](https://github.com/vizkov/oss/blob/main/Usage%20Guide.md)

---

[Framework Coverage & Enforcement](https://github.com/vizkov/oss/blob/main/Framework%20Coverage%20%26%20Enforcement.md)

