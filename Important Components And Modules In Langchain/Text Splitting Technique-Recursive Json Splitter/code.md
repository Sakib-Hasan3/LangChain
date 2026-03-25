
LangChain-এর official docs অনুযায়ী, `RecursiveJsonSplitter` এখন `langchain_text_splitters` package থেকে import করা হয়, এটি JSON data-কে depth-first ভাবে ছোট chunk-এ ভাগ করে, `max_chunk_size` দিয়ে chunk size control করা যায়, আর list content split করতে `convert_lists=True` ব্যবহার করা যায়। ([LangChain Docs][1])

---

# 1) প্রথমে package install

### Cell 1

```bash
pip install -U langchain-text-splitters
```

### Explanation

এই cell দিয়ে `langchain_text_splitters` package install হবে।
`RecursiveJsonSplitter` এই package-এর মধ্যেই থাকে। ([LangChain Docs][1])

---

# 2) প্রয়োজনীয় library import

### Cell 2

```python
import json
from langchain_text_splitters import RecursiveJsonSplitter
```

### Explanation

এখানে:

* `json` ব্যবহার হবে JSON data print বা string-এ convert করার জন্য
* `RecursiveJsonSplitter` হলো main splitter class

---

# 3) একটি sample nested JSON data তৈরি

### Cell 3

```python
json_data = {
    "course": "LangChain",
    "level": "Beginner",
    "modules": [
        {
            "title": "Introduction",
            "duration": "10 min",
            "topics": [
                "What is LangChain?",
                "Why use LangChain?"
            ]
        },
        {
            "title": "Text Splitters",
            "duration": "20 min",
            "topics": [
                "CharacterTextSplitter",
                "RecursiveCharacterTextSplitter",
                "RecursiveJsonSplitter"
            ]
        }
    ],
    "instructor": {
        "name": "Sakib",
        "experience": "2 years",
        "skills": ["Python", "NLP", "LangChain"]
    }
}
```

### Explanation

এখানে আমরা manually একটা nested JSON বানালাম।
এটার মধ্যে আছে:

* normal key-value
* nested object
* list
* list-এর ভিতরে object

এই type-এর structured data split করার জন্যই `RecursiveJsonSplitter` useful।

---

# 4) JSON data সুন্দরভাবে print করে দেখা

### Cell 4

```python
print(json.dumps(json_data, indent=4))
```

### Explanation

এই cell শুধু data-টা readable format-এ দেখাবে।
`indent=4` দিলে output সুন্দরভাবে line-by-line দেখা যায়।

---

# 5) Splitter object তৈরি করা

### Cell 5

```python
splitter = RecursiveJsonSplitter(max_chunk_size=120)
```

### Explanation

এখানে আমরা splitter object বানাচ্ছি।

* `max_chunk_size=120` মানে splitter চেষ্টা করবে chunk-গুলোকে 120 character-এর মধ্যে রাখতে

LangChain docs অনুযায়ী, এই splitter chunk size character count দিয়ে measure করে। ([LangChain Docs][1])

---

# 6) JSON কে smaller JSON chunks-এ split করা

### Cell 6

```python
json_chunks = splitter.split_json(json_data=json_data)
```

### Explanation

এই cell-এ `split_json()` method ব্যবহার করে original JSON data-কে ছোট ছোট JSON chunk-এ split করা হচ্ছে।

এটা useful যখন তুমি chunk-গুলোকে আবার JSON object হিসেবেই রাখতে চাও।
LangChain docs-এ `split_json` method এভাবেই ব্যবহার করা হয়েছে। ([LangChain Docs][1])

---

# 7) Split হওয়া chunk গুলো দেখা

### Cell 7

```python
for i, chunk in enumerate(json_chunks, start=1):
    print(f"\n--- Chunk {i} ---")
    print(json.dumps(chunk, indent=4))
```

### Explanation

এখানে আমরা প্রতিটি chunk print করছি।

* `enumerate(..., start=1)` → chunk numbering শুরু হবে 1 থেকে
* `json.dumps(..., indent=4)` → JSON সুন্দরভাবে show করবে

এতে তুমি clearly দেখতে পারবে কিভাবে বড় JSON ছোট logical অংশে ভাগ হয়েছে।

---

# 8) String format-এ chunk নেওয়া

### Cell 8

```python
text_chunks = splitter.split_text(json_data=json_data)

for i, chunk in enumerate(text_chunks, start=1):
    print(f"\n--- Text Chunk {i} ---")
    print(chunk)
```

### Explanation

যদি তুমি JSON object না নিয়ে plain text/string আকারে chunk চাও, তাহলে `split_text()` ব্যবহার করবে।

LangChain docs অনুযায়ী, `split_text()` string content return করে। ([LangChain Docs][1])

---

# 9) প্রতিটি chunk-এর length check করা

### Cell 9

```python
for i, chunk in enumerate(text_chunks, start=1):
    print(f"Chunk {i} length = {len(chunk)}")
```

### Explanation

