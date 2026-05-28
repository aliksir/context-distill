# context-distill — Claude Code 向けセッションコンテキスト蒸留

セッション中のコンテキストを構造化 YAML スナップショットに変換し、セッション間の AI-to-AI ナレッジ転送を実現する Claude Code プラグイン。

## 何ができるか

作業セッション終了時に `/context-distill` を実行すると、設計判断・失敗と復旧・再利用可能なパターンを構造化 YAML に保存。次のセッションの AI がパースして再利用できる。

**人間向けの引き継ぎメモではなく、AI が検索・学習できるセッション記憶。**

## クイックスタート

```bash
claude plugin install context-distill
```

Claude Code で:
```
/context-distill
```

## 仕組み

1. **キャプチャ** — 読み書きしたファイル、設計判断とその理由を収集
2. **構造化** — YAML スナップショットに変換（マークダウンの散文ではない）
3. **永続化** — `memory/snapshots/{日付}_{トピック}/snapshot.yaml` に保存
4. **インデックス** — `memory/snapshots/index.jsonl` に追記し将来の検索に対応

### 出力例

```yaml
key_decisions:
  - decision: "ステータスカラムではなくイベントテーブルを使用"
    why: "状態遷移に業務上の意味がある（契約ライフサイクル）"
    evidence: "設計レビュー時のドメイン分析で判明"
    reusable: true

failures_and_recovery:
  - failure: "マイグレーションスクリプトが既存データで失敗"
    recovery: "新カラムにデフォルト値を追加し再実行"
    prevention: "本番相当のデータでマイグレーションを事前テストする"

reusable_patterns:
  - pattern: "ZIP ファイル内容の全書き換え方式での置換"
    snippet: |
      with zipfile.ZipFile(src) as zin, zipfile.ZipFile(dst, 'w') as zout:
          for item in zin.namelist():
              data = new_content if item == target else zin.read(item)
              zout.writestr(item, data)
    when: "ZIP ベースフォーマット（OOXML, JAR 等）内のファイル変更時"
```

## 4 原則

| 原則 | ルール |
|------|--------|
| **fabrication 禁止** | 不明な値 → `unknown`、推測で埋めない |
| **YAML のみ** | 機械パース前提、マークダウン段落は書かない |
| **PII 警戒** | 機密セッション → パスや名前を匿名化 |
| **append-only** | 既存スナップショットの上書き禁止 |

## handover と snapshot の違い

| | handover.md | snapshot.yaml |
|---|---|---|
| 読者 | 人間 + AI | AI 専用 |
| 形式 | Markdown | YAML |
| 用途 | セッション引き継ぎの全体像 | 機械パースによるナレッジ検索・再利用 |
| ライフサイクル | 毎セッション上書き | append-only、全セッション保持 |

両方を併用できる。handover は人間が追いつくため、snapshot は AI が過去セッションから学ぶため。

## 要件

- Claude Code
- プロジェクト内の `memory/` ディレクトリ（初回実行時に自動作成）

## ライセンス

MIT
