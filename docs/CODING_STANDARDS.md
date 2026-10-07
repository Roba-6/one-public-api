# One Public API コーディング規約

この文書は，One Public API（OPA）のコード品質，可読性，保守性，安全性を一定に保つためのコーディング規約を定めます．

OPA は Python / FastAPI を中心とした API サーバーとして開発します．本規約では，Python の一般的な慣習を尊重しながら，API，ドメインロジック，外部サービス，永続化処理などの責務を明確に分離し，将来の変更に対応しやすい実装を目指します．

対象：

- 人間の開発者
- Codex
- ChatGPT
- その他の AI コーディングエージェント

コード変更を行う場合は，本規約と `AGENTS.md` の両方に従います．

## 1．基本方針

OPA の実装では，以下を優先します．

1．読みやすさ  
2．安全性  
3．単純さ  
4．テストしやすさ  
5．将来の変更しやすさ

短いコードより，意図が明確なコードを優先します．

過度な抽象化や，将来使うかもしれないという理由だけの仕組みは追加しません．

Framework や library の機能を使うこと自体を目的にせず，OPA に必要な最小限の構造を選択します．

### 1.1．疎結合

実装は可能な範囲で疎結合にします．

原則：

- API layer は DB や外部サービスの具体実装へ直接依存しすぎない
- ドメインロジックを FastAPI 固有の機能へ依存させない
- HTTP request / response と内部の業務ロジックを分離する
- 外部 API や永続化処理は，必要に応じて service，repository 等の境界越しに利用する
- module 間では必要最小限の interface だけを共有する
- 変更理由が異なる責務を同じ module へまとめない
- Test で外部依存を差し替えられる構造を優先する
- 循環 import を作らない

ただし，疎結合のためだけに不要な interface や layer を追加しません．

抽象化を追加する場合は，責務分離，交換可能性，Test 容易性などの具体的な利点があることを確認します．

## 2．使用言語

基本言語は Python とします．

原則：

- Python の型 annotation を積極的に使用する
- public な function，method，class では型を明確にする
- 型推論で十分な局所変数に冗長な annotation を付けない
- `Any` は原則として使用しない
- `# type: ignore` を安易に使用しない
- formatter で自動整形できる規則は，formatter の設定を正本とする
- 外部入力は型 annotation だけで信用せず，Pydantic 等による runtime validation を行う

悪い例：

```python
def save_user(user: Any):
    ...
```

良い例：

```python
from pydantic import BaseModel


class User(BaseModel):
    id: str
    name: str


def save_user(user: User) -> None:
    ...
```

## 3．Python style

Python の一般的な慣習に従い，PEP 8 と PEP 257 の考え方を基本とします．

自動整形や lint で管理できる規則は，手作業による独自ルールを増やさず，採用した tool の設定を優先します．

原則：

- indentation は space 4文字
- tab による indentation は使用しない
- 不要な括弧や複雑な1行表現を避ける
- 複数処理を `;` で1行にまとめない
- 読みやすさを損なう過度な one-liner を避ける

## 4．命名規則

### 4.1．変数・関数・method

`snake_case` を使用します．

### 4.2．class

`PascalCase` を使用します．

### 4.3．定数

module 全体で固定される定数は `UPPER_SNAKE_CASE` を使用します．

### 4.4．module・file

Python module の file 名は `snake_case.py` とします．

意味のない `utils.py`，`helpers.py` に処理を集約しすぎません．可能な限り責務が分かる file 名を使用します．

## 5．Boolean の命名

Boolean 値は意味が分かる名前にします．

推奨：

- `is_`
- `has_`
- `can_`
- `should_`

## 6．関数

function は1つの責務に集中させます．ただし，細かく分割しすぎて処理の流れが読みにくくなる場合は分割しません．

## 7．関数の長さ

厳密な行数制限は設けません．複数の責務，条件分岐の増加，深い nesting，Test の書きにくさなどがある場合は分割を検討します．

## 8．引数

function の引数が増えすぎる場合は，責務が大きすぎないか確認します．関連する値をまとめる意味が明確な場合は，Pydantic model，dataclass 等を使用できます．

## 9．戻り値

戻り値の意味を明確にします．複数の意味を持つ値を tuple の位置だけで表現することは避けます．必要に応じて dataclass，Pydantic model，専用 class 等を使用します．

## 10．コメント

