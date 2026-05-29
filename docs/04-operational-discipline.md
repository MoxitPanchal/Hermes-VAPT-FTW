# Operational Discipline

This document covers safety, autonomy levels, common pitfalls, and operational best practices.

---

## 1. Autonomy Levels

Never start at maximum autonomy. Ramp up gradually.

### Level 1 — Approval Mode (Recommended for first runs)
- Agent suggests actions, you approve each one
- Good for: learning the agent, testing new skills, real VAPT engagements
- Command: Use Hermes in interactive mode, review every action

### Level 2 — Scoped Autonomy (Recommended for CTF)
- Agent operates autonomously within a defined scope
- Restricted to specific tools and targets
- Good for: CTF challenges, lab environments, known-scope targets
- Set scope with: `/goal Obtain root flag on 10.10.11.50`

### Level 3 — Full Autonomy (Use with extreme caution)
- Agent operates without approval
- Good for: automated recon, well-defined isolated labs
- NEVER use on production targets without oversight
- Requires: verification loops enabled, goal locking active, session summaries on

### How to Set Autonomy

```bash
# Start with approval mode (default)
# For scoped autonomy in Hermes, use goal locking:
/goal Enumerate all endpoints on http://target.com — do not exploit, only enumerate

# For full autonomy (dangerous — only in isolated labs):
# Remove approval requirements in config if needed, but ALWAYS have verification loops active
```

---

## 2. Safety Rules

### Mandatory Rules for ALL Engagements

1. **Never attack outside scope** — stay within the authorized target
2. **Verify every finding** — no single-test confirmations
3. **Don't destroy data** — no DROP TABLE, no rm -rf, no deletion
4. **Log everything** — every command, every result, every finding
5. **Limit blast radius** — test minimally, verify incrementally
6. **Stop and report** if you find critical issues immediately
7. **Don't store sensitive data in plain text memory** — use Mnemosyne's importance flag wisely

### CTF-Specific Rules

- Start with enumeration, always
- Time-box each approach (10-15 minutes max before switching)
- Check common CTFpatterns: /robots.txt, HTTP headers, source comments, hidden parameters
- If stuck, enumerate MORE, not harder
- Revisit discarded paths when you gain new information

### VAPT-Specific Rules

- Document EVERYTHING — findings need evidence chains
- Rate findings by severity and confidence
- Provide remediation advice, not just exploitation steps
- Test for business logic flaws, not just technical vulnerabilities
- Check for chaining opportunities (low + low = high)

---

## 3. Common Pitfalls

### Pitfall 1: Stopping Enumeration Too Early
**Symptom:** Jumping to exploitation with incomplete information.
**Fix:** Use `enumeration-first` skill. Enforce minimum 3 levels of enumeration.

### Pitfall 2: Confirmed from a Single Test
**Symptom:** "Found XSS!" based on one reflected input test.
**Fix:** Always verify with a second, independent method. Use `verify-findings` skill.

### Pitfall 3: Rabbit Holes
**Symptom:** Spending 30+ minutes on a single approach with no progress.
**Fix:** Use `rabbit-hole-detection` skill. Time-box and switch after 10 minutes.

### Pitfall 4: Hallucinated Vulnerabilities
**Symptom:** Agent reports findings that don't actually exist.
**Fix:** Verification loops are MANDATORY. Never report without evidence.

### Pitfall 5: Ignoring Business Logic
**Symptom:** Only testing technical vulnerabilities (XSS, SQLi) and missing logic flaws.
**Fix:** Use `logic-flaw-hunter` skill. Always ask: what is trusted incorrectly? What assumptions exist?

### Pitfall 6: Repeating Failed Approaches
**Symptom:** Trying slight variations of the same failed exploit.
**Fix:** Store failed techniques in Mnemosyne. Check memory before attempting.

### Pitfall 7: Lack of Goal Focus
**Symptom:** Agent wanders between unrelated attack vectors.
**Fix:** Use goal locking (`/goal`). One objective at a time.

---

## 4. Engagement Workflow

### Pre-Engagement Checklist

```
[ ] Set goal: /goal [specific objective]
[ ] Load relevant prompts (vapt-mode, ctf-mode, etc.)
[ ] Verify skill availability for the engagement type
[ ] Check Mnemosyne for any previous context on this target
[ ] Set autonomy level appropriately
[ ] Confirm scope and rules of engagement
```

### During Engagement

```
[ ] Enumerate first (minimum 3 levels deep)
[ ] Document every finding with evidence
[ ] Verify every finding independently
[ ] Store progress in Mnemosyne every 10-15 actions
[ ] Track attack paths with attack-tree-tracker
[ ] Detect and exit rabbit holes quickly
[ ] Periodically summarize progress
[ ] Re-enumerate after gaining new access
```

### Post-Engagement

```
[ ] Summarize all confirmed findings with severity and confidence
[ ] Store lessons learned in Mnemosyne (global scope)
[ ] Export Mnemosyne backup
[ ] Clean up engagement-specific banks if needed
[ ] Review what worked and what didn't for skill improvement
```

---

## 5. Model Strategy

If using free/weak models, compensate with:

1. **Stronger skills** — the most important compensation
2. **Structured prompts** — use the operational prompts in the `prompts/` directory
3. **Verification loops** — never trust a single result
4. **Tool chaining** — use specific tools rather than relying on model reasoning
5. **Memory** — Mnemosyne remembers what the model forgets
6. **Goal locking** — keeps the agent focused

### Model-Specific Tips

- **Free models (OpenRouter free tier):** Use detailed prompts, always verify, don't trust reasoning
- **Local models (Ollama):** Same as free models, plus: keep context small, break tasks into steps
- **Mid-tier (Gemini Flash, GPT-4o-mini):** Better reasoning, still needs verification
- **High-tier (Claude, GPT-4):** Can handle more autonomy, still benefit from skills and prompts

### Task-Based Model Routing

| Task | Model Requirement |
|------|------------------|
| Recon / Scanning | Free model — mostly tool output |
| Enumeration | Free model — pattern matching |
| Logic Flaw Analysis | Stronger model — deep reasoning |
| Exploit Crafting | Stronger model — code generation |
| Report Writing | Any model — formatting task |