A [methodology](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#the-framework) I came up with for doing security research and secure code review. It uses [Comment Anchors](https://marketplace.visualstudio.com/items?itemName=ExodiusStudios.comment-anchors) (for visualization/navigation), [VS Code Snippets](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#vs-code-user-snippets-code-snippetsjson) (for syntax injection/convenience), and [git](https://github.com/vizkov/oss/blob/main/Workflow%20and%20Setup.md#using-git-during-security-research) to create a structured audit approach.

Enables identification and breakdown of: 
- [Data Flows](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#data-flow-source--sink)
- [Business Logic](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#mirroring-logic--logic)
- [Object states and lifecycle](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#lifecycle-flow--flow)
- [Architecture](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#parent-section--)
- [Attack surfaces](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#reference-area--standard)
- [Exploit Chains](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#exploit-chain-bug--multiple-tags)
- [Remediations](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#remediation-pair-bug--fix-suggestion)
- [Assumptions and Dependencies](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#assumption-dependency-cross-file-logic)
- [Design Patterns](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#intersection-relation--relation--combined-pattern)

---
Example code and comment syntax:

<img width="1092" height="346" alt="image" src="https://github.com/user-attachments/assets/a9e11e04-c9a0-4b53-b0f7-e419b8c6fff1" />

---

Visual render in VS Code:

<img width="1012" height="1146" alt="image" src="https://github.com/user-attachments/assets/b193c0a7-31c1-4f5f-8211-830eb3d679b5" />

---

## Quick Start

1. Install Comment Anchors extension.
2. Append [settings.json](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#comment-anchors-settingsjson) config to User Settings.
3. Create a new Code Snippet. 
4. Copy [code-snippets.json](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#vs-code-user-snippets-code-snippetsjson) config to Code Snippet.
5. Test: Type `s-src` in any code file `+ Tab`.

---
