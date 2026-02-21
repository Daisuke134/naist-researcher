# naist-researcher プロジェクトルール

## 依存関係

このスキルは K-Dense scientific-writer に依存している。

```bash
# K-Denseが正しくインストールされているか確認
ls ~/.claude/skills/claude-scientific-writer/skills/research-lookup/research_lookup.py
ls ~/.claude/skills/claude-scientific-writer/skills/parallel-web/scripts/parallel_web.py
```

## 必須コマンドルール

| タスク | 使うコマンド | 禁止 |
|-------|------------|------|
| 学術論文検索 | `python ~/.claude/skills/claude-scientific-writer/skills/research-lookup/research_lookup.py "..." -o sources/papers_XXX.md` | WebSearch |
| Webサーチ | `python ~/.claude/skills/claude-scientific-writer/skills/parallel-web/scripts/parallel_web.py search "..." -o sources/search_XXX.md` | WebSearch |
| 深掘りリサーチ | `python ~/.claude/skills/claude-scientific-writer/skills/parallel-web/scripts/parallel_web.py research "..." -o sources/research_XXX.md` | WebSearch |

## 出力ルール

- 全サーチ結果は必ず `sources/` に `-o` フラグで保存する
- ファイル名パターン: `sources/{type}_{YYYYMMDD}_{HHMMSS}_{topic}.md`
- 生成物バージョン: `drafts/v1_draft.md`, `v2_draft.md`... （上書き禁止）
- 最終稿: `final/dc1_draft_final.md`

## 採点・終了ルール

- peer-review スコア ≥ 85 → `RALPH_STATUS: EXIT_SIGNAL: true` を出力
- peer-review スコア < 85 → `.ralph/fix_plan.md` を更新して継続
- 5ラウンド超 → Circuit Breaker 発動、Slack に通知して停止

## 言語ルール

- 研究計画書は日本語（常体）
- コメント・進捗報告は日本語
