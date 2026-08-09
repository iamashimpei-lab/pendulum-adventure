# Project Handoff — ふりこの大冒険 (仮題)

Pendulumania を土台にした子ども向け振り子アクションゲーム。単一 HTML ファイルで iPhone / iPad の Safari 系環境で動かす。

## Status

- 段階 0「縦切り試作」完成・検収済 (2026-08-09)。実装 = Sol (gpt-5.6-sol) 委譲、検収 = Claude (コード精読 + ブラウザ実動確認)
- 成果物 = `index.html` 1 ファイル。タイトル → 1 区画 (的 3 + ガラス的 1) → 切断やり直し → クリア
- **配信中**: GitHub Pages https://iamashimpei-lab.github.io/pendulum-adventure/ (公開リポジトリ、2026-08-09 晋平さん承認済)。更新は main へ push するだけ (反映 1 分前後)
- 晋平さんの実機 (iPhone / iPad) での手触り確認は未実施

## Next up

- 晋平さんの実機テスト: 振り回して気持ちいいか / 子どもが「もう 1 回」と言うか
- 手触り調整: 画面左上を素早く 3 回タップ (PC は D キー) で開発パネルが開き、重力・ばね・減衰・切断長などをスライダーで実機調整できる。良い値が見つかったら index.html の既定値に反映
- 手触り合格後: ワールド 1 (3 ステージ + 強化 + 難易度 + セーブ + 工房)。タイトル絵は gpt-image-2 (`codex $imagegen`) で生成予定

## Documentation index

| 質問 | まず読む docs |
| --- | --- |
| ゲームの仕様・数式・出荷条件 | [docs/game-design.md](docs/game-design.md) (企画書 v2) |
| 設計レビューの指摘と裁定 | [docs/design-review-sol-2026-08-09.md](docs/design-review-sol-2026-08-09.md) |
| 動作確認の画面写真 | [docs/screenshots/](docs/screenshots/) |

## Key operational notes

- 壊してはいけない 5 本柱と「伸び偽装 2 経路」の対策は企画書 §2 §5。実装変更時はここを崩さない
- Mac での動作確認は `python3 -m http.server` でローカル配信して開く (file 直開きはブラウザ自動化が拒否する)
- 自動テストで振り子を振るときは固有周期 (約 2.5 秒) に合わせないと勢いが育たない (速く揺らすと当たらないのは正常挙動)
- 実装は Sol 委譲 + Claude 検収の体制。実装後の敵対的レビュー第 2 回 (実プレイ所感込み) はワールド 1 完成時に実施予定
