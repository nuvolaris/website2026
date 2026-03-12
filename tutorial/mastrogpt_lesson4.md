# Lesson 4 — Building a **Stateful Assistant** (MastroGPT • Apache Open Serverless)

In this lesson you’ll turn a stateless chat into a **stateful assistant** that remembers the conversation using the **OpenAI‑compatible API** (via Ollama) and **Redis** for history.

---

## 0) Prepare your environment

1. Open your **fork** of the MastroGPT starter on GitHub.  
2. If you see “**Update branch**” → **Sync fork** to pull the latest starter changes.  
3. Start your **Codespace** (or local dev env).  
4. In VS Code, open the 💬 extension and **log in**.  
5. From **Lessons**, download **Lesson 4** (PDF + Markdown).

> Lessons are self‑contained—you don’t need files from previous lessons to run this one.

---

## 1) What is an Assistant?

So far, chats were **stateless** (each request independent).  
An **assistant** is simply a chat that **keeps messaging history** and uses it to respond coherently.

We’ll use:
- **OpenAI‑compatible API** → standard Chat Completions interface (works with many LLMs, incl. **Ollama**).  
- **Redis** → to store and load conversation history between requests.

---

## 2) OpenAI‑compatible client (with Ollama)

Most providers expose a Chat Completions API with the same shape. With **Ollama**, auth is embedded in the base URL; the “API key” value is ignored by Ollama but included here for portability.

```python
# minimal client setup
from openai import OpenAI
import os

BASE_URL = os.getenv("OPENAI_BASE_URL")   # e.g. https://USER:PASS@ollama.example/api
API_KEY  = os.getenv("OPENAI_API_KEY")    # often ignored by Ollama

client = OpenAI(base_url=BASE_URL, api_key=API_KEY)

# non-streaming call
resp = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role":"user","content":"What is the capital of Italy?"}],
    stream=False,
)
print(resp.choices[0].message.content)
```

### Streaming response
```python
stream = client.chat.completions.create(
    model="llama3.1:8b",
    messages=[{"role":"user","content":"Write a tiny poem about oceans."}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content or ""
    print(delta, end="")
```

> Messages are a **list**. Each item has a `role` (`system`, `user`, `assistant`) and `content`.

---

## 3) A thin Python wrapper for chats

Wrap client usage in a simple class so you can append messages and complete repeatedly.

```python
# chat_wrap.py
from openai import OpenAI

class Chat:
    def __init__(self, base_url, api_key, model="llama3.1:8b", system=None):
        self.client = OpenAI(base_url=base_url, api_key=api_key)
        self.model = model
        self.messages = []
        if system:
            self.messages.append({"role":"system","content":system})

    def add(self, role_colon_text: str):
        # Accepts strings like "user: Hello" or "system: You answer with capitals"
        role, text = role_colon_text.split(":", 1)
        self.messages.append({"role": role.strip(), "content": text.strip()})

    def complete(self) -> str:
        resp = self.client.chat.completions.create(
            model=self.model, messages=self.messages, stream=False
        )
        msg = resp.choices[0].message.content
        self.messages.append({"role":"assistant","content":msg})
        return msg
```

Optional streaming method (exercise below) can be added as `complete_stream()`.

---

## 4) Exercise A — Add **streaming** to the wrapper

Implement `complete_stream()`:
1. Call the API with `stream=True`.
2. Iterate chunks; extract `delta.content` (may be `None`).
3. Accumulate text and **append a final `assistant` message** to history.
4. Return the full text.

Skeleton:
```python
def complete_stream(self):
    acc = []
    stream = self.client.chat.completions.create(
        model=self.model, messages=self.messages, stream=True
    )
    for chunk in stream:
        piece = (chunk.choices[0].delta.content or "")
        acc.append(piece)
        yield piece          # optionally yield live pieces to a socket/UI
    final = "".join(acc)
    self.messages.append({"role":"assistant","content":final})
```

