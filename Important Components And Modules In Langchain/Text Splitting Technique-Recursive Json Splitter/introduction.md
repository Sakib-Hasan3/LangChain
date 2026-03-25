
---

# Recursive JSON Splitter কী?

**Recursive JSON Splitter** হলো এমন একটি text splitting technique, যা **JSON data**-কে তার **structure অনুযায়ী ছোট ছোট chunk**-এ ভাগ করে।

এটা সাধারণ text splitter-এর মতো character count দেখে split করে না।
বরং JSON-এর:

* keys
* nested objects
* lists
* hierarchy

এসব বুঝে logically split করে।

---

# আগে JSON কী?

JSON = **JavaScript Object Notation**

এটা structured data store করার একটি popular format।

উদাহরণ:

```json
{
  "course": "LangChain",
  "modules": [
    {
      "title": "Introduction",
      "duration": "10 min"
    },
    {
      "title": "Text Splitters",
      "duration": "20 min"
    }
  ]
}
```

এখানে data random text না, বরং structure-based।

---

# তাহলে normal text splitter কেন যথেষ্ট না?

ধরো তুমি JSON data-কে normal character splitter দিয়ে split করলে।

তাহলে chunk এমন হতে পারে:

* এক chunk এ key থাকবে
* next chunk এ value থাকবে
* nested object ভেঙে যাবে
* list item আলাদা হয়ে যাবে

ফলে:

* meaning নষ্ট হয়
* context ভেঙে যায়
* retrieval quality কমে যায়
* LLM বুঝতে problem করে

এই সমস্যা সমাধানের জন্য **Recursive JSON Splitter** ব্যবহার করা হয়।

---

# “Recursive” শব্দটার মানে কী?

**Recursive** মানে হলো step by step nested structure-এর ভিতরে ঢুকে split করা।

JSON-এর মধ্যে যদি থাকে:

* object-এর ভিতরে object
* list-এর ভিতরে object
* nested dictionary

তাহলে splitter parent structure follow করে নিচের level পর্যন্ত যায়, তারপর logical chunk তৈরি করে।

অর্থাৎ, এটা surface level-এ split করে না—
বরং hierarchy ধরে split করে।

---

# এটা কীভাবে কাজ করে?

Recursive JSON Splitter সাধারণত এভাবে কাজ করে:

## Step 1:

পুরো JSON structure read করে

## Step 2:

Top-level keys detect করে

## Step 3:

যদি nested object বা list থাকে, তাহলে তার ভিতরে যায়

## Step 4:

Logical boundary অনুযায়ী smaller chunks বানায়

## Step 5:

প্রতিটি chunk usable format-এ return করে

---

# কেন এটা দরকার?

## 1. Structured data preserve করে

JSON-এর original hierarchy বজায় থাকে

## 2. Context নষ্ট হয় না

key-value সম্পর্ক intact থাকে

## 3. Nested data handle করতে পারে

Deep object বা array থাকলেও split করতে পারে

## 4. Retrieval ভালো হয়

RAG system relevant JSON section retrieve করতে পারে

## 5. LLM-friendly output দেয়

chunk ছোট হয় কিন্তু meaning intact থাকে

---

# কোথায় ব্যবহার হয়?

Recursive JSON Splitter useful যখন তোমার data JSON format-এ থাকে। যেমন:

* API response
* configuration files
* NoSQL documents
* metadata records
* product catalogs
* chat/export logs
* nested knowledge base entries

---

# সাধারণ Text Splitter আর Recursive JSON Splitter-এর পার্থক্য

## CharacterTextSplitter

* character count দিয়ে split করে
* JSON structure বোঝে না

## RecursiveCharacterTextSplitter

* separator-based smart splitting করে
* কিন্তু JSON hierarchy naturally preserve করে না

## Recursive JSON Splitter

* JSON object structure বুঝে
* nested fields অনুযায়ী split করে
* logical relation preserve করে

---

