নিচে **“Understanding Retrievers and Chains”** টপিকের ওপর একটা **short, runnable notebook-style project** দিলাম — **cell by cell**, সাথে explanation।

এটা তোমাকে শুধু definition না, **actual workflow** বুঝাবে:

* **Retriever** কী করে
* **Chain** কী করে
* দুটো একসাথে কীভাবে **RAG pipeline** বানায়

**একটা গুরুত্বপূর্ণ বাস্তবতা আগে:** LangChain v1-এ পুরনো অনেক **chain** utility মূল `langchain` package থেকে সরিয়ে `langchain-classic`-এ নেওয়া হয়েছে। Retrieval still core concept, কিন্তু `create_retrieval_chain`-এর মতো helper এখন `langchain_classic` থেকে ব্যবহার করা হয়। এছাড়া official docs retriever-কে এমন interface হিসেবে define করে যা **unstructured query** নিয়ে documents return করে। ([LangChain Docs][1])

---

# Project: Simple Retriever + Chain Demo

## What this project does

আমরা একটি ছোট knowledge base বানাবো, তারপর:

1. text কে chunks-এ ভাগ করব
2. vector store-এ রাখব
3. retriever দিয়ে relevant chunks আনব
4. chain দিয়ে final answer generate করব

---

## Cell 1 — Install packages

```python
!pip install -q langchain langchain-classic langchain-community langchain-openai faiss-cpu
```

### Explanation

এই packages দরকার হবে:

* `langchain` → core interfaces
* `langchain-classic` → legacy/utility chains যেমন retrieval chain helpers
* `langchain-community` → FAISS vector store
* `langchain-openai` → OpenAI model + embeddings
* `faiss-cpu` → local vector database

---

## Cell 2 — Set API key

```python
import os

os.environ["OPENAI_API_KEY"] = "your_openai_api_key_here"
```

### Explanation

এখানে তোমার OpenAI API key বসাতে হবে।

ভুলটা যেটা beginners করে:

* code-এ permanent key রেখে দেয়
* GitHub-এ push করে ফেলে

এটা করো না। শেখার জন্য temporary ঠিক আছে, production-এর জন্য `.env` use করো।

---

## Cell 3 — Create sample documents

```python
from langchain_core.documents import Document

docs = [
    Document(page_content="A retriever finds relevant documents for a user query."),
    Document(page_content="A chain connects multiple steps into one workflow."),
    Document(page_content="Vector stores keep embeddings and support similarity search."),
    Document(page_content="Embeddings convert text into numerical vectors."),
    Document(page_content="Retrieval-Augmented Generation combines retrieval with answer generation."),
]
```

### Explanation

এখানে আমরা ছোট knowledge base বানালাম।

প্রতিটা `Document` object একটা chunk হিসেবে কাজ করবে।

---

## Cell 4 — Inspect the raw documents

```python
for i, doc in enumerate(docs, start=1):
    print(f"Document {i}: {doc.page_content}")
```

### Explanation

এটা শুধু sanity check।

অনেকে data load করার পর inspect-ই করে না।
তারপর problem হলে model-কে blame দেয়।
সেটা lazy debugging।

---

## Cell 5 — Split documents into chunks

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=60,
    chunk_overlap=10
)

split_docs = splitter.split_documents(docs)

for i, doc in enumerate(split_docs, start=1):
    print(f"Chunk {i}: {doc.page_content}")
```

### Explanation

Retriever বড় text-এর ওপর কাজ করার আগে text split করা হয়, যাতে ছোট retrievable units তৈরি হয়। LangChain retrieval docs-এও text splitters-কে retrieval pipeline-এর core building block হিসেবে দেখানো হয়েছে। ([LangChain Docs][1])

`chunk_size` ছোট রাখছি যাতে তুমি effect বুঝতে পারো।

---

## Cell 6 — Create embeddings

```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
```

### Explanation

Embedding model text-কে vector-এ convert করে। Official retrieval docs এটাকেই semantic similarity search-এর ভিত্তি হিসেবে explain করে। ([LangChain Docs][1])

---

## Cell 7 — Store chunks in FAISS

```python
from langchain_community.vectorstores import FAISS

vector_store = FAISS.from_documents(split_docs, embeddings)
```

### Explanation

এখানে chunks + embeddings FAISS-এ store হচ্ছে।

এটাই তোমার **vector store**।

Retriever usually vector store-এর ওপর বসে relevant document search করে।

---

## Cell 8 — Build a retriever

```python
retriever = vector_store.as_retriever(search_kwargs={"k": 2})
```

### Explanation

এখানে retriever তৈরি হলো।

`k=2` মানে top 2 relevant chunks return করবে।

Retriever-এর মূল কাজ answer লেখা না — **relevant context আনা**। এই point official docs-এও clear। ([LangChain Docs][1])

---

## Cell 9 — Test the retriever directly

```python
query = "What does a retriever do?"

retrieved_docs = retriever.invoke(query)

for i, doc in enumerate(retrieved_docs, start=1):
    print(f"Retrieved {i}: {doc.page_content}")
```

### Explanation

এখানে শুধু retriever test করছি।

এটা খুব important step।

কারণ:

* answer খারাপ হলে problem chain-এ?
* না retriever-এ?
* না data-তেই?

এই debugging discipline না থাকলে তুমি শুধু copy-paste coder হয়ে থাকবে।

---

## Cell 10 — Create the language model

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini", temperature=0)
```

