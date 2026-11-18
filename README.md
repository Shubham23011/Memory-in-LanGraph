
# Memory-in-LangGraph

A structured guide to understanding **Memory Concepts in Generative AI**, focusing on Short-Term Memory (STM), Long-Term Memory (LTM), persistence, context management, and their implementation using LangGraph.

## 📚 Topics Covered

### 1. Short-Term Memory (STM)

Short-Term Memory maintains the state and context of an ongoing conversation or workflow.

**Key Concepts:**
- Checkpointer
- InMemorySaver
- Thread ID
- Graph State

**How It Works:**
- A Checkpointer saves the graph state during execution.
- InMemorySaver stores checkpoint data in application RAM.
- Thread ID identifies a conversation thread.
- Reusing the same Thread ID allows the application to continue a previous conversation.

### 2. Limitations of InMemorySaver

- Stores checkpoint data in RAM.
- Data is lost when the application process restarts.
- State is not automatically shared across multiple application instances.
- Not suitable as the only persistence mechanism for production environments.

### 3. Persistence in Production

Production-grade applications require persistent storage to retain conversation state across application restarts and failures.

**Database-backed persistence:**
- PostgreSQL
- Other supported database backends

A persistent Checkpointer helps maintain conversation state even after application restarts.

### 4. Context Window and Context Overflow

An LLM has a limited context window, which determines how much information it can process in a single request.

When the conversation grows beyond this limit, context management techniques are required.

#### A. Trimming Technique

- Set a maximum token limit.
- Remove older messages when the limit is reached.
- Retain the most recent N messages in the conversation state.

**Limitation:**
- Older messages may contain important information.
- Removing them can result in loss of context.
- This may reduce the accuracy and quality of LLM responses.

#### B. Summarisation Technique

- Generate a summary of older conversation messages.
- Retain the most recent N messages in their original form.
- Remove older raw messages from the active context.
- Pass the summary along with recent messages to the LLM.

**Benefit:**
- Preserves important historical information in a compressed format.
- Reduces token consumption.
- Helps manage long conversations efficiently.

**Limitation:**
- Summaries may omit important details or specific information.

---

## 5. Long-Term Memory (LTM)

Long-Term Memory stores useful information that can be retrieved across multiple conversations or sessions.

In LangGraph, Long-Term Memory is implemented using the **Store** abstraction.

The `BaseStore` abstract class defines the interface for memory storage and retrieval.

### Store Operations

| Operation | Method | Description |
|---|---|---|
| Create / Update | `put()` | Creates or updates a memory |
| Retrieve | `get()` | Retrieves memory using namespace and key |
| Search | `search()` | Searches for relevant memories |
| Delete | `delete()` | Deletes a stored memory |

### Store Implementations

1. **InMemoryStore**
   - Stores memories in RAM.
   - Useful for development and testing.
   - Data is not durable across process restarts.

2. **PostgresStore**
   - Stores memories in PostgreSQL.
   - Provides persistent storage.

3. **Redis-backed Store**
   - Uses Redis as the storage backend.
   - Provides fast access to stored memories, depending on configuration.

---

## 6. Namespace in Long-Term Memory

A Namespace is used to organize memories inside a Store.

It works similarly to folders in Google Drive, where documents are organized into different folders.

**Example:**

```python
namespace = ("users", "user-123", "preferences")
```

Namespaces help organize memories based on:
- Users
- Projects
- Applications
- Teams
- Memory categories

### Memory Operations

**Create / Update → `put()`**

Stores a new memory or updates an existing memory.

**Retrieve → `get()`**

Retrieves a memory using its namespace and key.

**Search → `search()`**

Finds relevant memories within a namespace.

**Delete → `delete()`**

Removes a stored memory.

---

## 7. Short-Term Memory vs Long-Term Memory

| Short-Term Memory | Long-Term Memory |
|---|---|
| Maintains active conversation state | Stores reusable information |
| Uses Graph State and Checkpointer | Uses Store / BaseStore |
| Identified using Thread ID | Organized using Namespace and Key |
| Maintains conversation continuity | Maintains memory across conversations |
| Can use InMemorySaver or persistent checkpointers | Can use InMemoryStore or persistent Store backends |

---

## 8. Production Architecture

A production-grade LangGraph application can use both Short-Term and Long-Term Memory.

- **Checkpointer:** Persists conversation state.
- **Store:** Persists reusable long-term memories.
- **Context Management:** Controls how much conversation history is sent to the LLM.
- **PostgreSQL:** Can provide persistent storage for checkpoints and long-term memories.

### Overall Workflow

```text
                 User Request
                      |
                      v
              Load Thread State
                      |
                      v
          Retrieve Relevant Memories
                 (Long-Term)
                      |
                      v
             Context Management
                      |
                      v
                 LLM Execution
                      |
                      v
               Update Graph State
                      |
                      v
                Save Checkpoint
                (Short-Term)
                      |
                      v
            Update Long-Term Store
                 (If Required)
```

---

## 🎯 Key Takeaways

- Short-Term Memory maintains the state of an ongoing conversation.
- Long-Term Memory retains useful information across conversations.
- Checkpointer is used to save and restore graph execution state.
- InMemorySaver stores checkpoint data in RAM.
- Thread ID identifies a conversation thread.
- PostgreSQL can provide persistent checkpoint storage.
- Trimming removes older messages to manage context size.
- Summarisation compresses older conversation history.
- Store / BaseStore provides the abstraction for Long-Term Memory.
- Namespace organizes memories inside a Store.
- `put()`, `get()`, `search()` and `delete()` are important Store operations.

---

**Note:** These notes cover the fundamentals of Memory Management in Generative AI using LangGraph.
