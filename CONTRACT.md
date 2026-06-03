# The Personalization Contract

*This is the rule that lets the blueprint and the home stay separate forever.*

The README explains the idea: there are two things, the **blueprint** and the **home**, and they must never blur. This file makes that idea operational. It is the contract every change must pass before it moves between the two.

---

## Two repositories, one boundary

There are two instances of this design, and they are not the same kind of thing.

- **The template** (`the_intern`, this repo) — the *shareable skeleton*. The framing, the room layout, the conventions, the empty schemas. It is meant to be cloned, read, and built upon by anyone. It contains no life.
- **The master** (`~/p/master`, private, never published) — the *private overlay* laid over that skeleton. The real instructions, the real people, the real memory, the real projects. It is the home actually lived in. It is never shared.

The template is **SHAPE**. The master is SHAPE **plus** SOUL. The whole job of this contract is to keep those two ledgers honest:

> **Back-port shape. Never soul.**

When the master's *structure* improves — a better section in `CLAUDE.md`, a new doc convention, a cleaner memory prefix — that improvement is **shape**, and it should flow *back* into the template so the blueprint stays current. When the master's *content* grows — a new contact, a memory about a real person, a project holding a real trip — that is **soul**, and it must **never** leave the master. Not in a commit, not in an example, not in a comment, not in a screenshot.

The arrow runs **one way only**:

```
   master (~/p/master)                 template (the_intern)
   SHAPE + SOUL                         SHAPE only
        │                                    ▲
        │   back-port SHAPE  ────────────────┘
        │   (structure, conventions, schemas)
        │
        └─ SOUL stays here. Forever.
           (real people, real memory, real secrets, real projects)
```

There is no reverse arrow carrying soul out. The template improves the master only by being a clean reference; the master improves the template only by donating structure stripped of all content.

---

## The line, stated plainly

**SHAPE** is anything that would be *equally true in someone else's house*. The shape of a room is the same whether you or a stranger lives in it. If a change describes *how the house is built* — its conventions, its skeletons, its empty forms — it is shape, and it belongs in the template.

**SOUL** is anything that is *true only in this house*. The people, the memories, the specifics of a real life. If a change describes *who lives here or what they did* — a name, a number, a habit, a trip, a secret — it is soul, and it stays in the master.

A one-line test for any line of any file:

> *Would this be just as correct in a stranger's clone of the repo?*
> Yes → shape, may be back-ported. No → soul, must never leave.

---

## Allowlist: what counts as SHAPE

These are the things that may be back-ported from master to template. The rule for each is the same — keep the *form*, remove the *content*.

