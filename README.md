# MCP Memory Service

This directory contains the persistent memory database for the MCP Memory Service used with this repository.

## About

The MCP Memory Service ([github.com/doobidoo/mcp-memory-service](https://github.com/doobidoo/mcp-memory-service)) is a semantic memory and persistent storage system that provides long-term memory capabilities for AI assistants. It enables storing and retrieving information with semantic search, allowing AI systems to maintain context and knowledge across different sessions.

## Setup

To use the MCP Memory Service with this repository, you need to set up the `MCP_MEMORY_CHROMA_PATH` environment variable in your MCP client settings:

```
MCP_MEMORY_CHROMA_PATH=<project_path>/memories/
```

Where `<project_path>` is the path to your project directory containing the memories folder.

## Benefits

- Maintains persistent knowledge across sessions
- Provides semantic search capabilities for relevant information retrieval
- Supports natural language time-based queries
- Enables tag-based organization and retrieval
- Helps track important information and decisions
- Improves consistency in responses and recommendations

## Usage

The memory service is automatically used by MCP-compatible AI assistants when properly configured. No additional steps are needed beyond the initial setup and configuration of your MCP client.

### Memory Operations

The MCP Memory Service provides the following operations:

#### Core Memory Operations

1. `store_memory` - Store new information with optional tags
2. `retrieve_memory` - Perform semantic search for relevant memories
3. `recall_memory` - Retrieve memories using natural language time expressions (e.g., "last week", "yesterday morning")
4. `search_by_tag` - Find memories using specific tags
5. `exact_match_retrieve` - Find memories with exact content match
6. `debug_retrieve` - Retrieve memories with similarity scores

#### Memory Management

7. `delete_memory` - Delete specific memory by hash
8. `delete_by_tag` - Delete all memories with specific tag
9. `cleanup_duplicates` - Remove duplicate entries
10. `delete_by_timeframe` - Delete memories within a specific timeframe
11. `delete_before_date` - Delete memories before a specific date

#### Database Management

12. `check_database_health` - Get database health metrics
13. `check_embedding_model` - Verify model status

### Tag Storage

When storing memories, you can add tags to categorize them for easier retrieval later. Tags can be provided in two formats:

1. Array format: `["tag1", "tag2"]`
2. String format: `"tag1,tag2"` (preferred)

Example with array format:
```json
{
    "content": "Memory content",
    "metadata": {
        "tags": ["important", "reference"],
        "type": "note"
    }
}
```

Example with string format (preferred):
```json
{
    "content": "Memory content",
    "metadata": {
        "tags": "important,reference",
        "type": "note"
    }
}
```

### Natural Language Time Expressions

The `recall_memory` operation supports various time-related expressions such as:
- "yesterday", "last week", "2 days ago"
- "last summer", "this month", "last January"
- "spring", "winter", "Christmas", "Thanksgiving"
- "morning", "evening", "yesterday afternoon"

Examples:
```
{
    "query": "recall what I stored last week"
}

{
    "query": "find information about databases from two months ago",
    "n_results": 5
}
```

### Timeframe-Based Operations

You can retrieve or delete memories within specific timeframes:

```
{
    "start_date": "2024-01-01",
    "end_date": "2024-01-31",
    "n_results": 5
}
```

To delete memories before a specific date:
```
{
    "before_date": "2024-01-01",
    "tag": "temporary"  // Optional: only delete memories with this tag
}
```

## Adding MCP Memory Server to Your Project

To add the MCP Memory Server to your own project, follow these steps:

### 1. Installation Options

#### Option A: Quick Installation with UV (Recommended)

```bash
# Install UV if not already installed
pip install uv

# Clone and install
git clone https://github.com/doobidoo/mcp-memory-service.git
cd mcp-memory-service
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -r requirements.txt
uv pip install -e .

# Run the service
uv run memory
```

#### Option B: Docker Installation

```bash
# Clone the repository
git clone https://github.com/doobidoo/mcp-memory-service.git
cd mcp-memory-service

# Build the Docker image
docker build -t mcp-memory-service .

# Create directories for persistent storage
mkdir -p ./data/chroma_db ./data/backups

# Run the container
docker run -d \
  --name mcp-memory \
  --restart unless-stopped \
  -v $(pwd)/data/chroma_db:/app/chroma_db \
  -v $(pwd)/data/backups:/app/backups \
  -e MCP_MEMORY_CHROMA_PATH=/app/chroma_db \
  -e MCP_MEMORY_BACKUPS_PATH=/app/backups \
  mcp-memory-service
```

### 2. Project Integration

1. Create a `memories` directory in your project root:
   ```bash
   mkdir -p memories
   ```

2. Configure your MCP client to use the memory service:

   Edit your MCP client configuration file and add the following configuration:
   ```json
   {
     "memory": {
       "command": "uv",
       "args": [
         "--directory",
         "path/to/mcp-memory-service",
         "run",
         "memory"
       ],
       "env": {
         "MCP_MEMORY_CHROMA_PATH": "path/to/your/project/memories",
         "MCP_MEMORY_BACKUPS_PATH": "path/to/your/project/memories/backups"
       }
     }
   }
   ```

3. For Docker-based setup, use:
   ```json
   {
     "memory": {
       "command": "docker",
       "args": [
         "run",
         "-i",
         "--rm",
         "-v", "path/to/your/project/memories:/app/chroma_db",
         "-v", "path/to/your/project/memories/backups:/app/backups",
         "mcp-memory-service"
       ],
       "env": {
         "MCP_MEMORY_CHROMA_PATH": "/app/chroma_db",
         "MCP_MEMORY_BACKUPS_PATH": "/app/backups"
       }
     }
   }
   ```

### 3. Verify Installation

1. Start your MCP client
2. You should see a message indicating the memory service has initialized
3. Test by asking your AI assistant to remember something and then recall it later

### 4. Configuration Options

The MCP Memory Service can be configured through environment variables:

| Environment Variable | Description | Default Value |
|----------------------|-------------|---------------|
| `MCP_MEMORY_CHROMA_PATH` | Path to ChromaDB storage | Required |
| `MCP_MEMORY_BACKUPS_PATH` | Path for backups | Optional |
| `MCP_MEMORY_AUTO_BACKUP_INTERVAL` | Backup interval in hours | 24 |
| `MCP_MEMORY_MAX_MEMORIES_BEFORE_OPTIMIZE` | Threshold for auto-optimization | 10000 |
| `MCP_MEMORY_SIMILARITY_THRESHOLD` | Default similarity threshold | 0.7 |
| `MCP_MEMORY_MAX_RESULTS_PER_QUERY` | Maximum results per query | 10 |
| `MCP_MEMORY_BACKUP_RETENTION_DAYS` | Number of days to keep backups | 7 |
| `MCP_MEMORY_LOG_LEVEL` | Logging level | INFO |

#### Hardware-specific environment variables:

| Environment Variable | Description | Default Value |
|----------------------|-------------|---------------|
| `PYTORCH_ENABLE_MPS_FALLBACK` | Enable MPS fallback for Apple Silicon | 1 |
| `MCP_MEMORY_USE_ONNX` | Use ONNX Runtime for CPU-only deployments | 0 |
| `MCP_MEMORY_USE_DIRECTML` | Use DirectML for Windows acceleration | 0 |
| `MCP_MEMORY_MODEL_NAME` | Override the default embedding model | paraphrase-MiniLM-L6-v2 |
| `MCP_MEMORY_BATCH_SIZE` | Override the default batch size | 8 |

## Cloudflare Worker Implementation

The MCP Memory Service also offers a serverless implementation using Cloudflare Workers. This implementation:

- Uses **Cloudflare D1** for storage (serverless SQLite)
- Uses **Workers AI** for embeddings generation
- Communicates via **Server-Sent Events (SSE)** for MCP protocol
- Requires no local installation or dependencies
- Scales automatically with usage

### Benefits of the Cloudflare Implementation

- **Zero local installation**: No Python, dependencies, or local storage needed
- **Cross-platform compatibility**: Works on any device that can connect to the internet
- **Automatic scaling**: Handles multiple users without configuration
- **Global distribution**: Low latency access from anywhere
- **No maintenance**: Updates and maintenance handled automatically

### Configuring Your MCP Client to Use the Cloudflare Memory Service

Add the following to your MCP client configuration to use the Cloudflare-based memory service:

```json
{
  "mcpServers": [
    {
      "name": "cloudflare-memory",
      "url": "https://your-worker-subdomain.workers.dev/mcp",
      "type": "sse"
    }
  ]
}
```

Replace `your-worker-subdomain` with your actual Cloudflare Worker subdomain.