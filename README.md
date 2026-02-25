# オフラインCAD図面（PDF）検索・閲覧システム 仕様パッケージ

このREADMEは、**オフライン環境でCAD図面（PDF）を検索・閲覧する運用を、1枚で導入できる手順書**として整理したものです。  
仕様ファイル本体は `cad_offline_viewer_spec/` 配下にあります。

---

## 0. この手順書でできること

- メンテPCでの準備（取り込み・索引生成・リリース・差分作成）
- 箱（Windows）への差分適用（A/Bスロット）
- 現場スマホでの閲覧開始
- 最低限の確認項目とトラブル時の切り分け

---

## 1. 全体像（最短フロー）

1. **メンテPC**で PDF群を取り込み、リリース/差分ZIPを作る
2. 差分ZIPを**箱（Windows）**へ搬入
3. 箱で `apply-delta` を実行し、**非稼働スロットへ適用→検証→切替**
4. 現場スマホは箱の `/app` を開いて検索・閲覧（オフライン）

> 役割分担
> - メンテPC: データ整備・差分作成
> - 箱: API/PWA配信、A/Bスロット運用
> - スマホ: 閲覧専用クライアント

---

## 2. 必要環境

### 2.1 メンテPC（必須）
- Git
- Python 3.10+（JSON検証、簡易サーバー）
- Ruby 3.x（OpenAPI YAML検証）
- `cadmaint`（メンテツール。`init/import/split/review/build-index/release/make-delta/validate/report/retry`）

### 2.2 箱（Windows）
- `cad-box-server`（Windowsサービス）
- `apply-delta.ps1`（差分適用）
- リポジトリ領域: 例 `D:\CadRepo`
- スロット運用: `A/B` + `active_slot.txt`

### 2.3 現場スマホ
- iOS Safari / Android Chrome
- 箱と同一ローカルWi-Fiに接続できること

---

## 3. インストール例（PC）

### Windows
- Git: `winget install --id Git.Git -e`
- Python: `winget install --id Python.Python.3.12 -e`
- Ruby: `winget install --id RubyInstallerTeam.RubyWithDevKit.3.2 -e`

### macOS
- `brew install git python ruby`

### Ubuntu/Debian
- `sudo apt update`
- `sudo apt install -y git python3 ruby`

---

## 4. 仕様パッケージ取得と確認

```bash
git clone <このリポジトリのURL>
cd zumen-2
```

### 4.1 UIモック表示（任意）
```bash
python -m http.server 4173
```
- ブラウザで `http://127.0.0.1:4173/cad_offline_viewer_spec/docs/pwa_ui_mock.html` を開く

### 4.2 仕様ファイル検証（任意）
```bash
for f in cad_offline_viewer_spec/schemas/*.json; do
  python -m json.tool "$f" >/dev/null
  echo "OK: $f"
done

ruby -ryaml -e 'YAML.load_file("cad_offline_viewer_spec/api/openapi.yaml"); puts "openapi.yaml: OK"'
```

---

## 5. メンテPC作業手順（本番運用）

以下は四半期更新などの標準フローです。

### 5.1 初期化
```bash
cadmaint init
```

### 5.2 PDF取り込み
```bash
cadmaint import --src <PDF格納ディレクトリ>
```

### 5.3 Slice正規化（分割）
```bash
cadmaint split
```

### 5.4 目視確認・補正
```bash
cadmaint review
```

### 5.5 索引生成
```bash
cadmaint build-index
```

### 5.6 リリース作成
```bash
cadmaint release --label <YYYY.QNなど>
```

### 5.7 差分ZIP作成
```bash
cadmaint make-delta --from <前回リリース> --to <今回リリース>
```

### 5.8 妥当性確認
```bash
cadmaint validate
cadmaint report
```

> 失敗時は `failures.csv`（timestamp, command, step, entity, id, path, error_code, message, retryable）を確認し、再実行可能な項目を `cadmaint retry` で処理します。

---

## 6. 箱（Windows）への適用手順

差分ZIPを箱へ搬入し、**必ず非稼働スロットに適用**します。

### 6.1 事前確認
- `cad-box-server` が稼働中であること
- 現在の `active_slot.txt`（AまたはB）を確認
- 差分ZIP内に次があること:
  - `delta_manifest.json`
  - `checksums.txt`
  - `deleted.json`
  - `added/`
  - `updated/`
  - `report.csv`

### 6.2 差分適用（概略）
1. `active_slot.txt` を読み、非稼働側をターゲットに決定
2. ターゲットスロットを初期化
3. 稼働中スロット内容をターゲットへコピー
4. `added/updated` を反映し、`deleted.json` に従って削除
5. `checksums` / `manifest` / （任意）検索スモークテストで検証
6. `active_slot.txt` をアトミックに切替

