# Lesson 2 — LLM Access & Streaming (MastroGPT • Apache Open Serverless)

Welcome to the **second lesson** of the OpenLLM course with **Apache Open Serverless** — aka the **MastroGPT course for Private AI**.  
In Lesson 1 you set up the environment and ran your first tests; here you’ll learn how to **call an LLM**, **handle secrets**, and **implement streaming**.


---

## 0) Start from your own fork (Codespaces or local)

1. Open **GitHub → MastroGPT starter** and **fork** it to your account (you won’t have write access to the original).  
2. Open your fork in **Codespaces** (recommended for the course) or work **locally** with Docker.  
3. When Codespaces starts, you should see the 💬 **speech bubble** extension. Use it to **log in** with the credentials provided in Lesson 1.  
4. From the extension’s **Lessons** panel, select **Lesson 2**. The lesson files (Markdown and PDF) will download into your repo.  
   - Each lesson is self‑contained; you don’t need Lesson 1’s files to work on Lesson 2.


---

## 1) Lesson plan

- **Access the LLM** (Ollama endpoint) with proper **secrets**.  
- **Streaming**: show partial tokens as they’re generated.  
- Put it all together: **LLM in streaming mode**.  
- Hands‑on **exercises** (secrets, streaming decoder, model switcher).


---

## 2) Speed tip (VS Code)

Map a shortcut to send selected text to the active terminal (super handy in this lesson):

- Open **Settings → Keyboard Shortcuts**.  
- Search: **“Run Selected Text in Active Terminal”**.  
- Bind a key (e.g., `Ctrl+Enter`).


---

## 3) Accessing the LLM (credentials & CLI)

The LLM endpoint (Ollama) is **protected** with the same credentials you use for Open Serverless. In code, read them from `args` first, then from env vars — this makes your functions work in **both** test and production.

```python
# Pattern used across this lesson:
def get_secret(args, key, default_env_key):
    return args.get(key) or os.getenv(default_env_key)

# Example names (adjust to your environment):
# - OLLAMA_HOST  (URL/host)
# - AUTH         (basic auth "user:pass" or token)
```

Quick check from the terminal (interactive Python via the course CLI):

```bash
ops ai cli
```

Inside the REPL:

```python
import os, requests

host = os.getenv("OLLAMA_HOST")          # provided by login/config
auth = os.getenv("AUTH")                 # provided by login/config (user:pass)

base = f"https://{auth}@{host}"
health = requests.get(f"{base}/api/health", timeout=10)
print(health.json() if health.headers.get("content-type","").startswith("application/json") else health.text)
```

If you see a “running/ok” response, credentials are working.


---

## 4) First non‑streaming call (requests)

```python
import os, requests, json

host  = os.getenv("OLLAMA_HOST")
auth  = os.getenv("AUTH")
base  = f"https://{auth}@{host}"
url   = f"{base}/api/generate"

payload = {
    "model": "llama3.1:8b",          # adjust tag to your environment
    "prompt": "Who are you?",
    "stream": False
}

res = requests.post(url, json=payload, timeout=120)
data = res.json()
print(data.get("response", ""))
```

> Later we’ll wrap this into an **action** and deploy it properly.


---

## 5) Where secrets come from (and how to pass them)

- **Config loaded at login** provides values to **tests** and **deploys**.  
- Env files:  
  - `.env` → base config  
  - `test.env` → used by tests/CLI  
  - `packages.env` → applies to deploys  
- Secrets not visible in the shell can still be read by **ops** and injected at deploy.

**Best practice in actions**: read from `args`, fallback to env:

```python
def main(args):
    host = args.get("OLLAMA_HOST") or os.getenv("OLLAMA_HOST")
    auth = args.get("AUTH") or os.getenv("AUTH")
    # ...
```

**Annotations** (in single‑file actions) propagate secrets automatically at deploy time:

```python
# --a param OLLAMA_HOST=${OLLAMA_HOST}
# --a param AUTH=${AUTH}
```

Deploy with the IDE helper:

```bash
ops ide deploy my-package/my-action.py     # single file
# or
ops ide deploy my-package/my-action        # folder gets zipped
```


---

## 6) Streaming: concept & minimal client

**Streaming** returns chunks progressively. In Python, treat it as an **iterator**.

```python
import os, requests

base = f"https://{os.getenv('AUTH')}@{os.getenv('OLLAMA_HOST')}"
url  = f"{base}/api/generate"

with requests.post(url, json={
    "model": "llama3.1:8b",
    "prompt": "Stream a short haiku about oceans.",
    "stream": True
}, stream=True, timeout=300) as r:
    for line in r.iter_lines():
        if not line:
            continue
        # each line is a JSON object
        chunk = line.decode("utf-8")
        # parse and extract only the token/text field
        # { "response": "<token>", "done": false, ... }
        # stop when done==true
```

**Serverless note:** runtime actions are async and sit behind a streamer. Your action will receive:

- `stream_host`, `stream_port` → open a socket and **send** partial results.  
- Return `{"streaming": true, ...}` on the first response to signal the client to switch to stream mode.


---

## 7) Testing streaming with a mock

We provide a **Stream Mock** to test locally (no sockets required). Typical pattern:

```python
from tests.streaming import StreamMock   # example import path
from my_action import stream_iter        # yields chunks

mock = StreamMock()
args = mock.args()                       # provides stream_host/port
mock.start()                             # start listener

# produce and send pieces
for piece in stream_iter(n=5):
    mock.send(piece)

captured = mock.stop()                   # list of pieces received
assert captured[-1]["done"] is True
```

This lets you validate your iterator and the “done” protocol end‑to‑end.


---

## 8) Exercises

### Exercise 1 — Add secrets to the LLM action
- Read `OLLAMA_HOST` and `AUTH` from `args` (fallback to env).  
- Add deploy annotations so secrets propagate automatically.  
- Confirm: the **URL test** passes.

### Exercise 2 — Fix the streaming decoder
- The Ollama stream returns **JSON lines**; parse each line and extract `response`.  
- Append partial `response` to an accumulator; handle `done==true`.  
- Confirm: the **streaming test** passes.

### Exercise 3 — Add a **model switcher**
- If input is `llama`, set model to `llama3.1:8b`.  
- If input is `deepseek`, set model to the corresponding tag.  
- Post‑process DeepSeek’s special “thinking” content: wrap it in `[...]` to keep it readable.  
- Deploy and verify in the UI.

Example skeleton:

```python
model = "llama3.1:8b"
if user_input.strip().lower() == "deepseek":
    model = "deepseek-r1:7b"             # example tag

# decode streamed JSON lines
line = line.decode("utf-8")
obj = json.loads(line)
piece = obj.get("response", "")

# Make hidden “thinking” visible for certain models
piece = normalize_thinking(piece)        # your helper
```


---

## 9) Deploy & use in the UI

- `ops ide deploy <your-package>` to ship changes (or use **dev mode** for incremental redeploys).  
- In **Pinocchio** (the UI), open your chat action, type `llama` or `deepseek` to switch models, and test streaming.  
- If a model takes too long or loops, adjust execution **timeout** in action parameters.


---

## 10) Key takeaways

- Always read secrets from `args`, fallback to env → works in **tests and prod**.  
- Use **annotations** so deploy injects secrets for you.  
- Streaming = JSON‑lines iterator; parse, extract `response`, stop at `done:true`.  
- Test with the **Stream Mock**; switch models at runtime with a simple command.  

You now have a robust pattern for **secure LLM calls** and **responsive streaming UIs**. 🚀
