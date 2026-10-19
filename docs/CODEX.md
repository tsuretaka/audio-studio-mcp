# Codex / ChatGPT setup

Audio Studio uses Stable Audio 3 on your Apple Silicon Mac. Codex or ChatGPT
interprets the request and calls the MCP tools; no Claude installation or
Anthropic API key is required.

## Codex on the Mac running the model

Run the normal installer (`./install.sh`) first, then register the installed
executable using its absolute path:

```sh
codex mcp add audio-studio -- /absolute/path/to/audio-studio-mcp/.venv/bin/audio-studio-mcp
```

In `~/.codex/config.toml`, configure the existing entry (do not add it twice):

```toml
[mcp_servers.audio-studio]
command = "/absolute/path/to/audio-studio-mcp/.venv/bin/audio-studio-mcp"
cwd = "/absolute/path/to/audio-studio-mcp"
startup_timeout_sec = 60
tool_timeout_sec = 900

[mcp_servers.audio-studio.env]
AUDIO_STUDIO_OUTPUT_DIR = "/absolute/path/to/audio-studio-mcp/outputs"
AUDIO_STUDIO_SA3_MODEL = "medium"
AUDIO_STUDIO_FFMPEG = "/absolute/path/to/ffmpeg"
```

Find ffmpeg with `command -v ffmpeg`. Preserve all other MCP entries and existing
configuration. Start a new Codex conversation after registering the server; use
`codex mcp get audio-studio` or `/mcp` to check it. The timeout accommodates model
loading and longer generations; the first installation downloads about 5.9 GB.

Example request:

> 85 BPM、20秒の落ち着いたカフェ用BGMを作って。歌なし。WAVで保存して。

The server instructions guide Codex to translate the sound description into
English while preserving the original Japanese request in `original_idea`.
Outputs include WAV audio and a JSON sidecar with the prompt, seed and settings.

Run the existing real-model acceptance workflow (generation, all edit modes,
export and model unloading) while keeping its outputs:

```sh
.venv/bin/python scripts/smoke_test.py --out outputs/acceptance
```

This performs real MLX inference through MCP stdio. It does not test Codex's
language-model tool selection; verify that in a new connected conversation.

## ChatGPT and remote devices

A desktop client supporting local MCP can use the same command on the same Mac.
A browser or phone cannot access a Mac's file paths through a normal hosted chat.
For that workflow, connect an authenticated remote MCP endpoint or a supported
Secure MCP Tunnel, and implement audio retrieval/playback. Do not expose the
existing unauthenticated loopback server directly to the internet. Remote access
also needs a suitable job workflow for requests longer than transport deadlines.
Configuring Codex on one Mac does not automatically configure another device.

## 日本語メモ

- 音声生成はMac側のStable Audio 3＋MLXが担当します。
- Claude用エンジンの置換は不要で、CodexのMCP接続設定を追加します。
- Codexへ登録した後は、新しい会話で接続を読み込んでください。
- 出力先・ffmpegは実在する絶対パスを指定します。
- コードのMITライセンスとモデルの利用条件は別です。元のブランド素材を
  自分の製品のロゴやキャラクターとして転用しないでください。

## Official references

- [Codex MCP configuration](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)
- [Secure MCP Tunnel](https://developers.openai.com/api/docs/guides/secure-mcp-tunnels)
