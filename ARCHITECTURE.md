# Agent flow

How a turn moves through `coding_agent.py`, from user input to final answer.

```mermaid
flowchart LR
    USER(["User input<br/>input()"]) --> APPEND["append to conversation<br/>role: user"]
    APPEND --> LLM["execute_llm_call<br/>models.generate_content"]
    LLM --> PARSE{"extract_tool_invocations<br/>any 'tool:' lines?"}

    PARSE -->|"no"| REPLY["print Assistant<br/>append role: model"]
    REPLY --> USER

    PARSE -->|"yes"| DISPATCH["look up TOOL_REGISTRY"]
    DISPATCH --> TOOLS["read_file<br/>list_files<br/>edit_file"]
    TOOLS --> RESULT["append tool_result<br/>role: user"]
    RESULT -.->|"agent loop"| LLM

    SYS["get_full_system_prompt<br/>built from TOOL_REGISTRY"] -.->|"system_instruction"| LLM
    KEY[".env GEMINI_API_KEY<br/>genai.Client"] -.-> LLM

    classDef io fill:#e8f0fe,stroke:#4285f4,color:#202124
    classDef model fill:#fef7e0,stroke:#f9ab00,color:#202124
    classDef tool fill:#e6f4ea,stroke:#34a853,color:#202124
    classDef cfg fill:#f1f3f4,stroke:#9aa0a6,color:#202124

    class USER,APPEND,REPLY io
    class LLM,PARSE model
    class DISPATCH,TOOLS,RESULT tool
    class SYS,KEY cfg
```

**Outer loop** — one pass per user message: read input, append it, run the inner
loop until the model answers without calling a tool.

**Inner loop** — the dashed `tool_result` edge back into `execute_llm_call`. The
model keeps calling tools, each result fed back as a `role: user` turn, until a
response contains no `tool:` line. That loop has no iteration cap.

**Not shown, because the code does not do it:** the model's own tool-call turn is
never appended to `conversation` before the tool result is. The model therefore
does not see the request it just made.
