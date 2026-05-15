# Nextcloud MCP Server

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server that provides tools for creating, reading, and managing files in Nextcloud via WebDAV.

## Features

This MCP server implements the following tools:

- **nextcloud_list** - List files and folders in a directory
- **nextcloud_read_file** - Read file contents
- **nextcloud_create_file** - Create or update a file
- **nextcloud_create_excel** - Create an Excel file with structured data
- **nextcloud_create_folder** - Create a folder (and parent folders if needed)
- **nextcloud_search** - Search for files by name

## Prerequisites

- Python 3.10 or higher
- Nextcloud instance with WebDAV access
- Nextcloud App Password (recommended) or user password

## Installation

### Local Installation

1. Clone the repository:
```bash
git clone https://github.com/your-username/mcp-nextcloud.git
cd mcp-nextcloud
```

2. Create a virtual environment and install dependencies:
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. Set environment variables:
```bash
export NEXTCLOUD_URL="https://<your-domain-here>"
export NEXTCLOUD_USER="assistant"
export NEXTCLOUD_APP_PASSWORD="<your-app-password>"
```

4. Run the server:
```bash
python3 server.py
```

### Docker Installation

1. Build the Docker image:
```bash
docker build -t mcp-nextcloud .
```

2. Run the container:
```bash
docker run -i \
  -e NEXTCLOUD_URL="https://<your-domain-here>" \
  -e NEXTCLOUD_USER="assistant" \
  -e NEXTCLOUD_APP_PASSWORD="<your-app-password>" \
  mcp-nextcloud
```

### Docker Compose

1. Copy the example environment file:
```bash
cp .env.example .env
```

2. Edit `.env` and set your Nextcloud credentials:
```env
NEXTCLOUD_URL=https://<your-domain-here>
NEXTCLOUD_USER=assistant
NEXTCLOUD_APP_PASSWORD=<your-app-password>
```

3. Start the server:
```bash
docker compose up -d
```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NEXTCLOUD_URL` | Yes | `https://<your-domain-here>` | Nextcloud instance URL |
| `NEXTCLOUD_USER` | Yes | `assistant` | Nextcloud username |
| `NEXTCLOUD_APP_PASSWORD` | Yes | - | App password or user password |

### Creating an App Password

For increased security, use an App Password instead of your main password:

1. Go to Nextcloud → Settings → Security
2. Under "Devices & sessions", create a new app password
3. Name it (e.g., "MCP Server")
4. Copy the generated password and use it as `NEXTCLOUD_APP_PASSWORD`

## Usage Examples

### List Files
```json
{
  "tool": "nextcloud_list",
  "arguments": {
    "path": "/Project1/documents"
  }
}
```

### Read File
```json
{
  "tool": "nextcloud_read_file",
  "arguments": {
    "path": "/Project1/planning.txt"
  }
}
```

### Create Text File
```json
{
  "tool": "nextcloud_create_file",
  "arguments": {
    "path": "/Reports/summary.md",
    "content": "# Report Summary\n\nThis is a markdown file."
  }
}
```

### Create Excel File
```json
{
  "tool": "nextcloud_create_excel",
  "arguments": {
    "path": "/Reports/data.xlsx",
    "data": {
      "sheets": [
        {
          "name": "Sales",
          "columns": ["Product", "Quantity", "Price"],
          "rows": [
            ["Widget A", 100, 9.99],
            ["Widget B", 50, 19.99]
          ]
        }
      ]
    }
  }
}
```

### Search Files
```json
{
  "tool": "nextcloud_search",
  "arguments": {
    "query": "budget",
    "path": "/Finance"
  }
}
```

## Integration with OpenClaw or Other MCP Clients

### Using with OpenClaw (Docker)

If you're running OpenClaw in a Docker container, add this MCP server to the network:

1. **docker-compose.yml for OpenClaw:**
```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw:latest
    volumes:
      - ./config:/config
    environment:
      - MCP_CONFIG=/config/mcp-config.json
    depends_on:
      - nextcloud-mcp
    networks:
      - mcp-network

  nextcloud-mcp:
    image: mcp-nextcloud
    environment:
      - NEXTCLOUD_URL=${NEXTCLOUD_URL}
      - NEXTCLOUD_USER=${NEXTCLOUD_USER}
      - NEXTCLOUD_APP_PASSWORD=${NEXTCLOUD_APP_PASSWORD}
    networks:
      - mcp-network
    stdin_open: true
    tty: true

networks:
  mcp-network:
    driver: bridge
```

2. **MCP Configuration (mcp-config.json):**
```json
{
  "mcpServers": {
    "nextcloud": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "nextcloud-mcp",
        "python3",
        "/app/server.py"
      ]
    }
  }
}
```

### Using with Claude Desktop or VS Code

Add to your MCP configuration file:

**Claude Desktop (~/.claude/config.json):**
```json
{
  "mcpServers": {
    "nextcloud": {
      "command": "python3",
      "args": ["/path/to/mcp-nextcloud/server.py"],
      "env": {
        "NEXTCLOUD_URL": "https://<your-domain-here>",
        "NEXTCLOUD_USER": "assistant",
        "NEXTCLOUD_APP_PASSWORD": "<your-app-password>"
      }
    }
  }
}
```

**VS Code (settings.json):**
```json
{
  "mcp.servers": {
    "nextcloud": {
      "command": "python3",
      "args": ["/path/to/mcp-nextcloud/server.py"],
      "env": {
        "NEXTCLOUD_URL": "https://<your-domain-here>",
        "NEXTCLOUD_USER": "assistant",
        "NEXTCLOUD_APP_PASSWORD": "<your-app-password>"
      }
    }
  }
}
```

## Integration with Azure DevOps MCP

For a complete workflow that handles both code (Azure DevOps) and documents (Nextcloud), combine both MCP servers:

```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "python3",
      "args": ["/path/to/mcp-azure-devops/server.py"],
      "env": {
        "AZURE_DEVOPS_ORG": "<org>",
        "AZURE_DEVOPS_PROJECT": "<project>",
        "AZURE_DEVOPS_PAT": "<pat>"
      }
    },
    "nextcloud": {
      "command": "python3",
      "args": ["/path/to/mcp-nextcloud/server.py"],
      "env": {
        "NEXTCLOUD_URL": "https://<your-domain-here>",
        "NEXTCLOUD_USER": "assistant",
        "NEXTCLOUD_APP_PASSWORD": "<app-password>"
      }
    }
  }
}
```

## Troubleshooting

### Connection Issues

Test WebDAV connection:
```bash
curl -u "$NEXTCLOUD_USER:$NEXTCLOUD_APP_PASSWORD" \
  "$NEXTCLOUD_URL/remote.php/dav/files/$NEXTCLOUD_USER/"
```

### SSL Certificate Issues

If you're using a self-signed certificate, you may need to disable SSL verification (not recommended for production):

1. Modify `server.py` and set:
```python
webdav_options = {
    ...
    'disable_check': True
}
```

### File Not Found Errors

- Ensure paths start with `/`
- Check that the user has access to the specified folder
- Verify folder exists: use `nextcloud_list` to check parent directory

## Development

### Running Tests

```bash
# Install dev dependencies
pip install -r requirements.txt

# Run the server in debug mode
LOG_LEVEL=DEBUG python3 server.py
```

### Adding New Tools

1. Add the function implementation in `server.py`
2. Register the tool in `list_tools()`
3. Add the handler in `call_tool()`
4. Update README with usage examples

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/specification)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [Nextcloud WebDAV Documentation](https://docs.nextcloud.com/server/latest/developer_manual/client_apis/WebDAV/index.html)
- [WebDAV Client Library](https://github.com/ezhov-evgeny/webdav-client-python-3)
- [OpenPyXL Documentation](https://openpyxl.readthedocs.io/)