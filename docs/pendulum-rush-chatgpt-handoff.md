# PENDULUM RUSH — Claude Code 開発引き継ぎ書

最終更新: 2026-08-11
対象ファイル: `Pendulum_Rush.html`

## 1. このゲームについて

`PENDULUM RUSH` は、昔のPCゲーム「Pendulamania（ペンジュラマニア）」の「リングを動かして振り子を操り、ターゲットを破壊し、振り子が徐々に大きく・重くなっていく」感触を参考にした、スマホ向け縦画面のオリジナルHTML5ゲーム。

実装は **単一HTMLファイル** にまとまっており、外部画像・外部音源・ゲームエンジン・npm依存はない。Canvas 2D + Web Audio API + Pointer Events + localStorage だけで動く。

開発時の最重要方針:

- スマホで片手で遊べること
- 操作は直感的で、説明を読まなくても触れば分かること
- 振り子の「重さ」「慣性」「危険な張力」がプレイ感の中心
- ターゲット破壊時に強い爽快感を出すこと
- UIがプレーエリアに重ならないこと
- ターゲットは「画面内」ではなく、実際に振り子が届きやすい場所に出すこと
- 既存機能を壊さず、基本的には `Pendulum_Rush.html` を直接改良すること

---

## 2. 現在の完成機能

### 基本ゲーム

- 縦画面スマホ向け
- 画面を指でドラッグして「リング（anchor）」を移動
- リングから伸縮ロープで重り（ball）がぶら下がる
- 重力・慣性・バネ・減衰・空気抵抗を使った振り子物理
- ターゲットに重りをぶつけて破壊
- 破壊するたびに重りの `mass` と `radius` が増加
- 重くなるにつれて衝撃力とスコア効率が上がる
- ロープの張力が高すぎる状態を続けると `ropeIntegrity` が低下
- 限界超過でロープ切断 → ゲーム終了
- 60秒制
- 一時停止 / 再開 / タイトルへ戻る

### ターゲット

4種類あり、進行度に応じて出現。

- `normal`: 標準
- `gold`: ボーナス。時間 +3秒、ロープ耐久回復
- `prism`: 高得点・RUSHゲージ増加量大
- `drifter`: 移動するターゲット

出現条件は `makeTarget()` にある。

重要: ターゲットはプレー領域の最上部には出さず、実際に振り子が届きやすい高さに制限済み。

### スコア / コンボ

スコア計算は主に `destroyTarget(target, impact)`。

倍率要素:

- ターゲット種別
- 衝突速度 / impact
- 張力（危険な状態ほど高得点）
- コンボ数
- RUSH MODE 中は ×2

短時間に連続破壊するとコンボ継続。

### RUSH MODE

- ターゲット破壊で `rush` が増加
- 100% で `activateFever()`
- 約7秒間の強化状態
- 得点倍率 ×2
- ロープ保護
- BGMの密度アップ
- エフェクト強化

コード内部では昔の命名が残っており、RUSH MODEを `fever` / `feverTime` と呼んでいる箇所がある。

### ビジュアル演出

Canvas 2Dで以下を実装。

- 重りの残像
- ターゲット破壊パーティクル
- 二重ショックウェーブ
- スコアのフローティング表示
- 発光
- 画面揺れ
- フラッシュ
- ヒットストップ (`hitFreeze`)
- RUSH時の背景変化
- ロープ張力による色変化 / 波打ち
- ネオン系テクノ背景

### サウンド / BGM

すべて **Web Audio APIでリアルタイム合成**。外部mp3等は使っていない。

`AudioEngine` クラスを参照。

#### 撃破SE

`AudioEngine.hit(combo, impact, type, pan)`

現在は複数レイヤー:

- ノイズのアタック
- サブベースパンチ
- 上昇する sawtooth zap
- triangle / square の高域レイヤー
- コンボが増えるほど音程上昇
- 4コンボごとに和音アクセント
- gold / prism は追加SE

#### BGM

`AudioEngine.startMusic()`

- 132 BPM
- 16分音符ベースのステップシーケンサー
- 4つ打ちキック
- オフビートハイハット
- スネア/クラップ風ノイズ
- ローリングベース
- アルペジオ
- RUSH中は `audio.setMood('fever')` によりパターン密度アップ

注意: `setInterval` ベースなので、将来的に音楽精度を上げる場合は AudioContext の look-ahead scheduling へ移行すると良い。

### 振動

`navigator.vibrate()` が使える端末のみ。

設定は保存される。

iOS SafariではVibration APIが使えないケースがあるので、使えなくてもゲーム進行に影響させないこと。

### ランキング

ローカル端末内ランキングを実装済み。

- 各ゲーム終了時にスコアを保存
- 最大100記録保持
- ランキング画面では上位10件表示
- 今回のスコアをハイライト
- 終了画面に `今回 X位 / 全Y記録`
- タイトルから `RANKING` で閲覧可能
- 以前のBEST値をランキングへ一度だけ移行

