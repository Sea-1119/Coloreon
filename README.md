# Coloreon


<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <title>ネイルチップ診断</title>

<style>
body {
  background-color: #f9f6f2; /* アイボリー系 */
  color: #4a4a4a;
  font-family: 'Helvetica Neue', sans-serif;
  padding: 20px;
  line-height: 1.6;
}

h1, h2 {
  color: #5e4b3c; /* 柔らかいブラウン系 */
}

.question, .result, .start-screen {
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
  animation: fadeIn 0.6s ease-out;
}

/* ▼ 追加：画像と質問文を縦並びに */
.question {
    display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center; /* ← 追加：縦方向中央揃え（必要なら） */
  text-align: center; /* ← テキスト中央寄せ */
}

/* ▼ 追加：質問内の画像に下マージンと調整 */
.question img {
  display: block;        /* ← インラインのままだと中央寄せされない */
  margin: 0 auto 16px;   /* ← 下に余白をつけつつ中央寄せ */
  max-width: 100%;
  height: auto;
}

.choices {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 12px;
  margin-top: 20px;
}

.choice {
  background: #fff;
  border: 1px solid #dcd2c8;
  border-radius: 10px;
  padding: 14px;
  cursor: pointer;
  width: calc(50% - 16px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
  animation: popIn 0.4s ease-out;
}

.choice:hover {
  background: #f0ebe5;
  transform: scale(1.02);
}

.choice.selected {
  border-color: #b89f85;
  background: #f8eee7;
  color: #5a3e2b;
  font-weight: bold;
  box-shadow: 0 0 0 2px #d2baaa;
}

button {
  margin: 20px 10px;
  padding: 12px 24px;
  border: none;
  background: #b89f85; /* アクセントベージュ */
  color: #fff;
  border-radius: 6px;
  cursor: pointer;
  font-size: 15px;
  transition: background 0.3s ease;
}

button:hover {
  background: #a68c72;
}

button[disabled] {
  background: #ccc;
  cursor: not-allowed;
}

.results-container {
  display: flex;
  flex-wrap: nowrap; /* 横に並べるため折り返し無効に */
  overflow-x: auto;   /* はみ出したら横スクロール */
  gap: 20px;
  justify-content: flex-start; /* 左詰めにする */
  padding: 20px; /* スクロール時に余白を確保 */
  scroll-behavior: smooth; /* スムーズスクロール */
}

.design {
  flex: 0 0 auto;  /* 折り返し禁止＋横幅固定 */
  width: 180px;
  text-align: center;
  animation: fadeIn 0.6s ease-out;
}

.design img {
  width: 180px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.footer {
  margin-top: 40px;
  text-align: center;
}

/* ▼ 画像なし（テキストのみ）選択肢用 */
.choice.text-choice {
  width: 100%;
  padding: 16px 20px;
  text-align: left;
  font-size: 1rem;
  font-weight: normal;
  line-height: 1.4;
}

/* 中央寄せが良ければ下記を上書き */
.question.text-choice-container .choices {
  justify-content: flex-start;
}

/* フェードインアニメーション */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ポップインアニメーション */
@keyframes popIn {
  from { opacity: 0; transform: scale(0.9); }
  to { opacity: 1; transform: scale(1); }
}

.content-wrapper {
  max-width: 800px;
  margin: 60px auto;
  padding: 40px 30px;
  background-color: #ffffff;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  animation: fadeIn 0.6s ease-out;
}

</style>

</head>
<body>
 <div class="content-wrapper">
  <div class="start-screen">
    <h1>ネイルチップ診断</h1>
    <p> 約90種類のネイルデザインから、あなたの好みにぴったりなネイルチップデザインを提案します。14つの簡単な質問に答えてください。</p>
    <button id="start-btn">診断を始める</button>
  </div>

  <div class="question" style="display:none;">
    <h2 id="question-title"></h2>
    <div class="choices" id="choices"></div>
    <div>
      <button id="back-btn" style="display:none;">◀ 戻る</button>
      <button id="next-btn" disabled>次へ ▶</button>
    </div>
  </div>

  <div class="result" style="display:none;">
    <h2>おすすめデザイン</h2>
    <div class="results-container" id="results"></div>
    <div class="footer">
      <button onclick="location.reload()">もう一度診断する</button>
      <button onclick="window.open('https://coloreon.base.shop/','_blank')">オーダーはこちら</button>
    </div>
  </div>

  <script>

const questions = [
  {
    id: 'scene',
    title: 'Q1. ネイルチップを使いたいシーンは？（複数選択可）',
    multi: true,
  withImage: false, // 画像なし
    choices: [
      { text: 'お仕事' },
      { text: '休日のお出かけ・普段使い' },
      { text: 'デート' },
      { text: '結婚式・お呼ばれ' },
      { text: 'ライブ・推し活' },
      { text: '特別なイベント' }
    ]
  },
  {
    id: 'taste',
    title: 'Q2. 好きなテイストは？（複数選択可）',
    multi: true,
  withImage: false, // 画像なし
    choices: [
      { text: 'シンプル／ミニマル' },
      { text: 'ガーリー／フェミニン' },
      { text: 'モード／クール' },
      { text: 'カジュアル／ストリート' },
      { text: 'ゴシック／ロック' },
      { text: '韓国風' },
      { text: '派手めギャル系' },
      { text: '和風・レトロ' },
      { text: '特に決まってない／なんでもいい' }
    ]
  },
  {
    id: 'impression',
    title: 'Q3. なりたい印象に近いものを選んでください（複数選択可）',
    multi: true,
  withImage: false, // 画像なし
    choices: [
      { text: '大人っぽい・落ち着いてる' },
      { text: '可愛い・愛され感' },
      { text: 'かっこいい・洗練されてる' },
      { text: '上品・エレガント' },
      { text: '個性的・目立ちたい' },
      { text: '遊び心がある' },
      { text: 'ナチュラルで抜け感がある' },
      { text: '派手・ゴージャス' }
    ]
  },

  {
    id: 'color',
    title: 'Q4: 好きなカラー系統は？（複数選択可）',
    multi: true,
    withImage: true,
    choices: [
      { text: 'くすみカラー／ニュアンスカラー', img: 'images/nuance.png' },
      { text: 'ビビッド／原色系', img: 'images/vivid.png' },
      { text: 'シアー／クリア', img: 'images/sheer.png' },
      { text: 'ヌーディ／ベージュ系', img: 'images/nude.png' },
      { text: 'メタリック／ミラー／箔', img: 'images/metallic.png' },
      { text: 'パステル／ミルキートーン', img: 'images/pastel.png' },
      { text: 'モノトーン（白黒グレー）', img: 'images/monotone.png' },
      { text: '特に決まってない', img: 'images/unknown.png' }
    ]
  },

  {
    id: 'length',
    title: 'Q5: ネイルの長さはどれが好き？（複数選択可）',
    multi: true,
    withImage: true,
    choices: [
      { text: 'ショート', img: 'images/short.png' },
      { text: 'ミディアム', img: 'images/medium.png' },
      { text: 'ロング', img: 'images/long.png' },
      { text: '超ロング', img: 'images/extra_long.png' },
      { text: '長さは気にしない', img: 'images/any.png' }
    ]
  },

  {
    id: 'experience',
    title: 'Q6. ネイルチップは初めてですか？',
    multi: false,
　withImage: false, // 画像なし
    choices: [
      { text: 'はい（初めて）' },
      { text: 'いいえ（使ったことがある）' }
    ]
  },

  {
    id: 'shape',
    title: 'Q7: 好きなチップの形は？（複数選択可）',
    multi: true,
    withImage: true,
    choices: [
      { text: 'オーバル', img: 'images/oval.png' },
      { text: 'スクエア', img: 'images/square.png' },
      { text: 'バレリーナ（先細り四角）', img: 'images/ballerina.png' },
      { text: 'ポイント（先端尖り）', img: 'images/point.png' },
      { text: 'こだわりなし', img: 'images/none.png' }
    ]
  },
  {
    id: 'season',
    title: 'Q8. 今の季節、または気分に合う季節は？',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: '春' },
      { text: '夏' },
      { text: '秋' },
      { text: '冬' },
      { text: '季節に関係なく選びたい' }
    ]
  },
  {
    id: 'attention',
    title: 'Q9. どれくらいネイルチップを目立たせたいですか？（複数選択可）',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: 'しっかり印象に残るデザインにしたい' },
      { text: 'さりげなく目立たせたい' },
      { text: '控えめにしたい' }
    ]
  },
  {
    id: 'texture',
    title: 'Q10. 好きな質感は？',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: 'ツヤツヤ・うるうる' },
      { text: 'マット・ふんわり' },
      { text: 'キラキラ・ラメ系' },
      { text: 'メタリック・ミラー系' },
      { text: '立体感・ぷっくり' },
      { text: 'シンプル・フラットな仕上がり' },
      { text: '特にこだわりはない' }
    ]
  },
  {
    id: 'feeling',
    title: 'Q11. ネイルに込めたい気持ちは？',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: '自信をつけたい' },
      { text: '気分を上げたい' },
      { text: '癒されたい' },
      { text: '背伸びしたい' },
      { text: '特にない' },
      { text: '周囲と差をつけたい' }
    ]
  },
  {
    id: 'trend',
    title: 'Q12. トレンド感はどれくらい重視しますか？',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: '最新のトレンド重視' },
      { text: '定番・王道で使いやすいもの' },
      { text: '流行は気にしない' }
    ]
  },

 {
    id: 'motif',
    title: 'Q13: どんなモチーフやパーツが好き？',
    multi: true,
    withImage: true,
    choices: [
      { text: 'お花・植物・自然モチーフ', img: 'images/flower.png' },
      { text: '幾何学・ライン・アート', img: 'images/line.png' },
      { text: 'キャラ・推し・ロゴ', img: 'images/logo.png' },
      { text: '宝石風・ストーン・立体感', img: 'images/jewel.png' },
      { text: 'パール・レース・フェミニン系', img: 'images/pearl.png' },
      { text: '食べ物・面白デザイン', img: 'images/funny.png' },
      { text: 'シンプル・柄なし', img: 'images/simple.png' },
      { text: '特にない', img: 'images/unknown.png' }
    ]
  },
  {
    id: 'value',
    title: 'Q14. ネイルで大事にしていることは？',
    multi: true,
　withImage: false, // 画像なし
    choices: [
      { text: 'SNS映え・写真映え' },
      { text: '普段使いやすいこと' },
      { text: '手元がきれいに見えること' },
      { text: '自分の好きを詰め込むこと' },
      { text: '季節・イベント感に合うこと' },
      { text: 'アート性・デザイン性' }
    ]
  }
];



    const designs = [
  { 
    name: "マットネイル", 
    img: "https://via.placeholder.com/180?text=Matte", 
    desc: "落ち着いた質感の大人ネイル。普段使い・仕事にも◎", 
    tags: ["普段使い", "仕事用", "シンプル", "モード", "ショート", "初めて", "大人っぽい", "ナチュラル"]
  },
  { 
    name: "シアーネイル", 
    img: "https://via.placeholder.com/180?text=Sheer", 
    desc: "透明感ある大人ネイル。シンプル派におすすめ。", 
    tags: ["普段使い", "シンプル", "シアー", "初めて", "大人っぽい", "ナチュラル", "上品・エレガント"]
  },
  {
    name: "ジェリーネイル",
    img: "https://via.placeholder.com/180?text=Jelly",
    desc: "ぷるんとした透明感が魅力の韓国風ネイル。",
    tags: ["韓国風", "シアー", "ミディアム", "可愛い", "遊び心がある"]
  },
  {
    name: "マグネットネイル",
    img: "https://via.placeholder.com/180?text=Magnet",
    desc: "独自技術で磁力を利用した先進的なデザイン。",
    tags: ["モード", "かっこいい", "都会的", "普段使い",]
  },
  {
    name: "シロップネイル",
    img: "https://via.placeholder.com/180?text=Syrup",
    desc: "とろけるような柔らかい質感で、上品な印象。",
    tags: ["上品・エレガント", "ナチュラル", "ミディアム"]
  },
  {
    name: "うるうるネイル",
    img: "https://via.placeholder.com/180?text=Glassy",
    desc: "ツヤ感溢れる仕上がりで、華やかな印象。",
    tags: ["韓国風", "シアー", "ビビッド"]
  },
  {
    name: "オーロラネイル（フィルム系）",
    img: "https://via.placeholder.com/180?text=Aurora",
    desc: "虹色の輝きを見せる豪華なお呼ばれ向けデザイン。",
    tags: ["お呼ばれ", "ロング", "派手・ゴージャス" ,]
  },
  {
    name: "フロストネイル（すりガラス風）",
    img: "https://via.placeholder.com/180?text=Frost",
    desc: "すりガラスのような上品でクールな印象を与えるデザイン。",
    tags: ["普段使い", "シンプル", "ナチュラル"]
  },
  {
    name: "チークネイル",
    img: "https://via.placeholder.com/180?text=Cheek",
    desc: "ほんのりとしたピンクが特徴の、可愛らしいガーリーデザイン。",
    tags: ["ガーリー", "可愛い", "ミディアム"]
  },
  {
    name: "フレーク（貝）ネイル",
    img: "https://via.placeholder.com/180?text=Flake",
    desc: "貝殻の質感を思わせる、個性的なデザイン。",
    tags: ["派手・ゴージャス", "個性的", "ロング"]
  },
  {
    name: "ニュアンスネイル",
    img: "https://via.placeholder.com/180?text=Nuance",
    desc: "スモーキーなトーンが大人の魅力を引き出すデザイン。",
    tags: ["モード", "ナチュラル", "大人っぽい"]
  },
  {
    name: "ステンドグラスネイル",
    img: "https://via.placeholder.com/180?text=StainedGlass",
    desc: "カラフルなステンドグラスのような、華やかで印象的なデザイン。",
    tags: ["お呼ばれ", "個性的", "派手・ゴージャス" , "ビビッド" ]
  },
  {
    name: "押し花ネイル（ドライフラワー風）",
    img: "https://via.placeholder.com/180?text=PressedFlower",
    desc: "ドライフラワーをあしらった、ナチュラルでロマンティックなデザイン。",
    tags: ["ナチュラル", "可愛い", "ミディアム" ,"ガーリー"]
  },
  {
    name: "水彩アートネイル",
    img: "https://via.placeholder.com/180?text=Watercolor",
    desc: "水彩画のような柔らかなタッチが美しいデザイン。",
    tags: ["ナチュラル", "お呼ばれ", "ミディアム" , "上品・エレガント"]
  },
  {
    name: "インクネイル（にじみ・滲み）",
    img: "https://via.placeholder.com/180?text=Ink",
    desc: "にじみや滲みの美しさを活かした芸術的なデザイン。",
    tags: ["個性的", "モード", "ミディアム"]
  },
  {
    name: "マーブルネイル（複数色）",
    img: "https://via.placeholder.com/180?text=Marble",
    desc: "複数色が混ざり合うマーブル模様の豪華なデザイン。",
    tags: ["モード", "ナチュラル", "ロング"]
  },
  {
    name: "大理石ネイル",
    img: "https://via.placeholder.com/180?text=Marble2",
    desc: "大理石の美しい模様を再現した高級感あふれるデザイン。",
    tags: ["モード", "上品・エレガント", "ロング"]
  },
  {
    name: "アート筆跡風ネイル（筆模様）",
    img: "https://via.placeholder.com/180?text=BrushStroke",
    desc: "筆で描いたかのような自由奔放なタッチが魅力のデザイン。",
    tags: ["おもしろ", "個性的"]
  },
  {
    name: "フローティングネイル（浮いてる風）",
    img: "https://via.placeholder.com/180?text=Floating",
    desc: "チップが浮かんでいるような幻想的な印象のデザイン。",
    tags: ["モード", "個性的", "ミディアム"]
  },
  {
    name: "ミラーネイル",
    img: "https://via.placeholder.com/180?text=Mirror",
    desc: "鏡面仕上げで反射が美しい、洗練されたデザイン。",
    tags: ["お呼ばれ", "モード", "メタリック", "ロング", "かっこいい"]
  },
  {
    name: "シルバークロムネイル",
    img: "https://via.placeholder.com/180?text=SilverChrome",
    desc: "シルバークロムの光沢がクールな印象を与えるデザイン。",
    tags: ["モード", "メタリック", "かっこいい" , "推し活" ]
  },
  {
    name: "ゴールド箔ネイル",
    img: "https://via.placeholder.com/180?text=GoldLeaf",
    desc: "ゴールド箔をふんだんに使った、華やかで豪華なデザイン。",
    tags: ["お呼ばれ", "メタリック", "派手・ゴージャス", "ロング"]
  },
  {
    name: "ラメネイル",
    img: "https://via.placeholder.com/180?text=Glitter",
    desc: "ラメが輝く、華やかでキラキラなデザイン。",
    tags: ["ギャル", "派手・ゴージャス", "使用経験あり"]
  },
  {
    name: "グリッターフレンチ",
    img: "https://via.placeholder.com/180?text=GlitterFrench",
    desc: "フレンチスタイルにキラリと光るアクセントが加えられたデザイン。",
    tags: ["ガーリー", "可愛い", "上品・エレガント"]
  },
  {
    name: "ユニコーンパウダー仕上げ",
    img: "https://via.placeholder.com/180?text=UnicornPowder",
    desc: "幻想的なユニコーンパウダーで仕上げた夢のようなデザイン。",
    tags: ["おもしろ", "個性的", "ビビッド" ,"シンプル"]
  },
  {
    name: "ガラスフレークネイル",
    img: "https://via.placeholder.com/180?text=GlassFlake",
    desc: "ガラスのフレークが美しく輝く、洗練されたデザイン。",
    tags: ["モード", "個性的", "上品・エレガント"]
  },
  {
    name: "オーロラパウダーネイル",
    img: "https://via.placeholder.com/180?text=AuroraPowder",
    desc: "パウダーで表現された虹色の輝きが特徴のデザイン。",
    tags: ["お呼ばれ", "ロング", "派手・ゴージャス" ,"シンプル"]
  },
  {
    name: "ホイルアートネイル（レース）",
    img: "https://via.placeholder.com/180?text=FoilArt",
    desc: "レースのような繊細なホイルアートが印象的な上品デザイン。",
    tags: ["お呼ばれ", "モード", "上品・エレガント"]
  },
  {
    name: "3Dアートネイル（キュート、面白）",
    img: "https://via.placeholder.com/180?text=3DArt",
    desc: "立体感あふれるキュートで面白い3Dデザイン。",
    tags: ["おもしろ", "ギャル", "可愛い" , "使用経験あり"]
  },
  {
    name: "彫刻風ネイル（スカルプアート）",
    img: "https://via.placeholder.com/180?text=Sculpture",
    desc: "彫刻のようなシャープさと芸術性を表現したデザイン。",
    tags: ["使用経験あり", "モード", "かっこいい"]
  },
  {
    name: "バブルネイル（泡デザイン）",
    img: "https://via.placeholder.com/180?text=Bubble",
    desc: "泡をモチーフにしたポップでユニークなデザイン。",
    tags: ["おもしろ", "個性的", "ミディアム"]
  },
  {
    name: "ラテアートネイル（もこもこ質感）",
    img: "https://via.placeholder.com/180?text=LatteArt",
    desc: "もこもこの質感が温かみを感じさせるデザイン。",
    tags: ["個性的", "ナチュラル", "ミディアム"]
  },
  {
    name: "ビー玉ネイル（ぷっくりドーム）",
    img: "https://via.placeholder.com/180?text=Bead",
    desc: "ぷっくりドーム状のビー玉が印象的なエレガントなデザイン。",
    tags: ["モード", "個性的", "ロング"]
  },
  {
    name: "宝石（指輪）ネイル",
    img: "https://via.placeholder.com/180?text=GemRing",
    desc: "宝石を連想させるきらめきでリッチに仕上げたデザイン。ラインを引いて指輪風にも。",
    tags: ["お呼ばれ", "派手・ゴージャス", "ロング"]
  },
  {
    name: "液体ネイル（しずく封入）",
    img: "https://via.placeholder.com/180?text=Liquid",
    desc: "液体がしずくのように動く、ダイナミックなデザイン。",
    tags: ["個性的", "おもしろ", "モード", "ミディアム"]
  },
  {
    name: "チューブジェルネイル（透明ぷっくり線）",
    img: "https://via.placeholder.com/180?text=TubeGel",
    desc: "透明感とぷっくり感が際立つ、先進的なジェルデザイン。",
    tags: ["モード", "シアー", "使用経験あり" , "ガーリー"]
  },
  {
    name: "スクラッチアートネイル",
    img: "https://via.placeholder.com/180?text=Scratch",
    desc: "スクラッチの跡が個性的なアクセントとなるデザイン。",
    tags: ["おもしろ", "個性的", "ミディアム"]
  },
  {
    name: "チェックネイル",
    img: "https://via.placeholder.com/180?text=Check",
    desc: "クラシカルなチェック柄がシンプルかつ上品な印象を与える。シアーなどでも〇",
    tags: ["シンプル", "個性的", "ショート"]
  },
  {
    name: "ドットネイル",
    img: "https://via.placeholder.com/180?text=Dot",
    desc: "かわいらしいドットがアクセントとなったポップなデザイン。",
    tags: ["シンプル", "ガーリー", "ショート"]
  },
  {
    name: "ストライプネイル",
    img: "https://via.placeholder.com/180?text=Stripe",
    desc: "シャープなストライプが印象的な、シンプルながらモダンなデザイン。",
    tags: ["シンプル", "モード", "ショート"]
  },
  {
    name: "幾何学ネイル",
    img: "https://via.placeholder.com/180?text=Geometry",
    desc: "幾何学模様で個性と現代性を表現したデザイン。",
    tags: ["個性的", "モード", "ショート"]
  },
  {
    name: "アニマル柄ネイル",
    img: "https://via.placeholder.com/180?text=Animal",
    desc: "レオパードやゼブラなど、ワイルドなアニマル柄が特徴のデザイン。",
    tags: ["おもしろ", "個性的", "派手・ゴージャス" , "ギャル"]
  },
  {
    name: "スネークスキンネイル",
    img: "https://via.placeholder.com/180?text=SnakeSkin",
    desc: "蛇革の質感を模した、エッジの効いた個性的なデザイン。",
    tags: ["おもしろ", "個性的", "派手・ゴージャス"]
  },
  {
    name: "タイルネイル（モザイク風）",
    img: "https://via.placeholder.com/180?text=Tile",
    desc: "モザイク風に組み合わされたタイル柄が洗練された印象を演出する。",
    tags: ["個性的", "モード", "ロング"]
  },
  {
    name: "トライバル柄",
    img: "https://via.placeholder.com/180?text=Tribal",
    desc: "エスニックなトライバル柄で強い個性を表現するデザイン。",
    tags: ["おもしろ", "個性的", "派手・ゴージャス"]
  },
  {
    name: "フェザーネイル",
    img: "https://via.placeholder.com/180?text=Feather",
    desc: "柔らかい羽毛のようなテクスチャーが特徴の、軽やかなデザイン。",
    tags: ["ガーリー", "可愛い", "ミディアム"]
  },
  {
    name: "塗りかけネイル（スプラッシュ）",
    img: "https://via.placeholder.com/180?text=Splash",
    desc: "スプラッシュ感あふれるランダムな塗りが目を引くデザイン。",
    tags: ["おもしろ", "個性的", "ナチュラル"]
  },
  {
    name: "ブロッキングネイル",
    img: "https://via.placeholder.com/180?text=Blocking",
    desc: "大胆なブロック分割で構成された、力強い印象のモダンデザイン。",
    tags: ["モード", "かっこいい", "ロング"]
  },
  {
    name: "グミネイル",
    img: "https://via.placeholder.com/180?text=Gummy",
    desc: "グミのような柔らかい質感とポップな色使いが特徴のデザイン。",
    tags: ["ギャル", "可愛い", "ミディアム"]
  },
  {
    name: "キャンディーネイル",
    img: "https://via.placeholder.com/180?text=Candy",
    desc: "カラフルなキャンディを思わせる、明るくポップなデザイン。",
    tags: ["ギャル", "可愛い", "ミディアム"]
  },
  {
    name: "落書きネイル",
    img: "https://via.placeholder.com/180?text=Graffiti",
    desc: "ストリートアートのような自由奔放な落書き風デザイン。",
    tags: ["おもしろ", "個性的", "ショート"]
  },
  {
    name: "ハートホロネイル",
    img: "https://via.placeholder.com/180?text=HeartHollow",
    desc: "ホロ感のあるハートモチーフがキュートな印象を与えるデザイン。",
    tags: ["ガーリー", "可愛い", "ショート"]
  },
  {
    name: "痛ネイル",
    img: "https://via.placeholder.com/180?text=Itanail",
    desc: "推し活向けの、派手で個性的なデザイン。",
    tags: ["推し活", "個性的", "使用経験あり"]
  },
  {
    name: "星空ネイル",
    img: "https://via.placeholder.com/180?text=Starry",
    desc: "夜空の星の輝きをイメージした幻想的なデザイン。",
    tags: ["お呼ばれ", "上品・エレガント", "ロング"]
  },
  {
    name: "ロゴネイル（ブランド風）",
    img: "https://via.placeholder.com/180?text=Logo",
    desc: "ブランド風のロゴを大胆にあしらった洗練されたデザイン。",
    tags: ["推し活", "個性的", "使用経験あり"]
  },
  {
    name: "文字ネイル（英字、漢字など）",
    img: "https://via.placeholder.com/180?text=Text",
    desc: "タイポグラフィを活かした、クールでアートなデザイン。",
    tags: ["推し活", "個性的", "使用経験あり"]
  },
  {
    name: "パール付きネイル",
    img: "https://via.placeholder.com/180?text=Pearl",
    desc: "上品なパールがアクセントの、エレガントなデザイン。",
    tags: ["上品・エレガント", "可愛い", "ミディアム" , "お呼ばれ"]
  },
  {
    name: "ラインストーン・ラインチェーンネイル",
    img: "https://via.placeholder.com/180?text=Rhinestone",
    desc: "ラインストーンで縁取られた、贅沢な印象のデザイン。",
    tags: ["推し活", "派手・ゴージャス", "ロング"]
  },
  {
    name: "モノトーン（モダン）ネイル",
    img: "https://via.placeholder.com/180?text=Monotone",
    desc: "シンプルで現代的なモノトーンデザイン。",
    tags: ["シンプル", "モード", "ロング"]
  },
  {
    name: "ベルベットネイル（起毛）",
    img: "https://via.placeholder.com/180?text=Velvet",
    desc: "起毛感が特徴の、柔らかで温かみのあるデザイン。",
    tags: ["ガーリー", "可愛い", "ミディアム"]
  },
  {
    name: "ニットネイル（立体模様）",
    img: "https://via.placeholder.com/180?text=Knit",
    desc: "立体的な編み模様が印象的な、個性的で温かみのあるデザイン。",
    tags: ["モード", "個性的", "ミディアム"]
  },
  {
    name: "チュールネイル",
    img: "https://via.placeholder.com/180?text=Tulle",
    desc: "繊細な透け感が魅力の、ガーリーでフェミニンなデザイン。",
    tags: ["ガーリー", "可愛い", "ミディアム"]
  },
  {
    name: "フリルネイル（波打つ縁）",
    img: "https://via.placeholder.com/180?text=Frill",
    desc: "波打つフリルが華やかさを演出する、可愛らしいデザイン。",
    tags: ["ガーリー", "可愛い", "ショート"]
  },
  {
    name: "レザー風ネイル",
    img: "https://via.placeholder.com/180?text=Leather",
    desc: "レザーの質感を再現した、クールでかっこいいデザイン。",
    tags: ["モード", "かっこいい", "ロング"  ,"上品・エレガント"]
  },
  {
    name: "木目ネイル（ウッド調）",
    img: "https://via.placeholder.com/180?text=Wood",
    desc: "木目の温かみある質感が特徴の、ナチュラルなデザイン。",
    tags: ["ナチュラル", "個性的", "ショート"]
  },
  {
    name: "コンクリートネイル",
    img: "https://via.placeholder.com/180?text=Concrete",
    desc: "コンクリートの無骨さを取り入れた、モダンなデザイン。",
    tags: ["モード", "シンプル", "ロング"]
  },
  {
    name: "ラテネイル（カフェ風アート）",
    img: "https://via.placeholder.com/180?text=LatteArt",
    desc: "カフェの温もりとおしゃれなアートが融合したデザイン。",
    tags: ["ナチュラル", "個性的", "ミディアム"]
  },
  {
    name: "アイスキューブネイル（氷質感）",
    img: "https://via.placeholder.com/180?text=IceCube",
    desc: "氷のようなクールな輝きが特徴のお呼ばれ向けデザイン。",
    tags: ["お呼ばれ", "モード", "メタリック", "ロング"]
  }
];

    let answers = {};
    let qIndex = 0;

    const startBtn = document.getElementById('start-btn');
    const questionDiv = document.querySelector('.question');
    const resultDiv = document.querySelector('.result');
    const titleEl = document.getElementById('question-title');
    const choicesEl = document.getElementById('choices');
    const backBtn = document.getElementById('back-btn');
    const nextBtn = document.getElementById('next-btn');

    startBtn.onclick = () => {
  document.querySelector('.start-screen').style.display = 'none';
  questionDiv.style.display = 'block';
  showQuestion();
};

    backBtn.onclick = () => {
      if (qIndex > 0) {
        qIndex--;
        showQuestion();
      }
    };

    nextBtn.onclick = () => {
  const q = questions[qIndex];
  answers[q.id] = [...choicesEl.children]
    .filter(c => c.classList.contains('selected'))
    .map(c => c.textContent);
  qIndex++;
  if (qIndex < questions.length) {
    showQuestion();
  } else {
    showResult();
  }
};

    function showQuestion() {
  const q = questions[qIndex];
  titleEl.textContent = q.title;
  choicesEl.innerHTML = '';
  nextBtn.disabled = true;
  backBtn.style.display = qIndex === 0 ? 'none' : 'inline-block';

  q.choices.forEach(choice => {
    const div = document.createElement('div');
    div.className = 'choice';

    // 画像とテキストをまとめるコンテナ
    const choiceContent = document.createElement('div');
    choiceContent.classList.add('choice-content');

    // 画像を追加
    if (choice.img) {
      const imgElement = document.createElement('img');
      imgElement.src = choice.img;
      imgElement.alt = choice.text;
      imgElement.classList.add('choice-img');
      choiceContent.appendChild(imgElement);
    }

    // テキストを追加
    const textElement = document.createElement('span');
    textElement.textContent = choice.text;
    textElement.classList.add('choice-text');
    choiceContent.appendChild(textElement);

    div.appendChild(choiceContent);
    choicesEl.appendChild(div);

    // 選択時の処理
    div.onclick = () => {
      if (q.multi) {
        div.classList.toggle('selected');
      } else {
        [...choicesEl.children].forEach(c => c.classList.remove('selected'));
        div.classList.add('selected');
      }
      nextBtn.disabled = ![...choicesEl.children].some(c => c.classList.contains('selected'));
    };
  });

  // 以前の選択を復元
  if (answers[q.id]) {
    answers[q.id].forEach(val => {
      [...choicesEl.children].find(c => c.querySelector('.choice-text').textContent === val).classList.add('selected');
    });
    nextBtn.disabled = answers[q.id].length === 0;
  }
    }

function showResult() {
  // scoring
  const scores = designs.map(d => {
    let score = 0;
    Object.values(answers).flat().forEach(ans => {
      if (d.tags.includes(ans)) score += 2;
    });
    return { design: d, score };
  });
  scores.sort((a, b) => b.score - a.score);

  // top10
  const top10 = scores.slice(0, 10);
  document.getElementById('results').innerHTML = '';
  top10.forEach(({ design, score }) => {
    const div = document.createElement('div');
    div.className = 'design';

    // 画像のサイズ指定
    const img = document.createElement("img");
    img.src = design.img;
    img.alt = design.name;
    img.style.width = "200px";
    img.style.height = "200px";
    img.style.objectFit = "cover";

    const title = document.createElement("h4");
    title.textContent = design.name;

    const desc = document.createElement("p");
    desc.textContent = design.desc;

    div.appendChild(img);
    div.appendChild(title);
    div.appendChild(desc);

    document.getElementById('results').appendChild(div);
  });

  questionDiv.style.display = 'none';
  resultDiv.style.display = 'block';
}

  </script>
</body>
</html>
