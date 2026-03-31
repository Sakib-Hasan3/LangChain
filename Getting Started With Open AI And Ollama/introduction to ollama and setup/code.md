নিচে **Ollama + LangChain setup** Ollama এখন macOS, Windows, Linux-এ available, local API defaultভাবে `http://localhost:11434/api`-এ serve করে, আর LangChain-এর official Ollama integration এখন `langchain-ollama` package-এ থাকে। ([Ollama Docs][1])

## আগে এটা terminal-এ করো

### 1) Ollama install

* **Windows:** installer দিয়ে install করো. ([Ollama Docs][2])
* **macOS:** app install করো, বা Homebrew use করতে পারো. ([Ollama Docs][3])
* **Linux:** service হিসেবে চালানো যায়; `systemctl` examples official docs-এ আছে. ([Ollama Docs][4])

### 2) একটা chat model pull করো

```bash
ollama pull llama3.1
```

Official LangChain docs `ollama pull <name-of-model>` flow-টাই recommend করে; example হিসেবেও `llama3` family দেখানো আছে. ([LangChain Docs][5])

### 3) একটা embedding model pull করো

```bash
ollama pull nomic-embed-text
```

RAG-এর জন্য chat model আর embedding model আলাদা রাখা বেশি sensible। LangChain-এর Ollama embeddings integration local embedding models use করার জন্যই design করা। ([LangChain Docs][6])

### 4) verify করো

```bash
ollama run llama3.1
```

বা API test:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1",
  "messages": [{"role": "user", "content": "Hello"}]
}'
```

Ollama quickstart অনুযায়ী local API chat endpoint এভাবেই hit করা যায়. ([Ollama Docs][1])

---

# Notebook: Ollama + LangChain local RAG demo

## Cell 1 — Install Python packages

```python
!pip install -qU langchain langchain-community langchain-text-splitters langchain-ollama faiss-cpu
```

### Explanation

এখানে:

* `langchain-ollama` → official LangChain Ollama integration
* `langchain-community` → FAISS vector store
* `langchain-text-splitters` → chunking
  LangChain docs এখন provider-specific package model follow করে, আর Ollama integration `langchain-ollama`-তেই থাকে. ([LangChain Docs][7])

---

## Cell 2 — Check that Ollama is reachable

```python
import requests

resp = requests.get("http://localhost:11434/api/tags")
print(resp.status_code)
print(resp.json())
```

### Explanation

এটা fail করলে notebook চালিয়ে লাভ নেই। তোমার local server-ই alive না।
Base URL defaultভাবে `http://localhost:11434/api` — এটা official docs-এ explicit. ([Ollama Docs][8])

---

## Cell 3 — Create a tiny knowledge base

```python
from langchain_core.documents import Document

docs = [
    Document(page_content="Ollama lets you run open models locally on your own machine."),
    Document(page_content="LangChain helps developers build LLM applications using models, retrieval, and chains."),
    Document(page_content="A retriever finds relevant document chunks for a user query."),
    Document(page_content="Embeddings convert text into vectors so semantic similarity search can work."),
    Document(page_content="A chain combines multiple steps such as retrieval, prompting, and generation."),
]
```

### Explanation

এখানে আমরা sample knowledge base বানাচ্ছি।
শুরুতে external PDF না নিয়ে tiny dataset use করা ভালো, কারণ তুমি আগে pipeline verify করবে, fantasy না। এটা prototype discipline. Embeddings semantic similarity-এর জন্য text-কে vector-এ map করে — LangChain docs এটাকেই core idea হিসেবে explain করে. ([LangChain Docs][9])

---

## Cell 4 — Inspect documents

```python
for i, doc in enumerate(docs, start=1):
    print(f"{i}. {doc.page_content}")
```

### Explanation

এই step skip করলে debugging harder হবে।
তোমার data-ই যদি ভুল হয়, model blame করা বোকামি।

---

## Cell 5 — Split text into chunks

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=80,
    chunk_overlap=15
)

split_docs = splitter.split_documents(docs)

for i, doc in enumerate(split_docs, start=1):
    print(f"Chunk {i}: {doc.page_content}")
```

### Explanation

Retrieval pipeline-এ chunking central step। বড় text directly search না করে chunks-এ ভেঙে vectorize করা হয়, যাতে relevant অংশ ফেরত আনা যায়. LangChain retrieval/RAG materials এই flow-টাই use করে. ([LangChain Docs][10])

---

## Cell 6 — Create Ollama embeddings

```python
from langchain_ollama import OllamaEmbeddings

embeddings = OllamaEmbeddings(model="nomic-embed-text")
```

### Explanation

এখানে local embedding model use হচ্ছে।
LangChain docs অনুযায়ী `OllamaEmbeddings`-ও `langchain-ollama` package থেকেই আসে. ([LangChain Docs][6])

---

## Cell 7 — Build a FAISS vector store

```python
from langchain_community.vectorstores import FAISS

