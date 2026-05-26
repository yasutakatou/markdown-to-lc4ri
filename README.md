# markdown-to-lc4ri

Claude Code / Copilot CLI / Codex CLI / OpenCode に対応した Agent Skills の配布リポジトリです。
<br>
Markdown:
<img width="593" height="572" alt="image" src="https://github.com/user-attachments/assets/699a7fb1-97d6-4a8f-8447-50d723337a22" />
<br>
LC4RI:
<img width="582" height="611" alt="image" src="https://github.com/user-attachments/assets/7294bcce-27c4-40d6-a5bd-705778013c85" />

何故元のMarkdownのまま実行できるようにしないかは以下のような理由です。様々なターミナルオペレーション自動化のために記載を変える必要があります。<br>
- インデントによって実行結果が成功した場合をAND条件として記述(表現)できます。コマンドの列記では記載できません
- 数字の見出しをそのまま変数に格納する機能があります。こちらもコマンドの列記では記載できません
- ファイルから内容を読み込んだり、ファイルに記載内容を書き出すディレクティブがあります。これも出来ません

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
