# 2026_ds
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>ユニコーンと虹のかけら</title>
<style>
  :root{
    --bg1:#1b1040; --bg2:#4a2a7a; --card:rgba(255,255,255,.1);
    --pink:#ff8ad8; --blue:#8ad8ff; --yellow:#fff08a;
  }
  *{box-sizing:border-box}
  body{
    margin:0; min-height:100vh; color:#fff;
    font-family:"Hiragino Maru Gothic ProN","Yu Gothic",system-ui,sans-serif;
    background:linear-gradient(160deg,var(--bg1),var(--bg2) 60%,#c25aa8);
    display:flex; justify-content:center; align-items:center; padding:16px;
    overflow-x:hidden;
  }
  .star{position:fixed; width:3px; height:3px; background:#fff; border-radius:50%;
    animation:tw 3s infinite ease-in-out; pointer-events:none; opacity:.6}
  @keyframes tw{0%,100%{opacity:.1}50%{opacity:1}}
  .sparkle{position:fixed; pointer-events:none; font-size:20px; z-index:99;
    animation:pop .9s forwards}
  @keyframes pop{from{transform:translateY(0) scale(1);opacity:1}
    to{transform:translateY(-60px) scale(.3);opacity:0}}
  main{width:100%; max-width:640px; background:var(--card); border:1px solid rgba(255,255,255,.25);
    border-radius:24px; padding:24px; backdrop-filter:blur(8px);
    box-shadow:0 10px 40px rgba(0,0,0,.35); position:relative; z-index:1}
  h1{margin:0 0 4px; text-align:center; font-size:1.7rem;
    background:linear-gradient(90deg,#ff7a7a,#ffd27a,#8aff9a,#7ad8ff,#c88aff,#ff7a7a);
    background-size:200% auto; -webkit-background-clip:text; background-clip:text;
    color:transparent; animation:flow 4s linear infinite}
  @keyframes flow{to{background-position:200% center}}
  .sub{text-align:center; opacity:.8; margin:0 0 16px; font-size:.9rem}
  #emoji{font-size:4.5rem; text-align:center; animation:float 3s ease-in-out infinite}
  @keyframes float{50%{transform:translateY(-10px)}}
  #hud{display:flex; justify-content:space-between; font-size:.9rem; opacity:.9; margin-bottom:6px}
  #story{min-height:120px; line-height:1.9; font-size:1.05rem; white-space:pre-wrap;
    background:rgba(0,0,0,.2); border-radius:14px; padding:14px; cursor:pointer}
  #choices{margin-top:14px; display:flex; flex-direction:column; gap:10px}
  button{font:inherit; color:#2a1450; background:linear-gradient(90deg,var(--pink),var(--blue));
    border:none; border-radius:999px; padding:12px 18px; cursor:pointer; font-weight:bold;
    transition:transform .15s, box-shadow .15s}
  button:hover{transform:scale(1.03); box-shadow:0 0 16px var(--pink)}
  input{font:inherit; width:100%; padding:12px 14px; border-radius:12px; border:2px solid var(--blue);
    background:rgba(255,255,255,.92); color:#2a1450; margin-bottom:8px}
  label{font-size:.9rem; display:block; margin:8px 0 4px}
  .game{position:relative; height:200px; background:rgba(0,0,0,.25); border-radius:14px; overflow:hidden}
  .game button{position:absolute; font-size:2rem; padding:6px 10px; background:none; box-shadow:none;
    animation:float 1.2s infinite}
  .msg{min-height:1.4em; font-size:.95rem; color:var(--yellow); margin:4px 0}
  .banner{text-align:center; font-size:1.3rem; padding:14px; border-radius:14px;
    background:linear-gradient(90deg,rgba(255,122,122,.5),rgba(255,210,122,.5),rgba(138,255,154,.5),rgba(122,216,255,.5),rgba(200,138,255,.5));
    margin-top:12px; font-weight:bold}
  .hidden{display:none}
</style>
</head>
<body>
<div id="stars"></div>
<main>
  <h1>🌈 ユニコーンと虹のかけら 🌈</h1>
  <p class="sub">あなたの選択で物語が変わる、ちいさな冒険</p>

  <div id="hud" class="hidden">
    <span id="who"></span>
    <span id="shards">🌈 かけら 0/5</span>
  </div>
  <div id="emoji">🦄</div>
  <div id="story"></div>
  <div id="choices"></div>
</main>

<script>
const $ = s => document.querySelector(s);
const story = $('#story'), choicesEl = $('#choices'), emojiEl = $('#emoji');
const S = { name:'', uni:'', shards:0, love:0, word:'', tries:0 };

/* ---------- 背景の星 ---------- */
for (let i = 0; i < 60; i++) {
  const d = document.createElement('div');
  d.className = 'star';
  d.style.left = Math.random()*100 + 'vw';
  d.style.top = Math.random()*100 + 'vh';
  d.style.animationDelay = Math.random()*3 + 's';
  $('#stars').appendChild(d);
}
/* ---------- クリックでキラキラ ---------- */
document.addEventListener('click', e => {
  const s = document.createElement('div');
  s.className = 'sparkle';
  s.textContent = ['✨','⭐','💖','🌟'][Math.floor(Math.random()*4)];
  s.style.left = e.clientX - 10 + 'px';
  s.style.top = e.clientY - 10 + 'px';
  document.body.appendChild(s);
  setTimeout(() => s.remove(), 900);
});

/* ---------- タイプライター表示 ---------- */
let timer = null, fullText = '', onDone = null;
function typeText(text, done) {
  clearInterval(timer);
  fullText = text; onDone = done; story.textContent = '';
  let i = 0;
  timer = setInterval(() => {
    story.textContent = text.slice(0, ++i);
    if (i >= text.length) finish();
  }, 30);
}
function finish() {
  if (!timer) return;
  clearInterval(timer); timer = null;
  story.textContent = fullText;
  const cb = onDone; onDone = null; cb && cb();
}
story.addEventListener('click', finish);

/* ---------- 画面の部品 ---------- */
function setChoices(list) {
  choicesEl.innerHTML = '';
  list.forEach(c => {
    const b = document.createElement('button');
    b.textContent = c.label;
    b.onclick = c.fn;
    choicesEl.appendChild(b);
  });
}
function updateHud() {
  $('#hud').classList.remove('hidden');
  $('#who').textContent = `🦄 ${S.uni} ／ 👤 ${S.name}`;
  $('#shards').textContent = `🌈 かけら ${S.shards}/5`;
}
function addInput(placeholder, btnLabel, handler, msg) {
  choicesEl.innerHTML = '';
  const input = document.createElement('input');
  input.placeholder = placeholder;
  input.maxLength = 30;
  const m = document.createElement('div'); m.className = 'msg';
  if (msg) m.textContent = msg;
  const b = document.createElement('button');
  b.textContent = btnLabel;
  const run = () => handler(input.value.trim(), m);
  b.onclick = run;
  input.addEventListener('keydown', e => { if (e.key === 'Enter' && !e.isComposing) run(); });
  choicesEl.append(input, m, b);
  input.focus();
}

/* ---------- シーン ---------- */
function go(id) {
  const sc = scenes[id]();
  emojiEl.textContent = sc.emoji;
  updateHud();
  choicesEl.innerHTML = '';
  typeText(sc.text, () => sc.after && sc.after());
}

const scenes = {
  s1: () => ({
    emoji:'🌲',
    text:`${S.name}は、月あかりの「ほしぞらの森」に迷いこんだ。\n\nすると、木々のむこうで何かがキラリと光った。\nそっと近づくと……角の折れかけた、小さなユニコーンが泣いている!\n\n「ぐすん……ぼくは${S.uni}……」\n\nさて、どうする?`,
    after: () => setChoices([
      { label:'🍎 りんごをそっと差し出す', fn:() => { S.love += 2; go('s2a'); } },
      { label:'📣「だいじょうぶー!?」と大声で呼ぶ', fn:() => { S.love -= 1; go('s2b'); } },
      { label:'🌳 木のかげからのぞいてみる', fn:() => { S.love += 1; go('s2c'); } },
    ])
  }),
  s2a: () => ({
    emoji:'🍎',
    text:`${S.uni}はりんごを一口かじって、ぱああっと笑顔になった。\n\n「おいしい……! ありがとう、${S.name}!」\n\n${S.uni}は事情を話してくれた。\n虹の宝石「にじのかけら」が5つ、森のどこかに散らばってしまったらしい。\nそれがないと、角が光らず、虹の橋をかけられないのだ。`,
    after: () => setChoices([{ label:'💪「いっしょに探そう!」', fn:() => go('s3') }])
  }),
  s2b: () => ({
    emoji:'😱',
    text:`「ひゃあああ!」\n${S.uni}は驚いて3メートル飛びあがり、りんごの木に頭をぶつけた。\nりんごがぽろぽろ降ってくる。\n\n「……いたた。でも、きみ、悪い人じゃなさそうだね」\n\nどうやら事情があるらしい。虹の宝石「にじのかけら」が5つ、森に散らばってしまったという。\nそれがないと角が光らず、虹の橋をかけられないのだ。`,
    after: () => setChoices([{ label:'🙏「ごめんね……いっしょに探すよ」', fn:() => go('s3') }])
  }),
  s2c: () => ({
    emoji:'👀',
    text:`${S.uni}は、木のかげの気配にすぐ気づいた。\n\n「そこにいるの? 出ておいでよ、こわくないよ」\n\nおずおず出ていくと、${S.uni}はにっこり笑った。\n虹の宝石「にじのかけら」が5つ森に散らばってしまい、角が光らなくて困っているそうだ。`,
    after: () => setChoices([{ label:'✨「手伝うよ!」', fn:() => go('s3') }])
  }),
  s3: () => ({
    emoji:'⭐',
    text:`森のあちこちで、にじのかけらがチカチカ光っている。\nすばしっこく動きまわるので、タップして5つ集めよう!`,
    after: () => {
      choicesEl.innerHTML = '';
      const area = document.createElement('div'); area.className = 'game';
      const star = document.createElement('button'); star.textContent = '💎';
      const place = () => {
        star.style.left = Math.random()*80 + '%';
        star.style.top = Math.random()*75 + '%';
      };
      place();
      star.onclick = e => {
        e.stopPropagation();
        S.shards++; updateHud(); place();
        if (S.shards >= 5) { setTimeout(() => go('s4'), 400); }
      };
      area.appendChild(star); choicesEl.appendChild(area);
    }
  }),
  s4: () => ({
    emoji:'🔮',
    text:`かけらが5つそろうと、空中に光の文字があらわれた。\n\n『最後の扉をひらく者よ、なぞなぞに答えよ』\n\n「雨あがりの空にかかる、七色の橋。\n  さわれないけれど、みんな大好き。\n  わたしは、なあんだ?」\n\n答えを入力してね。`,
    after: () => { S.tries = 0; askRiddle(); }
  }),
  s5: () => ({
    emoji:'☁️',
    text:`扉の向こうには、ふわふわの雲の生きもの「モクモク」がいた。\nかけらを散らばせたのは、この子だったらしい。\n\n「だって……だれも、ぼくと遊んでくれないんだもん……」\n\nモクモクは、しょんぼり雨を降らせている。`,
    after: () => setChoices([
      { label:'😠「いたずらはダメでしょ!」とおこる', fn:() => { S.love -= 1; go('s6'); } },
      { label:'🤝「いっしょに遊ぼうよ!」と誘う', fn:() => { S.love += 2; go('s6'); } },
    ])
  }),
  s6: () => ({
    emoji:'🪄',
    text:`${S.uni}の角が、ぼんやり光りはじめた。\n\n「${S.name}、最後の魔法をかけたいんだ。\n  きみが好きな【魔法のことば】を、ぼくに教えて!」\n\n(なんでもOK。おいしい食べ物でも、好きな言葉でも!)`,
    after: () => addInput('例: ふわふわパンケーキ', '✨ ことばを贈る', (v, m) => {
      if (!v) { m.textContent = '何か入力してね!'; return; }
      S.word = v; go('end');
    })
  }),
  end: () => {
    const power = Math.min(S.word.length, 10);
    let title, body, emoji;
    if (S.love >= 5) {
      emoji = '🌈'; title = '🌟 ハッピーエンド:虹の王国のひみつ';
      body = `「${S.word}!」\n\n${S.uni}の角が七色にかがやき、空に巨大な虹の橋がかかった!\nモクモクは雨をやめて、橋の上でくるくる踊りだした。\n\nそのとき、橋のむこうから虹の王国のみんなが現れて言った。\n「ようこそ、${S.name}! きみは今日から名誉ユニコーンだ!」\n\nおでこに、ちいさな光の角がぽんっと生えた。\n(魔法パワー:${power*10}%)`;
    } else if (S.love >= 2) {
      emoji = '🦄'; title = '✨ グッドエンド:にぎやかな夜明け';
      body = `「${S.word}!」\n\n${S.uni}の角がぽわっと光り、小さな虹がかかった。\nモクモクはまだ少し拗ねているけれど、虹をすべり台にして遊びはじめた。\n\n「また来てね、${S.name}」\n${S.uni}は手をふった。ほしぞらの森に、朝がやってくる。\n(魔法パワー:${power*10}%)`;
    } else {
      emoji = '🥲'; title = '🍂 ほろ苦エンド:ちょっぴり曇りの虹';
      body = `「${S.word}……!」\n\n${S.uni}の角が光り、うすぼんやりした虹がかかった。\nモクモクはぷいっとそっぽを向いている。\n\n「次はもうちょっと、やさしくしてね」と${S.uni}。\nうーん、${S.name}、次の冒険でリベンジしよう!\n(魔法パワー:${power*10}%)`;
    }
    return { emoji, text: `${title}\n\n${body}`, after: () => {
      const b = document.createElement('div'); b.className = 'banner';
      b.textContent = `🌈 魔法のことば:「${S.word}」`;
      setChoices([
        { label:'📋 結果をコピーしてシェア', fn:() => {
          const t = `${S.name}と${S.uni}の冒険は「${title}」で終わりました!魔法のことば:「${S.word}」 #ユニコーンと虹のかけら`;
          navigator.clipboard.writeText(t).then(() => alert('コピーしました!'));
        }},
        { label:'🔁 もう一度あそぶ', fn:() => { location.reload(); } },
      ]);
      choicesEl.prepend(b);
    }};
  }
};

/* ---------- なぞなぞ ---------- */
function askRiddle() {
  addInput('こたえを入力', '🔮 こたえる', (v, m) => {
    const a = v.replace(/\s/g,'');
    if (['虹','にじ','ニジ','rainbow','Rainbow'].includes(a)) {
      S.love += 2;
      alert('せいかい! 扉がひらく……!');
      go('s5');
      return;
    }
    S.tries++;
    if (S.tries >= 3) {
      S.shards = 5;
      alert(`こたえは「にじ」でした!${S.uni}が代わりに扉をこじあけた。`);
      go('s5');
    } else if (S.tries === 1) {
      m.textContent = `「${a || '(無言)'}」……ちがうみたい。ヒント:雨のあとに見えるよ`;
    } else {
      m.textContent = `「${a}」もちがう!ヒント:7つの色でできてるよ(あと1回)`;
    }
  });
}

/* ---------- タイトル画面 ---------- */
function titleScreen() {
  emojiEl.textContent = '🦄';
  story.textContent = 'あなたの名前と、ユニコーンの名前をつけてあげてください。';
  choicesEl.innerHTML = '';
  const l1 = document.createElement('label'); l1.textContent = 'あなたの名前';
  const i1 = document.createElement('input'); i1.placeholder = '例: ゆうき'; i1.maxLength = 12;
  const l2 = document.createElement('label'); l2.textContent = 'ユニコーンの名前';
  const i2 = document.createElement('input'); i2.placeholder = '例: ルナ'; i2.maxLength = 12;
  const b = document.createElement('button'); b.textContent = '🌙 冒険をはじめる';
  b.onclick = () => {
    S.name = i1.value.trim() || 'あなた';
    S.uni = i2.value.trim() || 'ルナ';
    go('s1');
  };
  choicesEl.append(l1, i1, l2, i2, b);
}
titleScreen();
</script>
</body>
</html>
