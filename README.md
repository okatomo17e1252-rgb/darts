# darts[darts_countup.html](https://github.com/user-attachments/files/28778685/darts_countup.html)
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ダーツ カウントアップ</title>
    <style>
        :root {
            --bg: #0f0f1a;
            --surface: #1e1e30;
            --surface2: #252540;
            --bull: #ffd700;
            --miss: #777;
            --accent: #4fc3f7;
            --text: #e8e8e8;
            --green: #66bb6a;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; }

        html, body {
            height: 100vh;
            overflow: hidden;
            background: var(--bg);
            color: var(--text);
            font-family: 'Segoe UI', Meiryo, 'Yu Gothic', sans-serif;
        }

        /* 各画面は画面全体を覆うflex列 */
        .screen {
            position: fixed;
            inset: 0;
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 1.5rem 2rem;
            overflow: hidden;
        }
        .screen.active { display: flex; }

        /* ===== 共通部品 ===== */
        .key {
            background: var(--surface2);
            border: 1px solid #555;
            border-radius: 6px;
            padding: 0.2rem 0.65rem;
            font-size: 0.95rem;
            font-weight: bold;
            min-width: 2rem;
            text-align: center;
            flex-shrink: 0;
        }
        .key.k-bull { color: var(--bull); border-color: var(--bull); }
        .badge {
            background: var(--surface);
            padding: 0.3rem 0.9rem;
            border-radius: 20px;
            font-size: 0.8rem;
            color: var(--miss);
        }

        /* ===== スタート画面 ===== */
        #screen-start { gap: 0; }
        .start-title {
            font-size: 2.6rem;
            font-weight: 900;
            color: var(--bull);
            letter-spacing: 0.12em;
            margin-bottom: 0.2rem;
        }
        .start-sub {
            color: var(--miss);
            font-size: 0.9rem;
            margin-bottom: 2rem;
        }
        .mode-row {
            display: flex;
            gap: 1.5rem;
            justify-content: center;
            margin-bottom: 2rem;
        }
        .mode-btn {
            background: var(--surface);
            border: 2px solid #333;
            color: var(--text);
            padding: 1.6rem 2.4rem;
            font-size: 1.3rem;
            cursor: pointer;
            border-radius: 14px;
            text-align: center;
            transition: border-color 0.2s, background 0.2s;
            min-width: 155px;
        }
        .mode-btn:hover { background: var(--surface2); }
        .mode-btn.selected { border-color: var(--accent); background: var(--surface2); }
        .mode-btn .lbl { font-weight: bold; display: block; }
        .mode-btn .desc { font-size: 0.78rem; color: var(--miss); display: block; margin-top: 0.35rem; }

        .key-guide-box {
            background: var(--surface);
            border-radius: 10px;
            padding: 1.2rem 2rem;
            margin-bottom: 1rem;
            width: 100%;
            max-width: 480px;
        }
        .kg-title { font-size: 0.82rem; color: var(--miss); text-align: center; margin-bottom: 0.9rem; }
        .kg-rows { display: grid; grid-template-columns: repeat(2, 1fr); gap: 0.6rem 2rem; }
        .kg-item { display: flex; align-items: center; gap: 0.6rem; font-size: 0.88rem; }
        .start-hint { color: var(--accent); font-size: 0.82rem; margin-top: 0.8rem; }

        /* ===== ゲーム画面 ===== */
        #screen-game {
            align-items: stretch;
            justify-content: flex-start;
            gap: 0.7rem;
        }

        /* 上段：スコア（左）| ラウンド（中央）| ブル数（右） */
        .game-top {
            display: grid;
            grid-template-columns: 1fr auto 1fr;
            align-items: end;
            flex-shrink: 0;
            padding-bottom: 0.5rem;
            border-bottom: 1px solid #2a2a42;
            gap: 0 1rem;
        }
        .score-block {}
        .score-lbl { font-size: 0.75rem; color: var(--miss); letter-spacing: 0.08em; text-transform: uppercase; }
        .total-score {
            font-size: 5.5rem;
            font-weight: 900;
            color: var(--bull);
            line-height: 1;
            font-variant-numeric: tabular-nums;
            transition: color 0.25s;
        }
        .total-score.flash { color: var(--green); }
        .hs-line { font-size: 0.78rem; color: var(--miss); margin-top: 0.15rem; }

        .round-block { text-align: center; }
        .round-lbl {
            font-size: 0.72rem;
            color: var(--miss);
            letter-spacing: 0.15em;
            text-transform: uppercase;
        }
        .round-num {
            font-size: 4.8rem;
            font-weight: 900;
            color: var(--accent);
            line-height: 1;
            font-variant-numeric: tabular-nums;
        }
        .round-of {
            font-size: 2rem;
            font-weight: normal;
            color: var(--miss);
        }
        .mode-line { font-size: 0.8rem; color: var(--miss); margin-top: 0.2rem; }

        .bull-block { text-align: right; }
        .bull-lbl {
            font-size: 0.72rem;
            color: var(--miss);
            letter-spacing: 0.15em;
            text-transform: uppercase;
        }
        .bull-num {
            font-size: 2.4rem;
            font-weight: 900;
            color: var(--bull);
            line-height: 1;
            font-variant-numeric: tabular-nums;
        }
        .bull-of {
            font-size: 1.2rem;
            font-weight: normal;
            color: var(--miss);
        }

        /* 前ラウンド結果 */
        .last-panel {
            background: var(--surface);
            border-radius: 10px;
            padding: 0.7rem 1.2rem;
            flex-shrink: 0;
        }
        .last-lbl { font-size: 0.72rem; color: var(--miss); margin-bottom: 0.4rem; }
        .last-darts { display: flex; gap: 0.7rem; align-items: center; flex-wrap: wrap; }
        .dart-chip {
            padding: 0.25rem 0.7rem;
            border-radius: 7px;
            font-size: 0.95rem;
            font-weight: bold;
            background: var(--surface2);
        }
        .dc-bull { color: var(--bull); }
        .dc-miss { color: var(--miss); }
        .round-total-tag { margin-left: auto; font-size: 1.1rem; font-weight: bold; }

        /* 履歴テーブル（残りのスペースを埋める） */
        .history-wrap {
            background: var(--surface);
            border-radius: 10px;
            padding: 0.6rem;
            flex: 1;
            overflow-y: auto;
            min-height: 0;
        }
        table.ht { width: 100%; border-collapse: collapse; font-size: 0.86rem; }
        .ht th {
            text-align: center;
            padding: 0.3rem 0.4rem;
            color: var(--miss);
            border-bottom: 1px solid #333;
            position: sticky;
            top: 0;
            background: var(--surface);
        }
        .ht td { text-align: center; padding: 0.3rem 0.4rem; border-bottom: 1px solid #1a1a2e; }
        .ht tr:last-child td { border-bottom: none; }
        .ht .cb { color: var(--bull); }
        .ht .cc { font-weight: bold; }

        /* キー操作ガイド */
        .input-row {
            display: flex;
            justify-content: center;
            gap: 0.6rem;
            flex-shrink: 0;
            flex-wrap: wrap;
        }
        .input-chip {
            background: var(--surface);
            border: 1px solid #3a3a55;
            border-radius: 7px;
            padding: 0.4rem 0.9rem;
            font-size: 0.82rem;
            display: flex;
            align-items: center;
            gap: 0.4rem;
            color: var(--text);
        }

        /* ===== リザルト画面 ===== */
        #screen-result {
            align-items: stretch;
            justify-content: flex-start;
            gap: 0.8rem;
        }
        .result-top {
            display: flex;
            align-items: flex-end;
            justify-content: space-between;
            flex-shrink: 0;
            padding-bottom: 0.6rem;
            border-bottom: 1px solid #2a2a42;
        }
        .result-score-block {}
        .res-lbl { font-size: 0.75rem; color: var(--miss); letter-spacing: 0.08em; text-transform: uppercase; }
        .result-score {
            font-size: 5.5rem;
            font-weight: 900;
            color: var(--bull);
            line-height: 1;
        }
        .result-hs {
            font-size: 0.82rem;
            color: var(--accent);
            margin-top: 0.15rem;
        }
        .new-rec {
            display: inline-block;
            background: var(--bull);
            color: #0f0f1a;
            padding: 0.1rem 0.45rem;
            border-radius: 4px;
            font-size: 0.7rem;
            font-weight: bold;
            margin-left: 0.4rem;
            vertical-align: middle;
        }
        .result-right { text-align: right; }
        .result-mode-lbl { font-size: 0.75rem; color: var(--miss); letter-spacing: 0.1em; text-transform: uppercase; }
        .result-rounds-big {
            font-size: 2.2rem;
            font-weight: 900;
            color: var(--miss);
            line-height: 1;
        }
        .result-table-wrap {
            background: var(--surface);
            border-radius: 10px;
            padding: 0.6rem;
            flex: 1;
            overflow-y: auto;
            min-height: 0;
        }
        .result-btns {
            display: flex;
            gap: 1rem;
            justify-content: center;
            flex-shrink: 0;
        }
        .btn {
            padding: 0.7rem 2rem;
            border-radius: 8px;
            border: none;
            font-size: 0.95rem;
            cursor: pointer;
            transition: opacity 0.2s;
        }
        .btn:hover { opacity: 0.82; }
        .btn-primary { background: var(--accent); color: #0f0f1a; font-weight: bold; }
        .btn-sec { background: var(--surface); color: var(--text); border: 1px solid #444; }
        .result-hint { text-align: center; font-size: 0.78rem; color: var(--miss); flex-shrink: 0; }

        ::-webkit-scrollbar { width: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #3a3a55; border-radius: 2px; }
    </style>
</head>
<body>

<!-- ========== スタート画面 ========== -->
<div id="screen-start" class="screen active">
    <h1 class="start-title">DARTS COUNT UP</h1>
    <p class="start-sub">ダーツ カウントアップ練習アプリ</p>

    <div class="mode-row">
        <button class="mode-btn selected" data-mode="normal" onclick="selectMode('normal')">
            <span class="lbl">通常モード</span>
            <span class="desc">8ラウンド / 24投</span>
        </button>
        <button class="mode-btn" data-mode="musou" onclick="selectMode('musou')">
            <span class="lbl">無双モード</span>
            <span class="desc">80ラウンド / 240投</span>
        </button>
    </div>

    <div class="key-guide-box">
        <p class="kg-title">キーボード操作 — 1ラウンドにつき1回キーを押す</p>
        <div class="kg-rows">
            <div class="kg-item"><span class="key k-bull">0</span>ブル0本（はずれ3本）</div>
            <div class="kg-item"><span class="key k-bull">1</span>ブル1本 + はずれ2本</div>
            <div class="kg-item"><span class="key k-bull">2</span>ブル2本 + はずれ1本</div>
            <div class="kg-item"><span class="key k-bull">3</span>ブル3本（トリプルブル）</div>
        </div>
    </div>

    <p class="start-hint">← / → でモード選択 &nbsp;|&nbsp; Enter でスタート</p>
</div>

<!-- ========== ゲーム画面 ========== -->
<div id="screen-game" class="screen">

    <!-- 上段：スコア（左）| ラウンド（中央）| ブル数（右） -->
    <div class="game-top">
        <div class="score-block">
            <p class="score-lbl">Total Score</p>
            <div class="total-score" id="total-score">0</div>
            <p class="hs-line" id="hs-info">ベスト: ---</p>
        </div>
        <div class="round-block">
            <p class="round-lbl">Round</p>
            <div class="round-num">
                <span id="round-cur">1</span><span class="round-of"> / <span id="round-total-disp">8</span></span>
            </div>
            <p class="mode-line" id="mode-badge">通常モード</p>
        </div>
        <div class="bull-block">
            <p class="bull-lbl">Bull</p>
            <div class="bull-num">
                <span id="bulls-hit">0</span><span class="bull-of"> / <span id="darts-thrown">0</span></span>
            </div>
        </div>
    </div>

    <!-- 前ラウンド結果 -->
    <div class="last-panel">
        <p class="last-lbl" id="last-lbl">前のラウンド</p>
        <div class="last-darts" id="last-darts">
            <span style="color:var(--miss);font-size:0.85rem">まだありません</span>
        </div>
    </div>

    <!-- 履歴（残りスペースを埋める） -->
    <div class="history-wrap">
        <table class="ht">
            <thead>
                <tr>
                    <th>R</th>
                    <th>内訳</th>
                    <th>R得点</th>
                    <th>累計</th>
                </tr>
            </thead>
            <tbody id="history-body"></tbody>
        </table>
    </div>

    <!-- キーガイド -->
    <div class="input-row">
        <div class="input-chip"><span class="key k-bull" style="font-size:.8rem">0</span>はずれ×3</div>
        <div class="input-chip"><span class="key k-bull" style="font-size:.8rem">1</span>ブル×1</div>
        <div class="input-chip"><span class="key k-bull" style="font-size:.8rem">2</span>ブル×2</div>
        <div class="input-chip"><span class="key k-bull" style="font-size:.8rem">3</span>ブル×3</div>
        <div class="input-chip"><span class="key" style="font-size:.8rem">R</span>リスタート</div>
    </div>
</div>

<!-- ========== リザルト画面 ========== -->
<div id="screen-result" class="screen">

    <!-- 上段：スコア（左）+ ラウンド完了（右） -->
    <div class="result-top">
        <div class="result-score-block">
            <p class="res-lbl">Final Score</p>
            <div class="result-score" id="res-score">0</div>
            <p class="result-hs" id="res-hs"></p>
        </div>
        <div class="result-right">
            <p class="result-mode-lbl" id="res-mode-lbl">通常モード</p>
            <div class="result-rounds-big" id="res-rounds">8R 完了</div>
        </div>
    </div>

    <!-- 全ラウンド履歴 -->
    <div class="result-table-wrap">
        <table class="ht">
            <thead>
                <tr>
                    <th>R</th>
                    <th>内訳</th>
                    <th>R得点</th>
                    <th>累計</th>
                </tr>
            </thead>
            <tbody id="res-history-body"></tbody>
        </table>
    </div>

    <div class="result-btns">
        <button class="btn btn-primary" onclick="restartSame()">もう一度（同モード）</button>
        <button class="btn btn-sec" onclick="goStart()">モード選択へ</button>
    </div>
    <p class="result-hint">Enter: もう一度 &nbsp;|&nbsp; Esc: モード選択へ</p>
</div>

<script>
    const S = {
        screen: 'start',
        mode: 'normal',
        totalRounds: 8,
        currentRound: 0,
        totalScore: 0,
        bullsHit: 0,
        rounds: []
    };

    function getHS(mode) {
        return parseInt(localStorage.getItem('darts_hs_' + mode) || '0');
    }
    function saveHS(mode, score) {
        if (score > getHS(mode)) {
            localStorage.setItem('darts_hs_' + mode, score);
            return true;
        }
        return false;
    }

    function selectMode(mode) {
        S.mode = mode;
        document.querySelectorAll('.mode-btn').forEach(b => {
            b.classList.toggle('selected', b.dataset.mode === mode);
        });
    }

    function showScreen(name) {
        S.screen = name;
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById('screen-' + name).classList.add('active');
    }

    function startGame() {
        S.totalRounds = S.mode === 'normal' ? 8 : 80;
        S.currentRound = 0;
        S.totalScore = 0;
        S.rounds = [];

        document.getElementById('mode-badge').textContent = S.mode === 'normal' ? '通常モード' : '無双モード';
        document.getElementById('round-total-disp').textContent = S.totalRounds;
        const hs = getHS(S.mode);
        document.getElementById('hs-info').textContent = 'ベスト: ' + (hs > 0 ? hs : '---');
        S.bullsHit = 0;
        document.getElementById('last-lbl').textContent = '前のラウンド';
        document.getElementById('last-darts').innerHTML = '<span style="color:var(--miss);font-size:0.85rem">まだありません</span>';
        document.getElementById('history-body').innerHTML = '';
        document.getElementById('total-score').textContent = '0';
        document.getElementById('total-score').classList.remove('flash');
        document.getElementById('bulls-hit').textContent = '0';
        document.getElementById('darts-thrown').textContent = '0';

        showScreen('game');
        updateRoundCounter();
    }

    function inputRound(bulls) {
        if (S.screen !== 'game') return;
        if (S.currentRound >= S.totalRounds) return;

        const misses = 3 - bulls;
        const darts = [];
        for (let i = 0; i < bulls; i++) darts.push({ type: 'bull', score: 50 });
        for (let i = 0; i < misses; i++) darts.push({ type: 'miss', score: Math.floor(Math.random() * 20) + 1 });
        // 投げる順番をシャッフル
        for (let i = darts.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [darts[i], darts[j]] = [darts[j], darts[i]];
        }

        const roundTotal = darts.reduce((s, d) => s + d.score, 0);
        S.totalScore += roundTotal;
        S.bullsHit += bulls;
        S.currentRound++;

        const rd = { round: S.currentRound, darts, roundTotal, cumulative: S.totalScore };
        S.rounds.push(rd);

        // スコアフラッシュ
        const scoreEl = document.getElementById('total-score');
        scoreEl.textContent = S.totalScore;
        scoreEl.classList.add('flash');
        setTimeout(() => scoreEl.classList.remove('flash'), 280);

        // ブル数更新
        document.getElementById('bulls-hit').textContent = S.bullsHit;
        document.getElementById('darts-thrown').textContent = S.currentRound * 3;

        // 前ラウンド結果表示
        document.getElementById('last-lbl').textContent = `Round ${rd.round} の結果`;
        const dartsHTML = rd.darts.map(d =>
            `<span class="dart-chip ${d.type === 'bull' ? 'dc-bull' : 'dc-miss'}">${d.type === 'bull' ? 'BULL' : d.score}</span>`
        ).join('');
        document.getElementById('last-darts').innerHTML =
            dartsHTML + `<span class="round-total-tag">+${rd.roundTotal}</span>`;

        // 履歴行追加
        const tbody = document.getElementById('history-body');
        const tr = document.createElement('tr');
        const dartsCells = rd.darts.map(d =>
            d.type === 'bull' ? '<span class="cb">B</span>' : d.score
        ).join(' / ');
        tr.innerHTML = `<td>${rd.round}</td><td>${dartsCells}</td><td>${rd.roundTotal}</td><td class="cc">${rd.cumulative}</td>`;
        tbody.appendChild(tr);
        tbody.parentElement.scrollTop = tbody.parentElement.scrollHeight;

        if (S.currentRound >= S.totalRounds) {
            setTimeout(showResult, 500);
        } else {
            updateRoundCounter();
        }
    }

    function updateRoundCounter() {
        document.getElementById('round-cur').textContent = S.currentRound + 1;
    }

    function showResult() {
        const isNew = saveHS(S.mode, S.totalScore);
        const hs = getHS(S.mode);
        const modeLabel = S.mode === 'normal' ? '通常モード' : '無双モード';

        document.getElementById('res-mode-lbl').textContent = modeLabel;
        document.getElementById('res-score').textContent = S.totalScore;
        document.getElementById('res-rounds').textContent = `${S.totalRounds}R 完了`;
        document.getElementById('res-hs').innerHTML =
            `ベスト: ${hs}` + (isNew ? '<span class="new-rec">NEW RECORD</span>' : '');

        const tbody = document.getElementById('res-history-body');
        tbody.innerHTML = '';
        S.rounds.forEach(rd => {
            const tr = document.createElement('tr');
            const dartsCells = rd.darts.map(d =>
                d.type === 'bull' ? '<span class="cb">B</span>' : d.score
            ).join(' / ');
            tr.innerHTML = `<td>${rd.round}</td><td>${dartsCells}</td><td>${rd.roundTotal}</td><td class="cc">${rd.cumulative}</td>`;
            tbody.appendChild(tr);
        });

        showScreen('result');
    }

    function restartSame() { startGame(); }
    function goStart() { showScreen('start'); }

    function confirmRestart() {
        if (S.currentRound > 0 && S.currentRound < S.totalRounds) {
            if (confirm('ゲームをリスタートしますか？')) goStart();
        } else {
            goStart();
        }
    }

    document.addEventListener('keydown', e => {
        switch (S.screen) {
            case 'start':
                if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
                    selectMode(S.mode === 'normal' ? 'musou' : 'normal');
                } else if (e.key === 'Enter') {
                    startGame();
                }
                break;
            case 'game':
                if (e.key === '0') inputRound(0);
                else if (e.key === '1') inputRound(1);
                else if (e.key === '2') inputRound(2);
                else if (e.key === '3') inputRound(3);
                else if (e.key === 'r' || e.key === 'R') confirmRestart();
                break;
            case 'result':
                if (e.key === 'Enter') restartSame();
                else if (e.key === 'Escape') goStart();
                break;
        }
    });
</script>
</body>
</html>
