# 開発方針と AI 活用方針

OPA の開発における人間，ChatGPT，Codex の役割分担と，重要な技術判断の記録方法を定義します．

開発フロー，Issue，Pull Request，レビュー，マージ等の運用ルールは [CONTRIBUTING.md](../CONTRIBUTING.md) を正本とし，実装上の規約は [コーディング規約](CODING_STANDARDS.md) を正本とします．

## 役割分担

### 人間

人間は，プロジェクトの最終的な意思決定者です．

主な責務：

- プロジェクトの目的，優先順位，要求を決定する
- 重要な仕様や設計判断を承認する
- 新しい主要技術の採用や既存技術の置き換えを承認する
- AI が判断できない要件や方針を決定する
- Pull Request の最終的なマージ可否を決定する

AI は，人間の承認が必要な判断を独断で確定しません．

### ChatGPT

ChatGPT は，主に要求整理，設計支援，開発運用，レビュー支援を担当します．

主な責務：

- 要求や論点を整理する
- Issue の目的，対応内容，完了条件を整理する
- 設計案や選択肢を比較し，判断材料を提示する
- リポジトリ内のドキュメントを作成・更新する
- GitHub 上の開発運用を支援する
- Pull Request の変更全体をレビューする
- 既存ルールや決定事項との矛盾を検出する
- 人間の判断が必要な事項を明示する

ChatGPT は，重要な未決定事項を推測で確定したり，人間の明示的な許可なく Pull Request をマージしたりしません．

### Codex

Codex は，主に Issue と既存の設計・規約に基づく実装作業を担当します．

主な責務：

- 対象 Issue の範囲内でコードを実装・修正する
- 必要な Test を追加・更新する
- lint，format，type check，Test 等を実行する
- レビュー指摘に対応する
- 実装中に発見した問題や不明点を報告する
- 既存の設計・規約に沿って変更する

Codex は，Issue の範囲を独断で拡張せず，主要技術の導入，architecture の大幅な変更，Public API の破壊的変更等を独断で決定しません．

## 判断に迷った場合

既存のルール，Issue，関連ドキュメントから判断できない場合は，推測で恒久的な方針を作りません．

軽微で容易に戻せる実装判断は，既存方針と整合する範囲で進めることができます．

一方，長期的な影響を持つ判断，複数領域へ影響する判断，Public API やデータ互換性に影響する判断については，人間へ判断を求めます．

## 技術的な重要判断の記録

日常的な実装上の判断は，Issue または Pull Request に記録します．

長期的に参照する必要がある重要な技術判断は ADR（Architecture Decision Record）として記録します．

ADR の対象とする主な判断：

- 主要な framework，library，database，infrastructure の採用または置き換え
- application architecture や主要 layer の責務・境界
- authentication / authorization の基本方式
- Public API の versioning や互換性に関する基本方針
- database や永続化に関する重要な設計方針
- concurrency，cache，message queue 等の横断的な方式
- deployment や運用へ長期的な影響を与える技術判断
- 後から判断理由を確認する必要性が高い事項

以下のような判断は，原則として ADR を作成せず Issue または Pull Request に記録します．

- 局所的で容易に変更できる実装詳細
- formatter による整形
- 小規模な refactor
- 既存方針に従うだけの library 使用方法
- 一時的な調査結果

## ADR の運用

ADR は `docs/adr/` に保存します．

ファイル名は以下を基本とします．

```text
NNNN-short-title.md
```

例：

```text
0001-select-database.md
0002-api-versioning-policy.md
```

ADR には最低限，以下を記録します．

- Status
- Context
- Decision
- Consequences

Status は原則として以下を使用します．

- `Proposed`
- `Accepted`
- `Superseded`

検討中の内容は Issue で議論し，長期的な決定として確定した段階で ADR に反映します．

既存 ADR の判断を変更する場合は，過去の記録を削除して書き換えるのではなく，新しい ADR を作成し，旧 ADR を `Superseded` とします．
