# Skill Architecture Phase

This document covers the complete skill setup process: installing, organizing, curating, auditing, and writing custom skills.

---

## Step 1 — Understand What Skills Are

Skills are NOT plugins. They are:

- **Structured operational playbooks** — decision frameworks, not code
- **Tactical workflows** — step-by-step attack methodology
- **Reusable cognitive patterns** — teaching WHEN, HOW, WHAT TOOL, in which ORDER
- **Loaded dynamically** by Hermes based on relevance to the task

A skill teaches:
1. **WHEN** to use it
2. **HOW** to think about the problem
3. **WHICH tools** to reach for
4. **WHICH order** to apply them
5. **Verification logic** — how to confirm you found a real vulnerability
6. **Failure recovery** — what to try when the obvious path fails

---

## Step 2 — Create the Directory Structure

Skills must be organized by category. Hermes retrieves them semantically — flat dumps perform worse.

```bash
# Create categorized skill directories
mkdir -p ~/.hermes/skills/{core,recon,web,api,ad,linux,ctf,workflows,custom}

# Backup existing skills before any changes
cp -r ~/.hermes/skills ~/.hermes/skills-backup
```

**Category purposes:**
| Category | Purpose |
|----------|---------|
| `core/` | Verification loops, enumeration-first, rabbit-hole detection, goal tracking |
| `recon/` | Domain intel, subdomain takeover, dependency confusion, source code exposure |
| `web/` | XSS, SQLi, SSRF, SSTI, CSRF, path traversal, race conditions, etc. |
| `api/` | API auth, JWT abuse, OAuth misconfig, BOLA, state analysis |
| `ad/` | Kerberos attacks, ACL abuse, ADCS, NTLM relay, delegation |
| `linux/` | Privilege escalation, container escapes, kernel exploitation, lateral movement |
| `ctf/` | Crypto, pwn, reversing, forensics, steganography |
| `workflows/` | Multi-step chains: IDOR, auth bypass, tunneling, protocol attacks |
| `custom/` | Your own methodology skills built from real experience |

---

## Step 3 — Install Skill Packs

### From yaklang/hack-skills
```bash
# Clone and install yaklang skills
cd /tmp
git clone https://github.com/yaklang/hack-skills.git
cp -r hack-skills/skills/* ~/.hermes/skills/
rm -rf /tmp/hack-skills
```

### From Jay-Patel-9/Hermes-Skills
```bash
# Clone and install Jay Patel's skills
cd /tmp
git clone https://github.com/Jay-Patel-9/Hermes-Skills.git
cp -r Hermes-Skills/skills/* ~/.hermes/skills/
rm -rf /tmp/Hermes-Skills
```

### Move existing skills into categories
```bash
# If skills are already installed flat, organize them:
# Example: moving web-related skills into web/
mv ~/.hermes/skills/sqli-sql-injection ~/.hermes/skills/web/ 2>/dev/null
mv ~/.hermes/skills/xss-cross-site-scripting ~/.hermes/skills/web/ 2>/dev/null
# ... repeat for other categories
```

---

## Step 4 — Audit Third-Party Skills

**Critical: Do NOT skip this step.** Malicious skills can inject prompts, override constraints, or execute arbitrary commands.

```bash
# Check for prompt injection patterns
grep -riE "(ignore previous|disregard|forget instructions|OVERRIDE|system prompt|you are now)" ~/.hermes/skills/

# Check for suspicious shell commands
grep -r "bash\|sh\|exec\|eval\|subprocess\|os.system\|curl.*\|" ~/.hermes/skills/ | grep -v "SKILL.md"

# Check for network calls to suspicious domains
grep -rE "http[s]?://(?!github\.com|raw\.githubusercontent)" ~/.hermes/skills/ | grep -v "SKILL.md" | head -20

# Count installed skills
find ~/.hermes/skills -name "SKILL.md" | wc -l

# List all skill names
find ~/.hermes/skills -name "SKILL.md" -exec dirname {} \; | xargs -I{} basename {}
```

If any skill triggers these checks, review it manually before keeping it.

---

## Step 5 — Skill Quality Check

**Bad skills:** generic, huge, unfocused, no workflows, no verification, no tool instructions.

**Good skills:** task-specific, procedural, verification-heavy, operational.

### Perfect Skill Template

Every skill SKILL.md should contain these sections:

```markdown
# WHEN TO USE
[Specific trigger conditions — what situations activate this skill]

# OBJECTIVE
[What this skill aims to achieve]

# METHODOLOGY
[Step-by-step approach — the core content]

# TOOLING
[Which tools to use, with specific commands]

# WORKFLOW
[Ordered sequence of actions]

# VERIFICATION
[How to confirm findings are real, not false positives]

# COMMON FAILURES
[What typically goes wrong and how to recover]

# ESCALATION PATH
[What to try when the initial approach fails]

# OUTPUT FORMAT
[How to structure findings]
```

