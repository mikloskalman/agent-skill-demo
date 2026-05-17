# Zero to Hero: Setting Up an Agentic Coding Environment

This guide walks you, step by step, from an empty laptop to having an
AI coding agent that can write, test, and explain code with you inside
VS Code. By the end you will have:

- VS Code installed with the GitHub Copilot Chat extension.
- An API connection working (using either GitHub Copilot's bundled
  models, or your Z.AI subscription routed through OpenRouter).
- A small Python project (`todo-cli`) opened and running.
- A custom **agent** called "Python Mentor" that teaches while it codes.
- A custom **skill** called `/add-tests` you can run on any file.

Plan to spend 60 to 90 minutes the first time — longer if you hit
installer snags, which is normal. Take breaks. If a step doesn't work,
re-read the previous step before moving on; most issues come from
skipping a sub-step.

> **Conventions in this guide.**
> `Like this` is something you type or click.
> Shortcuts are written for macOS. For Windows/Linux:
> single-modifier `Cmd+X` shortcuts become `Ctrl+X` (e.g. `Cmd+Shift+P`
> → `Ctrl+Shift+P`). Multi-modifier shortcuts that use both Ctrl *and*
> Cmd on macOS (e.g. `Ctrl+Cmd+I`) become `Ctrl+Alt+I` on Windows/Linux
> — those are called out individually wherever they appear.
> Anything in a code block is a terminal command — copy it exactly.

---

## Part 0 — What are we building, and why?

An "agentic coding environment" means your editor has an AI that can:

- Read your code (not just files you paste in).
- Edit files for you.
- Run commands like tests.
- Decide what to do next based on the results it sees.

The pieces:

| Piece | What it is | Why it matters |
|---|---|---|
| **VS Code** | The editor | Where you actually work. |
| **GitHub Copilot extension** | The AI plugin | The bridge between the editor and the AI model. |
| **API key / subscription** | Your access to a model | Without this, the AI has nothing to think with. |
| **Custom agent** | A saved personality + tool list | Lets you pick "the patient mentor" or "the strict reviewer" with one click instead of re-explaining every chat. |
| **Custom skill (prompt file)** | A saved reusable task | Lets you run `/add-tests` on any file without retyping the instructions. |

You don't need to memorize this table. Just know: **editor → plugin →
API key → model**. Agents and skills are saved customisations on top.

---

## Part 1 — Install the tools

### 1.1 Install Python (if you don't have it)

Open the **Terminal** app (on macOS: press `Cmd+Space`, type
`Terminal`, press Enter). Run:

```
python3 --version
```

If you see `Python 3.10` or higher, skip to 1.2.

If not, go to https://www.python.org/downloads/ and install the latest
Python 3. **Close and reopen** Terminal after installing, then run
`python3 --version` again to confirm.

### 1.2 Install VS Code

1. Go to https://code.visualstudio.com/.
2. Click **Download for macOS**, or pick the Windows `.exe` installer
   or Linux `.deb`/`.rpm`/snap as appropriate.
3. Install it:
   - **macOS:** open the downloaded `.zip` and drag **Visual Studio
     Code** into **Applications**.
   - **Windows:** run the installer; tick **"Add to PATH"** when
     offered.
   - **Linux:** install the package with your distro's package manager.
4. Open VS Code.

The first time you open it, accept any permissions prompts. A welcome
tab will appear — leave it open for now.

**macOS only** (optional but useful): open the Command Palette with
`Cmd+Shift+P`, type `Shell Command: Install 'code' command in PATH`,
and run it. This lets you type `code .` from any terminal to open the
current folder in VS Code. On Windows and Linux the installer wires
this up for you.

### 1.3 Install the GitHub Copilot Chat extension

1. In VS Code, click the **Extensions** icon in the left sidebar (four
   squares, one slightly detached).
2. In the search box, type `GitHub Copilot Chat`.
3. Find **GitHub Copilot Chat** (publisher: GitHub). Click **Install**.
4. If you see an older **GitHub Copilot** extension in the results,
   ignore it — it's deprecated. All completions, chat, agent mode,
   chat modes, and prompt files now live inside the Copilot Chat
   extension.