コメントは‘何をしているか’ではなく‘なぜそうしているか’を中心に記述します．コードを読めば分かる内容はコメントしません．

## 11．docstring

外部から使用される function，class，module や，意図が読み取りにくい処理には必要に応じて docstring を記述します．単純な private function に形式的な docstring を大量に追加しません．

## 12．TODO

曖昧な TODO は禁止します．可能な限り Issue 番号を付けます．恒久的に残す説明を TODO として記述しません．

```python
# TODO(#123): rate limit を Redis backend へ切り替える
```

## 13．import

import は原則として以下の順に整理します．

1．Python standard library  
2．third-party library  
3．project 内 module

import 順序を自動整形できる場合は，採用した formatter / linter の設定を優先します．function 内 import は，循環依存回避など明確な理由がある場合を除いて避けます．

## 14．深い相対 import を避ける

過度に深い relative import は避け，project root を基準にした absolute import を基本とします．

## 15．FastAPI

FastAPI の route handler は HTTP layer の責務に集中させます．

route handler の主な責務：

- request の受け取り
- dependency の解決
- input validation
- application / service layer の呼び出し
- HTTP response への変換

大量の業務ロジックを route handler に直接記述しません．

推奨構造：

```text
router
↓
service / application logic
↓
repository / external client
↓
database / external API
```

## 16．Router

Router は resource や責務ごとに分割します．1つの巨大な router file にすべての endpoint を集約しません．ただし，endpoint 数が少ない段階で過度に細分化しません．

## 17．Dependency Injection

FastAPI の `Depends` は，request context，authentication，service の取得など，request lifecycle と関係する dependency に使用します．業務ロジックそのものを `Depends` の連鎖へ埋め込みすぎません．Test で dependency override が可能な構造を維持します．

## 18．Pydantic model

外部との境界では Pydantic model を使用して入力と出力を明確にします．用途の異なる model を1つにまとめすぎません．DB model をそのまま API response model として公開することは避け，内部構造と public API contract を分離します．

## 19．入力 validation

外部から入る値は必ず検証します．

対象：

- request body
- path parameter
- query parameter
- header
- cookie
- Webhook
- 外部 API response
- import data
- environment variable

Pydantic model や FastAPI の validation で表現できる制約は，可能な限り境界で検証します．業務上のルールは service / domain layer で検証します．

## 20．API

API の request / response は明確な schema を定義します．response body を場当たり的な dictionary で返しません．同じ意味の response structure を endpoint ごとに不必要に変えません．

## 21．Public API contract

公開 API の field 名，HTTP status，error code，response structure は contract として扱います．内部 refactor の都合だけで不用意に変更しません．

変更する場合は，以下を確認します．

- backward compatibility
- API version
- documentation
- Test
- client への影響
- migration path

## 22．HTTP status code

HTTP status code は意味に合わせて選択します．

- `200 OK`：通常の取得・更新成功
- `201 Created`：resource 作成成功
- `204 No Content`：body を返さない成功
- `400 Bad Request`：一般的な不正 request
- `401 Unauthorized`：authentication が必要または失敗
- `403 Forbidden`：authentication 済みだが権限がない
- `404 Not Found`：resource が存在しない
- `409 Conflict`：resource の状態競合
- `422 Unprocessable Entity`：validation error
- `429 Too Many Requests`：rate limit 超過
- `500 Internal Server Error`：想定外の server error
- `502 Bad Gateway`：upstream service の異常
- `503 Service Unavailable`：一時的に利用不能
- `504 Gateway Timeout`：upstream timeout

status code の使い分けを endpoint ごとにばらつかせません．

## 23．Error handling

error を握りつぶしません．必要な error は記録し，適切な domain / application error へ変換します．ただし，log へ secret や sensitive data を含めません．

## 24．Exception

`Exception` を application 全体の通常制御に使用しません．意味のある error class を定義します．FastAPI の `HTTPException` を domain / repository layer へ持ち込みません．HTTP error への変換は API layer または共通 exception handler で行います．

## 25．Error response

client に返す error と内部 exception を分離します．error response は可能な限り一貫した structure を使用します．内部 exception message，stack trace，SQL error 等をそのまま client へ返しません．

## 26．Logging

標準の `logging` または project で採用した logging framework を使用します．`print()` を application logging の代わりに使用しません．

