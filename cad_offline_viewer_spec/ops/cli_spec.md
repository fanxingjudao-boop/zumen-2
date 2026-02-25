# メンテツール CLI仕様（cadmaint）

## 目的
PDF（単体/マルチ混在）を取り込み、Slice正規化→メタ整備→索引生成→リリース→差分生成を行う。

## コマンド
- init
- import
- split
- review
- build-index
- release
- make-delta
- validate
- report
- retry

## 失敗ログ（failures.csv）
timestamp, command, step, entity, id, path, error_code, message, retryable

