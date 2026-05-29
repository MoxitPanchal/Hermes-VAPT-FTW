# Workflow Enforcement Phase

Workflow enforcement is what separates a dangerous agent from a ChatGPT with tools. This covers verification loops, goal locking, session management, and operational patterns.

---

## What Is Workflow Enforcement?

Without enforcement, agents:
- Stop enumerating too early
- Chase rabbit holes
- Report false positives
- Hallucinate vulnerabilities
- Lose track of findings across sessions

With enforcement, agents:
- Enumerate deeply before exploiting
- Verify every finding before reporting
- Track attack paths systematically
- Summarize progress periodically
- Recover from failures with alternative approaches

---

## Step 1 — Verification Loops (CRITICAL)

Verification is the most important enforcement mechanism. It prevents hallucinated vulnerabilities and false positives.

### How Verification Loops Work

For EVERY finding, the agent must:
1. **Confirm the finding** with a second, independent test
2. **Check for alternatives** — could this be a false positive?
3. **Document the evidence** — what command, what output, what reasoning
4. **Rate confidence** — confirmed / probable / possible / unlikely

### The `verify-findings` Skill

This skill (already in `~/.hermes/skills/core/`) enforces:
- Never report a vulnerability from a single test
- Always re-test with a different method
- Check for environmental factors that could cause false positives
- Provide evidence chain for every claim

### The `agent-verification-loops` Skill

This skill enforces systematic verification after every action:
- Observe result → Classify confidence → Verify if needed → Escalate if uncertain
- Never skip verification for "obvious" findings
- Use multiple tools to cross-validate

### Enable These Skills

```bash
# Verify they exist
ls ~/.hermes/skills/core/verify-findings/SKILL.md
ls ~/.hermes/skills/core/agent-verification-loops/SKILL.md

# If missing, they are likely already loaded by Hermes.
# Check they're active:
hermes skills list | grep -E "verify|verification"
```

---

## Step 2 — Goal Locking

Goal locking prevents the agent from drifting off-task. Use `/goal` to set a clear, specific objective.

### How to Use Goal Locking

Start every engagement by setting a goal:

```
/goal Obtain authenticated admin access through business logic flaws only
```

```
/goal Find the flag in this CTF challenge through enumeration-first methodology
```

```
/goal Map all API endpoints and identify authorization bypasses
```

### Goal Locking Rules

1. **Set a goal before starting** — every engagement needs one
2. **Make goals specific** — not "find bugs" but "find auth bypasses in the admin panel"
3. **Don't change goals mid-engagement** unless the current goal is truly completed
4. **Track progress toward the goal** — don't get distracted by interesting side paths
5. **When the goal is achieved**, set a new one or close the engagement

---

## Step 3 — The Ralph Loop (Session Continuity)

The Ralph Loop enables multi-session continuity. It uses the filesystem as state storage.

### How It Works

```
Session 1: Start → Enumerate → Find target → Dump state to file
Session 2: Load state → Continue from where you left off → Go deeper → Dump state
Session 3: Load state → Exploit → Dump state
```

### Using the Ralph Loop Skill

```bash
# Verify the skill exists
ls ~/.hermes/skills/core/agent-ralph-loop/SKILL.md
```

The skill teaches the agent to:
- Save progress to a state file at key checkpoints
- Load state from file when resuming
- Summarize findings, failed approaches, and next steps
- Never repeat work already completed

---

## Step 4 — Session Summary Skill

Long engagements generate massive amounts of context. The session summary skill forces periodic summarization.

### When to Summarize

- Every 10-15 significant actions
- After completing an attack phase (recon, enumeration, exploitation)
- Before switching to a new attack vector
- When context is getting large
- At the end of a session

### What to Summarize