以下を log へ出しません．

- password
- API key
- access token
- refresh token
- authorization header
- cookie
- secret
- private credential
- request / response の sensitive data

## 27．Request ID

request を追跡できるよう，一意な request ID を利用できる構造を推奨します．request ID は log や downstream request の tracing に利用できます．client から渡された request ID を信用して security 判断には使用しません．

## 28．Security

以下を禁止します．

- API key，token，password 等を repository に commit する
- authentication を client 側の情報だけで信用する
- authorization check を省略する
- validation を回避して処理を続ける
- secret を source code に直接記述する
- exception response に内部情報を露出する
- user input をそのまま SQL や shell command へ埋め込む
- TLS 検証を理由なく無効化する

security 上重要な処理では，正常系だけでなく拒否される経路も Test します．

## 29．環境変数・設定

configuration は source code へ直接埋め込まず，環境変数等から取得します．Pydantic Settings 等を使用して application 起動時に validation できる構造を推奨します．application code の各所で直接 `os.environ` を読みません．設定の取得を1か所に集約します．

## 30．Database

Database を使用する場合は，永続化処理を HTTP layer へ直接記述しません．

原則：

- query は repository 等の永続化責務へまとめる
- transaction 境界を明確にする
- N+1 query を放置しない
- schema 変更は migration で管理する
- application 起動時に自動で破壊的 schema 変更を行わない
- raw SQL を使用する場合は parameter binding を使用する

## 31．Database 命名規則

SQL database では原則として `snake_case` を使用します．Python 側も基本的に `snake_case` のため，不要な命名変換を増やしません．

## 32．Migration

DB schema の変更は migration file で管理します．rollback 可能性，既存 data への影響，NOT NULL 追加時の既存 row，index 作成時の負荷，column rename / delete の compatibility，production data 量を考慮します．

## 33．Repository

Repository を採用する場合は，永続化技術を application logic から分離する役割を持たせます．Repository に HTTP response 生成や FastAPI 固有処理を入れません．すべての小さな処理に repository interface を作る必要はありません．

## 34．Domain / Service

OPA 固有の business rule は FastAPI router ではなく domain または service layer に置きます．

例：

- API key の有効性判定
- rate limit 判定
- upstream API 選択
- request normalization
- response normalization
- access policy
- retry policy

## 35．ID

ID の形式は resource ごとに明確にします．UUID を採用する場合でも，ID を知っていること自体を authorization として扱いません．

## 36．日付・時刻

日付・時刻は timezone を明確に扱います．

原則：

- datetime は timezone-aware にする
- server 内部では UTC を基本とする
- naive datetime を安易に使用しない
- 日付だけの値と日時を区別する

```python
from datetime import UTC, datetime

now = datetime.now(UTC)
```

API contract では ISO 8601 / RFC 3339 形式を基本とします．

## 37．Enum

意味のある状態を複数の Boolean で表現しすぎません．状態が排他的である場合は `Enum` 等を検討します．

## 38．None

`None` の意味を明確にします．値が存在しない，未指定である，lookup の結果が見つからない，などを混同しません．特に API request では field omitted と `null` の意味が異なる場合があります．

## 39．非同期処理

FastAPI では async を必要な箇所で使用します．

原則：

- async I/O には `async def` を使用する
- blocking I/O を event loop 上で直接実行しない
- CPU-bound な重い処理を async 化するだけで解決したと考えない
- coroutine は適切に `await` する
- 意図しない fire-and-forget を避ける
- background task の失敗経路を考慮する

## 40．外部 API

外部 API 呼び出しは dedicated client 等へまとめます．route handler 内へ HTTP request 処理を大量に直接記述しません．

外部 API では最低限以下を考慮します．

- timeout
- retry
- rate limit
- authentication
- error response
- schema validation
- logging
- circuit breaker の必要性
- idempotency

## 41．Timeout

外部 I/O に無制限 timeout を使用しません．HTTP client，DB connection 等には適切な timeout を設定します．timeout 値を application の各所へ magic number として散在させません．

## 42．Retry

retry は一時的な失敗に限定します．validation error や authentication failure のような，retry しても改善しない error は原則 retry しません．exponential backoff や jitter の必要性を考慮します．

## 43．Rate limit

rate limit を実装する場合は，判定単位と window を明確にします．複数 instance で動作する可能性がある場合，memory 上だけの counter で十分か確認します．

