# Dream AI — Knowledge Base System

> **Overview:** This directory contains the RAG (Retrieval-Augmented Generation) knowledge bases that power Dream AI agents. These documents are ingested by AI systems to provide accurate, consistent, and on-brand responses.

---

## Directory Structure

```
knowledge-bases/
├── README.md                    ← You are here
├── sales-kb.md                  ← Sales scripts, objections, pricing, demos
├── support-kb.md                ← FAQ, troubleshooting, refund policy, escalation
├── niches-kb.md                 ← Industry-specific data (RE, MedSpa, HVAC)
├── system-prompt-template.md    ← Master system prompt template with placeholders
└── [client-specific]/           ← Per-client knowledge bases (created during onboarding)
    └── [client-name]-kb.md
```

---

## How to Use These Knowledge Bases

### For AI Agents
These files are loaded into AI agent systems (via API or direct injection) so the agent can:
1. **Retrieve** relevant information when a customer asks a question
2. **Augment** its responses with accurate, up-to-date content
3. **Generate** on-brand answers instead of hallucinating

### For Sales Teams
- **sales-kb.md** — Study for objection handling, demo scripts, and pricing. Update quarterly.
- **niches-kb.md** — Use for prospecting calls. Industry pain points and ROI calculators close deals.

### For Support Teams
- **support-kb.md** — Reference for customer issues. Update when new issues are discovered.
- Use the troubleshooting guides for L1 support. Escalate per the escalation matrix.

### For Client Onboarding
- **system-prompt-template.md** — Copy and fill in placeholders for each new client.
- Combine with niche-specific data from **niches-kb.md**.
- Add client-specific FAQ/solutions in a custom KB file.

---

## Integration Instructions

### 1. N8N (Workflow Automation)

**Method A: Direct File Injection**
```
Workflow: Webhook → Load Knowledge Base (Read File node) → AI Agent node → Response

In the AI Agent node:
- System Prompt: Use content from system-prompt-template.md (filled in)
- Knowledge Base: Include sales-kb.md, support-kb.md, niches-kb.md as context
- Model: GPT-4o or Claude 3.5 Sonnet
```

**Method B: Vector Store (Recommended for large KBs)**
```
1. Split knowledge base files into chunks (500–1000 tokens each)
2. Embed chunks using OpenAI embeddings (text-embedding-3-small)
3. Store in N8N's vector store node (Pinecone, Weaviate, or Qdrant)
4. On each conversation:
   - Embed the user's question
   - Retrieve top 3–5 relevant chunks
   - Inject into AI Agent node as context
```

**N8N Node Configuration:**
```json
{
  "node": "AI Agent",
  "parameters": {
    "systemMessage": "{{system_prompt}}",
    "memory": "buffer",
    "tools": ["calendar_check", "crm_create_lead", "transfer_to_human"],
    "model": "gpt-4o",
    "temperature": 0.3
  }
}
```

### 2. OpenAI Assistants API

**Method A: File Search (Recommended)**
```
1. Upload knowledge base files to OpenAI:
   POST https://api.openai.com/v1/files
   - purpose: "assistants"
   - file: sales-kb.md, support-kb.md, niches-kb.md

2. Create an Assistant with file_search tool:
   POST https://api.openai.com/v1/assistants
   {
     "name": "[Client] Dream AI Agent",
     "instructions": "<filled system prompt from template>",
     "tools": [{"type": "file_search"}],
     "model": "gpt-4o",
     "tool_resources": {
       "file_search": {
         "vector_stores": [{"file_ids": ["file-xxx", "file-yyy"]}]
       }
     }
   }

3. Use in conversations:
   POST https://api.openai.com/v1/threads/{thread_id}/messages
   { "role": "user", "content": "Customer question here" }
```

**Method B: Code Interpreter (for dynamic content)**
```
Upload knowledge bases as files + use code interpreter to query them.
Better for: interactive lookups, calculations (ROI), real-time data.
```

### 3. Claude Projects (Anthropic)

```
1. Create a new Project in Claude:
   - Name: "[Client] Dream AI Agent"
   - Project Instructions: Paste the filled system prompt template

2. Upload knowledge base files as Project Knowledge:
   - sales-kb.md
   - support-kb.md  
   - niches-kb.md
   - client-specific-kb.md

3. The Claude model will automatically reference these files
   when answering questions within the project.

4. For API usage:
   POST https://api.anthropic.com/v1/messages
   {
     "model": "claude-3-5-sonnet-20241022",
     "max_tokens": 1024,
     "system": "<system prompt + KB content>",
     "messages": [{"role": "user", "content": "..."}]
   }
```