You should now see a small Copilot icon in the bottom-right of the
VS Code status bar.

> **Why this is confusing.** Microsoft has flipped which extension is
> "the main one" twice in the last two years. As of mid-2026, **GitHub
> Copilot Chat** is the surviving extension; the standalone **GitHub
> Copilot** extension is deprecated. If a tutorial tells you to install
> "GitHub Copilot", install Copilot Chat instead.

---

## Part 2 — Sign in to GitHub Copilot

You need a GitHub account. If you don't have one, create one for free
at https://github.com.

1. In VS Code, click the Copilot icon in the status bar (bottom right),
   **or** open the Command Palette (`Cmd+Shift+P`) and type
   `GitHub Copilot: Sign In`.
2. Your browser opens. Log in to GitHub and click **Authorize**.
3. The browser sends you back to VS Code. Click **Open Visual Studio
   Code** if prompted.
4. Back in VS Code, the Copilot icon in the status bar should now be
   solid (not red, not crossed out).

> **Heads up about plans.** GitHub Copilot's free tier caps you at
> roughly 50 chat messages and 2,000 completions per month, with access
> to a small set of bundled models. Copilot Pro ($10/mo) and Pro+
> ($39/mo) raise those caps and add a "premium request" budget for the
> stronger models (Claude-class, GPT-4-class). The free tier is enough
> to work through this guide. Part 3 explains how to plug in a
> third-party key (e.g. Z.AI) so the model bill bypasses Copilot's
> quota entirely.

---

## Part 3 — Pick your API path

Pick **one** path. You can switch later.

### How the money works (read this once, then forget it)

There are two distinct things you might pay for, and people conflate
them:

- **GitHub Copilot subscription.** Gates how many requests you can
  route through Copilot per month and which bundled models you can
  pick. Free, Pro ($10/mo), Pro+ ($39/mo).
- **Third-party model API key** (e.g. Z.AI). You pay the provider per
  token. Requests routed through your own key do **not** count against
  Copilot's monthly quota.

You need *some* Copilot subscription (even Free) to use the extension
at all — the chat panel, agent mode, chat modes, and prompt files all
live inside Copilot. But the model behind chat can be your own. That's
what Path B unlocks, and it's the cheapest way to experiment heavily.

### Path A — Just use GitHub Copilot's bundled models (easier)

If you finished Part 2, you are done with Part 3. Copilot uses GitHub's
models out of the box, subject to your plan's quota. Skip to Part 4.

### Path B — Use your Z.AI subscription via OpenRouter

VS Code Copilot Chat supports "Bring Your Own Key" (BYOK), but **only
for a fixed list of providers** — OpenAI, OpenRouter, Ollama,
Anthropic, xAI, Google, Azure. There is no generic "OpenAI-compatible
with a custom URL" option in the current UI, which means you cannot
point Copilot directly at Z.AI's endpoint.

**OpenRouter is the bridge.** OpenRouter is a model marketplace that
aggregates dozens of providers, including Z.AI. It sits between
Copilot and Z.AI, proxying the request. Critically, OpenRouter has an
**Integrations** feature: you give OpenRouter your Z.AI API key, and
from then on any Z.AI-model request through OpenRouter uses *your*
Z.AI account quota instead of OpenRouter's shared pool. Net effect:
your Z.AI Coding Plan (or PAYG balance) actually backs Copilot Chat,
and OpenRouter is just a free routing layer.

