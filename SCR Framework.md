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

# Framework Coverage & Enforcement

## Layer Coverage

- [ ] **Data Layer**: Both SOURCE and SINK tags present for each data flow?
- [ ] **Logic Layer**: Both LOGIC rules and FLOW steps documented?
- [ ] **Analysis Layer**: Both AREA surface and BUG findings present?
- [ ] **Organization Layer**: AUDIT context at file level? REVIEW notes for unclear items?
- [ ] **Structural Layer**: SECTION boundaries used? LINK connections present?
- [ ] **Metadata Layer**: RELATION patterns documented? INSTANCE occurrences marked?

## Relationship Pattern Coverage

- [ ] **Data Flow**: Every SOURCE has corresponding SINK?
- [ ] **Mirroring**: Symmetric operations paired (sign ↔ verify)?
- [ ] **Lifecycle**: Sequential steps properly ordered (seq)?
- [ ] **Parent**: Sections properly contain related tags?
- [ ] **Reference**: Bugs linked to relevant areas?
- [ ] **Validation**: Logic rules linked to enforcement code?
- [ ] **Exploit Chain**: Bugs link to all contributing factors?
- [ ] **Remediation**: Bugs have fix suggestions in REVIEW?
- [ ] **Assumption**: Cross-file dependencies documented?
- [ ] **AltPath**: Branches (if/else) both documented?
- [ ] **Composition**: Complex instances broken into phases?
- [ ] **Intersection**: Combined patterns documented?

## Metadata Layer Detailed Checks

- [ ] **SEC-RELATION presence**: At least one pattern type documented?
- [ ] **SEC-INSTANCE presence**: At least one pattern instance marked?
- [ ] **RELATION → INSTANCE pairing**: Every RELATION has corresponding INSTANCE(s)?
- [ ] **Epic consistency**: RELATION and INSTANCE use same epic values?
- [ ] **Composition structure**: Parent INSTANCE has child instances linked?
- [ ] **Intersection structure**: Combined pattern links to source patterns?
- [ ] **Pattern documentation**: Each relationship type (Data_Flow, Mirroring, etc.) has RELATION if used?

## Syntax Compliance

- [ ] All anchors have `epic` attribute?
- [ ] All anchors have emojis (🟢🔴⭐🪜⚠️☠️📋🔍📁🔗🔀🛤️)?
- [ ] Attribute order: `[epic, seq, id]`?
- [ ] SEC-LINK on separate line (not embedded)?
- [ ] SEC-SECTION has closing `!SEC-SECTION`?
- [ ] All `id` values are unique within file?

## Best Practices

- [ ] SEC-SECTION used 5-10 times per file?
- [ ] One SEC-AUDIT per file at top?
- [ ] SEC-SOURCE always has `id` for linking?
- [ ] SEC-BUG has `seq` for severity?
- [ ] SEC-FLOW has `seq` for ordering?
- [ ] SEC-INSTANCE always has `id` attribute?
- [ ] Multiple instances use `seq` for ordering?
- [ ] SEC-RELATION placed before first instance?


## Automated Validation Script

