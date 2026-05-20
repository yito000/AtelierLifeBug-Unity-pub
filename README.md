# AtelierLifeBug Unity

https://github.com/yito000/AtelierLifeBug-Unity-pub/releases/tag/release

Unity 6.4 を使った、映像制作・映像内ゲーム撮影用のプロジェクトです。

このリポジトリは、完成した単体ゲームではなく、動画素材として使う小規模な「見せゲーム」を Unity 上で管理・再生・録画するための制作環境です。`Assets/Showcase` 以下の共通基盤で、各見せゲームを `ShowGameDefinition` と `TakePreset` として登録し、Editor の Game Launcher からテイクを選んで再生・撮影します。

## 現在の内容

調査対象のリリースフォルダには、Unity プロジェクト本体と CCGS / Codex / Claude 用の制作支援ドキュメントが同梱されています。

- Unity プロジェクト: `src/AtelierLifeBug-Unity`
- Showcase 共通 Runtime / Editor tooling
- Recorder 用プリセット
- `SLG_MoveOnly` の実装済みサンプル
- `RPG_BattleOnly` の Wizard 生成済みスタブ
- ADR、QA、制作ログ、エージェント運用ドキュメント

一部の QA ドキュメントや過去ログには `ADV_MessageOnly` への参照がありますが、調査した `Assets/Showcase/Games` と `ShowGameRegistry.asset` には `SLG_MoveOnly` と `RPG_BattleOnly` の 2 件だけが含まれています。

## 技術スタック

| 項目 | 内容 |
| --- | --- |
| Engine | Unity 6.4.7f1 (`6000.4.7f1`) |
| Language | C# |
| Render Pipeline | Universal Render Pipeline 17.4.0 |
| Capture | Unity Recorder 5.1.6 |
| Input | Unity Input System 1.19.0 |
| Camera / Sequencing | Cinemachine 3.1.6, Timeline 1.8.12 |
| UI | uGUI 2.0.0, TextMesh Pro |
| Narrative Package | Naninovel 1.21.260328 (embedded package) |
| Test | Unity Test Framework 1.6.0, NUnit |
| Editor Automation | Showcase Game Launcher, New Game Wizard, Recorder preset setup |
| Agent / MCP Support | CCGS workflow docs, MCPForUnity |

## Unity プロジェクトの開き方

Unity Hub から次のディレクトリを開いてください。

```text
src/AtelierLifeBug-Unity
```

エディタバージョンは `src/AtelierLifeBug-Unity/ProjectSettings/ProjectVersion.txt` で `6000.4.7f1` に固定されています。初回起動時は Unity が `Library/`, `Temp/`, `Obj/`, `Logs/`, `UserSettings/` などを生成します。

Package Manager は `src/AtelierLifeBug-Unity/Packages/manifest.json` を参照します。`com.coplaydev.unity-mcp` は GitHub URL 参照なので、初回復元時にネットワーク接続が必要です。Naninovel は `src/AtelierLifeBug-Unity/Packages/com.elringus.naninovel` に embedded package として含まれています。

## 基本操作

1. Unity Hub で `src/AtelierLifeBug-Unity` を開く。
2. Unity の import / package restore が完了するまで待つ。
3. メニューから `Showcase/Game Launcher` を開く。
4. `Refresh` を押して登録済みゲームを読み込む。
5. 対象ゲームを `Select` し、`Open Scene` または `Play with this Take` を実行する。

Recorder 設定は `Assets/Showcase/Common/Recorder` に 16:9 / 9:16 の MP4 / PNG Sequence 用プリセットとして配置されています。必要に応じて `Showcase/Setup Recorder Presets` から再生成できます。

## 登録済み Showcase

| Game ID | 表示名 | Scene | Take | 状態 |
| --- | --- | --- | --- | --- |
| `SLG_MoveOnly` | SLG 移動のみ | `Assets/Showcase/Games/SLG_MoveOnly/Scenes/SLG_MoveOnly.unity` | `Take_BasicMove`, `Take_EdgeMove`, `Take_RangePreview` | 8x8 グリッド、移動範囲表示、カメラプリセットを持つ実装済みサンプル |
| `RPG_BattleOnly` | バトル | `Assets/Showcase/Games/RPG_BattleOnly/Scenes/RPG_BattleOnly.unity` | `Take_Default` | `New Game Wizard` 生成ベースのスタブ |

## Showcase システム概要

`Showcase` は、撮影用の見せゲームを共通形式で扱うための基盤です。

- `ShowGameRegistry`
  - Editor ツールが参照する見せゲーム一覧です。
  - `ShowGameDefinition` を配列で保持します。

- `ShowGameDefinition`
  - 1つの見せゲームを表す ScriptableObject です。
  - ゲーム ID、表示名、対象シーン、推奨アスペクト比、使用可能な `TakePreset` を持ちます。
  - Editor 上では `SceneAsset` 参照を持ち、実行時用の `_scenePath` と同期します。

- `TakePreset`
  - 撮影テイク単位の設定です。
  - 解像度、HUD 表示、デバッグ表示、TimeScale、乱数 seed、ゲーム固有の引数を持ちます。
  - ゲーム固有の引数は `[SerializeReference] ITakeArguments` で型付き管理します。