これはオンラインランキングではない。端末 / ブラウザごとの localStorage。

---

## 3. localStorage キー

互換性維持のため、理由なく変更しないこと。

| Key | 内容 |
|---|---|
| `pendulumRushSound` | `on` / `off` |
| `pendulumRushHaptics` | `on` / `off` |
| `pendulumRushBest` | BESTスコア |
| `pendulumRushScores` | ランキング履歴JSON（最大100件） |
| `pendulumRushRankingMigrated` | 旧BEST移行済みフラグ |

`safeStorage` ラッパーを通してアクセスしている。

---

## 4. コード構成

全コードは `Pendulum_Rush.html` 1ファイル。

大きく以下の順番。

1. HTML / CSS
2. 各ゲーム画面
   - メニュー
   - HUD
   - Pause
   - Ranking
   - Game Over
3. JS utility
4. `safeStorage`
5. DOM参照 `el`
6. `AudioEngine`
7. `PendulumGame`
8. イベントハンドラ / boot

グローバルデバッグ用に以下が公開されている。

```js
window.PendulumRush = game;
```

DevToolsから状態確認可能。

---

## 5. 主要クラス / 関数

### `AudioEngine`

音声全般。

重要メソッド:

- `init()` — AudioContext生成 / resume
- `tone()` — オシレータ音
- `noise()` — ノイズ音
- `hit()` — ターゲット破壊SE
- `levelUp()` — 重量成長系SE
- `fever()` — RUSH突入SE
- `warning()` — 張力警告
- `snap()` — ロープ切断
- `timeUp()` — タイムアップ
- `startMusic()` — テクノBGM開始
- `stopMusic()`
- `setMood()` — normal / fever

音量ルーティング:

```text
musicGain ─┐
           ├→ master → compressor → destination
sfxGain ───┘
```

現状概算:

- master = 0.36
- music = 0.30
- sfx = 0.82

SEの爽快感を落とさないため、BGMを上げる場合はコンプレッサーとSFXの聞こえ方を必ず実機確認すること。

### `PendulumGame`

ゲーム本体。

主要state:

```text
menu
playing
paused
ending
gameover
```

重要メソッド:

- `resize()` — Canvas解像度・基準ロープ長
- `bindPointer()` — タッチ / マウス入力
- `start()` — 1プレー初期化
- `pause()` / `resume()`
- `returnToTitle()`
- `getPlayBounds()` — UIを除いた実プレー領域
- `getTargetBounds()` — ターゲットが出現可能な領域
- `makeTarget()` — ターゲット生成
- `promoteTarget()` — 次ターゲットへ切替
- `fixedUpdate(dt)` — 物理更新
- `destroyTarget()` — 撃破、スコア、成長、演出
- `activateFever()` — RUSH MODE
- `finish()` — 終了演出
- `showResults()` — 結果 / ランキング保存
- `updateHud()`
- `loop()` — requestAnimationFrame
- `render()` / draw系 — Canvas描画

### 物理更新

`fixedDt = 1/120` の固定タイムステップ。

主要パラメータ（現行コード）:

```js
springK = 50
radialDamping = 7.2
gravity ≈ 920
restitution = 0.74
maxAnchorSpeed = 1450 // RUSH時 1750
```

ロープは剛体ではなく「伸縮するバネ」の感触。

張力はロープ伸長率、外向き速度、mass等から算出している。危険域で `ropeIntegrity` が低下する。

即切断条件の一つ:

```js
newDist > restLength * 2.16
```

この数字を下げすぎるとゲームが急に理不尽になるので注意。

---

## 6. プレー領域とHUD — 重要

過去に **プレーエリアと下部ゲージが重なる問題** があったため、現在は物理側でもHUDを除外している。

`getPlayBounds()`:

```js
const top = clamp(112 + this.height * .015, 116, 138);
const bottomReserve = clamp(126 + this.height * .012, 132, 152);
```

概念:

```text
┌─────────────────┐
│ SCORE / TIME HUD│ ← 非プレー領域
├─────────────────┤
│                 │
│   PLAY AREA     │
│                 │
│                 │
├─────────────────┤
│ TENSION / RUSH  │ ← 非プレー領域
└─────────────────┘
```

重りの画面境界判定にも `getPlayBounds()` を使用。

今後HUDの高さを変えた場合、**CSSだけ直して終わらせず `getPlayBounds()` も合わせて調整**すること。

理想的には将来、HUD DOMの `getBoundingClientRect()` からプレー領域を動的計算するとより堅牢。

---

## 7. ターゲット出現範囲 — 重要

以前、画面上部にターゲットが出すぎて物理的に届かない問題があった。

現在は `getTargetBounds(radius)` で:

