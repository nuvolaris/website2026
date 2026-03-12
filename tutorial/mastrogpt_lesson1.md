# Lesson 1: Integrated Services and First Exercises

Welcome to the **first real lesson** of the course.  
Lesson 0 was about setting up and configuring the environment. Now, we begin the practical part.

---

## 1. Accessing the Lesson

1. Make sure your environment is started and configured.  
2. Log in using your credentials.  
3. Select **Lesson 1** → this downloads all the files for the lesson.  
4. Open the files and preview them.  

This marks the beginning of the first real module.

---

## 2. Integrated Services Overview

The system comes with **integrated services** and several **examples (hello packages)** to help you learn.  
These examples serve two purposes:

- They demonstrate how to interact with the services.  
- They provide useful tools for debugging, testing, and exploration.  

Services included:  

- **Cache** → Redis  
- **Store** → Milvus (vector database)  
- **Streaming** → real-time responses  
- **LLM** → Ollama (LLaMA model)  

There are also two deliberately broken tests, meant to help you practice debugging.

---

## 3. Running Tests

1. After downloading, check the tests (refresh if needed).  
2. Run the tests:  
   - Most should pass.  
   - Two fail on purpose.  

### Fixing the Broken Test

- Locate the incorrect test (wrong output string).  
- Correct it (e.g., expected `"hello"` instead of `"hi"`).  
- Save the file.  

Run the tests again:  

- The **unit test** should now pass.  
- The **integration test** still fails until you **deploy** the code.  
- After deployment, the integration test passes as well.  

💡 Tip: Use **development mode (dev)** so that changes are automatically deployed.

---

## 4. Exploring the Examples

When you deploy, the system also installs the example services.  

Each participant will receive a unique deployment URL. Use your password to log in.  

Available services:  

- **LLM (LLaMA 3.1 8B)** → Handles natural language queries.  
  - Example: *“What is the capital of Italy?”* → *Rome*  
  - Small but powerful model, sufficient for course objectives.  

- **Streaming Service** → Sends ASCII characters one by one with delays to demonstrate streaming.  

- **Redis (Cache)** →  
  - Example: `INFO` returns system info.  
  - Example: `KEYS *` lists stored keys.  

- **S3-like Storage (via Nuvolaris)** →  
  - Upload, list, search, and delete files.  
  - Example: Upload a document, then process it with the LLM.  

- **Vector Database (Milvus)** →  
  - Insert text and perform vector similarity search.  
  - Example: Insert phrases containing “test” → search returns them in order of semantic distance.  
  - Core concept for **RAG (Retrieval-Augmented Generation)**.  

---

## 5. Command-Line Tools (ops)

Much of the work can also be done via CLI tools, especially **ops**.  

### Key Commands

- `ops action list` → list available actions.  
- `ops action create` → create a new action.  
- `ops action invoke` → run an action with parameters.  
- `ops action delete` → remove an action.  

### IDE & AI Subcommands

- `ops ide deploy <action>` → deploy a specific action.  
- `ops ai lesson` → download course lessons.  
- `ops ai user` → manage users.  
- `ops ai chat` → interact with LLM from the CLI.  
- `ops ai new <service>` → create a new service package.  
- `ops ai cli` → Python interpreter with environment integration.  

💡 `ops ide clean` removes leftover temporary files.

---

## 6. Exercise: Reverse Chat Service

Let’s create a new service: **Reverse Text**.

### Steps

1. Generate a new service:  

```bash
ops ai new reverse --package mypkg
```

This creates a new package `mypkg` with a `reverse` service and tests.  

2. Run the tests:  
   - Unit test passes.  
   - Integration test fails (service not deployed).  

3. Deploy the service:  

```bash
ops ide deploy mypkg/reverse
```

Now the integration test passes.  

4. Add the service to the **index** (`mastrogpt-index`) so it appears in the UI.  

```bash
ops ide deploy mastrogpt-index
```

5. Log in to the UI → the new **Reverse Service** is available.  
   - Input: `Pippo` → Output: `oppiP`  

### Implementation Example (Python)

```python
def main(args):
    text = args.get("input", "")
    if not text:
        return {"output": "Please provide an input"}
    return {"output": text[::-1]}
```

This simple example shows how to build, deploy, and test a new service.

---

## 7. Summary

- Lesson 1 introduces integrated services (LLM, Redis, Storage, Streaming, Vector DB).  
- You learned how to run and fix tests.  
- You explored CLI tools for deployment and development.  
- You created and deployed a new **Reverse Service** as your first exercise.  

🚀 With this, Lesson 1 is complete. You’re ready to continue building more advanced services!
