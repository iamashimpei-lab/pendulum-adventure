# Project Handoff — ふりこの大冒険 (仮題)

Pendulumania を土台にした子ども向け振り子アクションゲーム。単一 HTML ファイルで iPhone / iPad の Safari 系環境で動かす。

## Status

- **PENDULUM RUSH が主戦場** (2026-08-11 晋平さん決定「ChatGPT 版に追加要素を加えて行く」)。`rush/index.html` に取り込み、ふりこ由来の 6 アイディアを実装済み・検収合格: ①張力警告の多重表現 (震え・火花・支点脈動) ②RUSH 中クラスター的 4 個 ③かけら経済 + かざり図鑑 9 品 (性能影響ゼロ) ④こどもモード (物理共通・復活 1 回 + 危険域スローモ 0.55 倍・ランキング除外) ⑤ボス (残り 20 秒・留め具 3 + コア・全て接触即破壊・ジャックポット +15 かけら) ⑥子ども実機テスト文化は出荷条件として運用。経緯と検証メモ = [docs/pendulum-rush-takeover.md](docs/pendulum-rush-takeover.md)
- **rush/ は未配信 (commit + push 待ち)**。push されれば https://iamashimpei-lab.github.io/pendulum-adventure/rush/
- ふりこの大冒険 (index.html) は配信継続・開発停止 (2026-08-11)。切断裁定 (切断長 460 問題) は RUSH 乗り換えにより保留のまま閉じる
- 体制 = 企画・発注・検収・配信は Claude、実装は Sol (gpt-5.6-sol) 委譲 (2026-08-10 晋平さん指示で固定)
- **配信中**: GitHub Pages https://iamashimpei-lab.github.io/pendulum-adventure/ (公開リポジトリ、2026-08-09 晋平さん承認済)。更新は main へ push するだけ (反映 1 分前後)

## Next up

- **rush/ の commit + push (晋平さん承認待ち)** → 配信開始
- **実機テスト (iPhone)**: ①RUSH MODE・ボス・クラスターの fps と手触り ②かざり装備の見た目と和太鼓音 ③こどもモードを子どもたちで — 出荷条件は「説明なしで遊べるか・『もう 1 回』と言うか」
- 追加候補: ENDLESS モード (切断まで無制限) / BGM の look-ahead scheduler 化 / fever 演出の低負荷化 (低スペック端末向け) / タイトル絵 (gpt-image-2 `$imagegen`)
- ふりこ側の開発パネル (画面左上 3 回タップ / D キー) は現状のまま凍結

## Documentation index

| 質問 | まず読む docs |
| --- | --- |
| ゲームの仕様・数式・出荷条件 | [docs/game-design.md](docs/game-design.md) (企画書 v2) |
| 設計レビューの指摘と裁定 | [docs/design-review-sol-2026-08-09.md](docs/design-review-sol-2026-08-09.md) |
| PENDULUM RUSH (別実装) の引き取り判断 | [docs/pendulum-rush-takeover.md](docs/pendulum-rush-takeover.md) |
| 動作確認の画面写真 | [docs/screenshots/](docs/screenshots/) |

## Key operational notes

- 壊してはいけない 5 本柱と操作の不可侵 (支点 = カーソル完全一致) は企画書 §2 §3。Sol への発注文には毎回「物理・操作を 1 行も変えない」を明記する (過去 2 回の作り直しの教訓)
- Mac での動作確認は `python3 -m http.server` でローカル配信して開く (file 直開きはブラウザ自動化が拒否する)
- 自動テストで振り子を振るときは固有周期 (約 2.5〜3 秒) に合わせないと勢いが育たない。ガラス的の破壊は自動化困難 (深伸ばしの狙い打ちが要る)
- gitleaks が `SAVE_KEY` 等の文字列に誤検知することがある → 該当行に `// gitleaks:allow` を付ける
- 一度だけ Playwright のタブが白画面になった (2026-08-10 検収中、同一操作で再現せず)。再発したらゲーム側のメモリ暴走を疑う