```
## Engagement Summary
- **Target:** [URL/IP]
- **Phase:** [recon/enumeration/exploitation/post-exploitation]
- **Findings:** [numbered list of confirmed findings]
- **Failed approaches:** [what didn't work and why]
- **Assumptions:** [what we're assuming and should verify]
- **Attack surface:** [what we've mapped so far]
- **Next steps:** [ordered list of what to try next]
```

### Save Summaries to Mnemosyne

```bash
# After summarizing, save to Mnemosyne for cross-session recall
mnemosyne remember "Target: example.com | Phase: recon | Findings: admin panel at /admin, API at /api/v2 | Failed: SQLi on login, XSS on search | Next: test IDOR on API endpoints"
```

---

## Step 5 — Attack Tree Tracking

The `attack-tree-tracker` skill maintains a structured tree of attack paths.

### Structure

```
Root Goal: Get admin access
├── Path 1: Auth bypass
│   ├── 1a: JWT manipulation → FAILED (no JWT)
│   ├── 1b: Session fixation → TESTING
│   └── 1c: IDOR on user roles → NOT STARTED
├── Path 2: Logic flaws
│   ├── 2a: Password reset manipulation → NOT STARTED
│   └── 2b: OTP bypass → NOT STARTED
└── Path 3: Direct admin panel access
    ├── 3a: Forceful browsing → CONFIRMED (302 redirect, try bypass)
    └── 3b: Default credentials → FAILED
```

### Why This Matters

- Prevents repeating failed approaches
- Shows progress visually
- Helps prioritize next steps
- Makes handing off between sessions clean

---

## Step 6 — Enumeration-First Discipline

The most common agent failure is stopping enumeration too early.

### The Rule

**ALWAYS enumerate deeper before attempting exploitation.**

The `enumeration-first` skill enforces:
1. Never attempt exploitation before mapping the full attack surface
2. For every endpoint discovered, check for hidden parameters, alternate methods, and related endpoints
3. Enumerate at least 3 levels deep
4. Document everything before moving to exploitation
5. Re-enumerate after gaining new access levels

### Enumeration Depth Checklist

```
Level 1: Port scan, directory brute, technology fingerprint
Level 2: Parameter discovery, hidden endpoints, API documentation
Level 3: Authentication flows, role differences, parameter relationships
Level 4: State transitions, business logic, trust boundaries
```

---

## Step 7 — Rabbit-Hole Detection

Agents waste time on dead ends. The `rabbit-hole-detection` skill forces:

1. **Time-box each approach** — if no progress in N minutes, switch
2. **Track effort-per-finding ratio** — low ratio = potential rabbit hole
3. **Force pausing** every 5-10 actions to reassess: "Is this the best use of time?"
4. **Escalate** to a different attack vector when stuck
5. **Never revisit** failed approaches without new information

### When to Abandon

- Same test with slight variations 3+ times with no new results
- Spending more than 10 minutes on a single endpoint without progress
- No new findings in the last 5 actions
- The finding requires conditions that don't exist on the target

---

## Step 8 — Putting It All Together

The complete workflow enforcement stack:

```
1. /goal [specific objective]                    # Set goal lock
2. Load context from Mnemosyne if resuming       # Memory continuity
3. Begin enumeration-first methodology            # Deep recon
4. Track attack paths with attack-tree-tracker    # Systematic approach
5. Verify EVERY finding with verify-findings      # No false positives
6. Periodically summarize with session-summary    # Context management
7. Detect rabbit holes and abandon dead ends     # Time efficiency
8. Save state with ralph-loop between sessions    # Persistence
9. Store all findings in Mnemosyne               # Cross-session memory
10. Confirm goal achieved or set new goal         # Progress tracking
```

### Verification Checklist

Run through this after every significant finding:

- [ ] Can I reproduce this with a different method?
- [ ] Could this be a false positive due to environment factors?
- [ ] Did I document the exact command, output, and reasoning?
- [ ] Am I reporting facts or assumptions?
- [ ] Is this the highest-priority finding to pursue right now?