オフラインCAD図面（PDF）検索・閲覧システム 仕様パッケージ
このZIPには以下が含まれます。
docs/requirements.md : 要件定義書（完全版）
docs/pwa_ui_spec.md : PWA（スマホ閲覧）UI仕様
docs/windows_box_impl_spec.md : Windows箱（cad-box-server）実装仕様化
api/openapi.yaml : 箱側API仕様（OpenAPI風）
schemas/*.json : メタデータ／マニフェスト／差分のJSON Schema
ops/apply-delta_spec.md : A/Bスロット差分適用（運用・手順）
ops/cli_spec.md : メンテツールCLI仕様
想定運用（四半期更新）:
メンテPCで import/split/build-index/release/make-delta を実行し差分ZIPを生成
差分ZIPを箱へ搬入して apply-delta により非稼働スロットへ適用→検証→切替
現場スマホは PWA で検索・閲覧（閲覧のみ）
