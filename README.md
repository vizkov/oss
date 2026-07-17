A methodology I came up with for doing security research and secure code review. It integrates [Comment Anchors](https://marketplace.visualstudio.com/items?itemName=ExodiusStudios.comment-anchors) (for visualization/navigation), [VS Code Snippets](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#vs-code-user-snippets-code-snippetsjson) (for syntax injection/convenience), and [git](https://github.com/vizkov/oss/blob/main/Workflow%20and%20Setup.md#using-git-during-security-research) to create a structured audit approach.

---

<img width="1092" height="346" alt="image" src="https://github.com/user-attachments/assets/a9e11e04-c9a0-4b53-b0f7-e419b8c6fff1" />


<img width="1012" height="1146" alt="image" src="https://github.com/user-attachments/assets/b193c0a7-31c1-4f5f-8211-830eb3d679b5" />

---

# Quick Start

1. Install Comment Anchors extension.
2. Append [settings.json](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#comment-anchors-settingsjson) config to User Settings.
3. Create a new Code Snippet. 
4. Copy [code-snippets.json](https://github.com/vizkov/oss/blob/main/SCR%20Framework.md#vs-code-user-snippets-code-snippetsjson) config to Code Snippet.
5. Test: Type `s-src` in any code file `+ Tab`.

---