# Example দিয়ে বুঝি

ধরো তোমার JSON data এমন:

```json
{
  "student": {
    "name": "Rahim",
    "department": "CSE",
    "courses": [
      {
        "title": "Machine Learning",
        "credit": 3
      },
      {
        "title": "Database Systems",
        "credit": 3
      }
    ]
  }
}
```

এখন normal splitter use করলে content random cut হতে পারে।

কিন্তু Recursive JSON Splitter use করলে logically split হতে পারে:

### Chunk 1

```json
{
  "student": {
    "name": "Rahim",
    "department": "CSE"
  }
}
```

### Chunk 2

```json
{
  "student": {
    "courses": [
      {
        "title": "Machine Learning",
        "credit": 3
      }
    ]
  }
}
```

### Chunk 3

```json
{
  "student": {
    "courses": [
      {
        "title": "Database Systems",
        "credit": 3
      }
    ]
  }
}
```

এখানে দেখা যাচ্ছে chunking হলেও structure remain করছে।

---

# এটা কেন RAG-এ useful?

RAG system-এ chunking খুব important।

যদি JSON data directly LLM-এ দাও:

* অনেক বড় হয়ে যেতে পারে
* irrelevant data mixed থাকবে
* token waste হবে

কিন্তু Recursive JSON Splitter:

* useful sub-part বের করে
* relevant section retriever-এ index করতে সাহায্য করে
* query অনুযায়ী exact part খুঁজে আনতে help করে

---

# Real-world use case

## 1. API response analysis

একটা ecommerce API response-এ অনেক product info আছে
→ chunk করে product-wise retrieve করা যায়

## 2. Chat/export logs

nested conversation data
→ message block অনুযায়ী split করা যায়

## 3. Config files

complex settings JSON
→ section-wise analysis করা যায়

## 4. Document metadata

author, section, tags, content block
→ structured retrieval possible হয়

---

# Conceptual comparison

ধরো তোমার কাছে এই JSON আছে:

```json
{
  "product": {
    "name": "Laptop",
    "price": 50000,
    "specs": {
      "ram": "16GB",
      "storage": "512GB SSD"
    }
  }
}
```

### Normal splitter problem:

এক chunk এ `"ram"`
আরেক chunk এ `"16GB"`
meaning weak হয়ে যায়

### Recursive JSON Splitter:

`specs` block আলাদা meaningful chunk হিসেবে রাখতে পারে

---

# Main advantage

সবচেয়ে বড় সুবিধা হলো:

> **এটা JSON-কে text হিসেবে না দেখে structured object হিসেবে treat করে।**

এই কারণেই এটা smart এবং reliable।

---

# কখন এটা ব্যবহার করা উচিত?

তুমি Recursive JSON Splitter ব্যবহার করবে যখন:

* data JSON format-এ
* nested structure আছে
* key-value relationship important
* RAG / retrieval task করছ
* structured API data নিয়ে কাজ করছ
* LLM-কে clean chunk দিতে চাও

---

# কখন এটা best না?

এটা best choice না যখন:

* তোমার data plain text
* JSON খুব ছোট
* structure important না
* simple string split enough

---

# সংক্ষেপে definition

**Recursive JSON Splitter** হলো এমন একটি splitting technique, যা JSON data-এর hierarchical structure follow করে recursively ছোট ছোট meaningful chunk তৈরি করে, যাতে context, nesting, এবং key-value relation বজায় থাকে।

---

# Key points to remember

* এটা JSON-specific splitter
* structure-aware
* nested object handle করতে পারে
* logical chunk তৈরি করে
* RAG ও retrieval-এ খুব useful
* normal text splitter-এর চেয়ে structured data-র জন্য অনেক better

---

# খুব সহজ ভাষায়

এক লাইনে বললে:

**Recursive JSON Splitter JSON file-কে তার structure না ভেঙে ছোট ছোট usable অংশে ভাগ করে।**

---


