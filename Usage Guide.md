
#  Usage Guide

## Tag Use Case Reference

| If you are...                                    | Use this Tag   | Action / Goal                                                       |
| ------------------------------------------------ | -------------- | ------------------------------------------------------------------- |
| **Beginning** a review of a specific code block  | `SEC-SECTION`  | **Wrap** the code to create a spatial "room" in your sidebar        |
| **Tracking** where user input reaches a handler  | `SEC-SOURCE`   | Mark the **Entry** (input)                                          |
| **Tracking** where user input reaches a database | `SEC-SINK`     | Mark the **Danger** (execution)                                     |
| **Tracing** a complex multi-step algorithm       | `SEC-FLOW`     | Label steps ($1, 2, 3$) to keep the execution **Path** clear        |
| **Checking** if a user has the right permissions | `SEC-LOGIC`    | Document the **Rules** and business constraints                     |
| **Identifying** a sensitive area (like Crypto)   | `SEC-AREA`     | Flag the **Surface** for a deep-dive review later (OWASP)           |
| **Confirming** a vulnerability exists            | `SEC-BUG`      | Log the **Finding** and assign a severity rank ($1$–$5$)            |
| **Asking** a question for the next pass          | `SEC-REVIEW`   | Leave a **Note** for yourself                                       |
| **Organizing** a complex file layout             | `SEC-SECTION`  | Create **Boundaries** to group related **findings, logic or code**  |
| **Connecting** two related points                | `SEC-LINK`     | **Teleport** between related code in different files                |
| **Documenting** what relationship pattern exists | `SEC-RELATION` | **Label** the relationship pattern type                             |
| **Marking** a specific instance of a pattern     | `SEC-INSTANCE` | **Mark** individual occurrences or phases of a relationship pattern |

## Attribute Use Case Reference