```js
const reachableTop = Math.max(p.top + 58, this.height * .285);
```

とし、上すぎる位置を禁止している。

`makeTarget()` ではさらに:

- 現在のballに近すぎない
- anchorに近すぎない
- 現ターゲットに近すぎない

という条件で最大28回再抽選。

今後ターゲット追加時も **必ず `getTargetBounds()` 内に収めること**。

「見えている = 届く」ではない。リングより少し下〜中下段を主戦場とするのが現状の意図。

---

## 8. ランキング実装

### 読み込み

`loadScoreHistory()`

- JSONを読み込み
- 不正要素を除外
- 旧BESTを一度だけ移行
- スコア降順
- 同点の場合は古い記録が先
- 最大100件

### 記録

`recordScore(score)`

entry:

```js
{
  id: `${Date.now()}-${random}`,
  score: integer,
  ts: Date.now()
}
```

### 結果表示

`showResults()` 内:

```js
const placement = this.recordScore(roundedScore);
el.rankPosition.textContent = `今回 ${placement}位 / 全${this.scoreHistory.length}記録`;
```

### ランク称号

ランキング順位とは別にスコア評価ランクがある。

`getRank(score)`:

- ULTRA ≥ 250,000
- SSS ≥ 125,000
- SS ≥ 70,000
- S ≥ 35,000
- A ≥ 18,000
- B ≥ 9,000
- C ≥ 3,500
- D

「順位」と「RANK S/SS等」を混同しないこと。

---

## 9. 現在のゲームバランス

初期値:

- 制限時間: 60秒
- 初期mass: 1
- 初期radius: 17px
- 初期ロープ長: 画面サイズに応じて約118〜184px

破壊ごとにballが重く・大きくなる。

`destroyTarget()` のスコア式に、impact / tension / combo / target type / feverが絡む。

gold:

- +3秒
- ropeIntegrity +0.22

RUSH:

- 得点×2
- ロープ耐久へのダメージ軽減

バランスを触る際には「重量成長」「ロープ切断」「ターゲット位置」の3つを同時に考えること。massだけ大幅に増やすと終盤のロープ切断率が急上昇する。

---

## 10. スマホ対応上の注意

### 必須

- `touch-action: none`
- `user-scalable=no`
- Pointer Eventsを維持
- Safe Area (`env(safe-area-inset-*)`)を維持
- 指でリングが隠れないようタッチ時はanchorを指より上へオフセット

現行:

```js
const fingerOffset = pointerType === 'mouse' ? 0 : clamp(height * .06, 38, 64);
```

### テストすべき画面サイズ

最低限:

- 320 × 568
- 375 × 667
- 390 × 844
- 393 × 852
- 430 × 932
- iPad縦画面

横画面は主対象ではない。

### iOS音声

AudioContextはユーザー操作後でないと開始できない。

`launchGame()` で:

```js
await audio.init();
```

を行っている。自動再生へ変更しないこと。

---

## 11. 現状の技術的な改善候補

優先順。

### A. HUD領域の完全自動計測

現在は `getPlayBounds()` の値が画面高ベースの手動調整。

改善案:

- top HUD / bottom dockにIDを付ける
- `getBoundingClientRect()` で実座標取得
- Canvas座標へ変換
- safe marginを加えてplay boundsを生成

これにより将来UIを増やしても重なりにくい。

### B. Audio scheduler

現BGMは `setInterval`。

改善案:

- 25ms程度のscheduler loop
- AudioContext currentTime基準
- 100ms程度look-ahead

スマホ負荷時でもBGMのリズムが安定する。

### C. オンラインランキング

現在はlocalStorageのみ。

オンライン化する場合は不正対策を考えること。クライアント送信スコアをそのまま信用しない。

### D. モード追加

候補:

- 60秒 SCORE ATTACK（現行）
- ENDLESS（ロープ切断まで）
- HEAVY MODE（成長速度大）
- DAILY CHALLENGE

ただし表面UIを複雑にしすぎない。

### E. 成長の視覚的な達成感

mass増加時に:

- 小さなレベルアップ表示
- 一定massごとに球の材質変化
- 専用SE
- 背景段階変化

を入れる余地あり。

### F. ターゲットの種類

追加候補:

- 分裂ターゲット
- 連鎖爆発ターゲット
- 一撃高得点の小型ターゲット
- シールド付きターゲット

ただし「当てる爽快感」を邪魔する耐久敵は多用しない。

---

## 12. 変更時に壊しやすいポイント

1. **HUD CSSを変更したのに `getPlayBounds()` を変更しない**
   → UIとball/targetが重なる。

2. **target生成を直接 width/height で行う**
   → 上すぎる届かないターゲットが復活する。
   → 必ず `getTargetBounds()` を使う。

3. **AudioContextをページロード時に開始**
   → iPhoneで音が出ない。

