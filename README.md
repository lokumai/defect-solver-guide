# Defect Solver 🪲

An AI-powered system for rapid bug localization in complex microservice architectures. This system uses a hierarchical understanding of codebases to intelligently route bug reports to the correct microservice and find the specific files causing the defect.

## 📖 Table of Contents
- [Defect Solver 🪲](#defect-solver-)
  - [📖 Table of Contents](#-table-of-contents)
  - [How to Use Defect Solver?](#how-to-use-defect-solver)
  - [🚀 Quick Start](#-quick-start)
  - [🏛️ Defect Solver Pipeline (Architecture)](#️-defect-solver-pipeline-architecture)
  - [📄 References](#-references)
  

---

> **⚠️ Disclaimer:** Some links in this guide point to private company repositories and are only accessible to authorized team members. External users may encounter restricted access or broken links. Please contact us for public resources or further information.

## How to Use Defect Solver?

We offer a simple API to interact with the Defect Solver system. You can write your bug description in your natural language and the system will return a ranked list of `microservices` or `files` that are likely to contain the bug.

For example:
> User -> "Why is the user profile not loading?"

> Defect Solver -> "The bug is likely in the `user-service` microservice, specifically in the `UserProfileController.java` file."


For now, the API is **ONLY** exposed via our MCP Server. This allows direct and easy integration with any AI development environment that supports [Model-Context Protocol](https://modelcontextprotocol.io/introduction). This includes popular IDEs like **VSCode**, **IntelliJ**, **Claude Desktop** or any MCP-compatible **Agent/LLM**.

For using our API through the MCP Server, you can either:
- Run the server locally 
  
or 

- Use our hosted version.

## 🚀 Quick Start
1. First deploy the MCP Server. You can either run it locally or use our hosted version.
   - For local deployment, follow the [MCP Server Local Deployment Guide](./docs/deployment/mcp_server_local_deployment.md).
   - For hosted version, you can use our public MCP Server at `https://mcp.defect-solver.com` (Requires API Key).

## 🏛️ Defect Solver Pipeline (Architecture)

The pipeline follows a two-phase pipeline to narrow down the search space and localize the bugs.

```mermaid
graph TD
    subgraph "Phase 1: Select Search Space"
        A[Bug Description] --> B{Search Space Router}
        C[Microservice Knowledge] --> B
        B -- Identifies Top-N Suspicious Microservices --> D["Selected Search Space"]
    end

    subgraph "Phase 2: Bug Localization"
        D --> E{BugLocalizer}
        E -- Strategy Selection --> G{Strategy Execution}
        G --> H1[Filtering Strategy]
        G --> H2[Direct Strategy]
        H1 --> K[Ranked List of Files]
        H2 --> K
        K --> L((Final Result))
    end
```
> For a complete breakdown of the algorithmic logic, see the **[System Architecture Document](./docs/algorithm_details/algorithm_details.md)**.

## 📄 References

This project is inspired by the principles of hierarchical code understanding to overcome the limitations of standard LLM context windows in large-scale software projects.
For more details on the underlying research, see our published paper:
[Repository-Level Code Understanding by LLMs via Hierarchical Summarization: Improving Code Search and Bug Localization](https://www.researchgate.net/publication/391739021_Repository-Level_Code_Understanding_by_LLMs_via_Hierarchical_Summarization_Improving_Code_Search_and_Bug_Localization)

