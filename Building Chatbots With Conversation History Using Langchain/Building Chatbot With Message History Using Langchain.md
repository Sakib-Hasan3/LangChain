## Building Chatbot With Message History Using LangChain

এটা basically এমন chatbot বানানো, যেটা আগের কথোপকথন **মনে রাখে**। LangChain-এ chat models messages-এর sequence নিয়ে কাজ করে, আর multi-turn conversation-এর জন্য messages-ই core unit of context। ([LangChain Docs][1])

### Message history কেন দরকার

Message history না থাকলে chatbot-এর অবস্থা হয় এইরকম:

* তুমি প্রথমে নিজের নাম বললে
* পরে জিজ্ঞেস করলে: “আমার নাম কী?”
* bot blank হয়ে যাবে

কারণ তার কাছে আগের turn নেই।

Message history থাকলে chatbot আগের user messages, AI messages, system instructions—সব মিলে conversation context ধরে রাখতে পারে। LangChain docs অনুযায়ী models সাধারণত messages-এর list input হিসেবে নেয়, আর message prompts multi-turn conversation manage করার জন্যই useful। ([LangChain Docs][1])

---

## LangChain-এ message বলতে কী

LangChain messages-এর কিছু common type আছে:

* `SystemMessage`
* `HumanMessage`
* `AIMessage`
* `ToolMessage`

এগুলো conversation state represent করে। প্রতিটি message-এ role, content, আর optional metadata থাকতে পারে। ([LangChain Docs][1])

---

## Chatbot with history-এর basic flow

```text
User message
   ↓
Previous conversation history add হয়
   ↓
Model full message list পায়
   ↓
AI response generate করে
   ↓
New response history-তে save হয়
```

এখানে মূল idea হলো: model-কে শুধু current question না দিয়ে **conversation so far** দেওয়া হয়। LangChain docs এটাকে growing list of messages pattern হিসেবে describe করে। ([LangChain Docs][1])

---

## LangChain-এ এটা কীভাবে build হয়

Modern LangChain-এ এই কাজের জন্য commonly used abstraction হলো **`RunnableWithMessageHistory`**। এটা এমন একটি wrapper যা runnable-এর সঙ্গে conversation history attach করে, যাতে per-session chat memory manage করা যায়। ([LangChain API][2])

সোজা ভাষায়:

* তোমার একটা base chain থাকবে
* তার ওপর message history wrapper বসবে
* session ID অনুযায়ী history store হবে

---

## Mental model

### Base chatbot

শুধু current input দেখে উত্তর দেয়

### History-enabled chatbot

current input + old conversation দেখে উত্তর দেয়

এটাই difference.
আর এই difference ছোট না — পুরো UX change করে দেয়।

---

## কোথায় history useful

Message history সবচেয়ে useful হয় যখন chatbot:

* follow-up question handle করে
* user preferences মনে রাখে
* short-term conversational context ধরে রাখে
* একই topic-এর ওপর ধারাবাহিক discussion চালায়

---

## Brutal truth

অনেকে ভাবে chatbot বানানো মানে:

> input নাও, model call করো, output দেখাও

এটা toy demo. বাস্তব chatbot না।

কারণ বাস্তব chatbot-এ লাগে:

* message history
* context trimming
* session separation
* prompt discipline
* memory overflow handling

শুধু one-shot prompt দিয়ে তুমি chatbot বানাওনি। তুমি একটা glorified textbox বানিয়েছ।

---

## Minimal conceptual example

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])

llm = ChatOpenAI(model="gpt-4.1-mini")

chain = prompt | llm
```

এখানে `MessagesPlaceholder` history বসানোর জায়গা রাখে। তারপর wrapper history inject করে। Messages-based prompting LangChain-এর standard pattern-এর সাথেই এটা consistent। ([LangChain Docs][1])

---

## One-line summary

**Building a chatbot with message history using LangChain** মানে হলো এমন একটি conversation system বানানো, যেখানে bot আগের messages ধরে রেখে multi-turn context-aware response দিতে পারে। এটা LangChain-এর messages model এবং history-aware runnable pattern-এর ওপর দাঁড়িয়ে থাকে। ([LangChain Docs][1])


[1]: https://docs.langchain.com/oss/python/langchain/messages?utm_source=chatgpt.com "Messages - Docs by LangChain"
[2]: https://api.python.langchain.com/en/latest/core/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html?utm_source=chatgpt.com "RunnableWithMessageHistory"