---

## 5) Redis primer (for history)

**Redis** stores bytes; decode to/from UTF‑8 for strings. In Open Serverless, Redis is shared—**use your user prefix** to namespace keys.

Basic patterns:
```python
import os, json, redis

PREFIX = os.getenv("PREFIX","user:demo")
RURL   = os.getenv("REDIS_URL","redis://localhost:6379/0")
rd = redis.Redis.from_url(RURL)

# string
rd.set(f"{PREFIX}:greeting", "hello".encode())
val = rd.get(f"{PREFIX}:greeting").decode()

# list
rd.rpush(f"{PREFIX}:mylist", json.dumps({"a":1}).encode())
item = json.loads(rd.lindex(f"{PREFIX}:mylist", 0).decode())
```

---

## 6) History class (store/retrieve conversations)

A Redis‑backed **History** generates a unique id per conversation and stores the message list. Keys auto‑expire after 1 day.

```python
# history.py
import os, json, uuid, redis

class History:
    def __init__(self, conv_id=None, ttl_sec=86400):
        self.prefix = os.getenv("PREFIX","user:demo")
        self.key    = f"{self.prefix}:conv:{conv_id or uuid.uuid4()}"
        self.rd     = redis.Redis.from_url(os.getenv("REDIS_URL"))
        if conv_id is None:
            self.rd.expire(self.key, ttl_sec)

    def id(self):
        return self.key.split(":conv:",1)[1]

    def save(self, messages):
        self.rd.set(self.key, json.dumps(messages).encode())

    def load(self):
        raw = self.rd.get(self.key)
        return json.loads(raw.decode()) if raw else []
```

---

## 7) Putting it together — a **Stateful Assistant**

```python
# assistant.py
import os
from chat_wrap import Chat
from history import History

def run_assistant(user_input: str, conv_id: str|None):
    base = os.getenv("OPENAI_BASE_URL")
    key  = os.getenv("OPENAI_API_KEY")
    hist = History(conv_id=conv_id)
    chat = Chat(base_url=base, api_key=key, model="llama3.1:8b",
                system="You are a helpful assistant. Be concise.")

    # load old messages, if any
    prev = hist.load()
    if prev:
        chat.messages = prev

    # add latest user turn
    chat.add(f"user: {user_input}")
    answer = chat.complete()  # or chat.complete_stream()

    # persist full history
    hist.save(chat.messages)

    # return answer + new conversation id (to keep state in the client/UI)
    return {"output": answer, "state": {"conv_id": hist.id()}}
```

In your web action:
- Read `conv_id` from `args["state"]` if present, pass it to `run_assistant`.  
- Return the updated `state` so the UI can keep the same conversation thread.

---

## 8) Exercise B — Convert your **stateless chat** to **stateful**

1. **Load** history by `conv_id` if present.  
2. **Append** the new `{role:"user", content: input}` to messages.  
3. **Complete** via the OpenAI‑compatible API (optionally streaming).  
4. **Save** messages back to Redis and **return `state`** with `conv_id`.

**Bonus:** add a `/new` command that resets the history (new `conv_id`).

---

## 9) Tips & caveats

- With **Ollama**, auth is embedded in the **base URL**; other providers use headers. Keep code portable.  
- Keep histories **short** or summarize older turns to control token usage.  
- Always **namespace** Redis keys with your user `PREFIX`.  
- If you implement UI streaming, return `{ "streaming": true }` on the first response so the client switches to stream mode.

---

## 10) Checklist

- [ ] Synced fork & downloaded Lesson 4  
- [ ] Non‑streaming OpenAI‑compatible call works  
- [ ] Streaming call implemented in class wrapper  
- [ ] Redis history working (save/load)  
- [ ] Web action returns `state.conv_id` and reloads history on next turn

---

You now have a reusable pattern to build **assistants that remember**. Onward! 🚀
