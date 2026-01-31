# Skills

AI agent skills for Claude Code and GitHub Copilot.

Each skill queries its corresponding Spec dataset via the Spice CLI to provide expert guidance.

## Available Skills

| Skill | Description | Dataset |
|-------|-------------|---------|
| `security-expert` | Security recommendations for HIPAA, HITRUST, SOC2 | `security_spec` |
| `cloud-expert` | Cloud architecture and infrastructure guidance | `cloud_spec` |
| `quality-expert` | Quality assurance and testing guidance | `quality_spec` |
| `ux-design-expert` | UX design recommendations and standards | `ux_design_spec` |

## Requirements

- Spice CLI installed and configured
- SpiceManager spicepod running (`spice run` in SpiceManager directory)

---

## Setup for Claude Code

Claude Code discovers skills from directories containing a `SKILL.md` file with YAML frontmatter.

### Installation

**Option 1: Personal skills (available in all projects)**

```bash
# Create skill directories
mkdir -p ~/.claude/skills/security-expert
mkdir -p ~/.claude/skills/cloud-expert
mkdir -p ~/.claude/skills/quality-expert
mkdir -p ~/.claude/skills/ux-design-expert

# Copy and convert each skill (add frontmatter)
for skill in security-expert cloud-expert quality-expert ux-design-expert; do
  cat > ~/.claude/skills/$skill/SKILL.md << EOF
---
name: $skill
description: $(head -1 /path/to/ExpertOrgPOC/Skills/$skill.md | sed 's/# //' | sed 's/ Skill//')
---

$(cat /path/to/ExpertOrgPOC/Skills/$skill.md)
EOF
done
```

**Option 2: Project-scoped skills (available only in this project)**

```bash
# From the project root
mkdir -p .claude/skills/security-expert
# ... repeat for each skill

# Copy with frontmatter as shown above
```

### SKILL.md Format

Each skill file requires YAML frontmatter:

```markdown
---
name: security-expert
description: Security expert for HIPAA, HITRUST, and SOC2 compliance
---

# Security Expert Skill

[skill content...]
```

### Usage

Once installed, skills appear in `/skills` and can be invoked:

```
/security-expert
/cloud-expert
/quality-expert
/ux-design-expert
```

Claude will also automatically load relevant skills based on conversation context.

### Frontmatter Options

| Option | Description |
|--------|-------------|
| `name` | Skill identifier (lowercase, hyphens allowed) |
| `description` | Brief description shown in skill list |
| `disable-model-invocation: true` | Only user can invoke (not auto-loaded) |
| `user-invocable: false` | Only Claude can invoke (background knowledge) |

---

## Setup for GitHub Copilot

GitHub Copilot uses instruction files to customize behavior.

### Option 1: Repository Instructions (Recommended)

Create a `.github/copilot-instructions.md` file in your repository root:

```markdown
# Copilot Instructions

## Expert Skills

When asked about security, cloud, quality, or UX topics, use the Spice CLI
to query the relevant dataset.

### Security Questions
Use hybrid RRF search on the security_spec dataset:
\`\`\`bash
echo "SELECT path, content, fused_score
FROM rrf(
    vector_search(security_spec, '<semantic query>'),
    text_search(security_spec, '<keywords>', content),
    join_key => 'path'
)
ORDER BY fused_score DESC;" | spice sql
\`\`\`

### Cloud Questions
Query cloud_spec dataset with hybrid search...

### Quality Questions
Query quality_spec dataset with hybrid search...

### UX Design Questions
Query ux_design_spec dataset with hybrid search...
```

### Option 2: VS Code Workspace Instructions

Add to `.vscode/settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "Skills/security-expert.md"
    },
    {
      "file": "Skills/cloud-expert.md"
    },
    {
      "file": "Skills/quality-expert.md"
    },
    {
      "file": "Skills/ux-design-expert.md"
    }
  ]
}
```

### Option 3: Custom Chat Participants (VS Code Extension)

For more advanced integration, create a VS Code extension that registers custom chat participants. See [GitHub Copilot Extensibility](https://docs.github.com/en/copilot/building-copilot-extensions).

### Usage

With repository instructions, Copilot Chat will automatically use the guidance when you ask relevant questions:

```
@workspace How should I implement authentication for HIPAA compliance?
```

---

## Skill File Reference

Each `.md` file in this directory follows this structure:

1. **Title** - Skill name and role
2. **Instructions** - When and how to use the skill
3. **Commands** - Spice SQL queries with hybrid RRF search
4. **Behavior** - Step-by-step guidance for the AI
