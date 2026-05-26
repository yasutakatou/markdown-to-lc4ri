# markdown-to-lc4ri

Claude Code / Copilot CLI / Codex CLI / OpenCode に対応した Agent Skills の配布リポジトリです。

## スキル一覧

| Skill | Description |
|-------|-------------|
| [markdown-to-lc4ri](./plugins/markdown-to-lc4ri/) | 通常のMarkdown形式の手順書を、自動実行・再現可能なLC4RI (Literate Computing for Reproducible Infrastructure) 形式へ変換するスキル |

## インストール方法

### Claude Code — `/plugin` コマンド経由（推奨）

```
/plugin marketplace add yasutakatou/markdown-to-lc4ri
/plugin install yasutakatou/markdown-to-lc4ri
```

---

## リポジトリ構造

```
skill-marketplace-template/
├── README.md                          # このファイル
├── LICENSE                            # MIT License
├── .gitignore
├── .claude-plugin/
│   └── marketplace.json               # マーケットプレイス定義
└── plugins/
    └── markdown-to-lc4ri/                 # 各スキルは独立したプラグイン
        └── skills/
            └── markdown-to-lc4ri/
                └── SKILL.md           # スキル本体
```

## ライセンス

MIT License — 詳細は [LICENSE](./LICENSE) を参照してください。
