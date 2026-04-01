## 2. Working With Prompt Template and Message Chat History Using LangChain

এটার মানে হলো **LangChain ব্যবহার করে এমন chat app বানানো**, যেখানে:

1. **Prompt Template** দিয়ে তুমি model-কে structured instruction দাও
2. **Message Chat History** দিয়ে আগের conversation context ধরে রাখো

LangChain-এ **messages** হলো chat model-এর fundamental context unit। প্রতিটি message-এ role, content, আর optional metadata থাকতে পারে। সাধারণ role হলো `system`, `user`, `assistant`। ([LangChain Docs][1])

---

## Prompt Template কী?

**Prompt Template** হলো reusable prompt blueprint।

একই prompt বারবার hardcode না করে variable use করা হয়। যেমন:

```python
"Explain {topic} in simple terms."
```

এখানে `{topic}` dynamic value।

Chat-style prompt-এ সাধারণত multiple message role থাকে, যেমন:

* **System** → rules / behavior
* **Human** → user input
* **AI** → example response
* **Messages List / Placeholder** → conversation history inject করার জায়গা ([LangChain Docs][2])

### কেন দরকার?

কারণ raw prompt string দিয়ে কাজ চালানো যায়, কিন্তু সেটা messy হয়ে যায়। Template use করলে:

* prompt reusable হয়
* input variables clean থাকে
* history inject করা সহজ হয়
* chain maintain করা সহজ হয়

---

## Message Chat History কী?

**Message history** মানে আগের chat turns save রাখা, যাতে bot follow-up question বুঝতে পারে।

উদাহরণ:

* User: আমার নাম আশাব।
* User: আমার নাম কী?

History না থাকলে bot জানবে না।
History থাকলে আগের `HumanMessage` আর `AIMessage` দেখে উত্তর দিতে পারবে।

LangChain docs অনুযায়ী models-এ **list of messages** পাঠানো যায় conversation represent করার জন্য। এটাই multi-turn chat-এর ভিত্তি। ([LangChain Docs][3])

---

## Prompt Template + History একসাথে কীভাবে কাজ করে

Flowটা সাধারণত এমন:

```text
System instruction
+ Previous chat history
+ Current user message
→ Model
→ Response
```

LangChain-এ এই pattern-এ `MessagesPlaceholder` ব্যবহার করা হয়, যাতে prompt-এর ভিতরে history বসানো যায়। docs-এ message lists এবং history-aware prompting-এর ধারণা এভাবেই বর্ণনা করা হয়েছে। ([LangChain Docs][1])

---

## Basic mental model

### Without history

শুধু current question যায়

```text
User input → Prompt → Model → Answer
```

### With history

আগের messages-সহ current question যায়

```text
Chat history + current input → Prompt → Model → Better contextual answer
```

---

## LangChain-এ common building blocks

### 1. `ChatPromptTemplate`

Chat-style prompt বানানোর জন্য

### 2. `MessagesPlaceholder`

Prompt-এর ভিতরে history বসানোর জন্য

### 3. Chat model

যেমন `ChatOpenAI`, `ChatOllama` ইত্যাদি

### 4. `RunnableWithMessageHistory`

একটা runnable/chain-এর সাথে per-session history manage করার wrapper হিসেবে ব্যবহৃত হয়। ([LangChain][4])

---

## Conceptual example

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}")
])
```

এখানে:

* `system` message bot-এর behavior define করছে
* `MessagesPlaceholder` আগের chat history ঢোকাবে
* `human` current input নেবে

এটাই prompt template + message history-এর core pattern। ([LangChain Docs][1])

---

## এটা কেন important

যদি তুমি শুধু one-shot prompt use করো, তাহলে তুমি chatbot না, **single-turn responder** বানাচ্ছ।

Real chatbot-এর minimum requirement:

* previous turns remember করা
* follow-up বোঝা
* session-wise conversation রাখা
* stale context control করা

LangChain memory overview-এও short-term memory বা thread-scoped history-কে ongoing conversation track করার জন্য central বলে দেখানো হয়েছে, আর long conversations context window ও quality দুটোতেই সমস্যা তৈরি করতে পারে। ([LangChain Docs][5])

---

## Brutal truth

অনেকে “chatbot” বলে যা বানায়, সেটা আসলে:

```text
textbox → model → textbox
```

এটা chatbot না।
এটা contextless prompt wrapper।

যতক্ষণ না তুমি:

* prompt structure ঠিক করছ
* history inject করছ
* session আলাদা রাখছ

ততক্ষণ তুমি toy demo stage-এ আছ।

---

## One-line summary

**Working with Prompt Template and Message Chat History in LangChain** মানে হলো structured chat prompts বানানো এবং আগের conversation messages inject করে context-aware chatbot তৈরি করা। ([LangChain Docs][1])


[1]: https://docs.langchain.com/oss/python/langchain/messages?utm_source=chatgpt.com "Messages - Docs by LangChain"
[2]: https://docs.langchain.com/langsmith/create-a-prompt?utm_source=chatgpt.com "Create a prompt - Docs by LangChain"
[3]: https://docs.langchain.com/oss/python/langchain/models?utm_source=chatgpt.com "Models - Docs by LangChain"
[4]: https://api.python.langchain.com/en/latest/core/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html?utm_source=chatgpt.com "RunnableWithMessageHistory"
[5]: https://docs.langchain.com/oss/python/concepts/memory?utm_source=chatgpt.com "Memory overview - Docs by LangChain"