### Explanation

এখানে LLM initialize করা হচ্ছে।

`temperature=0` দিলে output বেশি stable হয়।
Learning phase-এ এটা ভালো।

---

## Cell 11 — Build the prompt for the chain

```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer the user's question using only the provided context. If the answer is not in the context, say you don't know."),
    ("human", "Context:\n{context}\n\nQuestion:\n{input}")
])
```

### Explanation

এই prompt chain-কে discipline দেয়।

নইলে model নিজের মতো বানিয়ে উত্তর দিতে পারে।

এখানে clear rule:

* context only
* context-এ না থাকলে admit ignorance

এটাই grounded answer-এর minimum standard।

---

## Cell 12 — Create the document-combining chain

```python
from langchain_classic.chains.combine_documents import create_stuff_documents_chain

document_chain = create_stuff_documents_chain(llm, prompt)
```

### Explanation

এই chain retrieved documents নিয়ে LLM-কে দেয়।

`stuff` strategy মানে retrieved docs একসাথে prompt-এ ঢুকিয়ে answer generate করা।

---

## Cell 13 — Create the retrieval chain

```python
from langchain_classic.chains import create_retrieval_chain

retrieval_chain = create_retrieval_chain(retriever, document_chain)
```

### Explanation

এটাই আসল point।

এখানে:

* **Retriever** relevant docs আনে
* **Chain** docs + question combine করে
* **LLM** final answer লেখে

LangChain docs-এ retrieval থেকে RAG-এ যাওয়ার এই exact idea explain করা হয়েছে, আর `create_retrieval_chain` retrieved source docs output context হিসেবে propagate করতে পারে। ([LangChain Docs][1])

---

## Cell 14 — Ask a question through the full chain

```python
response = retrieval_chain.invoke({"input": "What is a chain?"})

print("Answer:\n")
print(response["answer"])
```

### Explanation

এখানে full pipeline run হচ্ছে।

এই stage-এ chain answer দেয়, কিন্তু answer-এর ভিতরের raw knowledge retriever থেকে এসেছে।

---

## Cell 15 — See which documents were actually retrieved

```python
response = retrieval_chain.invoke({"input": "What is retrieval-augmented generation?"})

print("Answer:\n")
print(response["answer"])

print("\nRetrieved Context:\n")
for i, doc in enumerate(response["context"], start=1):
    print(f"{i}. {doc.page_content}")
```

### Explanation

এটা খুব দরকারি।

কারণ real-world app-এ user trust build করতে source/context দেখানো useful। Official LangChain material-এও উল্লেখ আছে যে built-in retrieval chain output-এ retrieved docs `context` key-তে পাওয়া যায়। ([LangChain Docs][2])

---

## Cell 16 — Compare retriever vs full chain

```python
question = "What does a vector store do?"

print("STEP 1: Retriever output\n")
docs_only = retriever.invoke(question)
for i, doc in enumerate(docs_only, start=1):
    print(f"{i}. {doc.page_content}")

print("\n" + "="*50 + "\n")

print("STEP 2: Full chain answer\n")
final_response = retrieval_chain.invoke({"input": question})
print(final_response["answer"])
```

### Explanation

এই cell-টা conceptually সবচেয়ে important।

এখানে তুমি difference দেখবে:

* **Retriever** → raw relevant text আনে
* **Chain** → retrieved text use করে final answer বানায়

এই distinction না বুঝলে তুমি শুধু শব্দ মুখস্থ করছ, system না।

---

## Cell 17 — Interactive version

```python
while True:
    q = input("\nAsk a question (or type 'exit'): ")
    if q.lower() == "exit":
        break

    result = retrieval_chain.invoke({"input": q})
    print("\nAnswer:", result["answer"])
```

### Explanation

এখন এটা mini Q&A app হয়ে গেছে।

---

# What you learned

## Retriever

Retriever-এর কাজ:

* user query নেয়
* relevant docs আনে
* answer লেখে না

## Chain

Chain-এর কাজ:

* workflow define করে
* retriever output + prompt + model combine করে
* final answer generate করে

## Together

এই দুইটা মিলে basic **RAG app** তৈরি হয়। Official LangChain retrieval docs-এও retrieval pipeline থেকে RAG-এ যাওয়াকে এইভাবেই frame করা হয়েছে। ([LangChain Docs][1])

---

# Brutal truth

তুমি যদি এই notebook run করো আর ভাবো “আমি retriever আর chain বুঝে ফেলেছি,” তাহলে তুমি নিজেকে ঠকাচ্ছ।

কারণ real difficulty এখানে না।
আসল difficulty এখানে:

* chunk size tuning
* top-k tuning
* irrelevant retrieval
* noisy documents
* prompt leakage
* source attribution
* evaluation

এই notebook foundation।
Product না।
Portfolio-level demo-এর skeleton।

---


[1]: https://docs.langchain.com/oss/python/langchain/retrieval?utm_source=chatgpt.com "Retrieval - Docs by LangChain"
[2]: https://docs.langchain.com/oss/python/integrations/vectorstores/sqlserver?utm_source=chatgpt.com "Sqlserver integration - Docs by LangChain"
