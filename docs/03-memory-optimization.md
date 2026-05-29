# Memory Optimization with Mnemosyne

Mnemosyne provides persistent, cross-session memory for Hermes. It uses a 3-tier BEAM architecture (Working → Episodic → TripleStore) backed by a single SQLite file with hybrid search (vector + FTS5 + importance scoring).

---

## Step 1 — Install Mnemosyne

```bash
# Install core + Hermes integration
pip install mnemosyne-memory mnemosyne-hermes

# Configure Hermes to use Mnemosyne as memory provider
hermes config set memory.provider mnemosyne

# Initialize Mnemosyne
hermes memory setup

# Disable Hermes built-in file memory (avoids duplication)
hermes tools disable memory

# Verify it works
mnemosyne remember "Test: Hermes VAPT memory operational"
mnemosyne recall "Hermes"
mnemosyne stats
```

---

## Step 2 — Configure Mnemosyne

Create or edit the Mnemosyne config for VAPT/CTF workloads:

```bash
# Set data directory (default is fine, but you can customize)
export MNEMOSYNE_DATA_DIR="$HOME/.hermes/mnemosyne/data"

# Optimize weights for security work
# Vector similarity: 50% (semantic matching of attack patterns)
# FTS5 keywords: 30% (exact tool/command/technique recall)
# Importance: 20% (prioritize high-value findings)
export MNEMOSYNE_VEC_WEIGHT=0.5
export MNEMOSYNE_FTS_WEIGHT=0.3
export MNEMOSYNE_IMPORTANCE_WEIGHT=0.2

# Increase working memory limit for long engagements
export MNEMOSYNE_WM_MAX_ITEMS=10000

# Recency halflife: 168 hours (1 week) — findings from this week matter most
export MNEMOSYNE_RECENCY_HALFLIFE=168
```

Add these to your `~/.bashrc` or `~/.zshrc`:
```bash
# Mnemosyne configuration for VAPT agent
export MNEMOSYNE_DATA_DIR="$HOME/.hermes/mnemosyne/data"
export MNEMOSYNE_VEC_WEIGHT=0.5
export MNEMOSYNE_FTS_WEIGHT=0.3
export MNEMOSYNE_IMPORTANCE_WEIGHT=0.2
export MNEMOSYNE_WM_MAX_ITEMS=10000
export MNEMOSYNE_RECENCY_HALFLIFE=168
```

---

## Step 3 — Memory Categories for VAPT/CTF

Organize your memory into these categories. Store findings in the right category so recall is precise.

| Category | What to Store | Importance |
|-----------|--------------|------------|
| **Targets** | URLs, IPs, domains, scope | 0.9 |
| **Credentials** | Found credentials, tokens, API keys | 0.95 |
| **Endpoints** | Discovered endpoints, parameters, API routes | 0.8 |
| **Findings** | Confirmed vulnerabilities with evidence | 0.9 |
| **Failed Techniques** | What didn't work and why | 0.6 |
| **Attack Paths** | Active attack trees and progress | 0.85 |
| **Assumptions** | What we're assuming (must verify later) | 0.7 |
| **Lessons Learned** | Patterns across engagements | 0.75 |
| **Tool Results** | Scan outputs, brute force results | 0.5 |

### Storing Memories (Mnemosyne CLI)

```bash
# Store a target
mnemosyne remember "Target: app.example.com — Node.js/Express backend, PostgreSQL, React frontend" --importance 0.9

# Store credentials
mnemosyne remember "Credential: admin:Password123@app.example.com found via default credentials on /admin" --importance 0.95

# Store a confirmed finding
mnemosyne remember "Finding: IDOR on /api/v2/users/{id} — can access other users' profiles by changing the ID parameter, confirmed with user IDs 1-5" --importance 0.9

# Store a failed approach
mnemosyne remember "Failed: SQLi on /api/login — parameter is sanitized, tried boolean/time-based/error-based, all failed. WAF present." --importance 0.6

# Store with entity extraction
mnemosyne remember "Met with client about auth bypass on admin panel — they use OAuth2 with GitHub provider" --importance 0.8

# Store global facts (cross-session, cross-engagement)
mnemosyne remember "Common CTF pattern: flags often hidden in /robots.txt, HTTP headers, or comments in source" --importance 0.75 --scope global
```

