# কনভার্সেশনাল Q&A চ্যাটবট - কোড ব্যাখ্যা (বাংলায়)

---

## Cell 1: সেল পরিচিতি এবং অবলম্বন

```
### Conversation Q&A Chatbot
In many Q&A applications we want to allow the user to have a back-and-forth conversation, meaning the application needs some sort of "memory" of past questions and answers, and some logic for incorporating those into its current thinking.

In this guide we focus on adding logic for incorporating historical messages. Further details on chat history management is covered in the previous videos.

We will cover two approaches:

- Chains, in which we always execute a retrieval step;
- Agents, in which we give an LLM discretion over whether and how to execute a retrieval step (or multiple steps).
```

### ব্যাখ্যা:
এই প্রথম সেলটি একটি মার্কডাউন সেল যা কনভার্সেশনাল Q&A চ্যাটবটের উদ্দেশ্য এবং পদ্ধতি ব্যাখ্যা করে। এখানে বলা হচ্ছে যে:
- আমরা এমন একটি চ্যাটবট তৈরি করব যা ব্যবহারকারীর সাথে সংলাপ করতে পারে
- চ্যাটবটের অতীত প্রশ্ন এবং উত্তরের মেমরি থাকবে
- আমরা দুটি পদ্ধতি ব্যবহার করব: Chains এবং Agents

---

## Cell 2: Groq API কী এবং LLM মডেল সেটআপ

```python
import os
from dotenv import load_dotenv
load_dotenv()
from langchain_groq import ChatGroq

groq_api_key=os.getenv("GROQ_API_KEY")
llm=ChatGroq(groq_api_key=groq_api_key,model_name="Llama3-8b-8192")
llm
```

### ব্যাখ্যা:
- `import os`: পরিবেশ ভেরিয়েবল অ্যাক্সেস করার জন্য
- `from dotenv import load_dotenv`: `.env` ফাইল থেকে API কী লোড করতে
- `load_dotenv()`: `.env` ফাইল থেকে পরিবেশ ভেরিয়েবল লোড করে
- `from langchain_groq import ChatGroq`: ChatGroq ইমপোর্ট করে
- `os.getenv("GROQ_API_KEY")`: পরিবেশ ভেরিয়েবল থেকে Groq API কী পায়
- `ChatGroq()`: Llama3-8b মডেল ব্যবহার করে একটি LLM অবজেক্ট তৈরি করে

---

## Cell 3: BeautifulSoup লাইব্রেরি ইনস্টলেশন

```python
!pip install bs4
```

### ব্যাখ্যা:
- এই কমান্ড BeautifulSoup4 (`bs4`) লাইব্রেরি ইনস্টল করে
- BeautifulSoup ওয়েব পেজ থেকে HTML/XML ডেটা পার্স করতে ব্যবহৃত হয়
- `!` চিহ্ন Jupyter নোটবুকে সরাসরি টার্মিনাল কমান্ড চালাতে ব্যবহৃত হয়

---

## Cell 4: Hugging Face ইমবেডিংস সেটআপ

```python
os.environ['HF_TOKEN']=os.getenv("HF_TOKEN")
from langchain_huggingface import HuggingFaceEmbeddings
embeddings=HuggingFaceEmbeddings(model_name="all-MiniLM-L6-v2")
```

### ব্যাখ্যা:
- `os.environ['HF_TOKEN']`: Hugging Face টোকেন পরিবেশ ভেরিয়েবলে সেট করে
- `from langchain_huggingface import HuggingFaceEmbeddings`: Hugging Face ইমবেডিংস ক্লাস ইমপোর্ট করে
- `HuggingFaceEmbeddings()`: "all-MiniLM-L6-v2" মডেল ব্যবহার করে একটি ইমবেডিংস অবজেক্ট তৈরি করে
- এই মডেল টেক্সটকে ভেক্টরে রূপান্তরিত করে যা মেশিন লার্নিং মডেলে ব্যবহার করা যায়

---