এই cell দেখাবে প্রতিটি chunk-এর character length কত।

এটা useful কারণ:

* chunk size কত হচ্ছে বোঝা যায়
* `max_chunk_size` tune করতে সুবিধা হয়

---

# 10) Document object তৈরি করা

### Cell 10

```python
docs = splitter.create_documents(texts=[json_data])

for i, doc in enumerate(docs, start=1):
    print(f"\n--- Document {i} ---")
    print(doc.page_content)
```

### Explanation

যদি তুমি LangChain pipeline-এ later use করতে চাও, তাহলে `create_documents()` খুব useful।

এটা chunk-গুলোকে `Document` object আকারে দেয়।
LangChain docs-এও `create_documents(texts=[json_data])` ব্যবহার করা হয়েছে। ([LangChain Docs][1])

---

# 11) List content split manage করার জন্য `convert_lists=True`

### Cell 11

```python
text_chunks_with_lists = splitter.split_text(
    json_data=json_data,
    convert_lists=True
)

for i, chunk in enumerate(text_chunks_with_lists, start=1):
    print(f"\n--- Converted List Chunk {i} ---")
    print(chunk)
```

### Explanation

LangChain docs অনুযায়ী, default behavior-এ splitter list split করে না।
এই কারণে কিছু chunk `max_chunk_size` এর চেয়ে বড় হতে পারে।
`convert_lists=True` দিলে list-কে আগে dict-like structure-এ convert করে, তারপর split করে—ফলে chunk size better control করা যায়। ([LangChain Docs][1])

---

# 12) `convert_lists=True` ব্যবহার করলে chunk sizes compare করা

### Cell 12

```python
print("Without convert_lists:")
print([len(chunk) for chunk in text_chunks])

print("\nWith convert_lists=True:")
print([len(chunk) for chunk in text_chunks_with_lists])
```

### Explanation

এই cell-এ আমরা compare করছি:

* normal split
* `convert_lists=True` split

এতে দেখা যাবে list conversion দিলে chunk sizes আরও balanced হতে পারে।

---

# Full Combined Version

নিচে সব code একসাথে দিলাম:

```python
import json
from langchain_text_splitters import RecursiveJsonSplitter

json_data = {
    "course": "LangChain",
    "level": "Beginner",
    "modules": [
        {
            "title": "Introduction",
            "duration": "10 min",
            "topics": [
                "What is LangChain?",
                "Why use LangChain?"
            ]
        },
        {
            "title": "Text Splitters",
            "duration": "20 min",
            "topics": [
                "CharacterTextSplitter",
                "RecursiveCharacterTextSplitter",
                "RecursiveJsonSplitter"
            ]
        }
    ],
    "instructor": {
        "name": "Sakib",
        "experience": "2 years",
        "skills": ["Python", "NLP", "LangChain"]
    }
}

print("Original JSON:\n")
print(json.dumps(json_data, indent=4))

splitter = RecursiveJsonSplitter(max_chunk_size=120)

json_chunks = splitter.split_json(json_data=json_data)
print("\nJSON Chunks:\n")
for i, chunk in enumerate(json_chunks, start=1):
    print(f"\n--- Chunk {i} ---")
    print(json.dumps(chunk, indent=4))

text_chunks = splitter.split_text(json_data=json_data)
print("\nText Chunks:\n")
for i, chunk in enumerate(text_chunks, start=1):
    print(f"\n--- Text Chunk {i} ---")
    print(chunk)

print("\nChunk Lengths:")
for i, chunk in enumerate(text_chunks, start=1):
    print(f"Chunk {i} length = {len(chunk)}")

docs = splitter.create_documents(texts=[json_data])
print("\nDocument Objects:\n")
for i, doc in enumerate(docs, start=1):
    print(f"\n--- Document {i} ---")
    print(doc.page_content)

text_chunks_with_lists = splitter.split_text(
    json_data=json_data,
    convert_lists=True
)

print("\nChunks with convert_lists=True:\n")
for i, chunk in enumerate(text_chunks_with_lists, start=1):
    print(f"\n--- Converted List Chunk {i} ---")
    print(chunk)

print("\nCompare Lengths:")
print("Without convert_lists:")
print([len(chunk) for chunk in text_chunks])

print("\nWith convert_lists=True:")
print([len(chunk) for chunk in text_chunks_with_lists])
```

---

# Output থেকে তুমি কী শিখবে

এই notebook run করলে তুমি বুঝবে:

* `split_json()` → JSON object আকারে chunk দেয়
* `split_text()` → string আকারে chunk দেয়
* `create_documents()` → LangChain `Document` object দেয়
* `convert_lists=True` → list-heavy JSON split করতে help করে

এসব behavior LangChain-এর official splitter docs-এর সঙ্গে consistent। ([LangChain Docs][1])

---

# খুব সহজ summary

**RecursiveJsonSplitter** use করবে যখন:

* data JSON format-এ
* nested structure আছে
* meaningful chunk দরকার
* পরে retriever / vector DB / RAG-এ use করতে চাও