### Recalling Memories

```bash
# Recall by keyword
mnemosyne recall "IDOR"
mnemosyne recall "credentials example.com"

# Recall with temporal boost (recent findings weighted higher)
mnemosyne recall "findings" --temporal-weight 0.5 --temporal-halflife 48

# Check memory stats
mnemosyne stats
```

---

## Step 4 — Memory Strategy During Engagements

### When to Store

- **Immediately** when you find credentials, tokens, or access
- **After** confirming a vulnerability (not before — avoid storing false leads)
- **When** switching attack vectors (store current state)
- **Before** ending a session (dump all progress)
- **When** a technique fails (so you don't repeat it)

### When to Recall

- **At the start** of every session (load previous context)
- **Before** trying an attack vector (check if it was already attempted)
- **When** hitting a dead end (recall alternative approaches)
- **When** finding a new endpoint (recall related patterns)

### Memory Summarization Prompt

Every 10-15 actions during an engagement, run this:

```
Summarize current engagement state:
1. What have we confirmed?
2. What have we tried and failed?
3. What attack surface have we mapped?
4. What are our current assumptions?
5. What are the next 3 actions to take?
```

Then store the summary:
```bash
mnemosyne remember "Engagement summary for [target]: [1-2 sentence summary of phase, key findings, next steps]" --importance 0.85
```

---

## Step 5 — Memory Banks (Per-Domain Isolation)

Mnemosyne supports "banks" — isolated memory namespaces. Use them to separate engagement data.

```python
from mnemosyne.core.banks import BankManager
from mnemosyne import Mnemosyne

# Create a bank for a specific engagement
BankManager().create_bank("engagement-client1")
client1_mem = Mnemosyne(bank="engagement-client1")
client1_mem.remember("Target: client1.example.com — scope includes admin panel and API")

# Create a bank for CTF
BankManager().create_bank("ctf-htb")
htb_mem = Mnemosyne(bank="ctf-htb")
htb_mem.remember("HTB起点: 10.10.11.50 — web app on port 8080")
```

### When to Use Banks

- Separate client data in real VAPT engagements (compliance)
- Separate CTF challenges from each other
- Keep methodology notes in a global bank
- Isolate sensitive credential stores

### Bank Operations

```bash
# Create via CLI (if supported)
# Otherwise use Python API as shown above

# Export a bank for backup
mnemosyne export --output backup-client1.json

# Import from backup
mnemosyne import --input backup-client1.json
```

---

## Step 6 — Knowledge Graph (TripleStore)

For complex engagements, use the TripleStore to track entity relationships.

```python
from mnemosyne.core.triples import TripleStore

kg = TripleStore()

# Track attack path relationships
kg.add("admin-panel", "accessible_via", "forceful-browsing", valid_from="2024-01-15")
kg.add("forceful-browsing", "leads_to", "IDOR-vulnerability")
kg.add("IDOR-vulnerability", "exploits", "user-api-endpoint")
kg.add("user-api-endpoint", "authenticated_as", "regular-user")

# Query relationships
kg.query("admin-panel")          # What do we know about admin-panel?
kg.query("admin-panel", as_of="2024-01-20")  # As of a specific date
```

---

## Step 7 — Periodic Maintenance

```bash
# Run consolidation (moves working memory to episodic)
mnemosyne sleep

# Check memory health
mnemosyne stats

# Export backup before major changes
mnemosyne export --output mnemosyne-backup-$(date +%Y%m%d).json

# Diagnose issues
mnemosyne diagnose
```

### When to Run Maintenance

- **After** each engagement (consolidate and backup)
- **Weekly** if doing continuous testing
- **Before** upgrading Mnemosyne version
- **When** recall seems slow or irrelevant (consolidation helps)