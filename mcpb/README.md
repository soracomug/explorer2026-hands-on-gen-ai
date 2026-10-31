# .mcpb パッケージ

Claude Desktop に SORACOM MCP Server をローカル MCP として追加するための拡張機能パッケージです。Free プランでは Claude Code が使えないため、Claude Desktop の参加者向けに配布します。

- `soracom/manifest.json`：パッケージの定義。`npx -y @soracom-labs/soracom-mcp-server@<バージョン>` を起動し、`SORACOM_PROFILE` にインストール時に入力したプロファイル名（既定：handson）を渡します。認証情報はパッケージに含めず、SORACOM CLI のプロファイル（`~/.soracom/<名前>.json`）から読み込みます
- `../docs/soracom-handson.mcpb`：ビルド済みのパッケージ。GitHub Pages からダウンロードできます

参加者の PC には、Node.js v22 以上（npx）と、`soracom configure --profile handson` で作ったプロファイルが必要です。

## ビルド

manifest を変更したら、`version` を上げてから再ビルドし、`docs/soracom-handson.mcpb` もコミットします。

```bash
npx -y @anthropic-ai/mcpb@2.1.2 validate mcpb/soracom/manifest.json
npx -y @anthropic-ai/mcpb@2.1.2 pack mcpb/soracom docs/soracom-handson.mcpb
```
