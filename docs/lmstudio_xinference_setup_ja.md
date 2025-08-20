# Lm Studio と Xinference を利用した Memobase サーバー設定

このドキュメントでは、Memobase サーバーをローカルの Lm Studio（LLM）と Xinference（Embedding）に接続する方法を示します。

## 前提条件

- Docker で Memobase サーバーを起動することを想定しています。
- Lm Studio は OpenAI 互換 API を提供している必要があります。
- Xinference は OpenAI 互換の Embedding API を有効にした状態で起動しておきます。

## 1. Lm Studio の準備

1. [Lm Studio](https://lmstudio.ai/) をインストールし、アプリを起動します。
2. "Local Server" や "API" などのメニューから OpenAI 互換サーバーを開始し、ポート番号（例: `1234`）を確認します。
3. サーバーが起動すると `http://localhost:1234/v1` で OpenAI 互換エンドポイントが利用できます。
   - Docker コンテナ内からアクセスする場合は `http://host.docker.internal:1234/v1` を利用します。
4. 認証キーは任意の文字列で構いません（例: `lm-studio`）。

## 2. Xinference の準備

1. Xinference をインストールしローカルサーバーを起動します。
   ```bash
   pip install "xinference[all]"
   xinference-local --host 0.0.0.0 --port 9997
   ```
2. Web UI (例: `http://localhost:9997`) から埋め込みモデルを追加します。`text-embedding-bge-small` など OpenAI 互換の埋め込みモデルをロードしてください。
3. モデルの次元数を確認します（`text-embedding-bge-small` は 512 次元など）。
4. 認証キーは任意の文字列で構いません（例: `xinference`）。

## 3. Memobase サーバーの設定

設定は `config.yaml` を編集する方法と、環境変数で指定する方法の 2 通りがあります。

### 3.1 `config.yaml` を編集する

`src/server/api/config.yaml` を編集し、以下の項目を設定します。

```yaml
llm_api_key: lm-studio
llm_base_url: http://host.docker.internal:1234/v1
best_llm_model: llama-3.1-8b-instruct

embedding_provider: openai
embedding_api_key: xinference
embedding_base_url: http://host.docker.internal:9997/v1
embedding_model: text-embedding-bge-small
embedding_dim: 512  # モデルの次元数に合わせて調整
```

### 3.2 環境変数で設定する

`src/server/.env` に以下を追記すると、上記と同じ設定が環境変数として読み込まれます。

```dotenv
MEMOBASE_LLM_API_KEY=lm-studio
MEMOBASE_LLM_BASE_URL=http://host.docker.internal:1234/v1
MEMOBASE_BEST_LLM_MODEL=llama-3.1-8b-instruct

MEMOBASE_EMBEDDING_PROVIDER=openai
MEMOBASE_EMBEDDING_API_KEY=xinference
MEMOBASE_EMBEDDING_BASE_URL=http://host.docker.internal:9997/v1
MEMOBASE_EMBEDDING_MODEL=text-embedding-bge-small
MEMOBASE_EMBEDDING_DIM=512
```

環境変数は `config.yaml` より優先されます。

- コンテナを使わずに直接実行する場合は `host.docker.internal` を `localhost` に置き換えてください。
- `embedding_dim` は使用するモデルの仕様に合わせて変更してください。

## 4. サーバーの起動

設定が完了したら、`docker-compose` で Memobase サーバー本体と MCP サーバーを同時に起動します。

```bash
cd src/server
cp .env.example .env
cp api/config.yaml.example api/config.yaml  # まだ作成していない場合
# 上記の設定を api/config.yaml に反映

docker-compose build
docker-compose up
```

起動後、内部ネットワーク経由で MCP サーバーが `memobase-server-api` に接続され、`MCP_EXPORT_PORT`（既定値: `8050`）で SSE エンドポイントが公開されます。

これで、Memobase サーバーは Lm Studio を LLM として、Xinference を Embedding として利用しつつ、MCP 経由で長期記憶を操作できるようになります。
