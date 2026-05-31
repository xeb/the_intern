# The Intern

"Imagine going to live on a mountaintop by yourself, forever. You build a home that no one will ever visit. Still, you invest the time and effort to shape the space in which you’ll spend your days. The wood, the plates, the pillows—all magnificent. Curated to your taste.”
- Rick Rubin

---

There is a particular kind of care that only makes sense when no one is watching.

A home built for an audience would be a different home. It would flatter — anticipate the visitor, arrange itself for the photograph, hide the parts that are merely lived-in. A home built for one person, forever, has no such incentive. Every choice in it is honest, because every choice answers only to the person who made it.

*The Intern* is a house like that: **a model-agnostic, local-memory, family agent** that lives in one person's home directory and serves one household. The running instance is private and will never be visited. What you are reading is the *blueprint* — the shape of the rooms, the logic of the doors, the conventions that make the place habitable, with the life inside it deliberately removed.

This document is organized the way you would build the house: choose the doors, raise the bridge that carries messages through them, name the rooms, furnish the house, teach it to remember, and decide who is allowed to do what. Read it as architecture first. The reflection sits inside each part, never in front of it, because the meaning here is drawn out of how the thing is actually built.

---

## The shape of the house

The Intern is **local-first** and **model-agnostic**. There is no proprietary brain at its center. It is "just a CLI agent tool," and the contract it honors is a filesystem, not a vendor's API. Move one mind out and another in — one model today, a different one tomorrow, a small local model the day after — and the house does not notice. The keeper changes; the home stays.

That independence holds because the agent's entire personality, its rules, and its sense of who it serves all live as plain files on disk. To switch keepers you rewire nothing: you start a different agent in the same window, and because its instruction file is a symlink to the same source of taste, it simply picks up where the last one left off.

The house is made of two repositories and one piece of connective tissue:

- **`~/p/master`** — *the house itself*: its furnishings, its workrooms, its memory.
- **`~/p/sink`** — *the front door for visitors*: a small daemon that lets people knock.
- **tmux** — *the hallways and the rooms*, the thing that makes the house navigable.

```
   family & approved collaborators                 the owner
              │                                        │
       iMessage (text / voice)               private web terminal
              │                                (own tmux, in a browser)
              ▼                                        │
   ┌──────────────────────┐                           │
   │ BlueBubbles (a Mac)  │   iMessage → local API    │
   └──────────┬───────────┘                           │
              ▼                                        │
   ┌──────────────────────┐                           │
   │ sink (Rust daemon)   │   the front door          │
   └──────────┬───────────┘                           │
              │  inject + capture (tmux)               │
              └───────────────┬───────────────────────┘
                              ▼
   ┌───────────────────────────────────────────────────┐
   │ tmux — the rooms                                  │
   │   a living agent session  (e.g. "sink MASTER")    │
   │   the keeper: any CLI agent (Claude/Gemini/local) │
   └─────────────────────────┬─────────────────────────┘
                             │  reads & writes plain files
                             ▼
   ┌───────────────────────────────────────────────────┐
   │ ~/p/master — the house                            │
   │   CLAUDE.md · GEMINI.md    instructions + auth     │
   │   contacts.toml            who may do what         │
   │   projects/                the workrooms           │
   │   memory/ · memory-claude/ what it remembers       │
   │   tools/ · docs/           scripts + reference     │
   └───────────────────────────────────────────────────┘

   git underneath it all · backed up · local — except the inference
```

Neither half is interesting alone. A house with no door is a vault; a door with no house behind it opens onto nothing. The design is in how they meet. And one principle runs underneath all of it:

> **Everything is in git. Everything is backed up. Everything is local — except the inference.**

The files are yours, on your disk, in your history. Only the thinking is borrowed, and the borrowed part is the one part you can replace at will.

---

## How a message moves through the house

The parts above are static — rooms, doors, furnishings. Here is the house in motion: a single message, from knock to answer.