```python
#!/usr/bin/env python3
"""
SCR Framework Compliance Validator
Validates Comment Anchors syntax and coverage
"""

import re
import sys
from pathlib import Path
from collections import defaultdict

class SCRValidator:
    def __init__(self, filepath):
        self.filepath = filepath
        self.content = Path(filepath).read_text()
        self.errors = []
        self.warnings = []
        self.stats = defaultdict(int)
        
    def validate(self):
        """Run all validation checks"""
        self.check_syntax()
        self.check_coverage()
        self.check_metadata_layer()
        self.check_relationships()
        self.check_best_practices()
        return self.report()
    
    def check_syntax(self):
        """Validate syntax compliance"""
        # Find all SEC-* tags
        tags = re.findall(r'//\s*SEC-(\w+)\[(.*?)\]\s*(.*?)$', self.content, re.MULTILINE)
        
        for tag_name, attributes, description in tags:
            self.stats[f'SEC-{tag_name}'] += 1
            
            # Check for epic attribute (except SECTION, LINK)
            if tag_name not in ['SECTION', 'LINK']:
                if 'epic=' not in attributes:
                    self.errors.append(f"SEC-{tag_name} missing 'epic' attribute")
            
            # Check attribute order: epic, seq, id
            if 'epic=' in attributes:
                epic_pos = attributes.find('epic=')
                if 'seq=' in attributes:
                    seq_pos = attributes.find('seq=')
                    if seq_pos < epic_pos:
                        self.warnings.append(f"SEC-{tag_name}: 'seq' before 'epic' (should be epic, seq, id)")
                if 'id=' in attributes:
                    id_pos = attributes.find('id=')
                    if 'seq=' in attributes and id_pos < seq_pos:
                        self.warnings.append(f"SEC-{tag_name}: 'id' before 'seq'")
            
            # Check for emoji
            emojis = ['🟢', '🔴', '⭐', '🪜', '⚠️', '☠️', '📋', '🔍', '📁', '🔗', '🔀', '🛤️']
            if not any(emoji in description for emoji in emojis):
                self.warnings.append(f"SEC-{tag_name} missing emoji in description")
    
    def check_coverage(self):
        """Check layer coverage"""
        # Data Layer
        if self.stats['SEC-SOURCE'] == 0:
            self.warnings.append("No SEC-SOURCE tags found (Data Layer incomplete)")
        if self.stats['SEC-SINK'] == 0:
            self.warnings.append("No SEC-SINK tags found (Data Layer incomplete)")
        
        # Logic Layer
        if self.stats['SEC-LOGIC'] == 0 and self.stats['SEC-FLOW'] == 0:
            self.warnings.append("No SEC-LOGIC or SEC-FLOW tags (Logic Layer unused)")
        
        # Analysis Layer
        if self.stats['SEC-BUG'] == 0:
            self.warnings.append("No SEC-BUG tags found (no findings documented)")
        
        # Organization Layer
        if self.stats['SEC-AUDIT'] == 0:
            self.warnings.append("No SEC-AUDIT tag (missing file context)")
        if self.stats['SEC-AUDIT'] > 1:
            self.warnings.append(f"Multiple SEC-AUDIT tags ({self.stats['SEC-AUDIT']}) - recommend 1 per file")
        
        # Structural Layer
        if self.stats['SEC-SECTION'] == 0:
            self.warnings.append("No SEC-SECTION tags (file not organized)")
        if self.stats['SEC-SECTION'] < 3:
            self.warnings.append(f"Only {self.stats['SEC-SECTION']} SEC-SECTION tags - recommend 5-10 per file")
            
        # Metadata Layer
	    if self.stats['SEC-RELATION'] == 0 and self.stats['SEC-INSTANCE'] == 0:
	        self.warnings.append("No SEC-RELATION or SEC-INSTANCE tags (Metadata Layer unused)")
    
    def check_relationships(self):
        """Check relationship patterns"""
        # Extract all ids
        ids = re.findall(r'id=([a-zA-Z0-9_]+)', self.content)
        id_set = set(ids)
        
        # Check duplicate ids
        if len(ids) != len(id_set):
            duplicates = [id for id in id_set if ids.count(id) > 1]
            self.errors.append(f"Duplicate IDs found: {', '.join(duplicates)}")
        
        # Extract all SEC-LINK targets
        links = re.findall(r'SEC-LINK:.*?#([a-zA-Z0-9_]+)', self.content)
        
        # Check if link targets exist
        for link_target in links:
            if link_target not in id_set:
                self.errors.append(f"SEC-LINK references non-existent ID: #{link_target}")
        
        # Check SEC-SECTION closure
        sections = re.findall(r'SEC-SECTION:', self.content)
        closures = re.findall(r'!SEC-SECTION', self.content)
        if len(sections) != len(closures):
            self.errors.append(f"SEC-SECTION mismatch: {len(sections)} open, {len(closures)} close")
    
    def check_best_practices(self):
        """Check best practice compliance"""
        # Check if SEC-SOURCE has id
        sources = re.findall(r'SEC-SOURCE\[(.*?)\]', self.content)
        for attrs in sources:
            if 'id=' not in attrs:
                self.warnings.append("SEC-SOURCE without 'id' - cannot be linked")
        
        # Check if SEC-BUG has seq (severity)
        bugs = re.findall(r'SEC-BUG\[(.*?)\]', self.content)
        for attrs in bugs:
            if 'seq=' not in attrs:
                self.warnings.append("SEC-BUG without 'seq' - severity not specified")
        
        # Check if SEC-SINK has corresponding SEC-LINK
        sinks = self.content.count('SEC-SINK')
        links_after_sink = len(re.findall(r'SEC-SINK.*?\n.*?SEC-LINK:', self.content, re.DOTALL))
        if sinks > links_after_sink:
            self.warnings.append(f"{sinks - links_after_sink} SEC-SINK tags without SEC-LINK")
        
        # Metadata Layer best practices
	    if self.stats['SEC-RELATION'] > 0 and self.stats['SEC-INSTANCE'] == 0:
	        self.warnings.append("SEC-RELATION found but no SEC-INSTANCE - patterns documented without instances")
    
	    if self.stats['SEC-INSTANCE'] > 0 and self.stats['SEC-RELATION'] == 0:
	        self.warnings.append("SEC-INSTANCE found but no SEC-RELATION - instances without pattern definition")
	    
	def check_metadata_relationships(self):
        """Check metadata layer specific patterns"""
        
        # Check RELATION → INSTANCE pairing
        relations = re.findall(r'SEC-RELATION\[epic=([^,\]]+)', self.content)
        instances = re.findall(r'SEC-INSTANCE\[epic=([^,\]]+)', self.content)
        
        relation_epics = set(relations)
        instance_epics = set(instances)
        
        # Find relations without instances
        for epic in relation_epics:
            if epic not in instance_epics:
                self.warnings.append(f"SEC-RELATION[epic={epic}] has no corresponding SEC-INSTANCE")
        
        # Find instances without relations (less critical)
        for epic in instance_epics:
            if epic not in relation_epics:
                self.warnings.append(f"SEC-INSTANCE[epic={epic}] has no corresponding SEC-RELATION (pattern not documented)")
        
        # Check Composition pattern (INSTANCE → sub-INSTANCEs)
        composition_instances = re.findall(r'SEC-INSTANCE\[epic=Composition.*?id=([^,\]]+)', self.content)
        for parent_id in composition_instances:
            # Check if any other INSTANCE links to this parent
            child_links = re.findall(rf'SEC-LINK:.*?#{parent_id}', self.content)
            if len(child_links) == 0:
                self.warnings.append(f"Composition INSTANCE[id={parent_id}] has no child instances linked to it")
        
        # Check Intersection pattern (multiple RELATIONs)
        intersection_relations = re.findall(r'SEC-RELATION\[epic=Intersection', self.content)
        if len(intersection_relations) > 0:
            # Each intersection should link to 2+ other relations
            for i, match in enumerate(intersection_relations):
                # Find SEC-LINK references after this RELATION
                # Should have 2+ links to other relation IDs
                # This is a simplified check
                pass  # Can be enhanced with more sophisticated parsing
    
    def check_metadata_layer(self):
    """Check metadata layer coverage and patterns"""
    
    # Basic presence check
    if self.stats.get('SEC-RELATION', 0) == 0 and self.stats.get('SEC-INSTANCE', 0) == 0:
        self.warnings.append("Metadata Layer: No SEC-RELATION or SEC-INSTANCE tags found")
        return  # Skip further metadata checks if layer not used
    
    # Extract all RELATION and INSTANCE tags with epics
    relation_pattern = r'SEC-RELATION\[epic=([^,\]]+).*?id=([^,\]]+)?'
    instance_pattern = r'SEC-INSTANCE\[epic=([^,\]]+).*?id=([^,\]]+)?'
    
    relations = re.findall(relation_pattern, self.content)
    instances = re.findall(instance_pattern, self.content)
    
    # Group by epic
    relation_epics = {}
    instance_epics = {}
    
    for epic, rel_id in relations:
        if epic not in relation_epics:
            relation_epics[epic] = []
        relation_epics[epic].append(rel_id if rel_id else None)
    
    for epic, inst_id in instances:
        if epic not in instance_epics:
            instance_epics[epic] = []
        instance_epics[epic].append(inst_id if inst_id else None)
    
    # Rule 2: Pairing check
    for epic in relation_epics:
        if epic not in instance_epics:
            self.warnings.append(f"Metadata Layer: SEC-RELATION[epic={epic}] has no corresponding SEC-INSTANCE")
    
    for epic in instance_epics:
        if epic not in relation_epics:
            self.warnings.append(f"Metadata Layer: SEC-INSTANCE[epic={epic}] exists without SEC-RELATION (pattern not documented)")
    
    # Rule 3: Epic consistency - check against valid relationship types
    valid_epics = [
        'Data_Flow', 'Mirroring', 'Lifecycle', 'Parent', 'Reference',
        'Validation', 'Exploit_Chain', 'Remediation', 'Assumption',
        'AltPath', 'Composition', 'Intersection'
    ]
    
    all_meta_epics = set(list(relation_epics.keys()) + list(instance_epics.keys()))
    for epic in all_meta_epics:
        if epic not in valid_epics:
            self.warnings.append(f"Metadata Layer: Epic '{epic}' is not a standard relationship pattern")
    
    # Rule 6: ID requirement for INSTANCE
    instances_without_id = [epic for epic, ids in instance_epics.items() if None in ids]
    if instances_without_id:
        self.errors.append(f"Metadata Layer: SEC-INSTANCE tags missing required 'id' attribute in epics: {', '.join(instances_without_id)}")
    
    # Rule 4: Composition structure check
    if 'Composition' in instance_epics:
        composition_ids = [id for id in instance_epics['Composition'] if id]
        for parent_id in composition_ids:
            child_links = re.findall(rf'SEC-LINK:.*?#{parent_id}', self.content)
            if len(child_links) == 0:
                self.warnings.append(f"Metadata Layer: Composition INSTANCE[id={parent_id}] has no child instances linked to it")
    
    # Rule 5: Intersection structure check
    if 'Intersection' in relation_epics:
        intersection_ids = [id for id in relation_epics['Intersection'] if id]
        for int_id in intersection_ids:
            # Count SEC-LINK references after this RELATION
            # Should find 2+ links to component relation IDs
            links_after = re.findall(rf'SEC-RELATION\[epic=Intersection.*?id={int_id}.*?\n(.*?)\n.*?SEC-LINK', self.content, re.DOTALL)
            if links_after and len(links_after[0].split('SEC-LINK')) < 3:  # Should have 2+ links
                self.warnings.append(f"Metadata Layer: Intersection RELATION[id={int_id}] should link to 2+ component patterns")
    
    # Rule 7: Sequence usage check
    for epic, ids in instance_epics.items():
        if len(ids) > 1:
            # Check if seq is used
            instances_with_seq = re.findall(rf'SEC-INSTANCE\[epic={epic}.*?seq=', self.content)
            if len(instances_with_seq) == 0:
                self.warnings.append(f"Metadata Layer: Multiple SEC-INSTANCE[epic={epic}] found but no 'seq' attribute for ordering")
    
    def report(self):
        """Generate validation report"""
        print(f"\n{'='*60}")
        print(f"SCR Framework Validation Report: {self.filepath}")
        print(f"{'='*60}\n")
        
        # Statistics
        print("Tag Statistics:")
        for tag, count in sorted(self.stats.items()):
            print(f"  {tag}: {count}")
        print()
        
        # Errors
        if self.errors:
            print(f"❌ ERRORS ({len(self.errors)}):")
            for error in self.errors:
                print(f"  • {error}")
            print()
        
        # Warnings
        if self.warnings:
            print(f"⚠️  WARNINGS ({len(self.warnings)}):")
            for warning in self.warnings:
                print(f"  • {warning}")
            print()
        
        # Summary
        if not self.errors and not self.warnings:
            print("✅ All checks passed!")
            return 0
        elif not self.errors:
            print(f"✅ No errors, {len(self.warnings)} warnings")
            return 0
        else:
            print(f"❌ {len(self.errors)} errors, {len(self.warnings)} warnings")
            return 1

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: scr_validate.py <file>")
        sys.exit(1)
    
    validator = SCRValidator(sys.argv[1])
    sys.exit(validator.validate())
```