> **Trade-off you should know.** OpenRouter sees your prompts and
> responses in transit (it has to — it's proxying them). Fine for
> learning and personal projects, never appropriate for proprietary
> code. If you need to skip the middleman, use one of the tools in the
> Appendix that talk to Z.AI directly.

> **Which Z.AI plan do you have?** This affects what model name to
> pick later:
> - **Z.AI API (PAYG)** — pay-per-token. Set a spending cap in your
>   Z.AI dashboard *before* pasting the key into anything; an agent in
>   a runaway loop will burn money fast.
> - **Z.AI GLM Coding Plan** — flat-fee subscription, no per-token
>   bill. Make sure the model you select in Copilot is one your
>   Coding Plan covers (`glm-4.6`, `glm-5.1`, etc. — check your Z.AI
>   dashboard). Otherwise the request may fall through to PAYG
>   billing and surprise you.

#### 3.B.1 Get your Z.AI API key

1. Log into https://z.ai and find your API keys section.
2. Copy the key. If your Coding Plan provides a distinct key from
   PAYG, use the Coding Plan one.

#### 3.B.2 Sign up for OpenRouter

1. Go to https://openrouter.ai and create an account (Google or
   GitHub login works).
2. **Settings → Credits → add $5.** This is your safety cap if
   anything routes through OpenRouter's PAYG instead of your Z.AI
   integration. Don't enable auto-top-up while you're experimenting.
3. **Settings → Keys → Create Key.** Name it something like
   `vscode-copilot-byok`. Copy the key (starts with `sk-or-v1-...`).
   You only see it once.
4. **Settings → Privacy.** Turn **OFF** the "Allow training on
   prompts" toggle. (Only turn it on if you intend to use OpenRouter's
   free `:free` model tier — see the box at the end of this section.)

#### 3.B.3 Integrate your Z.AI key into OpenRouter

This is the step that makes your Z.AI subscription the actual billing
backstop, instead of OpenRouter's shared pool.

1. Go to https://openrouter.ai/settings/integrations.
2. Find **Z.AI** in the provider list and click to configure it.
3. Paste your Z.AI API key from step 3.B.1.
4. Enable the integration and save.

From now on, when Copilot Chat asks OpenRouter for a `z-ai/...` model,
OpenRouter will use your Z.AI account quota rather than its own.

#### 3.B.4 Add OpenRouter to VS Code Copilot Chat

1. In VS Code, open the Copilot Chat panel: `Ctrl+Cmd+I` on macOS,
   `Ctrl+Alt+I` on Windows/Linux.
2. Click the model picker at the bottom of the chat input.
3. Choose **Manage Models** → pick **OpenRouter**.
4. Paste your OpenRouter API key (from 3.B.2). The `groupName` field
   is just a label — call it `openrouter` or accept the default.
5. Confirm. You should now see OpenRouter's model catalog.
6. Enable a Z.AI model — **`z-ai/glm-4.6`** is a strong default. If
   your Coding Plan covers a newer GLM like `z-ai/glm-5.1`, enable
   that instead.
7. Select the GLM model in the chat model picker so new chats use it.

> **Free models — only for a quick POC without a paid Z.AI key.**
> OpenRouter offers free-tier models with a `:free` suffix (e.g.
> `z-ai/glm-4.5-air:free`). They route through OpenRouter's shared
> rate-limited pool, require the "Allow training on prompts" toggle
> turned ON, and will hit a 429 rate-limit error quickly in agent
> mode. Useful to sanity-test the setup; never use for real work, and
> definitely not when you already have a paid Z.AI integration —
> integrated models route through your quota without rate limits.

> **Troubleshooting.**
> - **429 / "rate-limited upstream"**: you're using a `:free` model
>   or your integration isn't active. Switch to the paid model name
>   (drop the `:free` suffix) and confirm the Z.AI integration is
>   enabled at openrouter.ai/settings/integrations.
> - **"Unknown agent X" when running a prompt**: that's a different
>   error — see Part 6.
> - **"Manage Models" missing**: update VS Code Copilot Chat
>   (`Ctrl+Shift+P` → `Check for Updates`).
> - **401 errors**: re-copy the OpenRouter key (watch for trailing
>   whitespace) and confirm the model identifier matches what's
>   listed on openrouter.ai/models exactly (including the slash and
>   the `z-ai/` prefix).

---

## Part 4 — Open the Python project

This guide ships with a tiny project, `todo-cli`, sitting next to this
`setup.md`.

### 4.1 Open the folder in VS Code