| Tag            | `epic` Usage                                          | `seq` Usage                                  | `id` Usage                                        | **Use Case**                                                               |
| -------------- | ----------------------------------------------------- | -------------------------------------------- | ------------------------------------------------- | -------------------------------------------------------------------------- |
| `SEC-SOURCE`   | Groups data flow by vuln class (e.g., `epic=SQLi`)    | Order of data movement (1→2→3)               | Required - unique name to be linked from SINK     | Mark untrusted entry points (req.body, headers, params)                    |
| `SEC-SINK`     | Groups data flow by vuln class (e.g., `epic=RCE`)     | Order of data movement (1→2→3)               | Optional - only if multiple sinks need linking    | Mark dangerous execution points (db.query, eval, innerHTML)                |
| `SEC-LOGIC`    | Groups business rules (e.g., `epic=Auth`)             | Not used (rules don't have sequence)         | Required - allows linking from enforcement points | Document intended business constraints and rules                           |
| `SEC-FLOW`     | Groups multi-step process (e.g., `epic=Payment`)      | Required - chronological step (1, 2, 3...)   | Optional - only if steps need linking             | Trace step-by-step execution flow in algorithms or state machines          |
| `SEC-AREA`     | Groups by attack surface (e.g., `epic=OWASP_A01`)     | Not typically used                           | Optional - for linking from bug reports           | Flag high-value logic requiring deep review (Auth, Crypto, Access Control) |
| `SEC-BUG`      | Groups findings by vuln type (e.g., `epic=XSS`)       | Required - severity (1=Crit, 2=High, 5=Info) | Required - allows linking from executive summary  | Document confirmed vulnerabilities and security bugs                       |
| `SEC-AUDIT`    | Groups by audit scope (e.g., `epic=Sprint_3`)         | Optional - audit priority (1, 2, 3)          | Not typically used                                | Provide high-level context for file/module scope                           |
| `SEC-REVIEW`   | Groups by question type (e.g., `epic=Clarification`)  | Not used (questions don't have priority)     | Not used (file-scoped, not linked)                | Leave temporary notes and questions for second pass review                 |
| `SEC-SECTION`  | Not used - sections use spatial containment instead   | Not used - sections use visual proximity     | Optional - parent ID for hierarchical grouping    | Create spatial boundaries to organize related code in sidebar              |
| `SEC-LINK`     | Not used - just points to target                      | Not used - links don't have sequence         | Not used - links point to other IDs               | Connect related anchors or regions                                         |
| `SEC-RELATION` | Required - relationship type (e.g., `epic=Data_Flow`) | Optional - pattern priority (1-5)            | Optional - if linking patterns together           | Document relationship patterns                                             |
| `SEC-INSTANCE` | Required - relationship type (e.g., `epic=Mirroring`) | Optional - instance sequence (1, 2, 3)       | Required - unique ID for each instance            | Mark specific instances of patterns                                        |

## Link Use Case Reference

| **Relationship** | **Layer**    | **Connects**                     | **Direction**  | Use Case                                         | **Typical Example**                                     |
| ---------------- | ------------ | -------------------------------- | -------------- | ------------------------------------------------ | ------------------------------------------------------- |
| Data Flow        | Data         | `SEC-SOURCE` → `SEC-SINK`        | Forward        | Traces input flow to execution (SOURCE → SINK)   | `req.params.id` → `userSvc.find(id)`                    |
| Mirroring        | Logic        | `SEC-LOGIC` ↔ `SEC-LOGIC`        | Bidirectional  | Connects symmetric pairs bidirectionally         | `JWT.sign()` ↔ `JWT.verify()`                           |
| Lifecycle        | Logic        | `SEC-FLOW` → `SEC-FLOW`          | Forward        | Connects sequential flow steps                   | `order.status = 'paid'` → `shipment.init()`             |
| Parent           | Structural   | `SEC-SECTION` → *                | Contains       | Creates parent-child or cross-file relationships | `utils/sanitizer.js` → `controllers/auth.js`            |
| Reference        | Analysis     | `SEC-BUG` → `SEC-AREA`           | Categorization | Connects bugs to key areas                       | `await query(sql)` → `SEC-AREA: OWASP_A03`              |
| Validation       | Logic        | `SEC-LOGIC` → `SEC-FLOW`         | Forward        | Connects rules to enforcement or flow steps      | `Rule: Admin only` → `if role != 'admin': raise`        |
| Exploit Chain    | Analysis     | `SEC-BUG` → \*                   | Forward        | Chains all contributing factors                  | `SQLi Bug` → `[input, no_validation, query]`            |
| Remediation      | Organization | `SEC-BUG` → `SEC-REVIEW`         | Forward        | Connects reviews to bugs needing follow-up       | `Command Injection` → `Use subprocess with shell=False` |
| Assumption       | Structural   | \* → \*                          | Depends        | Documents implicit cross-file contracts          | `JWT tokens use SERVER_SECRET` → `config.py:SECRET`     |
| AltPath          | Logic        | `SEC-FLOW` → `SEC-FLOW`          | Split          | Connects branches back to parent decision point  | `auth_flow` → `[happy_path, error_path]`                |
| Composition      | Metadata     | `SEC-INSTANCE` → [sub-instances] | Aggregation    | Complex instances composed of simpler instance   | `Full exploit` → `[phase1, phase2, phase3]`             |
| Intersection     | Metadata     | `RELATION` + `RELATION`          | Combination    | Multiple patterns occurring simultaneously       | `Data_Flow + Validation` → `Secure_Data_Flow`           |

Use these **first** to define your workspace before you look for bugs:

- **`SEC-SECTION`**: Use this immediately when you find a block of code (a function, a class, or a middleware) that looks interesting. Wrap the code to create a "Spatial Room" in your sidebar.
- **`SEC-LINK`**: Use this at the end of your trace to physically "point" back to a starting point (like linking a Sink back to its Source).

Use these when you are tracing **untrusted input**:

- **`SEC-SOURCE`**: Mark the variable where user data enters (e.g., `req.body`, `params`).
- **`SEC-SINK`**: Mark the dangerous function where that data ends up (e.g., `db.query()`, `eval()`, `res.send()`).

Use these when you aren't looking at data flow, but at **how the code is supposed to work**:

- **`SEC-LOGIC`**: Mark where a business rule is defined (e.g., "Only admins can delete posts").
- **`SEC-FLOW`**: Use this to document the steps of a complex algorithm (Step 1: Check cache, Step 2: Validate token...).

Use these to categorize your **conclusions**:

- **`SEC-AREA`**: Use this to flag "Attack Surface" areas like Auth or Crypto logic that look complex and need a second look later—think of this as your "OWASP Top 10" map.
- **`SEC-BUG`**: Use this only when you have **confirmed** a vulnerability. This is your high-priority "Finding".

Use these for **audit management**:

- **`SEC-AUDIT`**: Place this at the top of a file or project to define the mission or metadata.
- **`SEC-REVIEW`**: Use this as a digital "sticky note" for questions you need to ask the developers or things to check in a second pass.

Use these to document **relationship patterns**:

- **`SEC-RELATION`**: Place this at the beginning of a code area to document what relationship pattern exists (e.g., "Data Flow pattern in auth module").
- **`SEC-INSTANCE`**: Use this to mark individual occurrences or when decomposing complex instances into phases within a pattern (e.g., multiple data flows in one file).

## Usage Frequency 

- `SEC-SECTION`: Use these **constantly**. A single file might have 5–10 sections, each wrapping a different function to keep the sidebar organized.
- `SEC-LINK`: Think of it as a "See Also" reference. Use links to connect Data, Logic, Analysis, Organization, Structure or Metadata. 

These tags set the stage. Having multiples of these in one file usually creates confusion in your sidebar and reporting:

- **`SEC-AUDIT`**: You typically have **one** at the top of the file to define the "Mission" or "Task Description" for that specific document.
- **`SEC-AREA` (Broadly)**: Usually, a file represents one major attack surface (e.g., a "Controller" is an A01/A03 area). While you _can_ have multiple, it's best to use one to define the whole file's risk category.

## The "Audit Trail" Navigation

When you find a **Sink** (like `db.query`), don't just mark it. Look for the **Source** (where the variable came from).

1. Mark the source: `// SEC-SOURCE[epic=SQLi, seq=1, id=raw-input] 🟢 Source: User input`
2. Mark the sink: `// SEC-SINK[epic=SQLi, seq=2] 🔴 Sink: Database query`
3. Link them: `// SEC-LINK: [epic: Data_Flow] 🔗 Connector to #raw-input`

When searching for vulnerabilities, tag these functions as `SEC-SINK`:

|**Category**|**Sinks to Watch**|
|---|---|
|**RCE**|`exec()`, `spawn()`, `system()`, `eval()`|
|**SQLi**|`execute()`, `db.run()`, `db.query()`|
|**XSS**|`innerHTML`, `document.write()`, `dangerouslySetInnerHTML`|
|**SSRF**|`fetch()`, `axios.get()`, `request()`|
|**FS**|`readFile()`, `writeFile()`, `unlink()`|

---

# Common Patterns

## Cryptographic Requirements

```python
# SEC-AREA[epic=Crypto, id=hash_module] ⚠️ Attack Surface: Password hashing implementation

# SEC-LOGIC[epic=Crypto, id=strength_req] ⭐ Business Logic: Must use bcrypt/argon2 with cost >= 10
# Description: Constraint - PBKDF2 min 100k iterations OR bcrypt cost >= 10

# SEC-LOGIC[epic=Crypto, id=salt_req] ⭐ Business Logic: Each password must have unique random salt
# Description: Constraint - Salt must be cryptographically random, >= 16 bytes, unique per password

# SEC-FLOW[epic=Crypto, seq=1] 🪜 Step: Generate salt
salt = os.urandom(16)  # ✓ Meets requirement

# SEC-FLOW[epic=Crypto, seq=2] 🪜 Step: Hash with salt
hash = bcrypt.hashpw(password, salt)  # ✓ Meets requirement
```

**Key Technique:** Use SEC-LOGIC with detailed descriptions to specify constraints

## State Machine Violations

```python
# SEC-RELATION[epic=Lifecycle] 🔀 Pattern: Order state transitions

# SEC-LOGIC[epic=State_Machine, id=valid_transitions] ⭐ Business Logic: Valid transitions
# Description: pending → processing → (paid OR failed), paid → shipped, shipped → delivered
# Invalid: pending → shipped, pending → delivered, failed → shipped

# SEC-FLOW[epic=Order, seq=1, id=pending] 🪜 Step: Order created (pending)
order.state = "pending"

# SEC-FLOW[epic=Order, seq=2, id=processing] 🪜 Step: Payment processing
order.state = "processing"

# SEC-FLOW[epic=Order, seq=3, id=shipped] 🪜 Step: Order shipped
# SEC-BUG[epic=State_Machine, seq=1] ☠️ Finding: Invalid state transition - skipped payment verification
order.state = "shipped"  # ← Should be "paid" first!
# SEC-LINK: 🔗 Connector to #valid_transitions
```

**Key Technique:** Document valid transitions in SEC-LOGIC, mark violations with SEC-BUG

## Configuration Security

```python
# SEC-AREA[epic=Config, id=security_config] ⚠️ Attack Surface: Security configuration

# SEC-LOGIC[epic=Config, id=prod_requirements] ⭐ Business Logic: Production requirements
# Description: debug=False, CORS=specific origins, HTTPS=True, rate_limit=enabled

# SEC-AUDIT[epic=Config, seq=1] 📋 Context: Reviewing production configuration

# Configuration checks:
# SEC-BUG[epic=Config, seq=1, id=debug_on] ☠️ Finding: Debug mode enabled in production
DEBUG = True  # ← Should be False

# SEC-BUG[epic=Config, seq=2, id=permissive_cors] ☠️ Finding: Overly permissive CORS
CORS_ORIGINS = "*"  # ← Should be specific domains

# SEC-LOGIC[epic=Config, id=https_requirement] ⭐ Business Logic: Must enforce HTTPS
SECURE_SSL_REDIRECT = False  # ← Violation
# SEC-BUG[epic=Config, seq=3] ☠️ Finding: HTTPS not enforced
```

**Key Technique:** Use SEC-AREA for config files + SEC-LOGIC for requirements + SEC-BUG for violations

## Resource Management

```python
# SEC-RELATION[epic=Mirroring] 🔀 Pattern: Resource acquire/release pair

# SEC-FLOW[epic=Resource_Leak, seq=1, id=acquire] 🪜 Step: Acquire file handle
file = open("sensitive.txt", "r")

try:
    # ... processing ...
    data = file.read()
    
except Exception as e:
    # SEC-BUG[epic=Resource_Leak, seq=1] ☠️ Finding: File handle leaked on exception
    # Description: Exception path missing cleanup - file.close() never called
    return error_response
    
finally:
    # SEC-FLOW[epic=Resource_Leak, seq=2, id=release] 🪜 Step: Release file handle
    file.close()
    # SEC-LINK: 🔗 Connector to #acquire
```

**Key Technique:** Use Mirroring relationship + mark exception paths with SEC-BUG

## Side-Channel Vulnerabilities

```python
# SEC-AREA[epic=Timing_Attack, id=auth_compare] ⚠️ Attack Surface: Password comparison

# SEC-LOGIC[epic=Timing_Attack, id=constant_time_req] ⭐ Business Logic: Must use constant-time comparison
# Description: Constraint - comparison time must not leak information about password correctness

# SEC-FLOW[epic=Timing_Attack, seq=1, id=early_exit] 🪜 Step: Character-by-character comparison
def compare_password(input_pw, stored_pw):
    for i in range(len(input_pw)):
        if input_pw[i] != stored_pw[i]:
            # SEC-BUG[epic=Timing_Attack, seq=1] ☠️ Finding: Early exit leaks timing information
            # Description: Side-channel - use hmac.compare_digest() or secrets.compare_digest()
            return False
    return True
```

**Key Technique:** Use SEC-LOGIC for constant-time requirements + SEC-BUG with description of fix

## Trust Boundary Crossings

```python
# SEC-RELATION[epic=Data_Flow] 🔀 Pattern: Data crossing trust boundaries

# SEC-AREA[epic=Trust_Boundary, id=internal_external] ⚠️ Attack Surface: Internal-to-external API boundary
# Description: Data crossing from trusted internal network to untrusted external API

# SEC-SOURCE[epic=Trust, seq=1, id=internal_data] 🟢 Source: Internal database (trusted)
internal_data = db.get_sensitive_records()

# SEC-REVIEW[epic=Trust, id=boundary_crossing] 🔍 Note: Data crosses trust boundary here - validate/sanitize?

# SEC-SINK[epic=Trust, seq=2] 🔴 Sink: External API (untrusted)
external_api.send(internal_data)

# SEC-BUG[epic=Trust, seq=1] ☠️ Finding: Sensitive data sent to external API without sanitization
# Description: Trust boundary violation - should filter/mask sensitive fields
```

**Key Technique:** Use SEC-AREA to mark boundaries + SEC-REVIEW to flag crossing points

---

# The Executive Summary (Markdown)

To create a final report that links back to your code:

1. Create a `SUMMARY.md` file.
2. Narrative high-level findings.
3. Use the teleport link format: `[Finding Description](./path/to/file.ext#id-name)`.

> **Note:** The `id-name` must match the `[id=...]` attribute in your code comment.

---
