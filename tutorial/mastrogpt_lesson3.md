# Lesson 3 — Authentication, Forms & Displays (MastroGPT • Apache Open Serverless)

Welcome to the **third lesson** of the Private AI course with **Open Serverless** (aka the **MastroGPT** course).  
In this lesson you will:
- **Update your fork** of the starter safely
- Understand **authentication** for web actions and enable **Pinocchio token auth**
- Build **forms** for structured input and wire them into prompts
- Use **displays** to render structured output (e.g., a chessboard)
- Complete a guided **exercise** (Form‑driven Chess Puzzle Generator)

---

## 0) Update your forked starter (stay in sync)

The starter evolves. Before starting Lesson 3, **update your fork** so you don’t run into drift.

1. Open your starter repository on GitHub → you’ll likely see “**This branch is 1 commit behind**” (or similar).
2. Click **Sync fork** → **Update branch**. Your changes will be preserved (unless you modified the same files—resolve conflicts if needed).
3. (Optional but recommended) **Delete and recreate** your Codespace to ensure a fresh, updated environment.

> ⚠️ If you have **uncommitted changes** in the Codespace, commit or stash them before deleting the Codespace.

---

## 1) Start the environment & fetch Lesson 3

1. Open your **fork** in **Codespaces** (or use a local setup if you prefer).
2. When VS Code loads, log into **Open Serverless** via the 💬 *speech bubble* extension (credentials from Lesson 1).
3. From the extension’s **Lessons** panel, select **Lesson 3** to download the lesson files (Markdown + PDF).  
   - Lessons are **self‑contained**: you don’t need files from Lesson 1 or 2.

---

## 2) Authentication for Web Actions

### 2.1 Default Open Serverless behavior
- **Non‑web actions** are protected and require credentials via the OpenWhisk key stored in `~/.wskprops`.
- **Web actions** (`web: true`) are publicly invokable (no built‑in auth).  
  Pinocchio (the UI) needs **web actions**, so **we must add our own auth layer**.

### 2.2 Pinocchio token authentication
- On login, Pinocchio generates a **random token** and stores it in **Redis** under a user‑scoped key.
- Pinocchio sends this token (via cookie/header) with each request.
- Your web actions should **verify** the token against Redis before proceeding.

**High‑level flow:**
1. Client logs in → server sets `token` in Redis: `prefix:token:<username> = <secret>`.
2. Each web action reads the presented token and **compares** it with what’s in Redis.
3. If mismatch/missing → **deny** the request.

### 2.3 Practical snippets

**Shell helpers (optional quality‑of‑life):**
```bash
# Load OpenWhisk credentials into the shell
source ~/.wskprops

# Get the web URL of an action (example: mastrogpt-index)
URL="$(ops url mastrogpt-index | tail -n +2)"
curl -sS "$URL"
```

**Protected (non‑web) action invocation (illustrative):**
```bash
# Uses ~/.wskprops AUTH + APIHOST under the hood
ops action invoke my/ns/protected-action --result   -p name "World"
```

**Public web action:**
```bash
# Deployed with web: true → no built-in auth
curl -sS "$(ops url my/ns/public-action | tail -n +2)"
```

**Token check (inside your web action):**
```python
def unauthorized(msg="You are not authenticated"):
    return {"error": msg}, 401

def main(args):
    # 1) Extract token presented by Pinocchio (cookie/header/args)
    token = args.get("token")

    # 2) Resolve current user and read expected token from Redis
    user  = args.get("user")  # or derive from session
    expect = redis_get(f"{args.get('prefix','')}:token:{user}")  # pseudocode

    # 3) Verify
    if not token or token != expect:
        return unauthorized()

    # ...authorized flow...
    return {"output": "ok"}
```

> Add this check to **each public** (`web: true`) action that needs authentication.

---

## 3) Forms — Structured Input → Prompt Engineering

With forms you can request **structured input** (e.g., role, tone, topic), then build a **prompt** programmatically.

