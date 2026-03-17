

# ল্যাংচেইন (LangChain) পরিচিতি

---

## ল্যাংচেইন কী? 🤖

**ল্যাংচেইন (LangChain)** হলো একটি ওপেন-সোর্স ফ্রেমওয়ার্ক যা **Large Language Models (LLMs)** ব্যবহার করে শক্তিশালী অ্যাপ্লিকেশন তৈরি করতে সাহায্য করে।

এটি **২০২২ সালে Harrison Chase** তৈরি করেন এবং এটি **Python** ও **JavaScript** উভয় ভাষায় পাওয়া যায়।

---

## কেন ল্যাংচেইন দরকার? 🤔

সাধারণ LLM (যেমন: ChatGPT, GPT-4) এর কিছু সীমাবদ্ধতা আছে:

| সমস্যা | ল্যাংচেইনের সমাধান |
|---|---|
| ❌ রিয়েল-টাইম ডেটা জানে না | ✅ বাহ্যিক ডেটা সোর্সের সাথে সংযোগ |
| ❌ নিজস্ব ডাটাবেস ব্যবহার করতে পারে না | ✅ কাস্টম ডেটা ইন্টিগ্রেশন |
| ❌ জটিল কাজ একা করতে পারে না | ✅ চেইন ও এজেন্ট ব্যবহার |
| ❌ মেমোরি সীমিত | ✅ কথোপকথনের মেমোরি ব্যবস্থাপনা |

---

## ল্যাংচেইনের মূল উপাদানসমূহ 🧩

### ১. মডেলস (Models)
```
বিভিন্ন LLM এর সাথে কাজ করার সুবিধা দেয়।
```

```python
from langchain.llms import OpenAI

# OpenAI মডেল ব্যবহার
llm = OpenAI(model_name="gpt-3.5-turbo", temperature=0.7)
response = llm("বাংলাদেশের রাজধানী কী?")
print(response)
```

### ২. প্রম্পট টেমপ্লেট (Prompt Templates)
```
গতিশীল প্রম্পট তৈরি করার জন্য ব্যবহৃত হয়।
```

```python
from langchain.prompts import PromptTemplate

# প্রম্পট টেমপ্লেট তৈরি
template = """
তুমি একজন বাংলা ভাষার শিক্ষক।
নিচের বিষয়ে সহজ ভাষায় ব্যাখ্যা দাও:

বিষয়: {topic}

উত্তর:
"""

prompt = PromptTemplate(
    input_variables=["topic"],
    template=template
)

# প্রম্পট তৈরি
final_prompt = prompt.format(topic="কৃত্রিম বুদ্ধিমত্তা")
print(final_prompt)
```

### ৩. চেইনস (Chains)
```
একাধিক ধাপকে একসাথে জুড়ে দেওয়ার পদ্ধতি।
```

```python
from langchain.chains import LLMChain
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate

# সাধারণ চেইন তৈরি
llm = OpenAI(temperature=0.7)

prompt = PromptTemplate(
    input_variables=["product"],
    template="{product} এর জন্য একটি বিজ্ঞাপনের স্লোগান লিখো।"
)

chain = LLMChain(llm=llm, prompt=prompt)

# চেইন চালানো
result = chain.run("চা")
print(result)
```

### ৪. মেমোরি (Memory)
```
কথোপকথনের ইতিহাস মনে রাখার ব্যবস্থা।
```

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain
from langchain.llms import OpenAI

# মেমোরি সহ কথোপকথন
llm = OpenAI(temperature=0.7)
memory = ConversationBufferMemory()

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# প্রথম প্রশ্ন
conversation.predict(input="আমার নাম রাহিম।")

# দ্বিতীয় প্রশ্ন - আগের কথা মনে থাকবে
conversation.predict(input="আমার নাম কী?")
# উত্তর: "আপনার নাম রাহিম।"
```

### ৫. এজেন্ট (Agents)
```
LLM নিজে সিদ্ধান্ত নেয় কোন টুল ব্যবহার করবে।
```

```python
from langchain.agents import load_tools, initialize_agent, AgentType
from langchain.llms import OpenAI

llm = OpenAI(temperature=0)

# টুলস লোড করা (যেমন: ক্যালকুলেটর, সার্চ)
tools = load_tools(["calculator", "serpapi"], llm=llm)

# এজেন্ট তৈরি
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# এজেন্ট ব্যবহার
agent.run("বাংলাদেশের জনসংখ্যা কত এবং সেটাকে ২ দিয়ে ভাগ করলে কত হবে?")
```

### ৬. ডকুমেন্ট লোডার ও ভেক্টর স্টোর (Document Loaders & Vector Stores)
```
নিজস্ব ডকুমেন্ট থেকে তথ্য খোঁজার ব্যবস্থা।
```

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS

# ডকুমেন্ট লোড করা
loader = TextLoader("bangla_document.txt")
documents = loader.load()

# ডকুমেন্ট ভাগ করা
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
docs = text_splitter.split_documents(documents)

# ভেক্টর স্টোরে সংরক্ষণ
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(docs, embeddings)

# প্রশ্ন করা
query = "বাংলাদেশের মুক্তিযুদ্ধ কবে হয়েছিল?"
results = vectorstore.similarity_search(query)
print(results[0].page_content)
```

---

## ল্যাংচেইনের আর্কিটেকচার 🏗️

