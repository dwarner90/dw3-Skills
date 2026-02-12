# Expert Skills

AI agent skills for Claude Code that query organizational spec datasets via SpiceAI.

## Available Skills

| Skill | Description | Dataset |
|-------|-------------|---------|
| `security-expert` | HIPAA, HITRUST, SOC2 compliance guidance | `security_spec` |
| `cloud-expert` | AWS infrastructure and architecture | `cloud_spec` |
| `quality-expert` | QA and testing standards | `quality_spec` |
| `ux-design-expert` | UX design and accessibility | `ux_design_spec` |

## Requirements

- [Spice CLI](https://docs.spiceai.org/getting-started) installed
- SpiceManager running (`cd SpiceManager && spice run`) OR `SPICE_CLOUD_API_KEY` set

---

## Installation for Claude Code

### Option 1: Install from .skill packages (Recommended)

Download the `.skill` files from `dist/` and install:

```bash
# Personal skills (available in all projects)
unzip security-expert.skill -d ~/.claude/skills/
unzip cloud-expert.skill -d ~/.claude/skills/
unzip quality-expert.skill -d ~/.claude/skills/
unzip ux-design-expert.skill -d ~/.claude/skills/
```

Or for project-scoped skills:

```bash
# Project skills (available only in this project)
unzip security-expert.skill -d .claude/skills/
```

### Option 2: Copy skill directories directly

```bash
# Copy from this repo to your Claude skills directory
cp -r security-expert ~/.claude/skills/
cp -r cloud-expert ~/.claude/skills/
cp -r quality-expert ~/.claude/skills/
cp -r ux-design-expert ~/.claude/skills/
```

### Verify Installation

Run `/skills` in Claude Code to see installed skills, or invoke directly:

```
/security-expert
/cloud-expert
/quality-expert
/ux-design-expert
```

Skills also auto-trigger based on conversation context.

---

## Setup for GitHub Copilot

### Repository Instructions

Create `.github/copilot-instructions.md` in your repository:

```markdown
# Copilot Instructions

When asked about security, cloud, quality, or UX topics, use the Spice CLI
to query the relevant dataset with hybrid RRF search.

## Security Questions
Query security_spec dataset:
\`\`\`bash
echo "SELECT path, content, fused_score
FROM rrf(
    vector_search(security_spec, '<semantic query>'),
    text_search(security_spec, '<keywords>', content),
    join_key => 'path'
)
ORDER BY fused_score DESC
LIMIT 10;" | spice sql
\`\`\`

## Cloud Questions
Query cloud_spec dataset (same pattern)

## Quality Questions
Query quality_spec dataset (same pattern)

## UX Design Questions
Query ux_design_spec dataset (same pattern)
```

### VS Code Workspace Instructions

Add to `.vscode/settings.json`:

```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    { "file": "Skills/security-expert/SKILL.md" },
    { "file": "Skills/cloud-expert/SKILL.md" },
    { "file": "Skills/quality-expert/SKILL.md" },
    { "file": "Skills/ux-design-expert/SKILL.md" }
  ]
}
```

---

## Skill Structure

Each skill follows the Claude Code skill format:

```
skill-name/
└── SKILL.md          # Skill definition with YAML frontmatter
```

The SKILL.md includes:
- **Frontmatter**: `name` and `description` (triggers auto-loading)
- **Query template**: Hybrid RRF search pattern
- **Workflow**: Steps for synthesizing results
- **Organization context**: Standards and constraints

---

## Building .skill Packages

To rebuild the distributable packages:

```bash
cd Skills
python3 << 'EOF'
import zipfile
from pathlib import Path

for skill in ['security-expert', 'cloud-expert', 'quality-expert', 'ux-design-expert']:
    with zipfile.ZipFile(f'dist/{skill}.skill', 'w', zipfile.ZIP_DEFLATED) as z:
        for f in Path(skill).rglob('*'):
            if f.is_file():
                z.write(f, f.relative_to(Path(skill).parent))
EOF
```
