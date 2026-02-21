# naist-researcher

学振DC1研究計画書 自動生成スキル — 田中研 / ComBeNSラボ向け

## セットアップ（初回のみ、5分）

```bash
# Step 1: このリポジトリをインストール
git clone https://github.com/Daisuke134/naist-researcher ~/.claude/skills/naist-researcher

# Step 2: K-Dense scientific-writerをプラグインとしてインストール
/plugin marketplace add https://github.com/K-Dense-AI/claude-scientific-writer
/plugin install claude-scientific-writer
# 次に初期化（一度だけ）
/scientific-writer:init

# Step 3: APIキーを設定（~/.zshrc に追加）
export ANTHROPIC_API_KEY="sk-ant-..."     # 必須
export OPENROUTER_API_KEY="sk-or-..."     # Perplexity論文検索に必要
```

APIキーの取得先:
- ANTHROPIC_API_KEY: https://console.anthropic.com
- OPENROUTER_API_KEY: https://openrouter.ai

**注意:** OPENROUTER_API_KEY は論文検索強化に使用。無料プランでは Perplexity Sonar が使えないため
研究検索は Claude 内蔵検索にフォールバックします（品質はやや低下）。有料プランにすると最高品質になります。

## 使い方

### Mode A: Cursor / Claude Code（ローカル実行）

Claude Code を開いて以下をコピペ:

```
学振DC1の研究計画書を書きたい。
テーマ：[ここに研究テーマを書く]
ラボ：田中研 / ComBeNS
締切：[月]末
終わるまでやれ。
```

30分後に初稿が完成します。

### Mode B: Slack（インストール不要）

1. `#auto-research-自分の名前` チャンネルを作る（例: `#auto-research-narita`）
2. Anicca に以下を送る:

```
学振DC1の研究計画書を書きたい。
テーマ：[ここに研究テーマを書く]
ラボ：田中研 / ComBeNS
締切：[月]末
```

30分後に初稿が届きます。

## 裏でやっていること

| ステップ | 内容 |
|---------|------|
| 1 | 関連論文20本を自動取得（Perplexity Sonar Pro） |
| 2 | 採択例2本を参照してDC1フォーマットで初稿生成 |
| 3 | 採択例と自動比較してスコア算出（0〜100点） |
| 4 | 85点以上になるまで自動改善（最大5回） |
| 5 | Slackに完成版を配信 |

## 採択例データ

| ファイル | 内容 |
|---------|------|
| `data/jsps-examples/yoshida-dc1-2011.md` | 吉田 DC1 2011年（東大、面接免除採択） |
| `data/jsps-examples/tanisumi-dc1-2019.md` | 谷隅 DC1 2019年（同志社大、採択） |

## 使用ライブラリ（全部OSS）

| ライブラリ | 用途 | Stars |
|-----------|------|-------|
| K-Dense-AI/claude-scientific-writer | 文献検索・論文執筆・引用管理 | 8900+ |
| ralph-autonomous-dev（既存スキル） | 自動反復ループ制御 | — |

## 質問・バグ報告

Slack: @ダイス
GitHub Issues: https://github.com/Daisuke134/naist-researcher/issues