### 6.3 ロールバック
- 問題発生時は `active_slot.txt` を元スロットへ戻すことで即時復旧（設計上）

---

## 7. スマホ利用開始手順（現場）

1. 現場ローカルWi-Fiに接続
2. ブラウザで `http://<箱のIPまたはホスト>/app` を開く
3. 検索語を入力 → 結果一覧 → 図面詳細 → PDF閲覧
4. 必要に応じてヒットページへジャンプ

### 最低限の受入確認
- `/app` が開く
- `/api/v1/version` が応答する
- `/api/v1/search` で結果が返る
- PDFがRange配信され、ズーム/パンが実用速度で動作する

---

## 8. 運用チェックリスト（更新時）

- [ ] メンテPCで `validate` / `report` まで完了
- [ ] 差分ZIPの必須ファイルが揃っている
- [ ] 箱で非稼働スロットへ適用した
- [ ] 切替前に検索スモークテストを実施した
- [ ] 切替後に `/app` / `/api/v1/version` / `/api/v1/search` を確認した
- [ ] 必要時ロールバック手順を即実行できる状態にある

---

## 9. つまずきやすいポイント

- `python` が無い（Windows）
  - `py -m http.server 4173` を試す
- ポート競合
  - `python -m http.server 8080` などへ変更
- 文字化け
  - ターミナル/エディタをUTF-8に統一
- 差分適用後に検索不可
  - `checksums.txt` と `delta_manifest.json` の一致、`active_slot.txt` の切替結果、検索インデックス配置を確認

---

## 10. よくある質問（Q&A）

### Q1. まず何から始めればいいですか？
**A.** 最短では次の順序です。  
1) メンテPCに必要ソフト（Git/Python/Ruby/cadmaint）を導入  
2) PDFを `cadmaint import` で取り込み  
3) `build-index` と `release` / `make-delta` を実行  
4) 差分ZIPを箱へ適用し、`/app` をスマホで開いて確認

### Q2. 現場スマホで表示するURLはどれですか？
**A.** `http://<箱のIPまたはホスト>/app` です。箱と同じローカルWi-Fiに接続してください。

### Q3. 検索できないときは最初に何を見ればよいですか？
**A.** 次を順番に確認してください。  
- 箱サービス（`cad-box-server`）が稼働しているか  
- `active_slot.txt` の切替が想定通りか  
- 差分適用時に `checksums.txt` / `delta_manifest.json` の検証が通っているか  
- `/api/v1/version` と `/api/v1/search` に応答があるか

### Q4. PDFが重い/開かない場合はどうしますか？
**A.** Range配信前提なので、通信品質と箱側の配信状態を確認します。  
- スマホが箱のWi-Fiに安定接続できているか  
- `/api/v1/slices/{id}/pdf` が200/206を返すか  
- 切替直後ならインデックス/メタ配置の整合を再確認

### Q5. 更新に失敗した場合、すぐ戻せますか？
**A.** はい。A/Bスロット運用のため、問題時は `active_slot.txt` を元に戻すことでロールバック可能です（設計上）。

### Q6. CLI処理で失敗が出たらどうすればよいですか？
**A.** `failures.csv` を確認し、`retryable` な項目を `cadmaint retry` で再実行してください。再発する場合は対象PDFやメタ情報を `review` で修正します。

### Q7. 非エンジニアでも日常運用できますか？
**A.** できます。日常運用では「差分ZIPの受け取り→箱へ適用→確認」の定型手順が中心です。READMEの「6. 箱への適用」「7. スマホ利用開始」「8. 運用チェックリスト」をそのまま実施してください。

### Q8. このREADMEだけで進めてよいですか？
**A.** はい。通常の導入・更新・確認はこのREADMEの手順だけで実施できます。詳細仕様が必要な場合のみ末尾の参照ファイルへ進んでください。

---

## 11. 収録ファイル一覧（参照元）

- `cad_offline_viewer_spec/docs/requirements.md` : 要件定義書（完全版）
- `cad_offline_viewer_spec/docs/pwa_ui_spec.md` : PWA（スマホ閲覧）UI仕様
- `cad_offline_viewer_spec/docs/pwa_ui_mock.html` : PWA UI実画面イメージ（静的モック）
- `cad_offline_viewer_spec/docs/windows_box_impl_spec.md` : Windows箱（cad-box-server）実装仕様
- `cad_offline_viewer_spec/api/openapi.yaml` : 箱側API仕様（OpenAPI）
- `cad_offline_viewer_spec/schemas/*.json` : メタデータ／マニフェスト／差分のJSON Schema
- `cad_offline_viewer_spec/ops/apply-delta_spec.md` : A/Bスロット差分適用（運用・手順）
- `cad_offline_viewer_spec/ops/cli_spec.md` : メンテツールCLI仕様