Say a family member texts *"what's on the schedule today?"*

1. **BlueBubbles**, on the Mac, receives the iMessage and exposes it over its local API.
2. **sink** notices the new message and wraps it with a unique id and the sender's handle: `[CMD-ab12 from=<handle>] what's on the schedule today? [/CMD-ab12]`.
3. **sink injects** that line into the living agent session with `tmux send-keys` — into the room named `sink MASTER` — exactly as if someone had typed it at the keyboard.
4. **The agent**, already holding `CLAUDE.md` in mind, resolves `<handle>` against `contacts.toml` to a person and a role. This request is ordinary, so any household member may ask; a *privileged* request would be honored only for the owner.
5. **It does the work** — reads a project file, checks a schedule, whatever the request needs — and if it learned something worth keeping, writes or updates a memory file (git will carry it).
6. **It answers in the room**, wrapped as `[REPLY-ab12] … [/REPLY-ab12]` with the same id.
7. **sink captures** the pane, lifts out the reply by its id, and sends it back through BlueBubbles as an iMessage.
8. The family member gets a text. And if the owner happened to be watching, they saw the whole thing unfold, live, in the terminal.

The owner's own door skips the first three steps: they type directly into the room and read the reply where it appears. Same house, shorter hallway.

---

## What you'll need

The blueprint assumes a few materials. None are exotic.

- **A Mac running [BlueBubbles](https://bluebubbles.app/)** — to put iMessage on a local API. (Only needed for the iMessage door.)
- **A host that stays on** — the same Mac or a separate always-on machine — running tmux, a CLI agent, and the sink daemon.
- **A CLI agent to keep the house** — Claude Code, Gemini, or a local model that lives in a terminal.
- **git** — the timeline, and the backbone of every backup.
- **Optional comforts** — [mempalace](https://github.com/mempalace/mempalace) for search by meaning; a cron daemon for the daily question and scheduled backups; a tunnel/CDN and identity provider (e.g. Cloudflare and a Google sign-in) if you want the owner's web door reachable from anywhere.

---

## 1. The doors: choosing an interface

A house has more than one way in, and not every door is for everyone.

**The visitors' door is iMessage.** Family members and approved outside collaborators reach the Intern only by text. They never see the rooms; they knock, the house answers, and it feels like talking to a capable person who happens to live there. This is the door the whole household already knows how to use — no app to learn, no account to make. To stand it up, run **[BlueBubbles](https://bluebubbles.app/)** on a Mac, which exposes iMessage over a local API on a port of your choosing.

**The owner's door is a private terminal.** Only the owner uses it, and the owner does not knock — they walk straight into the rooms and work there, beside the agent. One way to build this door is a small web client in the spirit of **[tmux-terminal](https://github.com/xeb/tmux-terminal)**: it simply renders your own tmux windows in a browser. Fronted by a tunnel and CDN (Cloudflare, say) and gated behind your own identity provider (a Google sign-in, for instance), it becomes reachable from anywhere — including a work machine — without SSH into anything, because you are only ever viewing your own terminal. Its oversized input box is built for a phone, where voice dictation turns a thumb-typed errand into a spoken one.

Voice, in fact, is the quiet luxury of both doors. iMessage dictation and the web client's big mobile input mean the most natural way to reach the house is simply to *say* what you need. One house, two kinds of entrance: most of the household is received at the threshold; one person lives inside.

---

## 2. The bridge: sink and the living session

The visitors' door is held open by **[sink](https://github.com/xeb/sink)**, a small Rust daemon. Its defining decision is what it does *not* do: it does not spawn a fresh, forgetful agent for every message. Instead it injects each incoming message into a single agent session that is already awake inside a named room.

```
iMessage ─▶ BlueBubbles (Mac) ─▶ sink daemon
                                     │  tmux send-keys
                                     ▼
                          a living agent session
                          (a tmux window, e.g. "sink MASTER")
                                     │  capture pane
                                     ▼
