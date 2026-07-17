
# Global Settings 

## Setup

| **Component**         | **Location**             | **Purpose**                                               |
| --------------------- | ------------------------ | --------------------------------------------------------- |
| **Global Settings**   | `settings.json`          | Defines what the tags **look like** and **how they act**. |
| **Audit Snippet**     | `security.code-snippets` | Handles the **syntax** so you just enter data.            |
| **Research File**     | Any source code file     | Where you actually perform the audit.                     |
| **Executive Summary** | `REPORT.md`              | Your high-level narrative for clients/teams.              |

## Comment Anchors (`settings.json`)

To configure your environment, open the Command Palette (`Ctrl+Shift+P`), type **"Open User Settings (JSON)"**, and paste this block:

```json
{
"editor.snippetSuggestions": "top",
  "commentAnchors.tags.anchors": {
    // --- WORKSPACE SCOPE ---
    "SEC-SOURCE": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#4CAF50",
    },
    "SEC-SINK": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#1B5E20",
    },
    "SEC-LOGIC": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#9C27B0",
    },
    "SEC-FLOW": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#4A148C",
    },
    "SEC-AREA": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#FFC107",
    },
    "SEC-BUG": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#F57F17",
    },
    "SEC-AUDIT": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#2196F3",
    },
    "SEC-RELATION": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#00BCD4",
    },
    "SEC-TRACE": {
      "behavior": "anchor",
      "scope": "workspace",
      "highlightColor": "#006064",
    },

    // --- FILE SCOPE ---
    "SEC-REVIEW": {
      "behavior": "anchor",
      "scope": "file",
      "highlightColor": "#0D47A1",
    },
    "SEC-SECTION": {
      "behavior": "region",
      "scope": "file",
      "highlightColor": "#9E9E9E",
    },

    // --- HIDDEN SCOPE ---
    "SEC-LINK": {
      "behavior": "link",
      "scope": "hidden",
      "highlightColor": "#424242",
    },

    // --- Disabled: Default Tags ---
    "TODO": { "enabled": false, "scope": "workspace" },
    "FIXME": { "enabled": false, "scope": "workspace" },
    "STUB": { "enabled": false, "scope": "workspace" },
    "NOTE": { "enabled": false, "scope": "workspace" },
    "ANCHOR": { "enabled": false, "scope": "workspace" },
    "SECTION": { "enabled": false, "scope": "workspace" },
    "LINK": { "enabled": false, "scope": "workspace" },
    "REVIEW": { "enabled": false, "scope": "workspace" },
  },

  "commentAnchors.workspace.enabled": true,
  "commentAnchors.workspace.matchFiles": "**/*.{js,ts,py,php,go,c,cpp,h,java,md}",
  "commentAnchors.parseDelay": 150,
  "commentAnchors.tags.sortAlphabetically": true,
  "commentAnchors.tags.displayTagName": false
}
```

## VS Code User Snippets (`.code-snippets.json`)

Snippets allow you to drop a full audit template into your code instantly.

**How to set up:**
1. Command Palette -> **"Configure User Snippets"**.
2. Select **"New Global Snippets file"** (e.g., `security.code-snippets.json`).
3. Paste the following:

