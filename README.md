# オフラインCAD図面（PDF）検索・閲覧システム 仕様パッケージ

本リポジトリは、オフライン環境でCAD図面（PDF）を検索・閲覧するための**仕様書一式**です。  
実体ファイルは `cad_offline_viewer_spec/` 配下にあります。

## 収録ファイル

- `cad_offline_viewer_spec/docs/requirements.md` : 要件定義書（完全版）
- `cad_offline_viewer_spec/docs/pwa_ui_spec.md` : PWA（スマホ閲覧）UI仕様
- `cad_offline_viewer_spec/docs/pwa_ui_mock.html` : PWA UI実画面イメージ（静的モック）
- `cad_offline_viewer_spec/docs/windows_box_impl_spec.md` : Windows箱（cad-box-server）実装仕様
- `cad_offline_viewer_spec/api/openapi.yaml` : 箱側API仕様（OpenAPI）
- `cad_offline_viewer_spec/schemas/*.json` : メタデータ／マニフェスト／差分のJSON Schema
- `cad_offline_viewer_spec/ops/apply-delta_spec.md` : A/Bスロット差分適用（運用・手順）
- `cad_offline_viewer_spec/ops/cli_spec.md` : メンテツールCLI仕様

## 想定運用（四半期更新）

1. メンテPCで `import/split/build-index/release/make-delta` を実行し差分ZIPを生成
2. 差分ZIPを箱へ搬入して `apply-delta` により非稼働スロットへ適用→検証→切替
3. 現場スマホは PWA で検索・閲覧（閲覧のみ）

---

## パソコン上での導入手順（仕様確認用）

本章は「この仕様書パッケージをPCで確認・レビューする」ための手順です。  
アプリ本体のビルドではなく、**仕様閲覧 / UIモック確認 / 形式検証**を対象にしています。

### 1) 必要ソフトウェア

#### 必須
- **Git**（リポジトリ取得）
- **Python 3.10+**（ローカルHTTPサーバー起動、JSON検証）
- **Ruby 3.x**（`openapi.yaml` の構文確認）

#### 任意（あると便利）
- **VS Code**（Markdown/JSON/YAMLの閲覧）
- **Node.js 20+**（将来的にlintや表示確認ツールを追加する場合）

### 2) インストール例

#### Windows
- Git: `winget install --id Git.Git -e`
- Python: `winget install --id Python.Python.3.12 -e`
- Ruby: `winget install --id RubyInstallerTeam.RubyWithDevKit.3.2 -e`

#### macOS
- Homebrew未導入の場合は先にHomebrewを導入
- Git/Python/Ruby: `brew install git python ruby`

#### Ubuntu/Debian
- `sudo apt update`
- `sudo apt install -y git python3 ruby`

### 3) リポジトリ取得

```bash
git clone <このリポジトリのURL>
cd zumen-2
```

### 4) UIモック（実画面イメージ）の表示

`pwa_ui_mock.html` はブラウザで直接開いても表示できますが、相対パスや検証時の再現性のためHTTP配信を推奨します。

```bash
python -m http.server 4173
```

ブラウザで以下を開きます:

- `http://127.0.0.1:4173/cad_offline_viewer_spec/docs/pwa_ui_mock.html`

### 5) 仕様ファイルの検証（任意だが推奨）

#### JSON Schema の構文検証

```bash
for f in cad_offline_viewer_spec/schemas/*.json; do
  python -m json.tool "$f" >/dev/null
  echo "OK: $f"
done
```

#### OpenAPI YAML の構文検証

```bash
ruby -ryaml -e 'YAML.load_file("cad_offline_viewer_spec/api/openapi.yaml"); puts "openapi.yaml: OK"'
```

### 6) よくあるつまずき

- `python` コマンドが見つからない場合:
  - Windowsでは `py -m http.server 4173` も試してください。
- ポート4173が使用中の場合:
  - `python -m http.server 8080` のように別ポートへ変更してください。
- 文字化けする場合:
  - エディタとターミナルの文字コードをUTF-8に設定してください。

