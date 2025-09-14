### Building AI Applications with Large Language Models

In this video, Roy Derks explores how to build AI applications using **Large Language Models (LLMs)**. He breaks down the process into three key patterns: **basic prompting**, **Retrieval Augmented Generation (RAG)**, and **AI agents**. These methods enable developers to create applications that can understand and respond to complex questions, driving business value.

---

#### Key Patterns for Building AI Applications

1. **Basic Prompting**
   - Starts with a user interface where questions are asked.
   - The question is embedded in a **prompt**, which includes instructions for the LLM (e.g., "be a helpful assistant").
   - The prompt is sent to the LLM via an **API** or **SDK** provided by the LLM provider.
   - The LLM retrieves and returns the final answer.

2. **Retrieval Augmented Generation (RAG)**
   - Begins with a question, which is converted into an **embedding**.
   - The embedding is used by a **vector database** to find relevant context (top N matches).
   - The context and question are combined into a prompt, which is sent to the LLM.
   - The LLM generates an answer based on the enriched prompt.
   - **Data upload** to the vector database is crucial for RAG to function effectively.

3. **AI Agents**
   - Involves an **agent** that acts as an intermediary between the question and the answer.
   - The agent plans, acts, and reflects based on the question and available tools (e.g., APIs, databases, or web-crawling code).
   - Tools are used by the LLM to execute tasks and provide the final answer.
   - **Multi-agent frameworks** can also be implemented, where a supervisor agent determines which agent should handle the question.

---

#### Why Use These Patterns?
- **Basic Prompting**: Simplest approach, ideal for straightforward tasks.
- **RAG**: Enhances LLM responses by making them context-aware using vector databases.
- **AI Agents**: Enables complex problem-solving by leveraging multiple tools and frameworks.
