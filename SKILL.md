---
name: graphify
description: any input (code, docs, papers, images) → knowledge graph → clustered communities → HTML + JSON + audit report
trigger: /graphify
homepage: https://github.com/gregb100/graphify
---

# /graphify

Turn any folder of files into a navigable knowledge graph with community detection, an honest audit trail, and three outputs: interactive HTML, GraphRAG-ready JSON, and a plain-language GRAPH_REPORT.md.

## Installation

```bash
# Install upstream package (required for Python modules)
pip install graphifyy

# Install Otto's custom wrappers (recommended)
git clone https://github.com/gregb100/graphify.git
cd graphify
pip install -e .
# Or just copy the bin scripts:
cp bin/graphify-full ~/.local/bin/
cp bin/graphify-semantic ~/.local/bin/
chmod +x ~/.local/bin/graphify-*
```

**Fork:** https://github.com/gregb100/graphify (Otto's customized version)

## Usage

```bash
graphify-full <path>              # Full pipeline: detect → extract → build → cluster → report
graphify-full <path> --no-viz     # Skip HTML visualization
graphify-full <path> --merged     # Rebuild with existing semantic extraction

graphify-semantic <path>          # Extract semantic edges from docs (auto-detects size)
graphify-semantic <path> --force-ollama     # Force Ollama (small corpuses)
graphify-semantic <path> --force-openrouter # Force OpenRouter (all sizes)

graphify query "<question>"        # Query the graph (BFS traversal)
graphify query "<question>" --dfs  # DFS traversal
```

### Tiered Extraction Strategy

| Corpus Size | Provider | Model | Cost |
|------------|----------|-------|------|
| Small (<5K words, <10 docs) | Ollama | gemma2:2b | Free |
| Medium (5K-50K words) | OpenRouter | gemini-2.5-flash-lite | $0.10/1M in |
| Large (>50K words) | OpenRouter | deepseek-v3.2 | $0.26/1M in |

## What graphify is for

graphify is built around Andrej Karpathy's /raw folder workflow: drop anything into a folder - papers, tweets, screenshots, code, notes - and get a structured knowledge graph that shows you what you didn't know was connected.

Three things it does that Claude alone cannot:
1. **Persistent graph** - relationships are stored in `graphify-out/graph.json` and survive across sessions
2. **Honest audit trail** - every edge is tagged EXTRACTED, INFERRED, or AMBIGUOUS
3. **Cross-document surprise** - community detection finds hidden connections

## What You Must Do When Invoked

Use `graphify-full` for the complete pipeline:

```bash
graphify-full <path>              # Run full pipeline (AST only for code)
graphify-semantic <path>         # Extract semantic edges from docs (LLM-powered)
graphify-full <path> --merged    # Rebuild graph including semantic extraction
```

**After running**, read the key sections from `graphify-out/GRAPH_REPORT.md`:
- God Nodes (most connected concepts)
- Surprising Connections (unexpected links)
- Suggested Questions

Then offer to explore. Pick the most interesting suggested question and ask if the user wants tracing.

### Semantic Extraction Details

The `graphify-full` pipeline extracts AST edges from code files. For semantic extraction from docs/papers:

1. Run `graphify-semantic <path>` - auto-selects provider based on corpus size
2. Then re-run `graphify-full <path> --merged` to rebuild graph with semantic data

**Ollama setup** (for small corpuses, free extraction):
- Ollama server must be running (default: http://192.168.148.129:11434)
- Model: gemma2:2b

**OpenRouter setup** (for medium/large corpuses):
- API key required (stored in OpenClaw config)
- Models: gemini-2.5-flash-lite (fast/cheap) or deepseek-v3.2 (better for complex)

---
