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

### Use the Skill Curation Prompt

For curating skills, use `prompts/skill-curation.txt`:

```bash
# Start Hermes
hermes
# Then paste the contents of prompts/skill-curation.txt
```

---

## Step 3 — Verify Skill Installation

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
