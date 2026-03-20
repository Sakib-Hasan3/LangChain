
---

# 🧠 CharacterTextSplitter কী?

👉 এটা **simple splitting technique**
👉 কোনো recursion নেই
👉 নির্দিষ্ট separator দিয়ে text কেটে দেয়

---

# 🔥 Core Idea

```text
Text → fixed rule দিয়ে split → chunks
```

👉 Recursive splitter যেখানে smart ভাবে কাটে,
👉 Character splitter সেখানে **straightforward কাটে**

---

# 📦 Cell 1 — Import

```python
from langchain_text_splitters import CharacterTextSplitter
```

---

# 📄 Cell 2 — Sample text

```python
text = """Artificial intelligence (AI) is intelligence demonstrated by machines.
It is used in chatbots, automation, recommendation systems.
AI is transforming industries rapidly."""
```

---

# ⚙️ Cell 3 — Splitter তৈরি

```python
splitter = CharacterTextSplitter(
    separator="\n",
    chunk_size=100,
    chunk_overlap=20
)
```

---

# 🔍 Parameters Explanation

### 🔹 `separator="\n"`

👉 line break অনুযায়ী split করবে

---

### 🔹 `chunk_size=100`

👉 প্রতি chunk max 100 characters

---

### 🔹 `chunk_overlap=20`

👉 আগের chunk-এর শেষ 20 characters overlap করবে

---

# ✂️ Cell 4 — Text split করা

```python
chunks = splitter.split_text(text)
```

---

# 👀 Cell 5 — Output দেখো

```python
for i, chunk in enumerate(chunks):
    print(f"\n--- Chunk {i+1} ---")
    print(chunk)
```

---

# 📊 Visual Idea

ধরো text:

```text
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```

Split হলে:

```text
Chunk 1: AAAAAAAAAAAAAAAA...
Chunk 2:        BBBBBBBBBBB...
```

👉 overlap থাকলে আগের অংশও carry হবে

---

# 📄 PDF এর সাথে ব্যবহার

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain_text_splitters import CharacterTextSplitter

loader = PyPDFLoader("file.pdf")
docs = loader.load()

splitter = CharacterTextSplitter(
    separator="\n",
    chunk_size=800,
    chunk_overlap=100
)

chunks = splitter.split_documents(docs)

print(chunks[0].page_content)
```

---

# ⚠️ Important Limitation

❌ এটা smart না
❌ context break হয়ে যেতে পারে
❌ sentence মাঝখানে কেটে যেতে পারে

---

# 🆚 Character vs Recursive

| Feature          | Character   | Recursive  |
| ---------------- | ----------- | ---------- |
| Smart splitting  | ❌ No        | ✅ Yes      |
| Context preserve | ❌ Low       | ✅ High     |
| Speed            | ✅ Fast      | ⚖️ Medium  |
| Use case         | Simple text | Production |

---

# 🎯 কখন ব্যবহার করবে?

✔ ছোট text
✔ clean structured data
✔ testing / learning
✔ fixed format logs

---

# ❌ কখন ব্যবহার করবে না?

🚫 PDF chatbot
🚫 RAG system
🚫 research paper
🚫 messy text

---

# 🚀 Best Settings

```python
CharacterTextSplitter(
    separator="\n",
    chunk_size=800,
    chunk_overlap=100
)
```

---

# 🔥 Pro Insight

👉 Real-world AI systems এ:

```text
CharacterTextSplitter ❌
RecursiveCharacterTextSplitter ✅
```

---

# 💬 Summary

* Character splitter = simple cut
* Recursive splitter = smart cut
* Production = Recursive

---


