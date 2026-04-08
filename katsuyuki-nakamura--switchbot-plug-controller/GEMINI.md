## switchbot-plug-controller

> > SwitchBot Plug Mini を BLE 経由で Ubuntu Linux コンソールから操作する CLI ツール

# switchbot-plug-controller 開発ガイドライン

> SwitchBot Plug Mini を BLE 経由で Ubuntu Linux コンソールから操作する CLI ツール

## 1. 運用方針
- **言語**: 説明・コミットメッセージは日本語。スクリプト・コードは英語
- **コミット**: 明示指示がある場合のみ。一行目に日本語要約
- **作業開始前**: `git status` → テスト状態確認
- **作業管理**: タスク管理・進捗追跡・完了記録は GitHub Issues で行う（テンプレート: `.github/ISSUE_TEMPLATE/`）
- **Issue運用**: 作業開始時に対応 Issue を確認し、完了時はコメントで結果を記録して Close

## 2. アーキテクチャ概要

Python CLI アプリケーション。bleak ライブラリ経由で SwitchBot Plug Mini と BLE 通信を行う。

```
switchbot-plug-controller/
├── src/
│   └── switchbot_plug/
│       ├── __init__.py       # パッケージ初期化
│       ├── cli.py            # CLI エントリポイント (click)
│       ├── scanner.py        # BLE スキャン・Plug Mini 検出
│       ├── plug.py           # Plug Mini BLE 通信コア
│       └── protocol.py       # BLE プロトコル定数・パケット構築
├── tests/                    # pytest テスト
├── pyproject.toml            # プロジェクト定義・依存関係
└── docs/
    └── investigation/        # 調査結果
```

### BLE 通信フロー

```
CLI (click) → plug.py → bleak (BleakClient) → BlueZ D-Bus → BLE → Plug Mini
                                                                    ↓
                                               GATT Characteristic UUID:
                                               TX: cba20002-224d-11e6-9fb8-0002a5d5c51b
                                               RX: cba20003-224d-11e6-9fb8-0002a5d5c51b
```

### コマンド体系 (SwitchBot BLE API)

| 操作 | REQ パケット | RESP |
|---|---|---|
| 電源ON | `0x570f50010180` | `0x0180` |
| 電源OFF | `0x570f50010100` | `0x0100` |
| トグル | `0x570f50010280` | — |
| 状態取得 | `0x570f5101` | `0x01` + `0x00`(OFF)/`0x80`(ON) |

## 3. ビルド & テスト

### 環境セットアップ
```bash
# 仮想環境の作成・有効化
python3 -m venv .venv
source .venv/bin/activate

# 開発用インストール
pip install -e ".[dev]"
```

### テスト
```bash
# 全テスト実行
pytest

# カバレッジ付き
pytest --cov=switchbot_plug --cov-report=term-missing

# リント
ruff check src/ tests/

# 型チェック
mypy src/
```

- テストフレームワーク: pytest + pytest-asyncio + pytest-cov
- BLE通信のモック: `unittest.mock` で bleak の `BleakClient`/`BleakScanner` をモック
- **テストカバレッジ**: 変更点をカバーするテストが存在することを必ず確認。無ければ追加
- **🚨 テスト失敗時**: テストを安易に修正してパスさせるのは禁止。テストが正しいか実装が正しいかを先に判断

### CLI 使用例
```bash
switchbot-plug scan                      # 周囲の Plug Mini を検出
switchbot-plug on  --addr XX:XX:XX:XX    # 電源ON
switchbot-plug off --addr XX:XX:XX:XX    # 電源OFF
switchbot-plug toggle --addr XX:XX:XX:XX # トグル
switchbot-plug status --addr XX:XX:XX:XX # 状態取得
```

## 4. 技術スタック

| 項目 | 技術 | バージョン |
|---|---|---|
| 言語 | Python | 3.10+ |
| BLE通信 | bleak | 2.x |
| BLEバックエンド | BlueZ (D-Bus) | 5.55+ |
| CLI | click | 8.x |
| テスト | pytest + pytest-asyncio | — |
| リント | ruff | — |
| 型チェック | mypy | — |

## 5. BLE プロトコル参考資料

- **公式BLE API**: https://github.com/OpenWonderLabs/SwitchBotAPI-BLE
- **Plug Mini仕様**: https://github.com/OpenWonderLabs/SwitchBotAPI-BLE/blob/latest/devicetypes/plugmini.md
- **参考実装 (pySwitchbot)**: https://github.com/sblibs/pySwitchbot
- **調査結果**: `docs/investigation/001_ble_protocol_research.md`

### デバイス識別
- デバイスタイプ: `g` (0x67)
- Company ID: `0x0969`
- Service UUID: `0xfd3d`
- **暗号化: 不要** (Lockと異なりPlug Miniはキー不要)

## 6. プロジェクト固有の注意事項

- **BLE権限**: Linux で BLE スキャンするには `sudo` またはユーザーを `bluetooth` グループに追加が必要な場合がある
- **asyncio**: bleak は asyncio ベース。CLI エントリポイントで `asyncio.run()` を使用
- **接続タイムアウト**: BLE接続は不安定になりえるため、適切なリトライ・タイムアウト処理を実装
- **エラーハンドリング**: デバイス未検出・接続失敗・コマンド失敗時にユーザーフレンドリーなメッセージを表示
- **テストでのBLEモック**: 実際のBLEデバイスなしでテストできるよう、bleak をモックする設計を採用

## 7. 関連ドキュメント
- **調査結果**: `docs/investigation/` — BLE プロトコル調査
- **AIコードレビュー**: `.github/ai-workflow/docs/for-agents/CODE_REVIEW_GUIDELINES.md`, `.github/ai-workflow/project/review.md`
- **マルチエージェント**: `.github/ai-workflow/` (フレームワーク一式)
- **パフォーマンス**: `.github/ai-workflow/project/performance.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/katsuyuki-nakamura)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/katsuyuki-nakamura)
<!-- tomevault:4.0:gemini_md:2026-04-07 -->