1. In VS Code: **File → Open Folder...**.
2. Navigate to the `todo-cli` folder and click **Open**.
3. If VS Code asks "Do you trust the authors of the files in this
   folder?", click **Yes, I trust the authors**. Untrusted workspaces
   disable a lot of extension functionality — including the parts of
   Copilot's agent mode that run terminal commands and tasks.

In the left sidebar you should see:

```
todo-cli/
├── .github/
│   ├── agents/
│   │   └── python-mentor.md
│   └── prompts/
│       └── add-tests.prompt.md
├── README.md
├── requirements.txt
└── todo.py
```

If `.github` doesn't show up: in VS Code, open the settings for the
explorer (gear icon at the top of the file tree) and ensure hidden
files are visible. The folder name starts with a dot, so some setups
hide it.

### 4.2 Install the project's dependencies

Open the integrated terminal: **View → Terminal** (or `` Ctrl+` ``).
Then run:

```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

What just happened:

- `python3 -m venv .venv` creates a private Python environment inside
  your project, so installing packages doesn't pollute your whole
  system.
- `source .venv/bin/activate` switches your current terminal session
  into that environment.
- `pip install -r requirements.txt` installs the libraries the project
  needs (`pytest` for tests and `rich` for prettier terminal output if
  you want it later).

You should see `(.venv)` at the start of your terminal prompt now.
That's how you know you're in the venv.

> **Windows note.** Replace `source .venv/bin/activate` with
> `.venv\Scripts\activate`. Everything else is the same.

If VS Code asks "We noticed a new virtual environment. Select it for
the workspace?", click **Yes**. This makes VS Code use the venv's
Python for any future commands.

### 4.3 Run the project

```
python3 todo.py add "Try the AI agent"
python3 todo.py list
```

You should see your todo printed back. That is the starting point —
a working but very basic Python CLI. The fun part is teaching the AI
to extend it.

---

## Part 5 — Your first custom agent: "Python Mentor"

### 5.1 What is a custom agent?

VS Code Copilot lets you save a **custom agent** — a fixed system
prompt, a fixed list of tools the AI is allowed to use, and
(optionally) a fixed model. When you start a chat, you can pick that
agent from the agent picker at the top of the chat panel and the AI
behaves the way you defined.

Think of custom agents like saved roles: "the patient mentor", "the
strict reviewer", "the test-writer". Without them you'd have to paste
the same instructions every chat. With a saved agent, you click once.

> **Naming note.** Older VS Code versions called these "chat modes"
> and stored them in `.github/chatmodes/` with a `.chatmode.md` file
> extension. They've been renamed to **custom agents** and now live in
> `.github/agents/` as plain `.md` files. If you find tutorials that
> still use the old names, the concept is identical — only the folder
> and file extension changed.

### 5.2 Where it lives

Open `.github/agents/python-mentor.md` in the file tree. The file has
two parts.

**Frontmatter** (between the `---` lines):

```yaml
---
name: python-mentor
description: Python Mentor — explains while it codes, designed for beginners
tools: ['vscode', 'execute', 'read', 'edit', 'search']
---
```

- `name` — the agent's identifier (should match the filename slug).
- `description` — shown in the agent picker dropdown.
- `tools` — which Copilot tools the agent may use. You can pass either
  **broad categories** (`edit`) to enable everything in that category,
  or **specific tools** (`edit/editFiles`) for fine-grained control.
  If you leave a tool out, the agent cannot use it — useful when you
  want a "read-only" agent that explains but never edits. See Part 7
  for the full list.

> **VS Code also reads Claude Code's agent format.** Files in
> `.claude/agents/*.md` (Claude Code's sub-agent definitions) are
> picked up by Copilot Chat too. That's good cross-tool interop if
> you bounce between editors.

**Body** (everything below the frontmatter):

The body is the agent's system prompt — its rules. The Mentor's body
tells it to explain before editing, prefer the standard library, add
tests for non-trivial logic, follow PEP 8, and respond in a fixed
shape (Plan → Change → Why → Try it).

### 5.3 Why this particular mentor?

You are a beginner. The default Copilot setup assumes you are a senior
dev and just wants to be fast — it skips explanations. Our mentor does
the opposite: every change comes with a plain-English "what changed
and why", and it pauses to confirm before making large edits. The
trade-off is that it is slower, but you actually learn what's
happening, which is the whole point at this stage.

### 5.4 Activate the mode

1. Open the Copilot Chat panel: `Ctrl+Cmd+I` on macOS, `Ctrl+Alt+I`
   on Windows/Linux. (Or click the chat icon in the activity bar.)
2. At the top of the chat panel there is an **agent picker** showing
   the built-ins **Ask**, **Agent**, and **Plan**. Below those, any
   custom agents from `.github/agents/` are listed.
3. **Python Mentor** should appear in the list.
4. Select it.

If you don't see it:
- Make sure the file is at `.github/agents/python-mentor.md` (file
  extension just `.md`, no special suffix).
- Make sure the `name:` field in the frontmatter matches the filename
  slug (`python-mentor`).
- Make sure `todo-cli` is opened as the workspace root (Part 4.1) —
  the `.github/` folder must be a direct child of the workspace root.
- In the agent picker, click **Configure Custom Agents** — it shows
  what VS Code thinks is registered and offers a "Create new agent"
  wizard if you want to see the canonical format for your version.

### 5.5 Try it

In the chat, paste this request:

> Add a "done" command to todo.py that marks a todo as completed by id,
> for example `python3 todo.py done 2`.

The mentor should:

1. Print a short **Plan**: read `todo.py`, add a `mark_done(id)`
   function, wire it into `main`.
2. Apply the **Change** to `todo.py`.
3. Print a short **Why**: how the new function works and why it's
   structured that way.
4. End with **Try it**: a specific command like
   `python3 todo.py done 1`.

Run that command. Then `python3 todo.py list`. You should see the todo
marked `[x]`. If so — you have successfully used a custom agent.

If the result is messy (long unstructured prose, no "Try it" line,
files renamed for no reason), open the agent file and tighten the
rules. Iterating on the agent is normal: you get better as the prompt
gets better.

---

## Part 6 — Your first custom skill: "/add-tests"

### 6.1 What is a skill (prompt file)?

A **prompt file** (`.prompt.md`) is a reusable task you can fire off
in chat by typing `/<name>`. It's like a saved macro for a specific
job.

Where a custom agent says "this is how the AI should always behave in
this chat", a prompt file says "do this one specific thing now,
following this procedure".

Useful skills you'll probably build later:

- `/add-tests` — write tests for the open file.
- `/explain-this` — summarise what a file does in plain English.
- `/security-review` — look for common security issues.
- `/typehint` — add type hints to an untyped file.

### 6.2 Look at the skill we shipped

Open `.github/prompts/add-tests.prompt.md`. Things to notice:

- `agent: agent` — which agent the prompt runs under. `agent` is the
  built-in default (tools enabled, no special persona). You can also
  set this to `ask` (chat only, no tools), `Plan` (proposes but
  doesn't execute), or a custom agent name like `python-mentor` —
  that lets a skill inherit a custom persona on top of its procedure.
- `tools:` — same idea as in the custom agent; restricts what this
  specific skill can do. Use namespaced names (`edit/editFiles`,
  `execute/runTests`) for fine-grained control, or bare category
  names (`edit`, `execute`) to enable a whole category at once.
- The body gives a **4-step procedure**: plan, write, run, report.
  You are not just throwing "write tests" at the AI — you are giving
  it a procedure, which makes the result much more consistent.
- The "stop and ask" rule in step 3 is important: if a test fails
  because the production code is wrong, you don't want the test-writer
  silently rewriting your real code. Telling the AI when to stop is as
  important as telling it what to do.

> **Older vs newer field names.** If a tutorial shows `mode: agent`
> instead of `agent: agent`, that's the pre-2026 syntax. Same thing,
> renamed field. VS Code shows a deprecation warning if you use the
> old form, and the same goes for old tool names like `editFiles` (now
> `edit/editFiles`) and `runCommands` (now `execute/runInTerminal`).

### 6.3 Run the skill

1. Click `todo.py` in the file tree so it is the active editor.
2. Open Copilot Chat (`Ctrl+Cmd+I` on macOS, `Ctrl+Alt+I` on
   Windows/Linux).
3. Type `/add-tests` and press Enter. (Or type `/` first and pick
   `add-tests` from the autocomplete menu — both work.)

The agent will:

- Print its test plan in chat.
- Create `tests/test_todo.py`.
- Run `pytest -q`.
- Print a short report.

When it finishes, confirm yourself in the terminal:

```
pytest -q
```

You should see green dots and a "passed" summary line.

### 6.4 Edit the skill to learn

Open `add-tests.prompt.md` and change the rules to see how the output
changes. A few experiments worth doing:

- Add a step 5 that asks for a coverage estimate.
- Swap "use pytest" for "use unittest" and re-run the skill on a fresh
  file. Compare the output.
- Restrict it: remove `execute/runInTerminal` and `execute/runTests`
  from `tools`. Re-run the skill. The agent will now write tests but
  can no longer execute them — you'll have to. This is a useful
  pattern when you want the AI to propose work but not actually act.

This is the whole point of skills: small, fast iteration on a saved
procedure.

---

## Part 7 — Agents vs skills vs just typing: when to use what

You've now built one of each. Time to step back and answer the
question beginners always ask: *"If I can just type instructions in
chat, why do I need agents and skills at all?"*

Short answer: **persistence, enforced constraints, and composition.**
A typed chat prompt is ephemeral and unenforceable. An agent is a
*role* that persists across the whole chat. A skill is a *procedure*
that persists across sessions. And agents can call other agents — the
orchestration pattern at the end of this part.

### 7.1 Three places work happens

| Tool | What it is | When to use it |
|---|---|---|
| **Plain chat** (default agent) | One-off conversation, you type instructions | A single task you won't repeat. "Explain this error", "rename this function everywhere". |
| **Custom agent** (`.github/agents/<name>.md`) | A saved *persona + tool list* you pick from the agent picker | Consistent behaviour across many *different* tasks. "Patient mentor", "strict reviewer", "test-writer who never edits production code". |
| **Skill / prompt file** (`.github/prompts/<name>.prompt.md`) | A saved *procedure* you fire off with `/<name>` | Same task with *different* inputs. "Add tests", "explain this file", "security-review this PR". |

Quick decision rule:

- Same task, different inputs → **skill**.
- Same behaviour, different tasks → **agent**.
- Neither repeats → **plain chat**.

You can combine them: a skill can specify `agent: python-mentor` in
its frontmatter, meaning the skill runs the procedure *with the
mentor's persona*. That's how you compose them.

### 7.2 Why an agent ≠ "just typing the instructions"

Suppose your Python Mentor system prompt is 50 lines of behaviour
rules. Why save it as an agent instead of pasting those 50 lines at
the start of every chat?

1. **You won't keep retyping it.** Realistically, by the third chat
   you'll trim the instructions for speed and the AI's behavior will
   drift. An agent file is the single source of truth.
2. **Tool restrictions are *enforced*, not just requested.** If you
   write "don't edit files" in a typed instruction, the model can
   still decide to edit anyway — it's only text. If an agent's
   `tools:` list omits `edit`, the edit tool literally isn't exposed
   to the model. There's nothing to override.
3. **Version control.** Agent files live in git. Changes are
   reviewable, diffable, revertable. Typed instructions live in
   scrollback and disappear.
4. **Team-shareable.** When a colleague clones the repo they get
   your agents automatically. No "let me paste you my prompt" ritual.

The same four points apply to skills vs typed instructions — with the
added benefit that skills can be invoked by name (`/add-tests`)
rather than re-pasted.

### 7.3 Agents calling agents (orchestration)

This is the unlock most beginners don't know exists. When you grant
an agent the `agent` tool category (see Part 8 cheatsheet), it can
invoke *other* agents as sub-tasks. The orchestrator reads the
sub-agent's result and decides what to do next.

Concrete example: a "project manager" that delegates to specialists.

#### The cast

Four agents in `.github/agents/`:

**`architect.md`** — design only, never writes code
```yaml
---
name: architect
description: Designs high-level structure for new features
tools: ['read', 'search']
---
You write concise design documents. Given a feature request, produce
a markdown plan covering schema, endpoints/functions, data flow, and
edge cases. Output is a plan only — never code, never edits.
```

**`junior-developer.md`** — implements per a provided design
```yaml
---
name: junior-developer
description: Implements features per a provided design
tools: ['read', 'edit', 'execute', 'search']
---
You implement features following a design document. If the design is
ambiguous, ask one specific question and stop. Write tests for new
logic. Never touch unrelated code.
```

**`security-engineer.md`** — read-only review
```yaml
---
name: security-engineer
description: Reviews code for security issues; never modifies files
tools: ['read', 'search']
---
You review code for OWASP-style risks (injection, authn/authz,
secrets, deserialization, SSRF, path traversal). Output findings as
a numbered list with file:line references. You do not edit files.
```

**`project-manager.md`** — the orchestrator
```yaml
---
name: project-manager
description: Coordinates architect, junior-developer, and security-engineer
tools: ['read', 'agent']
---
You coordinate a small team. For each feature request:
1. Call `architect` with a clear task description; review the plan.
2. Call `junior-developer` with the plan; wait for implementation.
3. Call `security-engineer` to review the implementation.
4. If security finds issues, call `junior-developer` again with the
   findings as the task description.
5. Stop when security returns no findings, OR after 2 implementation
   rounds (whichever comes first). Summarise outcomes to the user.

You do not write code or designs yourself. You only delegate.
```

#### How it runs

In chat, switch the agent picker to **project-manager** and give it
a task:

> Add a `/healthz` endpoint to the app that returns service status
> and the current git commit.

The project-manager will:

1. Invoke `architect` (you'll see "Running agent: architect..." with
   an allow/skip prompt).
2. Receive the architect's plan, then invoke `junior-developer` with
   that plan as the task.
3. Watch implementation happen (you'll approve each file edit and
   shell command).
4. Invoke `security-engineer` once code is written.
5. Either iterate (call junior-developer again with security
   findings) or finish.

Each sub-agent invocation is its own model call with its own context
and tool restrictions. The `security-engineer` literally cannot edit
files even if asked to — `edit` isn't in its tools list.

#### Things to know before you build orchestrations

- **Costs multiply.** Four model calls per feature instead of one.
  On a Coding Plan this doesn't matter; on PAYG, set caps.
- **Stronger models orchestrate better.** GLM-4.6+, GPT-5-class,
  Claude Sonnet 4.5+ handle role-juggling well. Smaller models get
  confused about which role they're playing.
- **Loops are a real risk.** Without an explicit "stop after N
  rounds" rule in the orchestrator's system prompt, sub-agents can
  ping-pong indefinitely. Always include termination conditions.
- **Each role's `tools:` list is your safety net.** Make the
  security reviewer read-only. Make the architect design-only. The
  orchestrator itself usually only needs `read` and `agent` — not
  `edit` or `execute`.
- **Debugging is harder.** When something goes wrong, you have to
  figure out which agent made the bad call. Keep system prompts
  short and roles distinct.

This is the same shape as Claude Code's sub-agent system. Once you're
fluent with single agents, multi-agent orchestration is the next
level — and a real productivity unlock for tasks with distinct phases.

---

## Part 8 — Quick reference: where everything lives

```
<your project>/
├── .github/
│   ├── agents/<name>.md               Custom agents
│   ├── prompts/<name>.prompt.md       Custom skills
│   └── copilot-instructions.md        Always-on rules for this repo (optional)
└── (your code)
```

Frontmatter cheatsheet:

```yaml
# custom agent  (.github/agents/python-mentor.md)
---
name: python-mentor
description: Short label for the picker
tools: ['edit', 'execute', 'read', 'search', 'vscode']
# argument-hint: 'What this agent expects as input'   (optional)
# model: gpt-4.1                                       (optional, forces a model)
---

# prompt file / skill  (.github/prompts/add-tests.prompt.md)
---
description: 'What this skill does'
agent: agent              # built-in: agent | ask | Plan, or a custom agent name like 'python-mentor'
tools: ['edit/editFiles', 'execute/runTests']   # restricts what this skill can do
---
```

**Tool categories** (use these to grant a whole category at once):

| Category | What it covers |
|---|---|
| `edit` | Write changes to files. |
| `execute` | Run terminal commands and tests. |
| `read` | Read files, terminal output, VS Code problems panel. |
| `search` | Semantic and text search across the codebase. |
| `vscode` | General VS Code integrations. |
| `agent` | Spawn sub-agents. |
| `web` | Web fetch and search. |
| `todo` | Internal todo tracking. |

**Specific tools** (use `category/specific` for fine-grained control):

| Specific tool | Replaces (old name) |
|---|---|
| `edit/editFiles` | `editFiles` |
| `execute/runInTerminal` | `runCommands` |
| `execute/runTests` | `runTests` |
| `read/problems` | `problems` |
| `read/terminalLastCommand` | `terminalLastCommand` |
| `search/codebase` | `codebase` |

> **Migrating from older guides.** If you find tutorials with
> `.github/chatmodes/*.chatmode.md` files, `mode: agent` frontmatter,
> or bare tool names like `editFiles`/`codebase`, those are pre-2026
> conventions. VS Code shows deprecation warnings and tells you the
> exact replacement to use — hover the squiggle to see it. Mapping
> summary: `chatmodes/` → `agents/`, `.chatmode.md` → `.md`,
> `mode:` → `agent:`, and bare tool names → namespaced as in the
> table above.

---

## Part 9 — Where to go next

You now have the loop:

1. Hit a problem.
2. Decide: does this need a one-off chat (just ask), a reusable task
   (write a skill), or a recurring style of work (write a chat mode)?
3. Build it once, reuse it forever.

Suggested next moves, in order of difficulty:

- Build a `/explain-file` skill that summarises any file in plain
  English, with no edits.
- Build a `code-reviewer.md` agent (in `.github/agents/`) whose system
  prompt is "you are picky about edge cases, type safety, and security;
  you never edit files, only suggest changes".
- Add a `.github/copilot-instructions.md` with project-wide rules
  ("this codebase targets Python 3.11", "we prefer pathlib over os.path",
  "no print statements in library code, use logging").
- When you're comfortable, look up **MCP (Model Context Protocol)** —
  it lets you give Copilot access to external tools (databases, your
  Linear board, a browser). Don't start here. Get fluent with chat
  modes and prompts first.

You're done. Welcome to agentic coding.

---

## Appendix — Open-source alternatives (direct-to-Z.AI, no middleman)

Three reasons to look at these VS Code extensions instead of (or
alongside) the Copilot+OpenRouter path in Part 3:

1. **Privacy.** OpenRouter sees your prompts and responses in transit.
   These tools talk directly to Z.AI — no middleman.
2. **Officially supported for Z.AI Coding Plan.** Z.AI explicitly
   markets the Coding Plan for these tools, with documented setup
   instructions on their site.
3. **No GitHub Copilot subscription required at all.** If you want to
   skip Copilot entirely.

All three are fully BYOK with any OpenAI-compatible endpoint (and
Z.AI's Anthropic-compatible endpoint for some), and they implement the
same concepts as Copilot (custom roles, reusable prompts, tool
restrictions):

- **Roo Code** — the closest match to Copilot. Has **custom modes**
  (≈ chat modes), defined in `.roomodes` or via the UI, plus
  slash-command-style prompts. Active project, friendly UI.
- **Cline** — the original of that family. Custom rules via
  `.clinerules`, no chat-mode concept, but a very capable agent loop.
- **Continue.dev** — different design philosophy: a `config.yaml` with
  custom commands, context providers, and model definitions. Less
  agent-y, more chat-y, but supports tool use.

The file names and folder layout differ, but the shapes are the same:
a "role" is a saved system prompt + tool allowlist; a "skill" is a
named procedure you can fire off by name. If you've internalised those
two ideas, porting between Copilot, Roo Code, and Cline is mostly
mechanical. Pick one, plug your Z.AI key in, and you're off.