sink ─▶ BlueBubbles ─▶ iMessage reply
```

sink types the message in with `tmux send-keys`, watches the pane until the agent is done, lifts out the reply, and carries it back over iMessage. Three consequences follow, and they are all the kind you want in a home:

- **You can watch.** Stand in the doorway of that room and see the keeper think in real time.
- **Context persists.** It is the same session each time, so it does not forget between messages.
- **It is durable and debuggable.** Because the daemon drives the *real* interactive terminal exactly as a person would — typing keys, reading the screen — it needs no special API or scripting integration to maintain. To the underlying tool, it is indistinguishable from a human at the keyboard, which means it keeps working across vendors and across changes to the agent, and you can always step into the same window and type a command yourself.

So replies don't get confused in a scrolling buffer, every exchange wears a tag. The daemon wraps an inbound message as `[CMD-XXXX]…[/CMD-XXXX]` with a unique four-character id, and the agent answers with `[REPLY-XXXX]` using the same id — for example, `[CMD-ab12 from=<handle>]…[/CMD-ab12]`. Right after the opening tag, the daemon adds bracketed metadata: the sender's handle, a resolved display name if it has one, any attachment paths, an optional link. That metadata is the daemon speaking, not the person, and the agent treats it that way. The matching id is what lets the daemon pull the *right* reply out of a buffer full of older ones — a name on the envelope, matched coming and going.

The door is courteous about crowds. In a group chat, an optional fast, cheap model first decides whether a message was meant for the Intern at all; when unsure, it stays quiet. The house can also remember to follow up — pulling reminders out of a conversation and sending them back later in the same thread, which the recipient can snooze, dismiss, or silence during quiet hours. The daemon runs as a user `systemd` service, keeps its message and follow-up state in local SQLite, downloads attachments locally and hands their paths to the agent, and offers a small local web panel for watching the door.

---

## 3. The rooms: tmux windows

The rooms of this house are **tmux windows** — not a metaphor stretched over something else, but the literal architecture. The agent runs inside named windows. The front door targets a specific named window to deliver a message into (by convention, a window with an agreed name such as `sink MASTER`). The status tooling reads the windows to learn what the house is doing.

Which makes **naming load-bearing**. A consistent convention is how the door finds the right room, how the tooling reports on it, and how the owner refers to it without ambiguity. Get the names right and the house has an address system; get them wrong and the messenger knocks on an empty wall.

To take in every room at once there is a **scan tool**: it walks the live windows, matches each to the project it belongs to, captures recent pane output, reads the state of the work — `active`, `waiting`, `idle`, `error`, `completed` — and writes a small status file back into that project's folder. It is how the owner stands in the entryway and sees which fires are lit, which are waiting on someone, and which have gone cold.

---

## 4. The furnishings: the master folder

A house becomes a *home* in its furnishings — the choices no one else would have made the same way. The `~/p/master` folder is where they live. The owner commits it to git but never shares it; what you are reading is the blueprint of that folder, not the folder.

Here is the floorplan:

```
~/p/master/                   # the house — committed to git, never shared
├── CLAUDE.md                 # the agent's instructions: craft + authorization
├── GEMINI.md                 # → symlink to CLAUDE.md (any agent reads the same taste)
├── contacts.toml             # handle → person → role (owner / family / external / …)
├── projects/                 # the workrooms
│   └── <name>/               #   README · STATUS · TODO · PLAN · NOTES · PROCEDURES
├── memory/                   # short notes the agent writes for itself
├── memory-claude/            # the agent's session memory, symlinked in so git keeps it
│   └── MEMORY.md             #   one-line index of every memory
├── docs/                     # on-demand reference for tools & APIs (not preloaded)
└── tools/                    # bespoke scripts (the tmux scanner, backups, alerts, …)
```

The most important piece is a single file: **`CLAUDE.md`**. The agent reads it at the start of every session, and it carries two things at once — the craft (tone of voice, how time and dates are written, the shortcuts and small protocols the owner has settled into) and the rules of the house (the authorization logic, which we come to last). These are the wood, the plates, the pillows: chosen, not defaulted. A stranger would find them unremarkable; that is the correct reaction to furniture curated for an audience of one.

Because the house is model-agnostic, **`GEMINI.md` is a symlink to `CLAUDE.md`** — the same instructions serve whichever agent moves in, because nothing in them assumes a vendor. One file, one source of taste, read by any keeper. This is the quiet mechanism behind the agnosticism, not a slogan about it: switch keepers and you change nothing but who is reading.

The rest of the furnishings:

- **`tools/`** — a drawer of small, bespoke scripts: the tmux scanner, backups, alerts, whatever the day asked for. The house expects this drawer to fill up. A lived-in workshop accumulates jigs fitted to one pair of hands; that accumulation is not mess, it is fit.
- **`docs/`** — reference notes on tools and external APIs, kept in a closet rather than laid out on every surface. They are read only when a task needs them, never preloaded, so the always-on instruction surface stays small and the keeper's attention stays uncluttered.
- **`projects/`** — the workrooms where things actually get done. Every project is a folder under `master/projects/<name>/` with the same predictable doc set:

```
projects/<name>/
  README.md      # what this is
  STATUS.md      # where it stands
  TODO.md        # what is next
  PLAN.md        # how we get there
  NOTES.md       # (optional)
  PROCEDURES.md  # (optional)
