# LangChain External Tools

Practice notebooks for giving LangChain agents access to **external tools** (Wikipedia and Tavily search), using **Groq** as the language model.

## What's inside

- **Wikipedia tool**: a `WikipediaQueryRun` tool built on `WikipediaAPIWrapper`, configured with `top_k_results=3` and `doc_content_chars_max=2000`.
- **Tavily search tool**: a `TavilySearch` tool for live web search.
- **Agent setup**: an agent created with `create_agent`, using `ChatGroq(model="openai/gpt-oss-20b")`.
- **Inspection helpers**: code that prints which tools the agent called, what each tool returned, and the final answer.

## How it works

1. A **tool** gives the agent a name, a description, and an input schema. The model reads these to decide when to call it.
2. The **agent** receives the question and decides whether to answer from memory or call a tool.
3. If it calls a tool, the result goes back to the model, which writes the final answer.

## Setup

```bash
git clone https://github.com/razangit5-pixel/langchain-external-tools.git
cd langchain-external-tools
pip install -U langchain langchain-groq langchain-community langchain-tavily wikipedia python-dotenv
```

Create a `.env` file in the project folder:

```
GROQ_API_KEY=your_groq_key
TAVILY_API_KEY=your_tavily_key
```

Never commit `.env`. It is listed in `.gitignore`.

## Usage

Open the notebook and run the cells in order:

```python
agent = create_agent(model=ChatGroq(model="openai/gpt-oss-20b", temperature=0.1), tools=[wiki_tool])

result = agent.invoke({
    "messages": [{"role": "user", "content": "Use Wikipedia to answer: Who invented the telephone and when?"}]
})
print(result["messages"][-1].content)
```

The agent may answer easy questions from memory without calling a tool. To force tool use, ask for it in the question or add a system prompt such as `"Always use the Wikipedia tool before answering."`

## Troubleshooting

**`JSONDecodeError: Expecting value: line 1 column 1 (char 0)` from the Wikipedia tool**

Wikipedia rejects the `wikipedia` library's default User-Agent and returns a non-JSON response. Set a custom one before creating the tool:

```python
import wikipedia
wikipedia.wikipedia.USER_AGENT = "your-project-name/1.0"
```

**`AttributeError: 'WikipediaAPIWrapper' object has no attribute 'name'`**

`WikipediaAPIWrapper` is only the low-level client. Wrap it in `WikipediaQueryRun(api_wrapper=...)` to get a tool the agent can use.

## Tech stack

- Python
- LangChain (`langchain`, `langchain-community`, `langchain-groq`, `langchain-tavily`)
- Groq (`openai/gpt-oss-20b`)
- Wikipedia and Tavily APIs
