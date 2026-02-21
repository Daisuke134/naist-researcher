# Ralph ゴール定義 — 学振DC1研究計画書

## 目標

**peer-review スコア 85/100 以上の学振DC1研究計画書を生成する。**

## 終了条件

```
RALPH_STATUS: EXIT_SIGNAL: true
```

peer-review スコアが 85 点以上になったら上記を出力して終了。

## タスクリスト

`.ralph/fix_plan.md` を確認して未完了タスクを処理する。

## 現在のスコア

`writing_outputs/*/progress.md` を確認。

## Circuit Breaker

- 同一エラーが3回連続 → 停止してSlackに報告
- 5回反復してもスコア85未満 → 停止してSlackに報告

停止時の報告文:
```
【naist-researcher 停止通知】
5ラウンド実行しましたが、スコアが85点に達しませんでした。
最終スコア: {SCORE}/100
手動での確認をお願いします。
```

## ループフロー

```
1. prompts/generate.md を読む
2. K-Denseで文献調査 → 初稿生成 → 概念図生成
3. prompts/review.md を読む
4. peer-reviewでスコア算出
5. スコア確認:
   - ≥ 85 → EXIT_SIGNAL: true
   - < 85  → fix_plan.md 更新 → 1に戻る（最大5回）
```