### 3.1 Form schema (concept)
A form is a **list of fields**, each with:
- `name`, `label/description`
- `kind` (e.g., `text`, `textarea`, `radio`, `checkbox`, `file`)
- `required` (bool)
- `options` (for radios/checkboxes)
- Optional `accept` or `encoding` hints (for files, e.g., base64)

### 3.2 Returning a form
Return a form when there’s no user input yet:
```python
def main(args):
    if not args.get("input"):
        form = [
            {"name": "job",  "label": "Your role", "kind": "text", "required": True},
            {"name": "why",  "label": "Why you use it", "kind": "textarea"},
            {"name": "tone", "label": "Tone", "kind": "radio",
             "options": ["formal", "informal"], "required": True}
        ]
        return {"output": "Please fill the form", "form": form}
```

### 3.3 Processing form input → building the prompt
When the form is submitted, Pinocchio sends back the **form data** in `args["form"]`:
```python
def main(args):
    data = args.get("form")
    if data:
        prompt = (
            "Write a short promotional post about Apache Open Serverless.
"
            f"Role: {data.get('job')}
"
            f"Reason: {data.get('why')}
"
            f"Tone: {data.get('tone')}"
        )
        # call LLM here and return the result
        return {"output": call_llm(prompt)}
```

---

## 4) Displays — Structured Output Rendering

Pinocchio’s display system renders **extra keys** in your action response. Any key that is **not** one of:
- `output`, `state`, `form`  
is forwarded to the **MastroGPT display** (extensible renderer).

Example: return a chessboard descriptor under the key `chess`, and the display renders it.

```python
# Action response (simplified)
return {
  "output": "Here is your puzzle",
  "chess": {"fen": "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1"}
}
```

**CLI round‑trip demo (concept):**
```bash
# Invoke action that outputs a 'chess' key → save JSON
ops ai cli <<'PY'
import json, requests
res = {"output": "ok", "chess": {"fen": "<FEN_STRING>"}}
open("chess.json", "w").write(json.dumps(res))
PY

# Forward to the display to render the board
ops display render chess.json
```

Internally the display can generate an **SVG** (or other formats) from the provided data.

---

## 5) Example — Post Generator (Form → Prompt)

- Define the **form** (role, reason, tone).
- On submit, build a **prompt** using form values.
- Call the **LLM** and return `output` (and optionally keep `state`).

This is classic **prompt engineering** with structured inputs.


---

## 6) Example — Chess Puzzle (Display + Regex extraction)

- Ask the LLM to **output a chess puzzle** in **FEN** format.
- Extract the `FEN` string with a **regular expression** from the LLM output.
- Return the extra key (e.g., `{"chess": {"fen": "..."} }`) so the display renders the board.

> Regex extraction is a simple and widely‑useful technique for parsing LLM output.


---

## 7) Exercise — Form‑Driven Chess Puzzle Generator

**Goal:** Modify the puzzle action so it **first asks for a form**, then generates a puzzle according to selected pieces.

### Requirements
- Show a form with **checkboxes**: `queen`, `rook`, `knight`, `bishop` (and any others).
- On submit, build a prompt like:  
  *“Generate a chess puzzle in FEN format containing a **queen** and a **knight** …”*
- Extract the **FEN** string (regex) from the LLM output.
- Return:
  ```json
  {
    "output": "Here is your customized puzzle",
    "chess": { "fen": "<FEN_HERE>" }
  }
  ```
- Deploy and test in the UI. Selecting *queen + knight* should yield a puzzle featuring those pieces.

### Hints
- Keep the form minimal and clear.
- Validate that at least one piece is selected.
- If extraction fails, return a helpful message and include the raw text for debugging.

---

## 8) Key Takeaways

- **Web actions** must be protected — use **Pinocchio token auth** with a Redis check.
- **Forms** let you capture structured inputs and turn them into **bulletproof prompts**.
- **Displays** render structured outputs (e.g., chessboards) from extra response keys.
- Combine these patterns to build richer, safer, more user‑friendly AI apps.

---

Happy building — see you in the next lesson! 🚀
