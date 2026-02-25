# A/Bスロット差分適用仕様（箱側）

## 目的
差分ZIPを非稼働スロットへ適用し、検証後に切替。失敗時は稼働中を維持し、即時ロールバック可能にする。

## 必須ファイル
- delta_manifest.json
- checksums.txt
- deleted.json
- added/ updated/ report.csv

## 適用手順（要点）
1) active_slot.txt を読み、非稼働スロットをターゲットにする
2) ターゲットスロットを初期化
3) 稼働中スロットをターゲットへコピー
4) added/updated を上書き、deleted.json に従い削除
5) checksums検証、version/manifest検証、検索スモークテスト（任意）
6) active_slot.txt をアトミックに切替

