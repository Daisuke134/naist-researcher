---
name: naist-researcher
version: 1.0.0
description: >
  学振DC1研究計画書を自動生成するスキル。研究テーマを入力するだけで、
  K-Dense scientific-writerを使った文献調査→初稿生成→peer-reviewループ→Slack配信まで全自動で実行する。
  Use when: 「学振を書きたい」「研究計画書を作りたい」「DC1の申請書」「JSPS」「学振DC1」
  「終わるまでやれ」と言われた場合はRalphループを起動する。
keywords: [学振, DC1, DC2, 研究計画書, JSPS, grant, proposal, research, 申請書]
auto_activate: true
allowed-tools: [Read, Write, Edit, Bash, Grep, Glob]
---

# naist-researcher — 学振DC1研究計画書 自動生成スキル

## 前提確認（必須）

このスキルを実行する前に以下を確認する:

```bash
# APIキー確認
echo $ANTHROPIC_API_KEY | head -c 10
echo $OPENROUTER_API_KEY | head -c 10

# K-Dense scientific-writer確認
ls ~/.claude/skills/claude-scientific-writer/CLAUDE.md 2>/dev/null && echo "OK" || echo "NOT FOUND"
```

K-Dense が未インストールの場合（2ステップ）:
```bash
/plugin marketplace add https://github.com/K-Dense-AI/claude-scientific-writer
/plugin install claude-scientific-writer
# 次に初期化
/scientific-writer:init
```

## 入力形式

ユーザーが以下の形式で送ってくる:
```
学振DC1を書きたい。
テーマ：[研究テーマ]
ラボ：[ラボ名]
締切：[月]末
（オプション）終わるまでやれ。
```

## パイプライン実行

### Phase 1: 文献調査

出力ディレクトリ作成:
```bash
OUTDIR="writing_outputs/$(date +%Y%m%d_%H%M%S)_dc1"
mkdir -p "$OUTDIR/sources" "$OUTDIR/drafts" "$OUTDIR/figures" "$OUTDIR/final"
```

論文検索（research_lookup.py — OPENROUTER_API_KEY に有料プランが必要）:
```bash
python3 ~/.claude/skills/claude-scientific-writer/skills/research-lookup/research_lookup.py "[研究テーマ] neural circuit mechanism" -o "$OUTDIR/sources/papers_$(date +%Y%m%d_%H%M%S)_main.md" 2>&1
# 402エラーが出た場合はスキップして Phase 1b へ
```

**Phase 1b（フォールバック）: research_lookup.py が402エラーの場合**
`mcp__exa__web_search_exa` で以下を検索してソースファイルに保存:
1. `[研究テーマ] 先行研究 神経科学 日本語`
2. `[研究テーマ] research gap未解決問題`
3. `JSPS DC1 [研究テーマ] 採択 年報`

Webサーチ（parallel_web.py — PARALLEL_API_KEY が必要）:
```bash
python3 ~/.claude/skills/claude-scientific-writer/skills/parallel-web/scripts/parallel_web.py search "[研究テーマ] 先行研究 課題 未解決" -o "$OUTDIR/sources/search_$(date +%Y%m%d_%H%M%S)_background.md" 2>&1
# エラーの場合もスキップ可
```

### Phase 2: 初稿生成

`prompts/generate.md` を読んで学振DC1フォーマットで初稿を生成する。

採択例2本を参照:
- `data/jsps-examples/yoshida-dc1-2011.md` — 吉田 DC1 2011年（東大、面接免除採択）
- `data/jsps-examples/tanisumi-dc1-2019.md` — 谷隅 DC1 2019年（同志社大、採択）

概念図生成:
```bash
python3 ~/.claude/skills/claude-scientific-writer/skills/scientific-schematics/scripts/generate_schematic.py "研究フロー図: [研究テーマ]の概念と研究アプローチ" -o "$OUTDIR/figures/research_flow.png"
```

### Phase 3: peer-review（Eval ループ）

`prompts/review.md` を読んで採択例基準でスコアリング。

スコアリング基準:
| 評価項目 | 配点 |
|---------|------|
| 社会的背景（数値・罹患率等） | 20点 |
| ファネル構造（大→具体） | 15点 |
| 新規性の明確さ | 20点 |
| 研究方法の具体性 | 20点 |
| 図・概念図 | 10点 |
| 期待成果・社会貢献 | 15点 |

スコア < 85 → `.ralph/fix_plan.md` を更新して Phase 2 に戻る（最大5回）
スコア ≥ 85 → `RALPH_STATUS: EXIT_SIGNAL: true` を出力して終了

### 完成時の出力

```
✅ 学振DC1研究計画書 生成完了

スコア推移:
  Round 1: 72/100
  Round 2: 81/100
  Round 3: 87/100 ← EXIT

ファイル: writing_outputs/[timestamp]_dc1/final/dc1_draft_v3.md

強み:
- 社会的背景が明確（100万人罹患の記述あり）
- 研究目的が具体的
- 概念図で全体像を可視化

改善の余地（参考）:
- 期待成果の波及効果をさらに具体化
```

## Ralph ループ仕様

| 項目 | 値 |
|------|-----|
| 終了条件 | peer-review スコア ≥ 85/100 |
| 最大回数 | 5回 |
| Circuit Breaker | 同一エラー3回連続 or 5回で閾値未達 |
| 状態ファイル | `.ralph/fix_plan.md` |
| 進捗ファイル | `.ralph/progress.txt` |