- `ShowGameController`
  - 各見せゲームのシーンに配置する基底 Controller です。
  - `ApplyTakePreset`, `ResetTake`, HUD / Debug / Assist / Camera 操作などをゲームごとに実装します。

- `CaptureHotkeyService`
  - 撮影中のホットキー操作を受け取り、現在の `ShowGameController` に委譲します。
  - Input System の generated wrapper (`CaptureControls`) を使用します。

- `CaptureOverlay`
  - 撮影中の Game / Take / elapsed / FPS / help を表示する uGUI + TextMesh Pro のオーバーレイです。

## 主な Editor メニュー

| メニュー | 用途 |
| --- | --- |
| `Showcase/Game Launcher` | 登録済み見せゲームを開く、または指定 Take で Play 開始 |
| `Showcase/New Game Wizard...` | 新しい見せゲームの asmdef / Controller / TakeArguments / Scene / Definition / Take を生成 |
| `Showcase/Setup Recorder Presets` | 16:9 / 9:16 の MP4 / PNG Sequence 用 Recorder 設定を生成 |
| `Showcase/Reset Pending New Game Wizard State...` | Wizard が途中停止した場合の SessionState リセット |

## ディレクトリ構成

```text
.
├── src/
│   └── AtelierLifeBug-Unity/
│       ├── Assets/
│       │   ├── Scenes/
│       │   ├── Settings/
│       │   ├── Showcase/
│       │   │   ├── Common/
│       │   │   │   ├── Runtime/
│       │   │   │   ├── Editor/
│       │   │   │   ├── Input/
│       │   │   │   ├── Prefabs/
│       │   │   │   ├── Recorder/
│       │   │   │   └── Settings/
│       │   │   ├── Games/
│       │   │   └── Templates/
│       │   ├── TextMesh Pro/
│       │   └── TutorialInfo/
│       ├── Packages/
│       │   └── com.elringus.naninovel/
│       └── ProjectSettings/
├── docs/
│   ├── architecture/
│   ├── engine-reference/
│   └── examples/
├── design/
├── production/
│   ├── qa/
│   ├── session-logs/
│   └── session-state/
├── prompts/
├── .claude/
├── .codex/
└── CCGS Skill Testing Framework/
```

## 主要なコード配置

- 共通 Runtime: `src/AtelierLifeBug-Unity/Assets/Showcase/Common/Runtime`
- 共通 Editor tooling: `src/AtelierLifeBug-Unity/Assets/Showcase/Common/Editor`
- Input Actions / generated wrapper: `src/AtelierLifeBug-Unity/Assets/Showcase/Common/Input`
- Recorder presets: `src/AtelierLifeBug-Unity/Assets/Showcase/Common/Recorder`
- Capture prefabs: `src/AtelierLifeBug-Unity/Assets/Showcase/Common/Prefabs`
- 各見せゲーム: `src/AtelierLifeBug-Unity/Assets/Showcase/Games`
- New Game template scene: `src/AtelierLifeBug-Unity/Assets/Showcase/Templates`

## 開発・検証メモ

設計判断は `docs/architecture/ADR-*.md` にまとまっています。現在の `Showcase` 基盤では、特に次の方針が重要です。

- string ID ではなく、`GameIdAsset` / `TakePresetIdAsset` の参照で identity を扱う。
- `TakePreset` のゲーム固有引数は `[SerializeReference] ITakeArguments` で型付き管理する。
- `ShowGameDefinition` のシーン参照は Editor-only の `SceneAsset` から runtime-safe な path に同期する。
- `CaptureHotkeyService` は InputAction 名の string lookup ではなく generated wrapper を使う。
- 共通処理は `ShowGameController.ApplyCommonTakeFields()` に寄せ、ゲーム固有処理は各 Controller 側に残す。

手動 smoke test は `production/qa/smoke-2026-05-19.md` にあります。ただし、この文書は `ADV_MessageOnly` を含む 3 ゲーム構成を前提にした記述を含みます。調査したリリースフォルダでは `ADV_MessageOnly` の Unity asset は含まれていないため、検証時は現在の `ShowGameRegistry.asset` と実際の `Assets/Showcase/Games` を先に確認してください。

既知の調査メモとして、`production/qa/bugs/BUG-LAUNCHER-001-current-scene-take-desync.md` に Game Launcher の Current Scene Take dropdown に関する報告があります。

## ドキュメント

- `CLAUDE.md` — CCGS ベースのプロジェクト方針
- `AGENTS.md` — Codex / Claude の担当範囲とハンドオフ規約
- `docs/architecture/` — ADR、architecture review、traceability registry
- `docs/engine-reference/` — Unity / Unreal / Godot 参照メモ
- `production/qa/` — smoke test、bug report、Recorder baseline
- `prompts/` — 作業メモ、調査ログ、会話ログ
- `.claude/`, `.codex/` — エージェント、スキル、フック、運用ルール

## ライセンス

リポジトリ直下の `LICENSE` は MIT License です。Unity、Naninovel、その他 third-party package / asset にはそれぞれのライセンスが適用されます。
