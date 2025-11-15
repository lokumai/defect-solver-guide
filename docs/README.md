# Defect Solver Documentation

Welcome to the Defect Solver documentation. This system provides automated bug localization for large-scale microservice architectures using LLM-powered natural language reasoning.

## 📚 Documentation Index

### For End Users

**[User Guide](user_guide.md)** - Start here if you want to use Defect Solver through the MCP server  
Learn how to:
- Install and configure the MCP client in your IDE
- Use bug localization tools via AI assistants (Copilot, etc.)
- Write effective bug descriptions
- Interpret and act on localization results

*Perfect for: Developers using Defect Solver to find bugs*

---

### For Maintainers & Developers

**[Developer Guide](dev_guide.md)** - Essential for system operators and maintainers  
Learn how to:
- Set up and configure the Codebase Summarizer
- Run manual summarization pipelines
- Deploy automated summarization schedules
- Maintain the Bug Localizer API and MCP Server
- Monitor system health and troubleshoot issues

*Perfect for: DevOps, system administrators, and project maintainers*

---

### For Researchers & Technical Deep Dive

**[Algorithm Details](algorithm.md)** - Understand how Defect Solver works internally  
Learn about:
- Core methodology (NL-to-NL reasoning)
- Two-phase pipeline (Knowledge Base + Localization)
- Hierarchical summarization strategy
- Search space routing mechanism
- Evaluation results and comparisons
- System architecture and design decisions

*Perfect for: Researchers, algorithm developers, and anyone curious about the internals*

---

## Quick Start by Role

```
┌─────────────────────────────────────────────────────────────┐
│                    Choose Your Path                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🧑‍💻 I'm a developer using the tool                          │
│     → [User Guide](user_guide.md)                           │
│                                                             │
│  🛠️  I maintain/deploy the system                           │
│     → [Developer Guide](dev_guide.md)                       │
│                                                             │
│  🔬 I want to understand the algorithm                      │
│     → [Algorithm Details](algorithm.md)                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## System Overview

Defect Solver is a bug localization system designed for large microservice architectures. It uses hierarchical natural language summaries of codebases to perform intelligent bug localization across multiple repositories.

### Architecture

```
┌───────────────────────────────────────────────────────────┐
│                    Defect Solver System                   │
├───────────────────────────────────────────────────────────┤
│                                                           │
│   ┌─────────────────┐      ┌─────────────────┐            │
│   │   Codebase      │      │      Bug        │            │
│   │   Summarizer    │─────▶│   Localizer     │            │
│   │   (Offline)     │      │     API         │            │
│   └─────────────────┘      └────────┬────────┘            │
│           │                         │                     │
│           │                         │                     │
│           ▼                         ▼                     │
│   ┌─────────────────┐      ┌─────────────────┐            │
│   │  Hugging Face   │      │   MCP Server    │            │
│   │    Storage      │      │   (Gateway)     │            │
│   └─────────────────┘      └────────┬────────┘            │
│                                     │                     │
│                                     ▼                     │
│                             ┌─────────────────┐           │
│                             │   AI Agents     │           │
│                             │ (Copilot, etc.) │           │
│                             └─────────────────┘           │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Key Components

1. **Codebase Summarizer**  
   Generates hierarchical natural language summaries of microservices (offline)

2. **Bug Localizer API**  
   Performs two-phase bug localization:
   - Phase 1: Routes bug to likely microservices
   - Phase 2: Ranks files within selected microservices

3. **MCP Server**  
   Exposes bug localization tools to AI coding assistants via Model Context Protocol

---

## Key Features

✅ **Multi-Repository Support** - Handles 46+ microservices with 1.1M+ lines of code  
✅ **Natural Language Reasoning** - Matches bug descriptions to NL summaries (not raw code)  
✅ **Hierarchical Search** - Repository → Directory → File (transparent reasoning path)  
✅ **Automated Maintenance** - Scheduled re-summarization keeps knowledge base current  
✅ **IDE Integration** - Seamless use through GitHub Copilot and other AI assistants  
✅ **High Accuracy** - 82% Pass@10 (2.4x better than Copilot/Cursor)

---

## Documentation Structure

```
docs/
├── README.md (you are here)
│   └── Index and navigation guide
│
├── user_guide.md
│   ├── Installation & setup
│   ├── Tool usage (search_space_routing, single_module_bug_localization)
│   ├── MCP prompts (coming soon)
│   ├── Workflow examples
│   └── Troubleshooting
│
├── dev_guide.md
│   ├── Codebase Summarizer setup
│   ├── Manual summarization pipeline
│   ├── Automated scheduler deployment
│   ├── Bug Localizer API maintenance
│   ├── MCP Server configuration
│   ├── Authentication overview
│   └── Common maintenance workflows
│
└── algorithm.md
    ├── Core methodology (NL-to-NL reasoning)
    ├── Two-phase pipeline
    ├── Hierarchical summarization
    ├── Search space routing
    ├── Bug localization strategies
    ├── Evaluation results
    └── Limitations & future work
```

---

## Getting Started

### As a User (Developer)
1. Read the [User Guide](user_guide.md)
2. Get your API key from [Lokum AI](https://github.com/lokumai)
3. Configure your IDE's MCP client
4. Start localizing bugs with AI assistants

### As a Maintainer
1. Read the [Developer Guide](dev_guide.md)
2. Set up the Codebase Summarizer
3. Run the manual summarization pipeline
4. Deploy the automated scheduler
5. Configure and deploy the API and MCP Server

### As a Researcher
1. Read the [Algorithm Details](algorithm.md)
2. Review the research paper (`paper.tex`)
3. Explore the codebase repositories
4. Consider extensions and improvements

---

## Related Resources

### Repositories
- **Codebase Summarizer:** [GitHub](https://github.com/lokumai/defect-solver-codebase-summarizer)
- **Bug Localizer API:** [GitHub](https://github.com/lokumai/defect_solver_api)
- **MCP Server:** [GitHub](https://github.com/lokumai/dnext_coder_mcp_server)

### Research
- **Paper:** See `paper.tex` in the guide repository
- **Evaluation Dataset:** DNext microservice architecture (46 repos, 1.1M LOC)

### Support
- **Email:** support@pia-team.com
- **GitHub:** [Lokum AI](https://github.com/lokumai)

---

## Frequently Asked Questions

**Q: Do I need access to the full codebase to use Defect Solver?**  
A: No. The system works with pre-generated summaries. You only need an API key.

**Q: How often are summaries updated?**  
A: Configurable (default: every 90 days). Maintainers can trigger manual updates anytime.

**Q: Can I use this for non-Java codebases?**  
A: Yes. The system supports multiple languages (Java, Python, JavaScript). Configure in `configurations.yaml`.

**Q: What if the tool doesn't find my bug?**  
A: Refine your bug description with more technical details, or try the next ranked files. See [User Guide](user_guide.md) for best practices.

**Q: How do I add a new microservice?**  
A: Follow the maintenance workflow in [Developer Guide](dev_guide.md) → "Adding a New Microservice".

---

## Next Steps

Pick your starting point from the guide above and dive in! Each guide is designed to be self-contained and focused on your specific needs.

Happy bug hunting! 🐛🔍
