
---

# 📦 Cell 1 — Package install

```python
!pip install -q langchain langchain-community pypdf
```

👉 কাজ:

* LangChain install
* PDF পড়ার জন্য `pypdf`
* loaders ব্যবহারের জন্য `langchain-community`

---

# 📥 Cell 2 — Import libraries

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
```

👉 কাজ:

* `PyPDFLoader` → PDF কে text এ convert করবে
* `RecursiveCharacterTextSplitter` → text কে ছোট ছোট chunk করবে

---

# 📂 Cell 3 — PDF path set

```python
pdf_path = "your_file.pdf"
```

👉 এখানে তোমার PDF file এর নাম/পাথ দিবে

---

# 📄 Cell 4 — PDF load করা

```python
loader = PyPDFLoader(pdf_path)
docs = loader.load()
```

👉 কী হচ্ছে এখানে:

* পুরো PDF page-wise load হচ্ছে
* `docs` = list of pages

---

# 🔢 Cell 5 — কত পেজ আছে দেখো

```python
print("Total pages:", len(docs))
```

👉 বুঝতে পারবে PDF-এ কয়টা page আছে

---

# 👀 Cell 6 — প্রথম পেজ preview

```python
print(docs[0].page_content[:1000])
```

👉 প্রথম page-এর কিছু text দেখাবে
👉 check করার জন্য খুব important

---

# ⚙️ Cell 7 — Best PDF Splitter create

```python
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=150,
    separators=["\n\n", "\n", ".", " ", ""]
)
```

👉 এটা সবচেয়ে গুরুত্বপূর্ণ অংশ 🔥

### এখানে কী হচ্ছে:

### 🔹 chunk_size = 1000

* প্রতি chunk max 1000 characters

---

### 🔹 chunk_overlap = 150

* আগের chunk-এর শেষ 150 character পরের chunk-এ থাকবে
  👉 context maintain করার জন্য

---

### 🔹 separators

```python
["\n\n", "\n", ".", " ", ""]
```

মানে:

1. paragraph দিয়ে কাটবে
2. না পারলে line দিয়ে
3. না পারলে sentence দিয়ে
4. না পারলে word দিয়ে
5. শেষে character

👉 এজন্য একে "Recursive" বলে

---

# ✂️ Cell 8 — Text split করা

```python
chunks = splitter.split_documents(docs)
```

👉 কী হলো:

* পুরো PDF → ছোট ছোট chunk হয়ে গেল
* প্রতিটা chunk এখন LLM-friendly

---

# 📊 Cell 9 — মোট chunk দেখো

```python
print("Total chunks:", len(chunks))
```

👉 কতগুলো piece তৈরি হয়েছে দেখাবে

---

# 📖 Cell 10 — প্রথম chunk দেখো

```python
print(chunks[0].page_content)
```

👉 প্রথম chunk-এর text

---

# 🧾 Cell 11 — Metadata check

```python
print(chunks[0].metadata)
```

👉 সাধারণত থাকে:

* page number
* source file

---

# 🔍 Cell 12 — কয়েকটা chunk একসাথে দেখো

```python
for i in range(3):
    print(f"\n--- Chunk {i+1} ---")
    print(chunks[i].page_content[:300])
    print("Metadata:", chunks[i].metadata)
```

👉 splitting ঠিকভাবে হয়েছে কিনা বুঝতে পারবে

---

# 🧠 Visual Idea

ধরো original text:

```
[----------1000 chars----------]
```

Split হওয়ার পর:

```
Chunk 1: [----------1000----------]
Chunk 2:        [----------1000----------]
                 ↑ overlap (150)
```

---

# 🎯 কেন এটা "Best for PDF"?

✔ PDF-এ paragraph থাকে → first priority
✔ structure maintain করে
✔ context loss কম
✔ LLM ভালো বুঝে

---

# ⚠️ Common Mistake

### ❌ খুব ছোট chunk

```python
chunk_size=200
```

👉 context নষ্ট হয়ে যাবে

---

### ❌ overlap না রাখা

```python
chunk_overlap=0
```

👉 model confusion

---

### ❌ খুব বড় chunk

```python
chunk_size=2000
```

👉 token limit issue + slow embedding

---

# ✅ Best Settings (Recommended)

```python
chunk_size = 1000
chunk_overlap = 150
```

---

# 🚀 Final Pipeline

```text
PDF → Loader → Splitter → Embedding → Vector DB → Chatbot
```

---

# 🔥 Pro Insight (Important)

যদি তোমার PDF হয়:

* research paper (2 column)
* scanned image
* table-heavy

👉 তখন better use:

* `UnstructuredPDFLoader`
* `PDFPlumberLoader`

---
