# テストと CI/CD

OPA の Test，品質チェック，CI，CD に関する基本方針を定義します．

個々のコードに対する Test の書き方や対象は [コーディング規約](CODING_STANDARDS.md) を参照します．

## 基本方針

変更は，人手による確認だけに依存せず，自動化できる品質チェックを CI で実行します．

CI は Pull Request をレビューするための前提情報として使用し，CI が成功していることだけをもってレビュー完了とはしません．

## 採用する基本ツール

Python / FastAPI の品質チェックには，当面以下を使用します．

| 用途 | Tool |
| --- | --- |
| lint | Ruff |
| format | Ruff |
| type check | mypy |
| Test | pytest |

追加の tool が必要になった場合は，目的と既存 tool で代替できない理由を確認したうえで導入します．

## CI

GitHub Actions を CI に使用します．

Pull Request では，変更内容に応じて最低限以下を実行します．

```bash
ruff check .
ruff format --check .
mypy .
pytest
```

上記の command が project 構成の確定によって変更される場合は，本書と実際の CI workflow を同じ変更で更新します．

CI は原則として以下のタイミングで実行します．

- Pull Request の作成・更新時
- `main` への変更時

Pull Request のマージに必要な status check は，GitHub Rulesets と CI workflow が整備された段階で，上記の品質チェックを基準として設定します．

実行されていない check を成功扱いしません．

## Test

Test は変更による回帰を防ぎ，仕様を継続的に確認できる状態を作るために実施します．

不具合修正では，可能な範囲で不具合を再現する Test を追加してから修正します．

Test の分類や具体的な書き方は [コーディング規約](CODING_STANDARDS.md) を参照します．

## CI の失敗

必須 CI が失敗している Pull Request は，原則としてマージ可能とは扱いません．

失敗が変更内容と無関係であることが確認できる場合も，原因と判断を Pull Request に記録し，必要な対応を明確にします．

CI を通すことだけを目的として Test や品質チェックを無効化してはいけません．

## CD

現時点では deployment 先と release 方法が未決定のため，自動 deployment を必須とはしません．

deployment 方針が決定した後に，環境，承認条件，rollback，secret 管理等を含めて CD の方針を更新します．

CI の成功だけを条件として production へ無条件に自動 deployment する仕組みは，運用方針が決まるまで導入しません．
