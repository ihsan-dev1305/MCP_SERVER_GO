# DummyJSON Explorer &mdash; Model Context Protocol (MCP) Server in Go

[![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go)](https://golang.org)
[![MCP Protocol](https://img.shields.io/badge/MCP%20Protocol-2024--11--05-4B32C3?style=for-the-badge)](https://modelcontextprotocol.io)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=for-the-badge)](https://github.com/ihsan-dev1305/MCP_SERVER/pulls)

A high-performance, production-ready [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server written in Go. This server bridges AI assistants—such as **Claude Desktop**, **Cursor**, **Zed**, and other MCP-compliant clients—with the comprehensive [DummyJSON](https://dummyjson.com) REST ecosystem, exposing **27 specialized tools** for querying, searching, comparing, and aggregating real-world mock data.

---

## 🌟 Key Features

- **⚡ 27 Rich MCP Tools:** Complete coverage of DummyJSON entities: Products, Users, Shopping Carts, Social Posts, Comments, Recipes, and Todos.
- **🚀 Concurrent Composite Aggregations:** Features advanced composite tools (`get_user_overview`, `get_shopping_summary`, `compare_products`, `get_post_details`) that leverage Go's `golang.org/x/sync/errgroup` to fetch and normalize multiple endpoints in parallel.
- **🛡️ Universal Schema Portability:** Built-in schema normalization automatically transforms ambiguous nullable array definitions into strict `anyOf` branches, preventing schema rejection across diverse LLM client engines.
- **🔌 Native Stdio Transport:** Adheres strictly to the JSON-RPC 2.0 stdio protocol specifications—diagnostic logging is isolated to `stderr`, guaranteeing an unpolluted, deterministic `stdout` stream.
- **📦 Zero External Dependencies to Run:** Interacts with the public DummyJSON REST API without requiring API keys, local databases, or authentication setup.

---

## 🏗️ Architecture

```
┌────────────────────────────────────────────────────────┐
│                   AI Host / Client                     │
│      (Claude Desktop, Cursor, Zed, MCP Inspector)      │
└──────────────────────────┬─────────────────────────────┘
                           │ Standard Input / Output (JSON-RPC 2.0)
                           ▼
┌────────────────────────────────────────────────────────┐
│             DummyJSON MCP Server (Go)                  │
│  ┌──────────────────────────────────────────────────┐  │
│  │   mcpserver: Router, Schemas & Tool Handlers     │  │
│  └───────────────────────┬──────────────────────────┘  │
│                          │ Context-Aware Go Routines   │
│  ┌───────────────────────▼──────────────────────────┐  │
│  │    api: HTTP Client (Pagination, Filters, Sort)  │  │
│  └───────────────────────┬──────────────────────────┘  │
└──────────────────────────┼─────────────────────────────┘
                           │ HTTPS REST Requests
                           ▼
┌────────────────────────────────────────────────────────┐
│             DummyJSON Public REST API                  │
│              (https://dummyjson.com)                   │
└────────────────────────────────────────────────────────┘
```

---

## 📋 Table of Contents

- [Quick Start](#-quick-start)
  - [Prerequisites](#prerequisites)
  - [Installation & Building](#installation--building)
- [Client Configuration](#-client-configuration)
  - [Claude Desktop](#claude-desktop)
  - [Cursor](#cursor)
- [Testing with MCP Inspector](#-testing-with-mcp-inspector)
- [Complete Tool Catalog](#-complete-tool-catalog)
- [Project Layout](#-project-layout)
- [Development & Testing](#-development--testing)
- [Contributing](#-contributing)

---

## 🚀 Quick Start

### Prerequisites

- **Go:** Version 1.22 or higher installed. Check with:
  ```bash
  go version
  ```
- **Git**
- An MCP Client (e.g., [Claude Desktop](https://claude.ai/download), [Cursor](https://cursor.com), or [MCP Inspector](https://github.com/modelcontextprotocol/inspector)).

### Installation & Building

1. **Clone the repository:**
   ```bash
   git clone https://github.com/ihsan-dev1305/MCP_SERVER.git
   cd MCP_SERVER
   ```

2. **Download dependencies:**
   ```bash
   go mod download
   ```

3. **Build the executable binary:**
   - **Linux / macOS:**
     ```bash
     go build -o dummyjson-explorer .
     ```
   - **Windows:**
     ```powershell
     go build -o dummyjson-explorer.exe .
     ```

---

## ⚙️ Client Configuration

### Claude Desktop

Claude Desktop connects to MCP servers configured in `claude_desktop_config.json`.

#### Configuration File Locations:
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Linux:** `~/.config/Claude/claude_desktop_config.json`
- **Windows (Standard):** `%APPDATA%\Claude\claude_desktop_config.json`
- **Windows (MSIX / Windows Store):** `%LOCALAPPDATA%\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json`
- **Windows (Claude v5+ Data Directory):** `%LOCALAPPDATA%\Claude-Data\claude_pzs8sxrjxfjjc\claude_desktop_config.json`

> **Tip:** You can always open the active config file directly from Claude Desktop by going to **Settings (`Ctrl+,` or `Cmd+,`) > Developer > Edit Config**.

#### Option A: Running the Compiled Binary (Recommended)

Replace `/ABSOLUTE/PATH/TO` with your machine's full absolute path:

```json
{
  "mcpServers": {
    "dummyjson-explorer": {
      "command": "C:\\Users\\User\\Desktop\\MCP_GO\\MCP_SERVER\\dummyjson-explorer.exe",
      "args": []
    }
  }
}
```

*(On macOS/Linux, specify the unix path, e.g. `/home/user/MCP_SERVER/dummyjson-explorer`)*.

#### Option B: Running with `go run` (Active Development)

If you are modifying Go source files and do not wish to recompile every time:

```json
{
  "mcpServers": {
    "dummyjson-explorer": {
      "command": "go",
      "args": [
        "run",
        "C:\\Users\\User\\Desktop\\MCP_GO\\MCP_SERVER\\main.go"
      ]
    }
  }
}
```

> **Important Note for Windows Users:**
> - Ensure all path backslashes are escaped (`\\`).
> - Save the file as standard **UTF-8 (without BOM)**.

---

### Cursor

To configure with Cursor:
1. Open Cursor Settings > **Features** > **MCP Servers**.
2. Click **Add New MCP Server**.
3. Set Name to `dummyjson-explorer`, Type to `command`, and enter the absolute path to your compiled binary or `go run` command.

---

## 🔍 Testing with MCP Inspector

You can test and inspect all tools interactively in your browser using the official MCP Inspector:

```bash
npx @modelcontextprotocol/inspector go run main.go
```

Or test with the compiled binary:

```bash
npx @modelcontextprotocol/inspector ./dummyjson-explorer
```

Open `http://localhost:5173` in your browser, click **Connect**, navigate to **Tools**, and execute queries with live schema validation. For sample input parameters, refer to [API_FORMAT.MD](API_FORMAT.MD).

---

## 🛠️ Complete Tool Catalog

The server registers **27 tools** categorized below:

### 1. Composite & High-Level Aggregation Tools
These composite tools execute parallel queries to generate unified responses:

| Tool Name | Parameters | Description |
| :--- | :--- | :--- |
| `get_user_overview` | `userId` (int, required) | Concurrently fetches user profile, recent posts, assigned todos, and shopping carts into a unified dashboard. |
| `get_shopping_summary` | `userId` (int, required) | Aggregates all user carts to calculate total spending, item count, quantities, and discounts. |
| `get_post_details` | `postId` (int, required) | Fetches a post, its comments, and author profile concurrently. |
| `compare_products` | `productIds` ([]int, required) | Fetches multiple products simultaneously and produces a side-by-side comparison matrix. |

---

### 2. Products & Inventory
| Tool Name | Key Parameters | Description |
| :--- | :--- | :--- |
| `list_products` | `limit`, `skip`, `sortBy`, `order`, `select` | Paginated product listing. |
| `get_product` | `id` (int, required) | Retrieves full details for a product. |
| `get_products_by_category` | `category` (string, required), pagination | Filters products by category (e.g., `laptops`, `smartphones`). |
| `search_products` | `searchTerm` (string, required), pagination | Searches product catalog by keyword. |

---

### 3. Users & Accounts
| Tool Name | Key Parameters | Description |
| :--- | :--- | :--- |
| `list_users` | `limit`, `skip`, `sortBy`, `order`, `select` | Paginated user profiles. |
| `get_user` | `id` (int, required) | Retrieves profile for a specific user ID. |
| `search_users` | `searchTerm` (string, required), pagination | Searches users by name, email, or credentials. |

---

### 4. Carts & Orders
| Tool Name | Key Parameters | Description |
| :--- | :--- | :--- |
| `list_carts` | `limit`, `skip` | Returns all customer shopping carts. |
| `get_cart` | `id` (int, required) | Retrieves details of a specific cart ID. |
| `get_carts_by_user` | `userId` (int, required), pagination | Retrieves all carts created by a user. |

---

### 5. Social Posts & Comments
| Tool Name | Key Parameters | Description |
| :--- | :--- | :--- |
| `list_posts` | `limit`, `skip`, `sortBy`, `order`, `select` | Paginated community posts. |
| `get_post` | `id` (int, required) | Retrieves a specific post by ID. |
| `get_posts_by_user` | `userId` (int, required), pagination | Lists all posts authored by a user. |
| `search_posts` | `searchTerm` (string, required), pagination | Keyword search across posts. |
| `list_comments` | `limit`, `skip` | Lists comments across all posts. |
| `get_comments_by_post` | `postId` (int, required), pagination | Retrieves comments on a specific post. |
| `get_comments_by_user` | `userId` (int, required) | Reconstructs all comments posted by a given user. |

---

### 6. Recipes & Todos
| Tool Name | Key Parameters | Description |
| :--- | :--- | :--- |
| `list_recipes` | `limit`, `skip`, `sortBy`, `order`, `select` | Lists cooking recipes with ingredients and instructions. |
| `get_recipe` | `id` (int, required) | Retrieves a single recipe by ID. |
| `search_recipes` | `searchTerm` (string, required), pagination | Searches recipes by query. |
| `list_todos` | `limit`, `skip` | Lists task items across the system. |
| `get_todo` | `id` (int, required) | Retrieves a todo item by ID. |
| `get_todos_by_user` | `userId` (int, required), pagination | Retrieves tasks assigned to a specific user. |

---

## 📂 Project Layout

```text
MCP_SERVER/
├── api/                   # DummyJSON HTTP client package
│   ├── client.go          # Core REST client, pagination, parameter builders
│   └── client_test.go     # Unit & integration tests for API endpoints
├── mcpserver/             # Model Context Protocol layer
│   ├── server.go          # Tool definitions, schema inference, request handlers
│   └── server_test.go     # In-memory client/server integration tests
├── models/                # Domain data models & struct declarations
│   ├── product.go         # Product models
│   ├── recipe.go          # Recipe models
│   └── resources.go       # User, Cart, Post, Todo, Comment models
├── .gitignore             # Git exclusion rules
├── API_FORMAT.MD          # MCP Inspector test payloads guide
├── go.mod                 # Module dependencies definition
├── go.sum                 # Checksums for dependencies
├── LICENSE                # MIT License
├── main.go                # Server entry point & Stdio transport initialization
└── README.md              # Project documentation
```

---

## 🧪 Development & Testing

### Running Unit Tests

Run all unit tests across packages:

```bash
go test -v ./...
```

Run tests with race detection:

```bash
go test -race ./...
```

### Code Formatting & Vetting

Ensure idiomatic Go standards:

```bash
go fmt ./...
go vet ./...
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check the [issues page](https://github.com/ihsan-dev1305/MCP_SERVER/issues).

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request
