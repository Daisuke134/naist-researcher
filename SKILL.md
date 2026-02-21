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

K-Dense のワーキングディレクトリを設定:
```bash
export KDENSE_HOME=~/.claude/skills/claude-scientific-writer
cd $KDENSE_HOME
```

論文検索（research_lookup.py）:
```bash
python research_lookup.py "[研究テーマ] neural circuit mechanism" -o writing_outputs/[timestamp]_dc1/sources/papers_$(date +%Y%m%d_%H%M%S)_main.md
```

Webサーチ（parallel_web.py）:
```bash
python scripts/parallel_web.py search "[研究テーマ] 先行研究 課題 未解決" -o writing_outputs/[timestamp]_dc1/sources/search_$(date +%Y%m%d_%H%M%S)_background.md

python scripts/parallel_web.py research "[研究テーマ] gap analysis research gaps" --processor pro-fast -o writing_outputs/[timestamp]_dc1/sources/research_$(date +%Y%m%d_%H%M%S)_gaps.md
```

### Phase 2: 初稿生成

`prompts/generate.md` を読んで学振DC1フォーマットで初稿を生成する。

採択例2本を参照:
- `data/jsps-examples/yoshida-dc1-2011.md` — 吉田 DC1 2011年（東大、面接免除採択）
- `data/jsps-examples/tanisumi-dc1-2019.md` — 谷隅 DC1 2019年（同志社大、採択）

概念図生成:
```bash
cd $KDENSE_HOME
python scripts/generate_schematic.py "研究フロー図: [研究テーマ]の概念と研究アプローチ" -o writing_outputs/[timestamp]_dc1/figures/research_flow.png
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