## Cell 5: প্রয়োজনীয় লাইব্রেরি ইমপোর্ট

```python
from langchain_chroma import Chroma
from langchain_community.document_loaders import WebBaseLoader
from langchain_core.prompts import ChatPromptTemplate
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
```

### ব্যাখ্যা:
- `Chroma`: একটি ভেক্টর ডাটাবেস যা ইমবেডিংস সংরক্ষণ করে
- `WebBaseLoader`: ওয়েব থেকে ডকুমেন্ট লোড করে
- `ChatPromptTemplate`: প্রম্পট টেমপ্লেট তৈরি করে
- `RecursiveCharacterTextSplitter`: বড় টেক্সটকে ছোট খণ্ডে ভাগ করে
- `create_retrieval_chain` এবং `create_stuff_documents_chain`: রিট্রিভাল চেইন তৈরি করে

---

## Cell 6: ওয়েব থেকে ডকুমেন্ট লোডিং

```python
import bs4
loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=bs4.SoupStrainer(
            class_=("post-content", "post-title", "post-header")
        )
    ),
)

docs=loader.load()
docs
```

### ব্যাখ্যা:
- `WebBaseLoader()`: একটি ওয়েব লোডার অবজেক্ট তৈরি করে
- `web_paths`: যে ওয়েব পেজ থেকে ডেটা লোড করতে হবে তার URL
- `parse_only`: শুধুমাত্র নির্দিষ্ট HTML ক্লাস অনুযায়ী কন্টেন্ট নিবেশন করে
- `class_=("post-content", "post-title", "post-header")`: এই ক্লাসগুলির কন্টেন্ট লোড করা হয়
- `loader.load()`: ওয়েব পেজ থেকে ডকুমেন্ট লোড করে

---

## Cell 7: টেক্সট স্প্লিটিং এবং ভেক্টরস্টোর তৈরি

```python
text_splitter=RecursiveCharacterTextSplitter(chunk_size=1000,chunk_overlap=200)
splits=text_splitter.split_documents(docs)
vectorstore=Chroma.from_documents(documents=splits,embedding=embeddings)
retriever=vectorstore.as_retriever()
retriever
```

### ব্যাখ্যা:
- `RecursiveCharacterTextSplitter()`: টেক্সট স্প্লিটার অবজেক্ট তৈরি করে
- `chunk_size=1000`: প্রতিটি খণ্ড ১০০০ ক্যারেক্টার হবে
- `chunk_overlap=200`: খণ্ডগুলির মধ্যে ২০০ ক্যারেক্টার ওভারল্যাপ থাকবে (কন্টেক্সট রক্ষার জন্য)
- `split_documents()`: ডকুমেন্টগুলি ছোট খণ্ডে বিভক্ত করে
- `Chroma.from_documents()`: খণ্ডগুলি থেকে একটি ভেক্টর ডাটাবেস তৈরি করে
- `vectorstore.as_retriever()`: একটি রিট্রিভার অবজেক্ট তৈরি করে যা সার্চ করতে পারে

---

## Cell 8: প্রম্পট টেমপ্লেট তৈরি

```python
system_prompt = (
    "You are an assistant for question-answering tasks. "
    "Use the following pieces of retrieved context to answer "
    "the question. If you don't know the answer, say that you "
    "don't know. Use three sentences maximum and keep the "
    "answer concise."
    "\n\n"
    "{context}"
)

prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{input}"),
    ]
)
```

### ব্যাখ্যা:
- `system_prompt`: চ্যাটবটের জন্য সিস্টেম নির্দেশনা
- এটি বলে যে সহায়ক প্রশ্নের উত্তর দেবে এবং তিন বাক্যের মধ্যে সীমাবদ্ধ থাকবে
- `{context}`: পুনরুদ্ধারকৃত ডকুমেন্ট প্রসঙ্গ যোগ করার জন্য প্লেসহোল্ডার
- `ChatPromptTemplate.from_messages()`: একটি প্রম্পট টেমপ্লেট তৈরি করে যা সিস্টেম এবং মানুষের বার্তা রাখে