```json
{
	// ============================================================================
	// Security Research FRAMEWORK
	// ============================================================================
	// 
	// 6 LAYERS:
	//   DATA (src→sink) = Trace untrusted input to execution
	//   LOGIC (logic→flow) = Intent vs Reality (what should happen vs what does)
	//   ANALYSIS (area→bug) = Attack surface to specific findings
	//   ORGANIZATION (audit→review) = Context and questions
	//   STRUCTURAL (section→link) = Spatial organization and connections
	//   METADATA (relation→instance) = Pattern types and instances
	//
	// 12 RELATIONSHIP PATTERNS:
	//   Data_Flow: SOURCE→SINK 
	//	 Mirroring: LOGIC↔LOGIC 
	//   Lifecycle: FLOW→FLOW
	//   Parent: SECTION→* 
	//   Reference: BUG→AREA 
	//	 Validation: LOGIC→FLOW
	//   Exploit_Chain: BUG→[multiple] 
	//   Remediation: BUG→REVIEW
	//   Assumption: *→* 
	//   AltPath: FLOW→[branches]
	//   Composition: INSTANCE→[sub-instances] 
	//   Intersection: RELATION+RELATION
	//
	// QUICK START:
	//   1. s-src = Mark untrusted input (req.body, params)
	//   2. s-sink = Mark dangerous execution (db.query, eval) + auto-link to source
	//   3. s-bug = Confirmed vulnerabilities
	//   4. s-section = Wrap code blocks (use constantly!)
	// ============================================================================
	// --- LAYER 1: DATA (DATA FLOW) ---
	"Tag: Source": {
		"prefix": "s-src",
		"body": "// SEC-SOURCE[epic=${1:Login_Flow}, seq=${2:1}, id=${3:src_id}] 🟢 Source: ${4:Untrusted data input}$0",
		"description": "DATA LAYER: Mark untrusted entry points (req.body, params, headers). Must have id= to link to SEC-SINK. seq=1,2,3 shows data movement order."
	},
	"Tag: Sink": {
		"prefix": "s-sink",
		"body": [
			"// SEC-SINK[epic=${1:Login_Flow}, seq=${2:2}] 🔴 Sink: ${3:Execution point}",
			"// SEC-LINK: [epic: Data_Flow] 🔗 Connector to #${4:src_id}$0"
		],
		"description": "DATA LAYER: Mark dangerous execution points (db.query, eval, innerHTML, os.system). Links back to SEC-SOURCE. seq=2+ shows data destination."
	},
	// --- LAYER 2: LOGIC (BUSINESS RULES) ---
	"Tag: Logic Rule": {
		"prefix": "s-logic",
		"body": "// SEC-LOGIC[epic=${1:Logic_Rule}, id=${2:rule_id}] ⭐ Business Logic: ${3:Only admins can delete users}$0",
		"description": "LOGIC LAYER: Document what code SHOULD do (e.g., 'Only admins can delete'). Use id= so SEC-FLOW can link to this rule for validation."
	},
	"Tag: Flow Step": {
		"prefix": "s-flow",
		"body": "// SEC-FLOW[epic=${1:Logic_Rule}, seq=${2:1}] 🪜 Step: ${3:Step-by-step flow}$0",
		"description": "LOGIC LAYER: Document what code ACTUALLY does, step by step. Use seq=1,2,3 for execution order. Can link to SEC-LOGIC rules to show enforcement (or missing enforcement)."
	},
	// --- LAYER 3: ANALYSIS (FINDINGS) ---
	"Tag: Surface Area": {
		"prefix": "s-area",
		"body": "// SEC-AREA[epic=${1:OWASP_Vector}] ⚠️ Attack Surface: ${2:Sensitive logic requiring review}$0",
		"description": "ANALYSIS LAYER: Flag sensitive areas needing review (Auth, Crypto, Access Control). Use epic=OWASP_A01 for categorization. SEC-BUG tags can link here."
	},
	"Tag: Concern/Finding": {
		"prefix": "s-bug",
		"body": "// SEC-BUG[epic=${1:Finding}, seq=${2:1}, id=${3:finding_id}] ☠️ Finding: ${4:Vulnerability description}$0",
		"description": "ANALYSIS LAYER: Document confirmed vulnerabilities. seq = Severity (1=Critical, 5=Info). id = allows linking from reports. Can link to SEC-SOURCE, SEC-SINK, SEC-AREA, SEC-REVIEW."
	},
	// --- LAYER 4: ORGANIZATION (CONTEXT) ---
	"Tag: Audit Context": {
		"prefix": "s-aud",
		"body": "// SEC-AUDIT[epic=${1:Auth_Module}, seq=${2:1}] 📋 Context: ${3:Auditing authentication module - JWT validation}$0",
		"description": "ORGANIZATION LAYER: File/module context. Use ONE per file. epic = Module/feature name. seq = Audit priority (optional). Describe what you're reviewing."
	},
	"Tag: Review Note": {
		"prefix": "s-rev",
		"body": "// SEC-REVIEW[epic=${1:Query_type}] 🔍 Note: ${2:Questions for second pass}$0",
		"description": "ORGANIZATION LAYER: Questions/notes for later (file-scoped only). Can link to SEC-BUG for fix suggestions. Use for unclear logic or TODOs."
	},
	// --- LAYER 5: STRUCTURAL (LAYOUT) ---
	"Tag: Section Wrapper": {
		"prefix": "s-section",
		"body": [
			"// SEC-SECTION: 📁 ${1:Component_Breakdown}",
			"$TM_SELECTED_TEXT$0",
			"// !SEC-SECTION"
		],
		"description": "STRUCTURAL LAYER: Create spatial boundaries (regions). Use CONSTANTLY - wrap each function/component. File-scoped. Other tags can reference sections via Parent pattern."
	},
	"Tag: Link/Connector": {
		"prefix": "s-link",
		"body": [
			"// SEC-LINK: 🔗 Connector to #${1:target_id}$0"
		],
		"description": "STRUCTURAL LAYER: Hidden navigation link. Place AFTER SEC-SINK, SEC-LOGIC, SEC-BUG, etc. to point back to their SOURCE/LOGIC/AREA. Links enable 'teleporting' between related code."
	},
	// --- LAYER 6: METADATA (PATTERN) ---
	"Tag: Relationship Pattern": {
		"prefix": "s-rel",
		"body": "// SEC-RELATION[epic=${1|Data_Flow,Mirroring,Lifecycle,Parent,Reference,Validation,Exploit_Chain,Remediation,Assumption,AltPath,Composition,Intersection|}, seq=${2:1}] 🔀 Pattern: ${3:Pattern description}$0",
		"description": "METADATA LAYER: Makes relationship patterns VISIBLE in sidebar (unlike hidden SEC-LINK). Use to label what pattern exists (e.g., 'Data Flow pattern in auth'). Pairs with SEC-INSTANCE for instances."
	},
	"Tag: Trace Instance": {
		"prefix": "s-trace",
		"body": "// SEC-TRACE[epic=${1|Data_Flow,Mirroring,Lifecycle,Parent,Reference,Validation,Exploit_Chain,Remediation,Assumption,AltPath,Composition,Intersection|}, id=${2:trace_id}, seq=${3:1}] 🛤️ Trace: ${4:Specific trace description}$0",
		"description": "METADATA LAYER: Mark specific instances of a pattern (e.g., 'Username flow', 'Password flow'). Use when SEC-RELATION defines a pattern and you have multiple examples. Pairs with SEC-RELATION. seq = trace number (1,2,3)."
	}
}
```

---
