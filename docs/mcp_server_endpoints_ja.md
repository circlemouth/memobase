# MCP サーバーのエンドポイント

Memobase に付属する MCP サーバーが提供するエンドポイントと設定方法をまとめます。

## SSE トランスポート

HTTP の Server-Sent Events (SSE) を利用する場合、以下の URL でアクセスできます。

```
http://<HOST>:<PORT>/sse
```

- `HOST` と `PORT` は環境変数で指定します。既定値は `HOST=0.0.0.0`、`PORT=8050` です。
- Docker コンテナ内からホスト側の MCP サーバーに接続する場合は `localhost` の代わりに `host.docker.internal` を利用してください。
- `docker-compose` で起動した場合は、`.env` の `MCP_EXPORT_PORT`（既定値: `8050`）を公開ポートとして利用できます。

### 例: Cursor から接続する場合

`~/.cursor/mcp.json` に以下を追記します。

```json
{
  "mcpServers": {
    "memobase": {
      "transport": "sse",
      "url": "http://localhost:8050/sse"
    }
  }
}
```

## Stdio トランスポート

`TRANSPORT=stdio` を指定すると、HTTP エンドポイントではなく標準入出力で通信します。MCP クライアント側でコマンド実行と環境変数設定を行ってください。

### 例: Stdio を利用する設定

```json
{
  "mcpServers": {
    "memobase": {
      "command": "python",
      "args": ["path/to/src/mcp/src/main.py"],
      "env": {
        "TRANSPORT": "stdio",
        "MEMOBASE_API_KEY": "YOUR-API-KEY",
        "MEMOBASE_BASE_URL": "YOUR-MEMOBASE-URL"
      }
    }
  }
}
```

これらのエンドポイント設定により、Memobase の長期記憶機能を MCP 対応クライアントから利用できます。
