# Lesson 6 – Vector Databases

Welcome to the sixth lesson of the **Private AI course with Apache Open Serverless**.  
In this lesson, we will talk about **vector databases**, **embeddings**, **vector search**, and how to **import PDFs** for use with vector databases.

---

## 1. Setup

Start from your fork of the starter repository.  
Make sure to update it if it is behind. Then, as usual, launch your **codespace** and open **Lesson 6**.

---

## 2. What Are Vector Databases?

Vector databases are specialized databases designed to efficiently store and search **vector embeddings**.

We will use **Milvus**, a NoSQL database optimized for vector searches.

- **Vector Search** = finding semantically similar data points by comparing their vector embeddings.  
- Example: the embeddings of *cat* and *dog* are closer to each other than *cat* and *orange*.

This is the foundation for **RAG (Retrieval Augmented Generation)**, which we will study in the next lesson.

---

## 3. Accessing Milvus

To connect to Milvus we need:

- **Host** (where it is located)
- **Database name**
- **Access token**

Using these, we create a **Milvus client** in Python.

Example setup:

```python
from pymilvus import MilvusClient

client = MilvusClient(uri="MILVUS_HOST", token="TOKEN", db_name="DATABASE")
print(client.list_collections())
```

---

## 4. Collections and Schema

- A **collection** in Milvus is like a table in relational databases.  
- Each collection requires a **schema** and can have **indexes** for faster searches.

### Example schema
- **id** (auto-generated primary key)  
- **text** (string)  
- **embedding** (vector of fixed size, e.g. 1024)  

We also create an **index** on the embedding field, specifying the similarity metric (e.g. IP, L2).

---

## 5. Insert and Query

Insert a record with both text and its vector embedding.

```python
data = {
  "id": 1,
  "text": "Hello World",
  "embedding": [0.12, 0.55, ...]  # Example numbers
}
client.insert("test_collection", data)
```

Retrieve it with:

```python
results = client.query("test_collection", output_fields=["text"])
```

---

## 6. Embeddings

**Embeddings** = numerical representations of text, created by an **embedding model**.

- Similar words → close embeddings.  
- Example: *dog* is closer to *cat* than to *orange*.

We use an embedding API (OpenAI-compatible) to convert text into vectors.

```python
from openai import OpenAI

client = OpenAI(base_url="OLLAMA_URL", api_key="dummy")
embedding = client.embeddings.create(model="mistral-embed", input="Hello world")
```

The output is a **vector array** suitable for similarity search.

---

## 7. Vector Search

Steps:

1. Insert texts and their embeddings.  
2. Provide a query text.  
3. Embed the query.  
4. Perform similarity search.

Example:

```python
query_embedding = client.embeddings.create(model="mistral-embed", input="test")

results = milvus_client.search(
    collection_name="test_collection",
    data=[query_embedding],
    limit=3,
    output_fields=["text"]
)

for r in results[0]:
    print(r["entity"]["text"], "distance:", r["distance"])
```

This finds the most semantically similar texts.

---

## 8. PDF Import

We can load documents into the vector database by extracting their text.

### Example with Bitcoin whitepaper:

```python
import fitz  # PyMuPDF

doc = fitz.open("bitcoin.pdf")
page = doc[0]
text = page.get_text()
```

Split the text into **sentences** before embedding (shorter chunks work better).

### Using a loader

With the `opsai` loader:

```bash
opsai loader bitcoin.pdf vdb_load
```

- Extracts text from PDF.  
- Splits into sentences.  
- Embeds each sentence.  
- Stores results in the vector DB.

Now you can search for terms like *bitcoin* or *merkle* and retrieve related sentences.

---

## 9. Exercise

Modify the loader to also **import content from the web**:

1. If the input starts with `http`, fetch the webpage.  
2. Use **BeautifulSoup** to extract the text.  
3. Split into sentences (using regex instead of heavy tokenizers).  
4. Embed and insert into Milvus.

---

## 10. Summary

In this lesson we learned:

- What **vector databases** are.  
- How to use **Milvus** with collections and schemas.  
- What **embeddings** are and how to compute them.  
- How to perform **vector similarity searches**.  
- How to **import PDFs** into the vector database.  

This sets the stage for the next lesson: **Retrieval Augmented Generation (RAG)**.

---

**Next up → Lesson 7: Retrieval Augmented Generation**