```
┌─────────────────────────────────────────────────┐
│                  অ্যাপ্লিকেশন                      │
├─────────────────────────────────────────────────┤
│                                                   │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │  চেইনস   │  │ এজেন্ট  │  │ মেমোরি   │      │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│        │             │             │              │
│   ┌────▼─────────────▼─────────────▼─────┐      │
│   │          প্রম্পট টেমপ্লেট              │      │
│   └────────────────┬─────────────────────┘      │
│                    │                              │
│   ┌────────────────▼─────────────────────┐      │
│   │         LLM মডেলস                     │      │
│   │  (OpenAI, Hugging Face, Google, etc.) │      │
│   └────────────────┬─────────────────────┘      │
│                    │                              │
│   ┌────────────────▼─────────────────────┐      │
│   │       বাহ্যিক ডেটা সোর্স              │      │
│   │  (ডাটাবেস, API, ফাইল, ওয়েবসাইট)     │      │
│   └──────────────────────────────────────┘      │
│                                                   │
└─────────────────────────────────────────────────┘
```

---

## ল্যাংচেইন ইনস্টলেশন ⚙️

```bash
# মূল প্যাকেজ ইনস্টল
pip install langchain

# OpenAI সাপোর্ট
pip install langchain-openai

# কমিউনিটি প্যাকেজ
pip install langchain-community

# সব একসাথে
pip install langchain langchain-openai langchain-community faiss-cpu
```

---

## একটি সম্পূর্ণ উদাহরণ: বাংলা চ্যাটবট 💬

```python
import os
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.memory import ConversationBufferWindowMemory
from langchain.chains import ConversationChain

# API কী সেট করা
os.environ["OPENAI_API_KEY"] = "your-api-key-here"

# মডেল তৈরি
chat_model = ChatOpenAI(
    model_name="gpt-3.5-turbo",
    temperature=0.7
)

# মেমোরি (শেষ ৫টি কথোপকথন মনে রাখবে)
memory = ConversationBufferWindowMemory(k=5)

# কথোপকথন চেইন
conversation = ConversationChain(
    llm=chat_model,
    memory=memory,
    verbose=True
)

# চ্যাটবট চালানো
print("=== বাংলা চ্যাটবট ===")
print("('exit' লিখে বের হোন)")
print()

while True:
    user_input = input("আপনি: ")
    if user_input.lower() == 'exit':
        print("ধন্যবাদ! আবার আসবেন। 🙏")
        break
    
    response = conversation.predict(input=user_input)
    print(f"বট: {response}")
    print()
```

---

## RAG (Retrieval Augmented Generation) উদাহরণ 📚

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# ১. PDF ডকুমেন্ট লোড
loader = PyPDFLoader("bangla_book.pdf")
pages = loader.load()

# ২. ডকুমেন্ট ছোট ছোট অংশে ভাগ
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(pages)

# ৩. ভেক্টর ডাটাবেস তৈরি
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(splits, embeddings)

# ৪. প্রশ্ন-উত্তর চেইন তৈরি
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=vectorstore.as_retriever()
)

# ৫. প্রশ্ন করা
question = "এই বইয়ের মূল বিষয়বস্তু কী?"
answer = qa_chain.run(question)
print(f"উত্তর: {answer}")
```

---

## LCEL (LangChain Expression Language) - নতুন পদ্ধতি 🆕

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# LCEL ব্যবহার করে চেইন তৈরি (পাইপ অপারেটর |)
prompt = ChatPromptTemplate.from_template(
    "{topic} সম্পর্কে ৩টি মজার তথ্য বাংলায় বলো।"
)

model = ChatOpenAI(model="gpt-3.5-turbo")
output_parser = StrOutputParser()

# চেইন তৈরি (নতুন পদ্ধতি)
chain = prompt | model | output_parser

# চেইন চালানো
result = chain.invoke({"topic": "চাঁদ"})
print(result)
```

---

## ল্যাংচেইনের ব্যবহারিক ক্ষেত্র 🌐

```
📌 ১. চ্যাটবট তৈরি
📌 ২. প্রশ্ন-উত্তর সিস্টেম (নিজস্ব ডেটা থেকে)
📌 ৩. ডকুমেন্ট সারসংক্ষেপ
📌 ৪. কোড জেনারেশন
📌 ৫. ডেটা বিশ্লেষণ
📌 ৬. ইমেইল/কন্টেন্ট লেখা
📌 ৭. অনুবাদ সিস্টেম
📌 ৮. শিক্ষামূলক টুল
```

---

## ল্যাংচেইন ইকোসিস্টেম 🌍

```
┌─────────────────────────────────────┐
│         LangChain Ecosystem         │
├─────────────────────────────────────┤
│                                     │
│  🔗 LangChain - মূল ফ্রেমওয়ার্ক    │
│  🦜 LangServe - API ডিপ্লয়মেন্ট    │
│  🛠️ LangSmith - ডিবাগিং ও টেস্টিং  │
│  📦 LangHub - প্রম্পট শেয়ারিং      │
│  🔍 LangGraph - জটিল ওয়ার্কফ্লো    │
│                                     │
└─────────────────────────────────────┘
```

---

## সারসংক্ষেপ 📝

| বিষয় | বিবরণ |
|---|---|
| **কী** | LLM ভিত্তিক অ্যাপ তৈরির ফ্রেমওয়ার্ক |
| **ভাষা** | Python, JavaScript |
| **মূল ধারণা** | Models, Prompts, Chains, Memory, Agents |
| **সুবিধা** | মডুলার, ফ্লেক্সিবল, বিশাল ইকোসিস্টেম |
| **ব্যবহার** | চ্যাটবট, RAG, সারসংক্ষেপ, বিশ্লেষণ |

---

> **💡 টিপস:** ল্যাংচেইন শেখা শুরু করতে প্রথমে Python এর বেসিক এবং API কী কনসেপ্ট জানা দরকার। তারপর ছোট ছোট প্রজেক্ট দিয়ে শুরু করুন!