vectorstore = FAISS.from_documents(split_docs, embeddings)
```

### Explanation

এখানে chunks-এর embeddings compute হয়ে FAISS-এ store হচ্ছে।
এটাই semantic retrieval-এর searchable memory layer।

---

## Cell 8 — Create a retriever

```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 2})
```

### Explanation

Retriever answer লেখে না। শুধু relevant chunks আনে।
এই distinction না বুঝলে RAG নিয়ে তোমার বোঝাপড়া shallow থাকবে।

---

## Cell 9 — Test retrieval only

```python
query = "What does a retriever do?"
results = retriever.invoke(query)

for i, doc in enumerate(results, start=1):
    print(f"Result {i}: {doc.page_content}")
```

### Explanation

এখানে শুধু retrieval test হচ্ছে।
এটা mandatory mindset: আগে search quality যাচাই করো, তারপর generation। না হলে তুমি বুঝবেই না failure কোথায়।

---

## Cell 10 — Create the local chat model

```python
from langchain_ollama import ChatOllama

llm = ChatOllama(
    model="llama3.1",
    temperature=0
)
```

### Explanation

LangChain-এর current Ollama chat integration `ChatOllama` use করে. Official docs example-ও এটাই. ([LangChain Docs][7])

---

## Cell 11 — Quick direct model test

```python
response = llm.invoke("Explain Ollama in one short sentence.")
print(response.content)
```

### Explanation

এখানে chain-এর আগে raw model test হচ্ছে।
এটা fail করলে chain build করার মানে নেই।

---

## Cell 12 — Create a prompt template

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_template("""
You are a helpful assistant.
Answer the question using only the context below.
If the answer is not in the context, say: "I don't know based on the provided context."

Context:
{context}

Question:
{input}
""")
```

### Explanation

এটা hallucination কমানোর basic control।
Grounded answering without constraints প্রায়ই garbage হয়।

---

## Cell 13 — Create the document QA chain

```python
from langchain.chains.combine_documents import create_stuff_documents_chain

document_chain = create_stuff_documents_chain(llm, prompt)
```

### Explanation

`stuff` strategy মানে retrieved docs একসাথে prompt-এ ঢোকানো হবে।
Small demo-র জন্য এটা যথেষ্ট।

---

## Cell 14 — Create the full retrieval chain

```python
from langchain.chains import create_retrieval_chain

retrieval_chain = create_retrieval_chain(retriever, document_chain)
```

### Explanation

এটাই full RAG flow:

* retriever relevant docs আনে
* chain prompt বানায়
* model answer generate করে
  LangChain RAG docs retrieval + generation pipeline-কেই এইভাবে structure করে. ([LangChain Docs][10])

---

## Cell 15 — Ask a question

```python
response = retrieval_chain.invoke({"input": "What is Ollama?"})
print(response["answer"])
```

### Explanation

এখানে full local pipeline চলল।
No cloud API. No OpenAI key. Local model, local embeddings, local retrieval.

---

## Cell 16 — Show retrieved context too

```python
response = retrieval_chain.invoke({"input": "What is a chain?"})

print("Answer:\n")
print(response["answer"])

print("\nRetrieved context:\n")
for i, doc in enumerate(response["context"], start=1):
    print(f"{i}. {doc.page_content}")
```

### Explanation

এটা useful কারণ এখন তুমি দেখতে পারবে model কী context দেখে answer দিল।
Blind trust না — inspectable workflow।

---

## Cell 17 — Interactive version

```python
while True:
    q = input("\nAsk a question (type 'exit' to stop): ")
    if q.lower() == "exit":
        break

    result = retrieval_chain.invoke({"input": q})
    print("\nAnswer:", result["answer"])
```

### Explanation

এখন এটা tiny local chatbot।

---

# What this teaches you

## Ollama side

* local model serve করে
* local API দেয় at `localhost:11434/api`
* interactive run + API usage দুইটাই support করে. ([Ollama Docs][1])

## LangChain side

* `ChatOllama` → local chat model
* `OllamaEmbeddings` → local embeddings
* retriever + chain → local RAG app. ([LangChain Docs][7])

---





[1]: https://docs.ollama.com/quickstart?utm_source=chatgpt.com "Quickstart"
[2]: https://docs.ollama.com/windows?utm_source=chatgpt.com "Windows"
[3]: https://docs.ollama.com/macos?utm_source=chatgpt.com "macOS"
[4]: https://docs.ollama.com/linux?utm_source=chatgpt.com "Linux"
[5]: https://docs.langchain.com/oss/python/integrations/llms/ollama?utm_source=chatgpt.com "Ollama integration - Docs by LangChain"
[6]: https://docs.langchain.com/oss/python/integrations/embeddings/ollama?utm_source=chatgpt.com "OllamaEmbeddings integration - Docs by LangChain"
[7]: https://docs.langchain.com/oss/python/integrations/chat/ollama?utm_source=chatgpt.com "ChatOllama integration - Docs by LangChain"
[8]: https://docs.ollama.com/api/introduction?utm_source=chatgpt.com "Introduction"
[9]: https://docs.langchain.com/oss/python/integrations/embeddings?utm_source=chatgpt.com "Embedding model integrations - Docs by LangChain"
[10]: https://docs.langchain.com/oss/python/langchain/rag?utm_source=chatgpt.com "Build a RAG agent with LangChain"
