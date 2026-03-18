
---

# 🧠 1. কী বানাবো?

আমরা একটা simple AI app বানাবো:
👉 user input → LangChain → OpenAI → output

---

# ⚙️ 2. Environment Setup (Conda)

## Step 1: env create

```bash
conda create -n langchain_env python=3.11 -y
```

## Step 2: activate

```bash
conda activate langchain_env
```

---

# 📦 3. প্রয়োজনীয় Package Install

```bash
pip install langchain langchain-openai python-dotenv
```

👉 optional (later লাগবে):

```bash
pip install faiss-cpu chromadb
```

---

# 🔑 4. OpenAI API Key Setup

## `.env` file তৈরি করো

```
OPENAI_API_KEY=your_api_key_here
```

---

# 🧪 5. First LangChain Program

## `app.py`

```python
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

# LLM init
llm = ChatOpenAI(model="gpt-4o-mini")

# simple query
response = llm.invoke("LangChain কী সহজ ভাষায় বলো")

print(response.content)
```

---

## ▶️ Run করো

```bash
python app.py
```

---

# 🎯 6. Prompt Template ব্যবহার

👉 dynamic input নেওয়ার জন্য

```python
from langchain.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from dotenv import load_dotenv

load_dotenv()

prompt = ChatPromptTemplate.from_template(
    "{topic} বিষয়টি beginner level এ explain করো"
)

llm = ChatOpenAI(model="gpt-4o-mini")

chain = prompt | llm

result = chain.invoke({"topic": "LangChain"})
print(result.content)
```

---

# 🔗 7. Chain Concept (Core Idea)

👉 pipeline system

```
Input → Prompt → Model → Output
```

LangChain এ এটা লিখি:

```python
chain = prompt | llm
```

---

# 💬 8. Chat Style Prompt

```python
from langchain.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "তুমি একজন AI teacher"),
    ("human", "{question}")
])
```

---

# 📄 9. Mini Project (Simple AI Assistant)

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from dotenv import load_dotenv

load_dotenv()

llm = ChatOpenAI(model="gpt-4o-mini")

prompt = ChatPromptTemplate.from_messages([
    ("system", "তুমি একজন helpful assistant"),
    ("human", "{question}")
])

chain = prompt | llm

while True:
    q = input("Ask: ")
    if q == "exit":
        break

    res = chain.invoke({"question": q})
    print("AI:", res.content)
```

---

# 🧪 10. Common Error Fix

## ❌ API key error

👉 `.env` ঠিকমতো load হয়েছে কিনা check করো

---

## ❌ Module not found

```bash
pip install langchain-openai
```

---

## ❌ Wrong Python version

👉 Python 3.10–3.11 best

---

# ⚡ 11. Next Step (Important)

এখন তুমি ready for:

## 🔥 Level 2:

👉 RAG (PDF chatbot)

## 🔥 Level 3:

👉 AI Agent (tool use)

## 🔥 Level 4:

👉 LangGraph (advanced workflow)

---

# 💡 Final Insight

👉 LangChain শেখার shortcut নেই
👉 কিন্তু 3টা জিনিস master করলে সব clear:

1. Prompt
2. Chain
3. RAG

---

# 🚀 Builder Tip (তোমার জন্য 🔥)

তুমি YouTuber mindset হলে:

👉 এই setup দিয়ে বানাতে পারো:

* AI script generator
* video idea generator
* auto content tool

---