---

## Cell 9: রিট্রিভাল চেইন তৈরি

```python
question_answer_chain=create_stuff_documents_chain(llm,prompt)
rag_chain=create_retrieval_chain(retriever,question_answer_chain)
```

### ব্যাখ্যা:
- `create_stuff_documents_chain()`: প্রশ্ন-উত্তর চেইন তৈরি করে যা পুনরুদ্ধারকৃত ডকুমেন্ট LLMে পাঠায়
- `create_retrieval_chain()`: একটি সম্পূর্ণ RAG (Retrieval-Augmented Generation) চেইন তৈরি করে
- প্রথমে রিট্রিভার ডকুমেন্ট খুঁজে এবং তারপর LLM উত্তর তৈরি করে

---

## Cell 10: প্রথম প্রশ্ন এবং উত্তর পরীক্ষা

```python
response=rag_chain.invoke({"input":"What is Self-Reflection"})
response
```

### ব্যাখ্যা:
- `rag_chain.invoke()`: RAG চেইন চালায়
- `{"input":"What is Self-Reflection"}`: প্রশ্ন ইনপুট দেয়
- `response`: সম্পূর্ণ প্রতিক্রিয়া পায় যার মধ্যে উত্তর এবং পুনরুদ্ধারকৃত ডকুমেন্ট রয়েছে

---

## Cell 11: শুধুমাত্র উত্তর স্ট্রিং বের করা

```python
response['answer']
```

### ব্যাখ্যা:
- প্রতিক্রিয়া অবজেক্ট থেকে শুধুমাত্র 'answer' কী এর মান বের করে
- এটি শুধুমাত্র পাঠ্য উত্তর প্রদর্শন করে

---

## Cell 12: দ্বিতীয় প্রশ্ন পরীক্ষা

```python
rag_chain.invoke({"input":"Howw do we achieve it"})
```

### ব্যাখ্যা:
- আরেকটি প্রশ্ন জিজ্ঞাসা করা হয়: "কীভাবে আমরা এটি অর্জন করব?"
- এই প্রশ্নটি পূর্ববর্তী প্রশ্নের উপর নির্ভর করে কিন্তু চ্যাটবট এখনও ইতিহাস বুঝে না
- পরবর্তী সেলগুলিতে আমরা চ্যাট হিস্ট্রি যোগ করব

---

## Cell 13: চ্যাট হিস্ট্রি সেকশন শুরু

```
### Adding Chat History
```

### ব্যাখ্যা:
এটি একটি বিভাগ শিরোনাম যা নির্দেশ করে যে এখন আমরা চ্যাট হিস্ট্রি বৈশিষ্ট্য যোগ করতে শুরু করছি।

---

## Cell 14: চ্যাট হিস্ট্রি-সচেতন রিট্রিভার তৈরি

```python
from langchain.chains import create_history_aware_retriever
from langchain_core.prompts import MessagesPlaceholder

contextualize_q_system_prompt = (
    "Given a chat history and the latest user question "
    "which might reference context in the chat history, "
    "formulate a standalone question which can be understood "
    "without the chat history. Do NOT answer the question, "
    "just reformulate it if needed and otherwise return it as is."
)
contextualize_q_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", contextualize_q_system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
```

### ব্যাখ্যা:
- `create_history_aware_retriever`: চ্যাট হিস্ট্রি বুঝতে পারে এমন একটি রিট্রিভার তৈরি করে
- `MessagesPlaceholder`: চ্যাট হিস্ট্রির জন্য একটি স্থান নির্ধারণ করে
- `contextualize_q_system_prompt`: LLMকে বলে যে পূর্ববর্তী প্রশ্নের উপর ভিত্তি করে বর্তমান প্রশ্নকে পুনর্নির্মাণ করতে হবে
- এটি নিশ্চিত করে যে রিট্রিভার সঠিক কন্টেক্সট খুঁজে পায়

