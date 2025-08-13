# Setting Up the defect_solver_mcp_server Locally
This guide explains how to clone and run the defect_solver_mcp_server project locally using uv and configure the environment to connect with the hosted DS API on Hugging Face.

---
## Prerequisites
* Make sure you have the following installed:

  - Git

  - uv (Python package manager)

  - Python 3.10+ installed and recognized by uv
---
## 1. Clone the repository:
   ```sh
   git clone https://github.com/lokumai/defect_solver_mcp_server.git
   cd defect-solver-mcp-server
   ```
Note: If you experience any issues accessing the repository, please contact the Lokum AI team for assistance.
## 2. Install dependencies using uv:
   ```sh
   uv sync
   ```
   This installs all required dependencies listed in pyproject.toml.


## 3. Configure environment variables:
   - Copy the example environment file:
       ```sh
       cp .env.example .env
       ```

   - Open the .env file and set the following values:

       ```sh
        DS_API_BASE_URL=https://dnext-coder-api.pia-team.com
        DS_API_MULTIMODULE_ENDPOINT=/mcp_multi_module_bug_localization
        DS_API_SINGLEMODULE_ENDPOINT=/mcp_single_module_bug_localization
        DS_API_SEARCHSPACE_ENDPOINT=/mcp_search_space_routing
       ```
## Run the Server
 - Start the MCP server locally with:
   ```sh
   uv run main.py
   ```


### Note: 
Once running, your MCP server will be available at:
http://127.0.0.1:8000/mcp/

You can now use this server to interact with the defect solver API as described in the [Defect Solver Usage Guide](../how_to_use/usage_guide.md).

## MCP Inspector
The MCP Inspector is a tool to help you visualize and debug the interactions with the MCP server.
To use the MCP Inspector, simply run the command below in your terminal:

```sh
npx @modelcontextprotocol/inspector
```

The GUI will open in your default web browser, allowing you to monitor the communication with the MCP server in real-time:

<img src="https://raw.githubusercontent.com/modelcontextprotocol/inspector/main/mcp-inspector.png" alt="MCP Inspector">