4. **固定 timestep をやめる**
   → 端末FPS差で物理感が変化する可能性。

5. **massだけ大きくする**
   → tension / snapとのバランスが崩れる。

6. **Canvasだけ見てUIを配置する**
   → safe area / notch端末で崩れる。

7. **ランキング保存時にbestScoreとの二重記録を作る**
   → migrationロジックに注意。

8. **RUSHとfeverの名前を途中だけrename**
   → `feverTime`, `activateFever`, `audio.setMood('fever')` 等が複数存在。
   → renameするなら一括で。

---

## 13. 最低限の回帰テスト

変更後は以下を全部確認する。

- [ ] STARTを押してゲーム開始
- [ ] 指ドラッグでリングが自然に追従
- [ ] 指の真下ではなく少し上にリングが来る
- [ ] 重りが上下HUDへ侵入しない
- [ ] ターゲットが上部HUD内に出ない
- [ ] ターゲットが下部ゲージ内に出ない
- [ ] 上すぎて届かないターゲットが出ない
- [ ] normal targetを破壊できる
- [ ] goldで+3秒
- [ ] prismが出現する
- [ ] drifterが範囲外へ逃げない
- [ ] 連続破壊でコンボが上がる
- [ ] 破壊SEが鳴る
- [ ] BGMが鳴る
- [ ] SOUND OFFでBGM/SE両方停止
- [ ] RUSH 100%でRUSH発動
- [ ] RUSH時にBGMが変化
- [ ] tensionが上がるとゲージが上昇
- [ ] 無理な操作でロープ切断
- [ ] タイムアップで終了
- [ ] 終了時にスコア保存
- [ ] 「今回X位 / 全Y記録」が表示
- [ ] RANKING画面に記録が出る
- [ ] 再読み込み後もランキングが残る
- [ ] Pause → Resume後にBGMが復帰
- [ ] タブをバックグラウンドにすると自動Pause
- [ ] Retryで前ゲームの状態が残らない
- [ ] 320×568でUIが重ならない
- [ ] 430×932でUIが不自然に離れすぎない

---

## 14. Claude Codeへの開発指示テンプレート

このファイルと `Pendulum_Rush.html` を同じディレクトリに置いた上で、Claude Codeには以下の前提で依頼する。

```text
PENDULUM_RUSH_HANDOFF_FOR_CLAUDE_CODE.md を最初に読んでください。
現在の完成版は Pendulum_Rush.html です。

既存のゲームプレイ、ランキング、スマホ操作、サウンド、RUSH MODEを壊さずに改良してください。
変更前に関連コードを読み、変更後は引き継ぎ書の回帰テスト項目を確認してください。

特に以下は必ず維持してください。
- HUDとプレー領域を重ねない
- ターゲットはgetTargetBounds()の届く範囲に出す
- iPhoneでAudioContextはユーザー操作後に開始
- localStorageの既存ランキングとの互換性を壊さない
- 物理更新は固定timestepを維持

依頼する変更:
（ここに次の変更内容を書く）
```

---

## 15. Claude Codeでの推奨作業方法

単一HTMLなので、最初はビルド環境を導入せずそのまま編集するのが安全。

推奨:

```bash
python3 -m http.server 8080
```

その後ブラウザでローカル確認。

macOS + iPhoneで実機確認する場合、同一LANからMacのIPへアクセスするか、HTTPS対応の開発環境を使う。

PWA版を更新する場合は `Pendulum_Rush_App.zip` 内の `index.html` に最新版HTMLを反映し、必要に応じてService Workerのキャッシュバージョンも更新すること。

---

## 16. 今後のプロダクト方向性

このゲームの魅力は「複雑なルール」ではなく、以下のループ。

```text
振る
↓
気持ちよく破壊
↓
重くなる
↓
さらに衝撃が大きくなる
↓
張力が危険になる
↓
ギリギリを攻める
↓
高得点 / RUSH
```

新機能を入れるときも、このループを強化するものを優先する。

避けたい方向:

- メニュー階層を増やしすぎる
- 初見で分からない複雑な育成
- ターゲットが硬すぎて爽快感を止める
- 広告ゲーム的な大量UI
- プレイ中に頻繁に操作を止める演出

目指す方向:

- 触った瞬間分かる
- 30〜60秒でも気持ちいい
- 上手い人ほど危険な高張力プレイで得点を伸ばせる
- 重量成長そのものが楽しい
- 音・振動・画面演出がプレイ結果に同期する

---

## 17. 現在の正本

**正本は `Pendulum_Rush.html`。**

引き継ぎ時点では、このHTMLがランキング、修正版プレー領域、届くターゲット生成、爽快SE、132 BPMテクノBGMを含む最新バージョン。

Claude Codeは、まずこのファイルを読み、上書きではなく既存実装を理解してから差分修正すること。