---

## Cell 15: চ্যাট হিস্ট্রি-সচেতন রিট্রিভার চালু করা

```python
history_aware_retriever=create_history_aware_retriever(llm,retriever,contextualize_q_prompt)
history_aware_retriever
```

### ব্যাখ্যা:
- `create_history_aware_retriever()`: একটি চ্যাট হিস্ট্রি-সচেতন রিট্রিভার অবজেক্ট তৈরি করে
- এই রিট্রিভার LLM, মূল রিট্রিভার এবং প্রম্পট টেমপ্লেট ব্যবহার করে
- এটি চ্যাট হিস্ট্রির প্রসঙ্গে বর্তমান প্রশ্ন বিবেচনা করে

---

## Cell 16: চ্যাট হিস্ট্রি সহ নতুন প্রম্পট টেমপ্লেট

```python
qa_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        MessagesPlaceholder("chat_history"),
        ("human", "{input}"),
    ]
)
```

### ব্যাখ্যা:
- একটি নতুন প্রম্পট টেমপ্লেট তৈরি করা হয় যা চ্যাট হিস্ট্রি অন্তর্ভুক্ত করে
- এটি সিস্টেম নির্দেশনা, চ্যাট হিস্ট্রি এবং মানুষের ইনপুট সব কিছু রাখে

---

## Cell 17: নতুন RAG চেইন তৈরি

```python
question_answer_chain=create_stuff_documents_chain(llm,qa_prompt)
rag_chain=create_retrieval_chain(history_aware_retriever,question_answer_chain)
```

### ব্যাখ্যা:
- পুরানো প্রশ্ন-উত্তর চেইন নতুন প্রম্পট টেমপ্লেটের সাথে পুনর্নির্মাণ করা হয়
- এই নতুন RAG চেইন চ্যাট হিস্ট্রি-সচেতন রিট্রিভার ব্যবহার করে
- এখন চেইন আগের কথোপকথন মনে রাখতে পারে

---

## Cell 18: চ্যাট হিস্ট্রি সহ প্রশ্ন উত্তর

```python
from langchain_core.messages import AIMessage,HumanMessage
chat_history=[]
question="What is Self-Reflection"
response1=rag_chain.invoke({"input":question,"chat_history":chat_history})

chat_history.extend(
    [
        HumanMessage(content=question),
        AIMessage(content=response1["answer"])
    ]
)

question2="Tell me more about it?"
response2=rag_chain.invoke({"input":question,"chat_history":chat_history})
print(response2['answer'])
```

### ব্যাখ্যা:
- `AIMessage` এবং `HumanMessage`: বার্তা অবজেক্ট তৈরি করতে ব্যবহৃত হয়
- `chat_history=[]`: খালি চ্যাট হিস্ট্রি শুরু করে
- প্রথম প্রশ্ন করা হয় এবং উত্তর পাওয়া যায়
- `chat_history.extend()`: প্রথম প্রশ্ন এবং উত্তর হিস্ট্রিতে যোগ করে
- দ্বিতীয় প্রশ্ন করা হয় এবং এখন চ্যাটবট প্রথম প্রশ্নের সাথে যুক্ত করতে পারে
- উত্তর প্রিন্ট করা হয়

---

## Cell 19: চ্যাট হিস্ট্রি পরীক্ষা করুন

```python
chat_history
```

### ব্যাখ্যা:
- সমস্ত চ্যাট হিস্ট্রি বার্তা প্রদর্শন করে
- এতে মানুষের প্রশ্ন এবং কৃত্রিম বুদ্ধিমত্তার উত্তর উভয়ই রয়েছে

---

## Cell 20: স্টেটফুল চ্যাট ম্যানেজমেন্ট

```python
from langchain_community.chat_message_histories import ChatMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}


def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]


conversational_rag_chain = RunnableWithMessageHistory(
    rag_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
    output_messages_key="answer",
)
```

