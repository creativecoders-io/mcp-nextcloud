# Nextcloud MCP Server - Quick Start

## Quick Test

1. **Install dependencies:**
```bash
pip install -r requirements.txt
```

2. **Set environment variables:**
```bash
export NEXTCLOUD_URL="https://<your-domain-here>"
export NEXTCLOUD_USER="assistant"
export NEXTCLOUD_APP_PASSWORD="your-app-password"
```

3. **Test the server:**
```bash
python3 server.py
```

The server will start and wait for MCP protocol messages on stdin.

## Quick Docker Test

```bash
# Build
docker build -t mcp-nextcloud .

# Run (interactive mode to test)
docker run -i \
  -e NEXTCLOUD_URL="https://<your-domain-here>" \
  -e NEXTCLOUD_USER="assistant" \
  -e NEXTCLOUD_APP_PASSWORD="your-password" \
  mcp-nextcloud
```

## Next Steps

- See [README.md](README.md) for full documentation
- Check [MCP-SERVERS-GUIDE.md](../MCP-SERVERS-GUIDE.md) for integration instructions