- **`CLAUDE.md` structure** — section headings, the organization of the file, the *patterns* of instruction (how shortcuts are documented, how protocols are framed, the table-of-reference convention pointing at `docs/`). Back-port the skeleton and the generic prose. Strip every real path that names a person, every real handle, every real preference that reveals a fact about the owner.
- **`docs/` convention** — the *practice* of keeping on-demand reference notes in a closet rather than preloaded, the INDEX pattern, the one-topic-per-file shape, and any genuinely generic tool reference (e.g. a public API's usage that anyone would write the same way). Strip account names, tokens, internal hostnames, private endpoints, and anything that pairs a tool with this owner specifically.
- **Projects doc-set** — the predictable per-project layout (`README.md`, `STATUS.md`, `TODO.md`, `PLAN.md`, optional `NOTES.md` / `PROCEDURES.md`) and the `master/projects/<name>/` convention. Back-port the *empty* template of a project. Never back-port a real project's contents.
- **`contacts.toml` SCHEMA** — the *shape* of the file: the `[[contact]]` table form, the field names (`name`, `role`, `handles`, `scope`, flags like `gluten_free`), and the role vocabulary (`owner` / `family` / `external` / `self` / `unknown`). Back-ported as **`contacts.example.toml`** with placeholder handles only. Real names, real numbers, real emails, and real scopes are soul.
- **Memory taxonomy** — the *system*, not the entries: the filename-prefix categories (`user_`, `project_`, `reference_`, `feedback_`, `daily_`), the `MEMORY.md` one-line-index pattern, the symlink-into-the-repo mechanism, and the "curate, don't accumulate" discipline. Back-port the taxonomy and an empty or obviously-fake `MEMORY.md` example. Never back-port a real memory file.
- **CMD/REPLY protocol** — the wire format: `[CMD-XXXX]…[/CMD-XXXX]` / `[REPLY-XXXX]…[/REPLY-XXXX]`, the four-character id rule, the daemon-injected metadata tokens (`from=`, `name=`, `attachment=`, `link=`), and the rule that metadata is the daemon speaking, not the user. This is pure shape — back-port it freely, using placeholder handles in any example.
- **Daily-question pattern** — the *mechanism*: a cron-driven routine that asks one specific question a day over the iMessage door, files the answer into the right memory document, and never re-asks a known topic. Back-port the loop and the state-tracking idea. Never back-port the questions actually asked or the answers actually given.

If a candidate change is not on this list and not obviously a generic structural improvement, treat it as soul until proven otherwise. The default is *stay*.

---

## Denylist: what is SOUL and never leaves master

These never appear in the template, in any form — not abbreviated, not anonymized "just a little," not in a comment, not in commit history.

- **Real contacts** — actual names, phone numbers, emails, handles, scopes, or any flag that identifies a real person. (The template carries only `contacts.example.toml` with placeholders.)
- **`memory-claude/` content** — every real memory file under `memory-claude/` and the real lines of `MEMORY.md`. The taxonomy is shape; the entries are soul.
- **`memory/` content** — the agent's self-written notes about the real owner and household.
- **mempalace data** — the semantic-search store and temporal knowledge graph mined from the real memory. It is soul distilled; it is some of the most private data in the house.
- **Secrets** — keys, tokens, decrypt passwords, account credentials, internal hostnames, private endpoints, tunnel/CDN configuration tied to real domains, and the shared-mailbox address and its decrypt key.
- **Project specifics** — the real contents of anything under `master/projects/<name>/`: trips, finances, health/body data, church and family logistics, work decisions. The doc-set *shape* is shareable; what fills it is not.

When in doubt, it is soul. The home keeps more than it gives.

---

## Verification: prove no soul leaked

A clean template is one where the owner's life cannot be found in it. Before publishing — and as a periodic audit — verify that a fresh clone reveals nothing private. The bar is simple: **a stranger who clones the template can learn the shape of the house and nothing about who lives in it.**

```bash
# In a clean checkout, far from the master folder:
git clone <the_intern remote> /tmp/the_intern_check
cd /tmp/the_intern_check

# The owner's name must appear NOWHERE — files OR history.
git grep -ri "<owner-name>"                 # working tree
git log -p | grep -i "<owner-name>"         # full history

# Likewise for every soul-bearing token. Each must return nothing:
git grep -riE "<family-member-names>"
git grep -riE "\+1[0-9]{10}|[0-9]{10}"      # real phone numbers
git grep -riE "[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}"  # real emails
#   ^ allow only obvious placeholders: *@example.com, +1XXXXXXXXXX
git grep -ri "<private-domains>"            # real domains / hostnames
git grep -riE "BEGIN .*PRIVATE KEY|age1|password|secret|token"  # secrets
```

**Pass condition:** every command above returns **nothing** (or, for the email/phone patterns, only obvious placeholders like `owner@example.com` and `+1XXXXXXXXXX`). A single real hit is a failed audit — fix it, and because git remembers, scrub the history too (the soul must be gone from the *timeline*, not just the tip).

The shortest form of the check, and the one to remember:

> `git clone the_intern && grep -ri <owner-name>` → **nothing.**

If that returns a line, the soul has leaked, and the contract has been broken.

---

## The contract, in one breath

Back-port shape, never soul. The template is the blueprint and stays empty; the master is the home and stays private. Improvements to *how the house is built* flow back; everything about *who lives in it* stays put. And you know it held when a stranger can clone the plans, read every room, and still never learn a single name.