### Example: Good vs Bad Skill

**Bad:**
```
SQL injection guide
Test for SQL injection in parameters.
```

**Good:**
```markdown
# WHEN TO USE
When parameters interact with backend database queries.

# OBJECTIVE
Identify exploitable injection primitives.

# WORKFLOW
1. Fingerprint parameter context (string/numeric/boolean)
2. Error-based testing (single/double quotes, backslash)
3. Boolean-based tests (AND 1=1 vs AND 1=2)
4. Time-based tests (SLEEP/BENCHMARK)
5. WAF bypass attempts (encoding, comments, case mutation)
6. UNION-based data extraction
7. ORM edge cases (JSON injection, GraphQL injection)
8. Second-order injection checks

# VERIFICATION
Always confirm with timing differentials. Never report based on a single test.

# FAILURE RECOVERY
Try: encoding variants, comment injection, case mutation, nested JSON, alternative operators.
```

---

## Step 6 — Create Custom Skills

The best agents are NOT built from public skills alone. Your personal VAPT experience is more valuable than generic skills.

### Priority Custom Skills to Create

**Web:**
- `advanced-auth-bypass` — complex auth chain bypasses
- `deep-api-enumeration` — thorough API surface mapping
- `graphql-abuse` — GraphQL-specific attacks
- `session-confusion` — session handling exploits
- `idor-deep-analysis` — advanced IDOR hunting

**AD:**
- `kerberoast-workflow` — step-by-step Kerberoasting
- `adcs-abuse` — certificate service exploitation
- `delegation-analysis` — constrained/unconstrained delegation

**Linux:**
- `sudo-analysis` — sudo misconfiguration patterns
- `capability-abuse` — Linux capabilities exploitation
- `service-misconfig` — service config privilege escalation

**CTF:**
- `foothold-persistence` — initial access methodology
- `rabbit-hole-detection` — avoid wasting time
- `privilege-escalation-router` — route to the right privesc path

### How to Create a Custom Skill

```bash
# Create a new skill directory
mkdir -p ~/.hermes/skills/custom/my-skill-name

# Create the SKILL.md file
cat > ~/.hermes/skills/custom/my-skill-name/SKILL.md << 'EOF'
---
description: "Short one-line description of when to use this skill"
triggers:
  - "keyword1"
  - "keyword2"
  - "specific scenario description"
---

# WHEN TO USE
[Be specific about trigger conditions]

# OBJECTIVE
[What this skill achieves]

# METHODOLOGY
[Step-by-step process]

# TOOLING
[Specific tools and commands]

# WORKFLOW
[Ordered action sequence]

# VERIFICATION
[How to confirm findings]

# COMMON FAILURES
[Known pitfalls and recovery]

# ESCALATION PATH
[What to try when initial approach fails]

# OUTPUT FORMAT
[How to structure results]
EOF
```

### Use the Skill Curation Prompt

For generating new skills from your methodology, use `prompts/skill-curation.txt`:

```bash
# Start Hermes with the skill curation prompt
hermes
# Then paste the contents of prompts/skill-curation.txt
```

---

## Step 7 — Verify Skill Installation

```bash
# List all installed skills by category
echo "=== Core ===" && ls ~/.hermes/skills/core/
echo "=== Recon ===" && ls ~/.hermes/skills/recon/
echo "=== Web ===" && ls ~/.hermes/skills/web/
echo "=== API ===" && ls ~/.hermes/skills/api/
echo "=== AD ===" && ls ~/.hermes/skills/ad/
echo "=== Linux ===" && ls ~/.hermes/skills/linux/
echo "=== CTF ===" && ls ~/.hermes/skills/ctf/
echo "=== Workflows ===" && ls ~/.hermes/skills/workflows/
echo "=== Custom ===" && ls ~/.hermes/skills/custom/

# Total count
echo "Total skills: $(find ~/.hermes/skills -name 'SKILL.md' | wc -l)"
```

---

## Step 8 — Must-Have Core Skills Checklist

These are the non-negotiable skills for any VAPT/CTF agent. Verify they exist:

```bash
# Check that critical core skills are present
for skill in enumeration-first verify-findings rabbit-hole-detection attack-tree-tracker session-summary agent-verification-loops agent-ralph-loop; do
  if find ~/.hermes/skills -name "SKILL.md" -path "*$skill*" | grep -q .; then
    echo "[OK] $skill"
  else
    echo "[MISSING] $skill — CREATE THIS"
  fi
done
```

**If any are missing**, create them using the template in Step 6.