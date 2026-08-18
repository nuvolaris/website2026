# Lesson 7 – Building a RAG System

Welcome to the **seventh and final lesson** of this first edition of the Private AI course.  

---

## 1. Getting Started

Before starting, make sure to:

- **Update your fork** of the starter repository (to get the latest changes).  
- **Commit and push** your modifications to GitHub.  
- **Delete your Codespace** and restart it clean, to avoid issues caused by previous changes.  

Then:

1. Launch the Codespace.  
2. Log in.  
3. Select **Lesson 7**.  

---

## 2. What is RAG?

In this lesson we focus on **RAG – Retrieval Augmented Generation**.  
RAG is a method where Large Language Models (LLMs) are enhanced with **contextual information** retrieved from external data sources.  

- **Retrieval**: Find the most relevant pieces of information (chunks) using a vector database.  
- **Augmentation**: Provide these retrieved chunks as additional context to the LLM.  
- **Generation**: The LLM uses both the prompt and the retrieved knowledge to produce accurate answers.  

**Example:**  
If you ask *“Who is Lisa?”*, a standard LLM might give a generic answer.  
But with context (e.g., *Lisa Brennan, Steve Jobs’ daughter*), the model can provide **specific, correct responses**.

---

## 3. How It Works

1. **Embed text into vectors** using an embedding model.  
2. **Store vectors** in a **vector database** (we use **Milvus**).  
3. **Retrieve relevant vectors** with a similarity search when a query is made.  
4. **Concatenate retrieved chunks** as context to the user’s query.  
5. **Send query + context** to the LLM to generate an informed answer.  

This closes the loop of what we studied in the previous lessons (embedding, databases, loading data, etc.).  

---

## 4. Example: Lisa & Steve Jobs

We insert some records into the vector database:  

- Lisa is Steve Jobs’ first daughter.  
- Her full name is **Lisa Brennan**.  
- The computer **Apple Lisa** was named after her.  
- The name “Lisa” is said to be inspired by the **Mona Lisa**.  

When we ask *“Who is Lisa?”*:  
- **Without context** → the model returns a generic list of famous Lisas.  
- **With context** → the model explains both *Mona Lisa* and *Lisa Brennan*, and correctly identifies Lisa Brennan as Steve Jobs’ daughter.  

This shows the **power of context injection**.  

---

## 5. Chunking & Context Size

- Text must be split into **chunks** of ~500–800 words (~4,000 characters).  
- Chunks should be **semantically meaningful** (paragraphs, sections, etc.).  
- LLMs have a **context window limit** (e.g., Llama 3 supports up to 128k tokens).  
- Usually ~30 chunks (≈120k tokens) can be included, leaving space for the user query.  

Well-structured data → better results.  

---

## 6. The Loader

We created a **loader tool** that:  

- Manages multiple collections.  
- Supports inserting, searching, and deleting documents.  
- Imports **PDFs** and automatically splits them into chunks.  
- Works both **manually** and **automatically**.  

### Loader commands
- `* query` → Search for records.  
- `! substring` → Delete records containing substring.  
- `!! collection` → Delete entire collection.  
- `@collection` → Switch active collection.  

---

## 7. Importing Data

### Example: Bitcoin Whitepaper  
We download and load the **Bitcoin PDF** into the vector DB.  
- The document is split into ~4 chunks.  
- We can now ask questions like *“What is Bitcoin?”* and get detailed answers.  

### Example: Private Data (LinkedIn Contacts)  
We export LinkedIn contacts (CSV), convert them into text, and load them.  
Queries like:  
- *“List all contacts who are CEOs”*  
- *“List all contacts in software engineering”*  

⚠️ Public PDFs are usually **already known** by LLMs → private data gives more meaningful results.  

---

## 8. Multi-LLM & Multi-Collection Queries

The **RAG query action** supports:  

- **Multiple collections** → ask across different datasets.  
- **Multiple LLMs** → choose Llama, Phi, Mistral, DeepSeek, etc.  
- **Custom context size** → default is 30 chunks.  
- **Abbreviations** → use short prefixes for collection names.  

**Example usage:**  
```text
@jobs phi Who is Lisa?
```  
- `@jobs` → use the “jobs” collection.  
- `phi` → use Phi as LLM.  
- Query → *Who is Lisa?*  

---

## 9. Final Exercise

Build a **RAG for images**:  

1. Extend the database to store images.  
2. Upload images (via form or S3).  
3. Use an image recognition model to generate **descriptions**.  
4. Store descriptions in the vector database.  
5. Query the database with natural language → retrieve matching images.  

---

## 10. Conclusion

🎉 Congratulations! You have completed the first edition of the **Private AI Course with Apache Open Serverless**.  

We covered:  
- LLMs & assistants  
- Computer vision  
- Vector databases & embeddings  
- RAG systems  

This last lesson ties everything together into a **practical, useful system** you can extend for your own projects.  

Thank you for following the course. Stay tuned for the **next, enriched edition**! 🚀