### 4. LangChain / LangGraph

```python
from langchain_community.document_loaders import TextLoader
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain.chains import RetrievalQA

# Load knowledge bases
docs = []
for kb_file in ["sales-kb.md", "support-kb.md", "niches-kb.md"]:
    loader = TextLoader(f"knowledge-bases/{kb_file}")
    docs.extend(loader.load())

# Create vector store
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = FAISS.from_documents(docs, embeddings)

# Create retrieval chain
llm = ChatOpenAI(model="gpt-4o", temperature=0.3)
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=vectorstore.as_retriever(search_kwargs={"k": 5}),
    return_source_documents=True
)

# Query
response = qa_chain.invoke({"query": "What's your refund policy?"})
```

### 5. Direct Injection (Simplest)

For smaller knowledge bases or prototypes, inject the entire KB content into the system prompt:

```
System: You are a customer service agent for [Company]. Use the following 
knowledge base to answer questions accurately. If you don't know the answer, 
transfer to a human.

[ENTIRE KNOWLEDGE BASE CONTENT HERE]

User: [Customer message]
```

**Limitation:** Works up to ~20K tokens. Beyond that, use RAG/vector approach.

---

## Update Procedures

### When to Update

| Trigger | Action | Priority |
|---------|--------|----------|
| New product/feature launched | Update sales-kb.md + support-kb.md | High |
| Pricing change | Update sales-kb.md immediately | Critical |
| New client onboarded | Create client-specific KB | High |
| New FAQ from customers | Add to support-kb.md | Medium |
| Competitive landscape shift | Update competitive comparisons in sales-kb.md | Medium |
| Industry trend / use case | Update niches-kb.md | Low |
| New integration available | Update support-kb.md setup instructions | Medium |
| Client feedback on AI behavior | Review and update system-prompt-template.md | High |

### How to Update

#### Standard Update Process
1. **Edit** the relevant KB file (use version control / git)
2. **Test** changes by running test queries through the agent
3. **Validate** responses with a human team member
4. **Deploy** the updated KB to the agent system
5. **Monitor** first 48 hours of conversations for issues
6. **Log** the update in the version history below

#### Version History Format
```markdown
## Version History
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-03-18 | 1.0.0 | Dream AI Team | Initial release |
| 2026-04-01 | 1.0.1 | [Name] | Added Q about new product X |
```

### Quality Assurance
- **Weekly:** Review 10 random conversations against KB — spot-check accuracy
- **Monthly:** Full KB review for outdated content
- **Quarterly:** Major revision with new content, competitor updates, and client feedback
- **Annually:** Complete audit and rebuild if needed

---

## File Naming Conventions

```
Pattern: [type]-kb.md
Examples:
- sales-kb.md          ← General sales knowledge
- support-kb.md        ← General support knowledge
- niches-kb.md         ← Multi-niche reference
- acme-re-kb.md        ← Client-specific: Acme Realty
- glow-medspa-kb.md    ← Client-specific: Glow MedSpa

Pattern for system prompts:
- system-prompt-template.md     ← Master template
- [client]-system-prompt.md     ← Client-specific filled version
```

---

## Best Practices

1. **Keep it structured.** Tables, bullet lists, and headers are easier for AI to parse than walls of text.
2. **Be specific.** "Response time within 4 hours" is better than "We respond quickly."
3. **Include examples.** Scripts, sample conversations, and demos give the AI patterns to follow.
4. **Version everything.** Never edit production KBs without tracking changes.
5. **Test, test, test.** Every KB update should be validated with test queries before deploying.
6. **Separate concerns.** Sales KB ≠ Support KB ≠ Niche KB. Each has a different purpose.
7. **Keep it current.** A stale knowledge base is worse than no knowledge base.
8. **Client-specific KBs** should only contain what's unique to that client. Reference shared KBs for common content.

---

## Support

For questions about the knowledge base system:
- **Slack:** #dream-ai-ops
- **Email:** ops@dreamai.com
- **Internal Docs:** [Notion workspace link]

---

*Last updated: 2026-03-18*