```

The predictability is the point: any agent, of any make, can walk into any project and orient itself immediately, because every room is laid out the same way. And these workrooms are deliberately **disposable** — because everything is under version control, a folder can be deleted without ceremony and recovered later from history. Nothing is precious; everything is recoverable. The freedom to throw a thing away, knowing the timeline remembers it, is what keeps the house from silting up.

One line separates the Intern's work from the owner's other work, and it is drawn entirely by **file path**. Projects the Intern manages live under `~/p/master/projects/`. Standalone work — applications and the like — lives one level up, directly under `~/p/`. The only difference between an "intern project" and a "work project" is *where it sits*, and the Intern reaches into that outer space only when the owner explicitly says so. The house has a yard; the keeper stays indoors unless invited out.

---

## 5. The room that remembers: memory

Every home lived in long enough begins to remember its owner. Here, memory is not a database hidden in a wall — it is plain, human-readable markdown, kept close to the work in two stores. A **`memory/`** folder holds little documents the agent writes for itself. And **`memory-claude/`** captures the agent's own session memory: the agent's working-memory directory is *symlinked into the master folder*, so everything the keeper chooses to remember lands inside the repo and is carried along by git automatically — no separate export, no second backup to remember.

Memories are categorized by filename prefix — `user_`, `project_`, `reference_`, `feedback_`, and so on — with a `MEMORY.md` index holding one line per memory, like a card catalog. The governing discipline is three words: **curate, don't accumulate.** Update an existing memory rather than writing a near-duplicate beside it. Dedupe. And — the hard part — delete what turns out to be wrong. A house that keeps everything forever does not remember better; it remembers *worse*, buried under its own residue. Memory here is more garden than attic; wisdom is partly knowing what to forget.

On top of the plain files sits an optional layer: **[mempalace](https://github.com/mempalace/mempalace)**, a local, open-source memory engine that mines the markdown into semantic search and a temporal knowledge graph, so the house can find a thing by meaning rather than only by keyword. It is a *supplement*, never a replacement — the markdown stays the source of truth. The owner holds three systems in mind, each with one job and no overlap:

```
   master (markdown)  =  what is true NOW
   mempalace          =  what HAPPENED
   git                =  what CHANGED
