## LangSmith দিয়ে GenAI app tracking — পরিচিতি

**LangSmith** হলো **observability / tracing platform** for LLM apps। এর কাজ model call, prompt, response, latency, errors, tool calls, chain/agent steps—সব trace করে দেখানো, যাতে তুমি বুঝতে পারো app আসলে কী করছে। ([LangChain Docs][1])

### কেন এটা দরকার

তুমি যদি GenAI app build করো কিন্তু trace না রাখো, তাহলে তুমি অন্ধ হয়ে debug করছ। সাধারণ failure points:

* prompt wrong
* retriever irrelevant context আনছে
* tool call fail করছে
* latency বেশি
* hallucination কোথা থেকে আসছে বোঝা যাচ্ছে না

LangSmith tracing exactly এই visibility দেয়। এটা OpenAI apps, LangChain/LangGraph apps, custom instrumentation, এমনকি OpenTelemetry-based tracing-ও support করে। ([LangChain Docs][1])

---

## LangSmith কী track করে

LangSmith traces-এ সাধারণত তুমি দেখতে পারো:

* app run hierarchy
* individual LLM calls
* prompts and outputs
* chain / graph / agent steps
* child runs
* metadata, tags, project grouping

Observability quickstart অনুযায়ী wrapped model calls automatically traced child runs হিসেবে log হয়। ([LangChain Docs][2])

---

## Basic setup

LangSmith tracing enable করতে minimum এই env vars লাগে:

* `LANGSMITH_TRACING=true`
* `LANGSMITH_API_KEY=<your key>`

আর যদি তোমার API key multiple workspace-এর সাথে linked থাকে, তাহলে `LANGSMITH_WORKSPACE_ID`-ও set করতে হবে। Default project name হয় `default`, চাইলে অন্য project-এও log করতে পারো। ([LangChain Docs][3])

---

## সবচেয়ে simple tracking flow

### 1. Install

```bash
pip install -U langsmith openai
```

### 2. Set env vars

```python
import os

os.environ["LANGSMITH_TRACING"] = "true"
os.environ["LANGSMITH_API_KEY"] = "your_langsmith_api_key"
os.environ["OPENAI_API_KEY"] = "your_openai_api_key"
```

### 3. Wrap your client

```python
from openai import OpenAI
from langsmith.wrappers import wrap_openai

client = wrap_openai(OpenAI())
```

### 4. Make a normal model call

```python
response = client.chat.completions.create(
    model="gpt-4.1-mini",
    messages=[
        {"role": "user", "content": "Explain retrievers in simple words."}
    ]
)

print(response.choices[0].message.content)
```

LangSmith docs অনুযায়ী `wrap_openai` ব্যবহার করলে subsequent OpenAI calls automatically traced হয়, কিন্তু trace log হওয়ার জন্য `LANGSMITH_TRACING` অবশ্যই `true` হতে হবে। ([LangChain Docs][2])

---

## যদি তুমি LangChain use করো

LangChain / LangGraph use করলে tracing আরও natural হয়। LangSmith LangChain-specific integrations support করে, আর LangGraph-এর সাথেও smooth nesting trace works when underlying calls are instrumented properly. ([LangChain Docs][1])

---

## Custom functions-ও trace করতে পারো

যদি তোমার app-এ শুধু raw model call না থেকে custom preprocessing, retrieval logic, ranking, post-processing থাকে, LangSmith custom instrumentation allow করে—মানে তুমি নিজের function-level tracing add করতে পারো। এটা useful যখন তুমি জানতে চাও failure model-এ, না তোমার own logic-এ। ([LangChain Docs][4])

---

## Sensitive data নিয়ে সাবধান

সব input/output blindly trace করা বুদ্ধিমানের কাজ না। LangSmith docs নিজেই sensitive data masking / preventing logging-এর জন্য dedicated guidance দিয়েছে। Production app-এ এটা ignore করা amateur mistake। ([LangChain Docs][5])

---

## বাস্তব কথা

যদি তুমি GenAI app বানাও কিন্তু LangSmith বা equivalent observability না রাখো, তাহলে তুমি builder না—guessing machine।
কারণ তুমি output দেখছ, কিন্তু system behavior দেখছ না।

---

## এক লাইনে summary

**LangSmith = your GenAI app-এর black box recorder + debugger.**
এটা ছাড়া serious app iterate করা ধীর, messy, এবং mostly self-deception. ([LangChain Docs][1])


[1]: https://docs.langchain.com/langsmith/observability?utm_source=chatgpt.com "LangSmith Observability - Docs by LangChain"
[2]: https://docs.langchain.com/langsmith/observability-quickstart?utm_source=chatgpt.com "Tracing quickstart - Docs by LangChain"
[3]: https://docs.langchain.com/langsmith/trace-openai?utm_source=chatgpt.com "Trace OpenAI applications - Docs by LangChain"
[4]: https://docs.langchain.com/langsmith/annotate-code?utm_source=chatgpt.com "Custom instrumentation - Docs by LangChain"
[5]: https://docs.langchain.com/langsmith/mask-inputs-outputs?utm_source=chatgpt.com "Prevent logging of sensitive data in traces"