### ব্যাখ্যা:
- `ChatMessageHistory`: চ্যাট বার্তা সংরক্ষণের জন্য একটি ক্লাস
- `store = {}`: একটি অভ্যন্তরীণ স্টোর যা বিভিন্ন সেশনের জন্য চ্যাট ইতিহাস রাখে
- `get_session_history()`: একটি সেশন ID দেওয়া হলে সেই সেশনের চ্যাট ইতিহাস ফেরত দেয়
- `RunnableWithMessageHistory`: RAG চেইনকে স্বয়ংক্রিয়ভাবে চ্যাট ইতিহাস পরিচালনা করতে সক্ষম করে
- এখন প্রতিটি সেশনের নিজস্ব ইতিহাস থাকবে

---

## Cell 21: স্টেটফুল চ্যাট এ প্রথম প্রশ্ন

```python
conversational_rag_chain.invoke(
    {"input": "What is Task Decomposition?"},
    config={
        "configurable": {"session_id": "abc123"}
    },
)["answer"]
```

### ব্যাখ্যা:
- `conversational_rag_chain.invoke()`: কথোপকথনমূলক RAG চেইন চালায়
- `session_id: "abc123"`: একটি নির্দিষ্ট সেশন ID সেট করে
- এই সেশনের জন্য চ্যাট ইতিহাস স্বয়ংক্রিয়ভাবে পরিচালিত হয়
- শুধুমাত্র উত্তর (`["answer"]`) প্রদর্শন করা হয়

---

## Cell 22: একই সেশনে দ্বিতীয় প্রশ্ন

```python
conversational_rag_chain.invoke(
    {"input": "What are common ways of doing it?"},
    config={"configurable": {"session_id": "abc123"}},
)["answer"]
```

### ব্যাখ্যা:
- একই সেশন ID ("abc123") ব্যবহার করে অন্য প্রশ্ন জিজ্ঞাসা করা হয়
- চ্যাটবট পূর্ববর্তী "Task Decomposition" প্রশ্নের প্রসঙ্গে এই প্রশ্নের উত্তর দিতে পারে
- চ্যাট ইতিহাস স্বয়ংক্রিয়ভাবে সংরক্ষিত এবং পুনরুদ্ধার করা হয়

---

## Cell 23: স্টোর কন্টেন্ট পরীক্ষা করুন

```python
store
```

### ব্যাখ্যা:
- `store` অভ্যন্তরীণ স্টোর অবজেক্ট প্রদর্শন করে
- এতে সেশন ID "abc123" এর জন্য সমস্ত চ্যাট ইতিহাস বার্তা রয়েছে
- বিভিন্ন সেশন ID এর জন্য বিভিন্ন ইতিহাস বজায় থাকে

---

## Cell 24: খালি সেল

```python

```

### ব্যাখ্যা:
এটি একটি খালি সেল যা কোনো কোড চালায় না বা কোনো কিছু প্রদর্শন করে না।

---

## সংক্ষিপ্ত সারসংক্ষেপ

এই নোটবুক নিম্নলিখিত পদক্ষেপ চলে:

1. **সেটআপ**: Groq API এবং Hugging Face ইমবেডিংস কনফিগার করা
2. **ডেটা ইনজেকশন**: ওয়েব থেকে ডকুমেন্ট লোড করা এবং ভেক্টর ডাটাবেসে সংরক্ষণ করা
3. **RAG বাস্তবায়ন**: প্রশ্ন-উত্তর চেইন তৈরি এবং পরীক্ষা করা
4. **চ্যাট হিস্ট্রি সংযোজন**: কথোপকথনের প্রেক্ষাপট মনে রাখতে সক্ষম করা
5. **সেশন ম্যানেজমেন্ট**: একাধিক ব্যবহারকারী সেশন পরিচালনা করা

শেষ ফলাফল একটি সম্পূর্ণ কার্যকরী কনভার্সেশনাল Q&A চ্যাটবট যা অতীত কথোপকথন মনে রাখতে পারে এবং বিভিন্ন সেশন সমর্থন করে।