## 44．Cache

cache は性能上の必要性が確認されてから導入します．cache を利用する場合は，key，TTL，invalidation，stale data の許容範囲，failure 時の挙動を明確にします．

## 45．Test

以下は優先的に Test します．

- domain / service logic
- validation
- authentication
- authorization
- API contract
- error handling
- external API failure
- timeout
- retry
- rate limit
- date / timezone 境界
- database constraint
- regression bug

不具合修正では，可能な限り再現 Test を先に追加します．

## 46．Test の種類

目的に応じて Unit Test，Integration Test，API Test を使い分けます．FastAPI の `TestClient` または async client 等，採用構成に適した tool を使用します．

## 47．Test の命名

Test 名は，何を検証しているか分かる名前にします．Python identifier として扱いやすいため，基本的には英語の `snake_case` を推奨します．

## 48．Mock

必要以上に mock しません．特に pure な domain logic は，可能な限り実 data structure で Test します．主に external API，cloud service，clock，random generator，network 等の境界を mock / fake の対象とします．

## 49．Test fixture

pytest fixture は共有する価値がある setup に使用します．fixture を多段階に依存させすぎて，Test の前提が読み取れなくなる構成は避けます．

## 50．外部 library

library 追加前に確認します．

- Python standard library で十分ではないか
- FastAPI / Pydantic の既存機能で実現できないか
- 既存 dependency で実現できないか
- 現在も maintenance されているか
- security issue がないか
- license
- dependency tree への影響
- project の Python version を support しているか

## 51．依存関係

version は理由なく頻繁に最新版へ更新しません．dependency update は，可能な限り feature 変更と分離します．major update では breaking change と migration guide を確認します．

## 52．File size

巨大な file を避けます．複数の責務，HTTP layer と business logic の混在，DB query と API response 生成の混在などがある場合は分割を検討します．行数だけを基準に分割しません．

## 53．重複 code

単純な重複が2箇所あるだけなら，すぐ共通化しません．同じ意味の logic が複数箇所に現れる，または明確な共通概念が存在する場合に共通化を検討します．‘DRY のための DRY’を避けます．

## 54．早すぎる最適化

performance 問題が確認される前に，複雑な optimization を行いません．まず測定し，必要な箇所だけ改善します．

## 55．Performance

API server では必要に応じて response time，DB query count，external API latency，memory usage，request throughput，error rate を測定します．optimization 前後で測定可能な状態を優先します．

## 56．破壊的変更

既存の public API，DB schema，configuration，error code 等を変更する場合は，影響範囲を確認します．必要に応じて migration，Test，API contract，OpenAPI documentation，README，設計書，client documentation を更新します．

## 57．OpenAPI

FastAPI が生成する OpenAPI schema も public API の一部として扱います．endpoint summary，request schema，response schema，status code，error response，parameter description を意識します．

## 58．API documentation

public endpoint を追加または変更した場合は，必要に応じて API documentation も更新します．code と documentation の内容が矛盾しない状態を維持します．

## 59．開発運用

branch，commit，Pull Request，review，merge 等の開発運用は，[開発への参加と運用ルール](../CONTRIBUTING.md) を正本とします．

本規約では，開発運用ルールを重複して定義しません．

## 60．AI コーディングエージェント

AI コーディングエージェントの作業開始時の確認事項，GitHub アカウントの使い分け，参照すべき正本等は，[AGENTS.md](../AGENTS.md) に従います．

本規約では，AI エージェント向けの運用ルールを重複して定義しません．

## 61．日本語の文書・コメント表記

日本語で documentation，comment，説明文等を記述する場合は，`CONTRIBUTING.md` の句読点と記号の規則に従います．

日本語文章中では，原則として `．`，`，`，`‘ ’`，`“ ”`，`（ ）` を使用します．ただし，code，identifier，URL，CLI command，JSON，SQL，regular expression，inline code 等は本来の記法を変更しません．

## 62．完了条件

code 変更完了時の品質チェックと CI に関する正式な方針は，[テストと CI/CD](testing-ci-cd.md) に従います．

実行していない check を‘成功’と報告しません．

## 63．最終原則

OPA では，

> 賢そうな code より，次に読む人が理解できる code

を優先します．

そして，

> 今はシンプルに，将来は交換可能に

という設計原則を，code level でも維持します．