```

Memory is stored locally and backed up to git; the mempalace store is backed up daily to cloud storage. Git runs underneath the whole house as both timeline and safety net — it is why a room can be cleared without fear, and why the house can always remember what it used to be.

---

## 6. The daily question

A home learns its owner by living alongside them, one answer at a time. Once a day, a cron-driven routine wakes and sends the owner a single thoughtful, specific question — over the very same iMessage door everyone else uses. The owner answers in a sentence; the reply travels *back* through sink into the living session, and `CLAUDE.md` already knows how to file it into the right memory document. The routine keeps track of what it has asked and never re-asks a topic it already knows.

The premise is patient and a little humbling: one good question a day, over weeks, gathers more than any single long interview ever could. You do not furnish a home in an afternoon.

---

## 7. The locks: identity and authorization

Not every door opens for every hand. Identity begins at **`contacts.toml`**, which maps an inbound message handle to a person and to an authorization **role**:

| Role | Trust | Allowed |
| --- | --- | --- |
| `owner` | full | everything, including privileged, owner-gated actions |
| `family` | normal | ordinary conversational help; no privileged actions |
| `external` | least | one named scope, nothing else |
| `self` | n/a | the agent's own address |
| `unknown` | none | unlisted handle; untrusted, generic answers only |

`contacts.toml` says only *who* someone is. What that identity may *do* is decided elsewhere — and this is the part most worth understanding before you adopt the design. **Authorization is not enforced by code.** There is no firewall, no permission middleware, no gate that throws. The rules live in *prose*, in `CLAUDE.md`, and they are enforced by the agent reading them and choosing to obey. The door stamps each message with the sender's handle; the agent resolves it to a role and applies that role's privileges. A privileged request — toggling a household setting, say — is honored from the owner and declined from anyone else, because the instructions say to decline it, not because a guard rejects the call.

This only makes sense in a house with one builder. Enforcement-by-code is what you build when you cannot see the people inside, or do not trust them. Here the builder can see the whole house at once, and *write the rule plainly and follow it* is lighter, more legible, and easier to change than a cage of conditionals. The lock is real, but it is made of instructions a thinking resident is trusted to honor — exactly as strong as the care taken in writing the rules and choosing the keeper. State it plainly when you build your own: it is a home, not a bank.

---

## Principles

- **Local, except the inference.** Files, memory, history, and logic all live on your disk; only the thinking is borrowed, and the borrowed part is the one you can replace.
- **Model-agnostic by contract.** The agreement is the filesystem, not an API. Any CLI agent can keep the house.
- **Curate, don't accumulate.** Update, dedupe, and delete. A tended memory is worth more than a complete one.
- **Disposable rooms, permanent timeline.** Git lets you throw work away without losing it.
- **Build for one.** Every choice answers to the owner, not to an audience.

---

## References

- **[sink](https://github.com/xeb/sink)** — the iMessage-to-agent daemon; the front door.
- **[BlueBubbles](https://bluebubbles.app/)** — the iMessage API that runs on a Mac.
- **[tmux-terminal](https://github.com/xeb/tmux-terminal)** — the spirit of the owner's private web terminal.
- **[mempalace](https://github.com/mempalace/mempalace)** — local memory engine: semantic search and a temporal knowledge graph over the plain files.

---

## The blueprint and the home

These are the plans — the framing, the room layout, the placement of the doors and the logic of their locks. Anyone can take this drawing and raise their own house on their own mountaintop: pour the markdown, hang the doors, name the rooms, choose a mind to keep them. They are meant to be built upon.

But a blueprint is not a home, and everything above is at once true and empty. You have the rooms but not what is in them. You know there is a memory; you will never read it. You know there is a contacts file; you will never see a name in it. You know one question is asked each day; you will never hear an answer.

That absence is the design, not a redaction. What the plans cannot draw is the owner's memory, the people behind the handles, the projects that hold a life, the particular questions asked on particular mornings and the answers given back over a thousand quiet ones. That is the furniture carried out before the plans were shared. The blueprint shares the shape of the space — never the life lived in it. That is the soul, and the soul does not travel. It stays in the real house: the one on the mountaintop that no one will ever visit.

That was always the point. Build well. Curate, don't accumulate. And keep the door you live behind for yourself.
