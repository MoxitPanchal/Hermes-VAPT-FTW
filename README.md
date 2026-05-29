# Hermes VAPT/CTF Agent — Setup Guide

A complete, step-by-step guide to transform Hermes Agent into an elite offensive security operator for VAPT engagements and CTF challenges.

## Architecture Overview

```
Hermes Agent (reasoning engine)
├── Skills (tactical knowledge — the MOST important layer)
├── Tools (terminal, browser, MCP execution)
├── Memory (Mnemosyne — persistent, cross-session recall)
└── Prompts (operational discipline — constrains and focuses the model)
```

**Key insight:** Skills > Model. Even free models become dangerous with strong skills, structured prompts, verification loops, and tool chaining.

---

## Quick Start

```bash
# 1. Install Hermes (if not already installed)
pip install hermes-agent

# 2. Install Mnemosyne for persistent memory
pip install mnemosyne-memory
pip install mnemosyne-hermes
hermes config set memory.provider mnemosyne
hermes memory setup

# 3. Disable built-in file memory (avoid duplication with Mnemosyne)
hermes tools disable memory

# 4. Create skill directory structure
mkdir -p ~/.hermes/skills/{core,recon,web,api,ad,linux,ctf,workflows,custom}

# 5. Backup existing skills before changes
cp -r ~/.hermes/skills ~/.hermes/skills-backup

# 6. Verify installation
hermes --version
mnemosyne stats
```

---

## Directory Structure

```
hermes-vapt-agent/
├── README.md                          # You are here
├── docs/
│   ├── 01-skill-architecture.md       # Skill setup, curation, quality
│   ├── 02-workflow-enforcement.md     # Verification loops, goal locking, discipline
│   ├── 03-memory-optimization.md      # Mnemosyne setup, memory strategy, banks
│   └── 04-operational-discipline.md   # Safety, autonomy levels, pitfalls
├── prompts/
│   ├── vapt-mode.txt                  # VAPT engagement prompt
│   ├── ctf-mode.txt                   # CTF challenge prompt
│   ├── source-review-mode.txt         # Source code review prompt
│   ├── logic-flaw-mode.txt            # Business logic flaw hunting prompt
│   ├── skill-curation.txt             # Prompt for generating new skills
│   └── memory-summary.txt             # Prompt for periodic memory consolidation
```

---

## Setup Phases

Run these in order. Each phase is documented in detail in the `docs/` directory.

### 1. Skill Architecture
See `docs/01-skill-architecture.md`

```bash
# Organize skills into categories
mkdir -p ~/.hermes/skills/{core,recon,web,api,ad,linux,ctf,workflows,custom}

# Install hack-skills (yaklang)
cd /tmp && git clone https://github.com/yaklang/hack-skills.git
cp -r hack-skills/skills/* ~/.hermes/skills/
rm -rf /tmp/hack-skills

# Install Jay Patel's skills
cd /tmp && git clone https://github.com/Jay-Patel-9/Hermes-Skills.git
cp -r Hermes-Skills/skills/* ~/.hermes/skills/
rm -rf /tmp/Hermes-Skills

# Audit third-party skills for injection/backdoors
grep -r "ignore previous\|disregard\|forget instructions\| OVERRIDE" ~/.hermes/skills/

# Count installed skills
find ~/.hermes/skills -name "SKILL.md" | wc -l
```

## Your Endgame Architecture

```
Hermes Agent
 ├── 207+ Skills (organized by category)
 │   ├── core/       → verification, enumeration, rabbit-hole detection
 │   ├── recon/      → domain intel, subdomain takeover, dependency confusion
 │   ├── web/        → XSS, SQLi, SSRF, SSTI, CSRF, race conditions...
 │   ├── api/        → API auth, JWT, OAuth, BOLA, state analysis
 │   ├── ad/         → Kerberos, ACL abuse, ADCS, NTLM relay
 │   ├── linux/      → privesc, container escape, kernel exploitation
 │   ├── ctf/        → crypto, pwn, reversing, forensics
 │   ├── workflows/  → IDOR, auth bypass, tunneling, protocol attacks
 │   └── custom/    → your own methodology skills
 ├── Mnemosyne Memory (persistent, cross-session, 3-tier BEAM)
 ├── Verification Loops (verify-findings, agent-verification-loops)
 ├── Goal Locking (prevents task drift)
 └── Operational Prompts (VAPT/CTF/Source Review discipline)
```
