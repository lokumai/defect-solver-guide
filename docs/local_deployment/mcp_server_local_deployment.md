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
        DS_API_BASE_URL=https://dnext-ds-api.hf.space
        DS_API_MULTIMODULE_ENDPOINT=/mcp_multi_module_bug_localization
        DS_API_SINGLEMODULE_ENDPOINT=/mcp_single_module_bug_localization
        DS_API_SEARCHSPACE_ENDPOINT=/mcp_search_space_routing
        
        HF_ACCESS_TOKEN=your_huggingface_token_here
       ```

 -  Replace HF_ACCESS_TOKEN with your personal access token from Hugging Face.

## Run the Server
 - Start the MCP server locally with:
   ```sh
   uv run main.py
   ```


### Note: 
Once running, your MCP server will be available at:
http://127.0.0.1:8000/mcp/