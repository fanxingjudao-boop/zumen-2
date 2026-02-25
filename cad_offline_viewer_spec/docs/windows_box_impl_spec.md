# Windows箱 実装仕様（cad-box-server）

## 0. 実装方針
- ASP.NET Core Minimal API (.NET)
- Self-contained publish推奨
- Windowsサービス（UseWindowsService）
- RepoRoot: D:\CadRepo
- active_slot.txt により A/B スロット切替
- PWAを /app で静的配信
- APIを /api/v1 で提供
- PDF配信はRange対応

## 1. コンポーネント
- cad-box-server（Windowsサービス）
  - PWA配信、検索API、PDF/サムネ配信
  - active_slot監視→無停止切替
- repo（slots A/B）
- apply-delta.ps1（差分適用）

## 2. 受入基準（箱）
- サービス自動起動
- /app がスマホで開ける
- /api/v1/version が返る
- /api/v1/search が実用速度
- PDFがRangeで配信され、ズーム/パンが快適
- A/B切替とロールバックが成立

