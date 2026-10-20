<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Тетрис: Абсолютный Микс</title>
<style>
    * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; -webkit-tap-highlight-color: transparent; }
    body { background: #0a1929; color: #e0e6ed; font-family: 'Segoe UI', Arial, sans-serif; display: flex; flex-direction: column; align-items: center; height: 100vh; overflow: hidden; transition: background 0.3s; }
    #game-container { display: flex; flex-direction: column; width: 100%; height: 100%; position: relative; }
    #ui { display: flex; justify-content: space-between; align-items: center; width: 100%; padding: 10px 15px; background: rgba(17, 45, 78, 0.95); font-size: 14px; font-weight: bold; z-index: 5; }
    #modeName { color: #feca57; text-transform: uppercase; letter-spacing: 1px; }
    #canvas-wrap { flex: 1; display: flex; justify-content: center; align-items: center; padding: 10px; position: relative; }
    canvas { background: #0a1929; border: 2px solid #1f6feb; border-radius: 8px; box-shadow: 0 0 20px rgba(31, 111, 235, 0.5); max-height: 100%; max-width: 100%; }
    #controls { display: flex; justify-content: space-around; padding: 10px; background: rgba(17, 45, 78, 0.95); gap: 5px; }
    .btn { background: #1f6feb; color: white; border: none; border-radius: 8px; padding: 15px 0; flex: 1; font-size: 18px; font-weight: bold; touch-action: manipulation; cursor: pointer; }
    .btn:active { background: #4da6ff; }
    .btn-special { background: #e17055; }
    .btn-special:active { background: #ff9f80; }
    .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: #0a1929; display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 10; padding: 20px; text-align: center; transition: background 0.3s; }
    .screen h1 { color: #4da6ff; font-size: 38px; margin-bottom: 5px; text-shadow: 0 0 10px #4da6ff; }
    .screen p { color: #00ff88; font-size: 16px; margin-bottom: 30px; }
    .menu-btn { width: 260px; padding: 15px; margin: 8px; font-size: 16px; background: rgba(17, 45, 78, 0.9); color: white; border: 2px solid #1f6feb; border-radius: 10px; font-weight: bold; cursor: pointer; transition: 0.2s; }
    .menu-btn:active, .menu-btn:hover { background: #1f6feb; }
    #leaderboard table { width: 90%; max-width: 350px; border-collapse: collapse; margin: 20px 0; background: rgba(17, 45, 78, 0.9); border-radius: 8px; overflow: hidden; }
    #leaderboard td, #leaderboard th { border-bottom: 1px solid #1f6feb; padding: 10px; text-align: left; font-size: 14px; }
    #leaderboard th { background: #1f6feb; color: white; }
    .settings-row { display: flex; justify-content: space-between; align-items: center; width: 100%; max-width: 300px; padding: 12px 15px; background: rgba(17, 45, 78, 0.8); border-radius: 8px; margin-bottom: 10px; font-size: 16px; }
    .toggle-switch { position: relative; width: 50px; height: 28px; background: #555; border-radius: 14px; cursor: pointer; transition: 0.3s; }
    .toggle-switch.active { background: #1f6feb; }
    .toggle-switch::after { content: ''; position: absolute; width: 24px; height: 24px; border-radius: 50%; background: white; top: 2px; left: 2px; transition: 0.3s; }
    .toggle-switch.active::after { left: 24px; }
    .theme-btn { padding: 10px; border-radius: 8px; border: 2px solid transparent; cursor: pointer; font-weight: bold; margin: 0 5px; color: white; }
    .theme-btn.active { border-color: white; box-shadow: 0 0 10px rgba(255,255,255,0.5); }
    .hidden { display: none !important; }
</style>
</head>
<body>

<div id="game-container">
    <div id="ui">
        <div id="score">Очки: 0</div>
        <div id="modeName">АБСОЛЮТНЫЙ МИКС</div>
        <div id="level">Ур: 1</div>
    </div>
    <div id="canvas-wrap">
        <canvas id="gameCanvas"></canvas>
    </div>
    <div id="controls">
        <button class="btn" onclick="moveLeft()">←</button>
        <button class="btn" onclick="rotatePiece()">↻</button>
        <button class="btn" onclick="moveRight()">→</button>
        <button class="btn btn-special" onclick="hardDrop()">⤓</button>
    </div>
</div>

<div id="menu" class="screen">
    <h1>ТЕТРИС</h1>
    <p>АБСОЛЮТНЫЙ МИКС 🦎💣🔥</p>
    <button class="menu-btn" onclick="startGame()" style="font-size: 20px; padding: 20px;">▶ ИГРАТЬ</button>
    <button class="menu-btn" onclick="showLeaderboard()" style="background: transparent; border-color: #feca57; color: #feca57;">🏆 Лидеры</button>
    <button class="menu-btn" onclick="showSettings()" style="background: transparent; border-color: #4da6ff; color: #4da6ff;">⚙️ Настройки</button>
</div>

<div id="settings" class="screen hidden">
    <h2 style="color: #4da6ff; margin-bottom: 20px;">⚙️ НАСТРОЙКИ</h2>
    <div class="settings-row"><span>🔊 Звуки</span><div class="toggle-switch" id="toggle-sound" onclick="toggleSetting('sound')"></div></div>
    <div class="settings-row"><span>📳 Тряска экрана</span><div class="toggle-switch" id="toggle-shake" onclick="toggleSetting('shake')"></div></div>
    <div class="settings-row"><span>✨ Частицы (эффекты)</span><div class="toggle-switch" id="toggle-particles" onclick="toggleSetting('particles')"></div></div>
    <div style="margin-top: 20px; margin-bottom: 10px; color: #e0e6ed; font-weight: bold;">🎨 Тема меню</div>
    <div style="display: flex; margin-bottom: 30px;">
        <div class="theme-btn" style="background: #0a1929;" onclick="setTheme('space')">Космос</div>
        <div class="theme-btn" style="background: #1a001a;" onclick="setTheme('neon')">Неон</div>
        <div class="theme-btn" style="background: #0a1f0a;" onclick="setTheme('forest')">Лес</div>
        <div class="theme-btn" style="background: #001a33;" onclick="setTheme('ocean')">Океан</div>
    </div>
    <button class="menu-btn" onclick="hideSettings()">← Назад</button>
</div>

<div id="leaderboard" class="screen hidden">
    <h2 style="color: #4da6ff; margin-bottom: 20px;">🏆 ЛУЧШИЕ ИГРОКИ</h2>
    <table id="lbTable"><tr><th>#</th><th>Имя</th><th>Очки</th></tr></table>
    <button class="menu-btn" onclick="hideLeaderboard()" style="margin-top: 20px;">← Назад</button>
</div>

<script>
    let config = { sound: true, shake: true, particles: true, theme: 'space' };
    const THEMES = {
        space: { bg: "#0a1929", ui: "rgba(17, 45, 78, 0.95)" },
        neon: { bg: "#1a001a", ui: "rgba(51, 0, 51, 0.95)" },
        forest: { bg: "#0a1f0a", ui: "rgba(20, 48, 20, 0.95)" },
        ocean: { bg: "#001a33", ui: "rgba(0, 40, 80, 0.95)" }
    };

    function loadSettings() {
        const saved = localStorage.getItem('tetris_config');
        if (saved) { try { config = Object.assign(config, JSON.parse(saved)); } catch(e){} }
        applyTheme();
        document.getElementById('toggle-sound').classList.toggle('active', config.sound);
        document.getElementById('toggle-shake').classList.toggle('active', config.shake);
        document.getElementById('toggle-particles').classList.toggle('active', config.particles);
    }

    function saveSettings() { localStorage.setItem('tetris_config', JSON.stringify(config)); }
    function toggleSetting(key) { config[key] = !config[key]; saveSettings(); document.getElementById('toggle-' + key).classList.toggle('active', config[key]); if (key === 'sound' && config.sound) initAudio(); }
    function setTheme(theme) { config.theme = theme; saveSettings(); applyTheme(); }
    function applyTheme() {
        const t = THEMES[config.theme];
        document.body.style.background = t.bg;
        document.querySelectorAll('.screen').forEach(el => el.style.background = t.bg);
        document.getElementById('ui').style.background = t.ui;
        document.getElementById('controls').style.background = t.ui;
        document.querySelectorAll('.theme-btn').forEach(el => el.classList.remove('active'));
        const btn = document.querySelector(`.theme-btn[onclick="setTheme('${config.theme}')"]`);
        if (btn) btn.classList.add('active');
    }

    function showSettings() { document.getElementById('menu').classList.add('hidden'); document.getElementById('settings').classList.remove('hidden'); }
    function hideSettings() { document.getElementById('settings').classList.add('hidden'); document.getElementById('menu').classList.remove('hidden'); }
    function showLeaderboard() { updateLB(); document.getElementById('menu').classList.add('hidden'); document.getElementById('leaderboard').classList.remove('hidden'); }
    function hideLeaderboard() { document.getElementById('leaderboard').classList.add('hidden'); document.getElementById('menu').classList.remove('hidden'); }

    let audioCtx;
    function initAudio() { if (!config.sound) return; if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)(); if (audioCtx.state === 'suspended') audioCtx.resume(); }
    function playTone(freq, type, dur, vol=0.1) {
        if (!config.sound || !audioCtx) return;
        try {
            const osc = audioCtx.createOscillator(), gain = audioCtx.createGain();
            osc.type = type; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(vol, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
            osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime + dur);
        } catch(e) {}
    }
    const SFX = {
        move: () => playTone(200, 'square', 0.05, 0.05), rotate: () => playTone(350, 'square', 0.08, 0.05), drop: () => playTone(150, 'triangle', 0.1, 0.1),
        clear1: () => { playTone(400, 'square', 0.1, 0.1); setTimeout(()=>playTone(600, 'square', 0.15, 0.1), 80); },
        clear2: () => { [400, 600, 800].forEach((f,i) => setTimeout(()=>playTone(f, 'square', 0.12, 0.1), i*70)); },
        clear3: () => { [400, 600, 800, 1000].forEach((f,i) => setTimeout(()=>playTone(f, 'square', 0.15, 0.1), i*60)); },
        tetris: () => { [523, 659, 784, 1046].forEach((f,i) => setTimeout(()=>playTone(f, 'square', 0.2, 0.15), i*50)); },
        bomb: () => { 
            if(!config.sound || !audioCtx) return;
            try { const osc = audioCtx.createOscillator(), gain = audioCtx.createGain(); osc.type = 'sawtooth'; osc.frequency.setValueAtTime(100, audioCtx.currentTime); osc.frequency.exponentialRampToValueAtTime(10, audioCtx.currentTime + 0.3); gain.gain.setValueAtTime(0.3, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.3); osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime + 0.3); } catch(e){}
        },
        chameleon: () => { [800, 1000, 1200].forEach((f,i) => setTimeout(()=>playTone(f, 'sine', 0.15, 0.15), i*60)); },
        adrenaline: () => { [300, 400, 600, 800].forEach((f,i) => setTimeout(()=>playTone(f, 'sawtooth', 0.1, 0.1), i*40)); },
        gameover: () => { [400, 300, 200, 100].forEach((f,i) => setTimeout(()=>playTone(f, 'square', 0.3, 0.2), i*200)); }
    };

    const COLS = 10, ROWS = 20;
    let CELL = 30;
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const PALETTES = [
        { bg: "#0a1929", grid: "#112d4e", I: "#00f0f0", O: "#f0f000", T: "#a000f0", S: "#00f000", Z: "#f00000", J: "#0000f0", L: "#f0a000" },
        { bg: "#1a001a", grid: "#330033", I: "#00ffff", O: "#ffff00", T: "#ff00ff", S: "#00ff00", Z: "#ff0000", J: "#0000ff", L: "#ff8800" },
        { bg: "#0a1f0a", grid: "#143014", I: "#00ccaa", O: "#cccc00", T: "#aa00aa", S: "#00cc00", Z: "#cc0000", J: "#0000cc", L: "#cc8800" }
    ];
    let currentPalette = PALETTES[0];
    const SHAPES = { 'I': [[1, 1, 1, 1]], 'O': [[1, 1], [1, 1]], 'T': [[0, 1, 0], [1, 1, 1]], 'S': [[0, 1, 1], [1, 1, 0]], 'Z': [[1, 1, 0], [0, 1, 1]], 'J': [[1, 0, 0], [1, 1, 1]], 'L': [[0, 0, 1], [1, 1, 1]] };
    
    const images = {};
    function createImage(key, color) {
        const svg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><rect x="5" y="5" width="90" height="90" fill="${color}" stroke="#fff" stroke-width="4" rx="8"/><rect x="15" y="15" width="70" height="70" fill="none" stroke="#fff" stroke-width="2" rx="4"/><polygon points="15,15 50,15 15,50" fill="#ffffff60"/></svg>`;
        const img = new Image(); img.src = 'data:image/svg+xml;utf8,' + encodeURIComponent(svg); images[key] = img;
    }
    for (let k in PALETTES[0]) { if (k !== 'bg' && k !== 'grid') createImage(k, PALETTES[0][k]); }
    const bombSvg = `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="55" r="35" fill="#2d3436" stroke="#e17055" stroke-width="5"/><line x1="50" y1="20" x2="65" y2="5" stroke="#e17055" stroke-width="6"/><circle cx="67" cy="5" r="6" fill="#ffeb3b"/></svg>`;
    const bombImg = new Image(); bombImg.src = 'data:image/svg+xml;utf8,' + encodeURIComponent(bombSvg); images['BOMB'] = bombImg;

    let board, current, next, score, lines, level, paused, gameOver, speed, timer, bag = [], adrenalineTimer = 0, adrenalineMult = 1, particles = [], shakeTimer = 0;

    function resizeCanvas() {
        const wrap = document.getElementById('canvas-wrap');
        CELL = Math.min(Math.floor((wrap.clientWidth - 20) / COLS), Math.floor((wrap.clientHeight - 20) / ROWS));
        canvas.width = COLS * CELL; canvas.height = ROWS * CELL; draw();
    }
    window.addEventListener('resize', resizeCanvas);

    function startGame() {
        initAudio();
        document.getElementById('menu').classList.add('hidden');
        document.getElementById('settings').classList.add('hidden');
        document.getElementById('leaderboard').classList.add('hidden');
        resizeCanvas();
        board = Array.from({ length: ROWS }, () => Array(COLS).fill(null));
        score = 0; lines = 0; level = 1; paused = false; gameOver = false; speed = 450;
        bag = []; adrenalineTimer = 0; adrenalineMult = 1; currentPalette = PALETTES[0]; particles = []; shakeTimer = 0;
        current = newPiece(); next = newPiece(); current.x = Math.floor((COLS - current.shape[0].length) / 2); current.y = 0;
        updateUI(); if (timer) clearInterval(timer); timer = setInterval(tick, speed);
        requestAnimationFrame(animateEffects); draw();
    }

    function newPiece() {
        if (bag.length === 0) bag = Object.keys(SHAPES).sort(() => Math.random() - 0.5);
        const key = bag.pop(); let special = null;
        if (Math.random() < 0.20) special = Math.random() < 0.5 ? 'bomb' : 'chameleon';
        if (special === 'bomb') return { key: 'BOMB', shape: [[1]], special: 'bomb' };
        return { key: key, shape: SHAPES[key].map(r => [...r]), special: special };
    }

    function tick() {
        if (paused || gameOver) return;
        if (adrenalineTimer > 0) { adrenalineTimer--; if (adrenalineTimer === 0) adrenalineMult = 1; }
        move(0, 1); draw();
    }

    function move(dx, dy) {
        if (gameOver || paused) return;
        if (!collide(current.shape, current.x + dx, current.y + dy)) { current.x += dx; current.y += dy; if (dx !== 0) SFX.move(); }
        else if (dy === 1) lockPiece();
    }

    function rotatePiece() {
        if (gameOver || paused || current.special === 'bomb') return;
        const rotated = current.shape[0].map((_, i) => current.shape.map(row => row[i]).reverse());
        for (let dx of [0, -1, 1, -2, 2]) {
            if (!collide(rotated, current.x + dx, current.y)) { current.shape = rotated; current.x += dx; SFX.rotate(); draw(); return; }
        }
    }

    function hardDrop() {
        if (gameOver || paused) return;
        while (!collide(current.shape, current.x, current.y + 1)) { current.y++; score += 2 * adrenalineMult; }
        SFX.drop(); lockPiece(); updateUI(); draw();
    }

    function collide(shape, x, y) {
        for (let r = 0; r < shape.length; r++) for (let c = 0; c < shape[r].length; c++) {
            if (shape[r][c]) { const nx = x + c, ny = y + r; if (nx < 0 || nx >= COLS || ny >= ROWS) return true; if (ny >= 0 && board[ny][nx]) return true; }
        }
        return false;
    }

    function createExplosion(x, y, color) {
        if (!config.particles) return;
        for (let i = 0; i < 12; i++) particles.push({ x: x * CELL + CELL/2, y: y * CELL + CELL/2, vx: (Math.random() - 0.5) * 10, vy: (Math.random() - 0.5) * 10, life: 1, color: color });
    }

    function lockPiece() {
        if (current.special === 'bomb') {
            SFX.bomb(); if (config.shake) shakeTimer = 15;
            for (let r = Math.max(0, current.y - 1); r <= Math.min(ROWS - 1, current.y + 1); r++) {
                for (let c = Math.max(0, current.x - 1); c <= Math.min(COLS - 1, current.x + 1); c++) {
                    if (board[r][c]) createExplosion(c, r, board[r][c].color); board[r][c] = null;
                }
            }
            createExplosion(current.x, current.y, '#e17055'); score += 50; spawnNext(); return;
        }
        for (let r = 0; r < current.shape.length; r++) for (let c = 0; c < current.shape[r].length; c++) {
            if (current.shape[r][c]) {
                const ny = current.y + r; if (ny < 0) { endGame(); return; }
                board[ny][current.x + c] = { key: current.key, color: currentPalette[current.key] || '#fff', special: current.special };
            }
        }
        clearLines(); spawnNext();
    }

    function spawnNext() {
        current = next; next = newPiece(); current.x = Math.floor((COLS - current.shape[0].length) / 2); current.y = 0;
        if (collide(current.shape, current.x, current.y)) endGame();
    }

    function clearLines() {
        let cleared = 0, hasChameleon = false;
        for (let r = ROWS - 1; r >= 0; r--) {
            if (board[r].every(cell => cell)) {
                if (board[r].some(cell => cell.special === 'chameleon')) hasChameleon = true;
                for (let c = 0; c < COLS; c++) if (board[r][c]) createExplosion(c, r, board[r][c].color);
                board.splice(r, 1); board.unshift(Array(COLS).fill(null)); cleared++; r++;
            }
        }
        if (cleared > 0) {
            if (cleared === 1) SFX.clear1(); else if (cleared === 2) SFX.clear2(); else if (cleared === 3) SFX.clear3(); else if (cleared >= 4) SFX.tetris();
            lines += cleared; let base = [0, 100, 300, 500, 800][cleared] || 0;
            if (hasChameleon) { SFX.chameleon(); base *= 2; speed = Math.min(800, speed + 100); }
            if (cleared >= 2) { SFX.adrenaline(); adrenalineTimer = 180; adrenalineMult = 2; } else { adrenalineTimer = 0; adrenalineMult = 1; }
            score += base * level * adrenalineMult;
            let oldLevel = level; level = Math.floor(lines / 10) + 1;
            if (level !== oldLevel) {
                let paletteIdx = Math.min(Math.floor((level - 1) / 5), PALETTES.length - 1); currentPalette = PALETTES[paletteIdx];
                for (let k in currentPalette) { if (k !== 'bg' && k !== 'grid') createImage(k, currentPalette[k]); } speed = Math.max(80, speed - 40);
            }
            clearInterval(timer); timer = setInterval(tick, speed); updateUI();
        }
    }

    function endGame() {
        SFX.gameover(); gameOver = true; clearInterval(timer); draw();
        setTimeout(() => {
            const name = prompt("GAME OVER!\nСчёт: " + score + "\nВведите имя:", "Игрок");
            if (name) saveScore(name || "Аноним", score);
            document.getElementById('menu').classList.remove('hidden');
        }, 100);
    }

    function animateEffects() {
        if (config.shake && shakeTimer > 0) shakeTimer--;
        for (let i = particles.length - 1; i >= 0; i--) { let p = particles[i]; p.x += p.vx; p.y += p.vy; p.vy += 0.5; p.life -= 0.02; if (p.life <= 0) particles.splice(i, 1); }
        if (!gameOver && !paused) draw(); requestAnimationFrame(animateEffects);
    }

    function drawBlock(x, y, color, special, ghost = false) {
        const px = x * CELL, py = y * CELL;
        if (ghost) { ctx.strokeStyle = color; ctx.lineWidth = 2; ctx.setLineDash([4, 2]); ctx.strokeRect(px + 2, py + 2, CELL - 4, CELL - 4); ctx.setLineDash([]); return; }
        if (special === 'chameleon') {
            const hue = Math.floor(Date.now() / 10) % 360; ctx.fillStyle = `hsl(${hue}, 100%, 50%)`; ctx.fillRect(px + 1, py + 1, CELL - 2, CELL - 2);
            ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.strokeRect(px + 1, py + 1, CELL - 2, CELL - 2); ctx.fillStyle = '#fff'; ctx.font = `${CELL * 0.5}px Arial`; ctx.textAlign = 'center'; ctx.textBaseline = 'middle'; ctx.fillText('🦎', px + CELL/2, py + CELL/2); return;
        }
        let imgKey = null; if (special === 'bomb') imgKey = 'BOMB'; else for (let k in currentPalette) if (currentPalette[k] === color) { imgKey = k; break; }
        if (imgKey && images[imgKey] && images[imgKey].complete) { ctx.drawImage(images[imgKey], px, py, CELL, CELL); }
        else { ctx.fillStyle = color; ctx.fillRect(px + 1, py + 1, CELL - 2, CELL - 2); ctx.strokeStyle = '#fff'; ctx.lineWidth = 2; ctx.strokeRect(px + 1, py + 1, CELL - 2, CELL - 2); }
    }

    function draw() {
        ctx.save(); if (config.shake && shakeTimer > 0) ctx.translate((Math.random() - 0.5) * 8, (Math.random() - 0.5) * 8);
        ctx.fillStyle = currentPalette.bg; ctx.fillRect(-10, -10, canvas.width + 20, canvas.height + 20);
        ctx.strokeStyle = currentPalette.grid; ctx.lineWidth = 1;
        for (let r = 0; r <= ROWS; r++) { ctx.beginPath(); ctx.moveTo(0, r * CELL); ctx.lineTo(canvas.width, r * CELL); ctx.stroke(); }
        for (let c = 0; c <= COLS; c++) { ctx.beginPath(); ctx.moveTo(c * CELL, 0); ctx.lineTo(c * CELL, canvas.height); ctx.stroke(); }
        if (board) for (let r = 0; r < ROWS; r++) for (let c = 0; c < COLS; c++) if (board[r][c]) drawBlock(c, r, board[r][c].color, board[r][c].special);
        if (current && !gameOver) {
            let gy = current.y; while (!collide(current.shape, current.x, gy + 1)) gy++;
            if (gy !== current.y) for (let r = 0; r < current.shape.length; r++) for (let c = 0; c < current.shape[r].length; c++)
                if (current.shape[r][c]) drawBlock(current.x + c, gy + r, currentPalette[current.key] || '#fff', null, true);
            for (let r = 0; r < current.shape.length; r++) for (let c = 0; c < current.shape[r].length; c++)
                if (current.shape[r][c]) drawBlock(current.x + c, current.y + r, currentPalette[current.key] || '#fff', current.special);
        }
        if (next && !gameOver) {
            const size = CELL * 0.5, ox = canvas.width - 4 * size - 10, oy = 10;
            ctx.fillStyle = 'rgba(0,0,0,0.5)'; ctx.fillRect(ox - 5, oy - 5, 4 * size + 10, 4 * size + 10);
            ctx.strokeStyle = '#4da6ff'; ctx.strokeRect(ox - 5, oy - 5, 4 * size + 10, 4 * size + 10);
            const nx = Math.floor((4 - next.shape[0].length) / 2);
            for (let r = 0; r < next.shape.length; r++) for (let c = 0; c < next.shape[r].length; c++)
                if (next.shape[r][c]) { ctx.fillStyle = currentPalette[next.key] || '#fff'; ctx.fillRect(ox + (nx + c) * size, oy + r * size, size - 1, size - 1); }
        }
        if (config.particles) { for (let p of particles) { ctx.globalAlpha = p.life; ctx.fillStyle = p.color; ctx.beginPath(); ctx.arc(p.x, p.y, 3 + p.life * 3, 0, Math.PI * 2); ctx.fill(); } ctx.globalAlpha = 1; }
        if (adrenalineMult > 1) { ctx.fillStyle = 'rgba(255, 71, 87, 0.8)'; ctx.fillRect(10, 10, 120, 30); ctx.fillStyle = '#fff'; ctx.font = 'bold 16px Arial'; ctx.textAlign = 'left'; ctx.fillText('🔥 АДРЕНАЛИН x2', 15, 32); }
        ctx.restore();
    }

    function updateUI() { document.getElementById('score').textContent = 'Очки: ' + score; document.getElementById('level').textContent = 'Ур: ' + level; }
    function moveLeft() { move(-1, 0); draw(); }
    function moveRight() { move(1, 0); draw(); }

    function saveScore(name, score) {
        let lb = JSON.parse(localStorage.getItem('tetris_mix_lb') || '[]');
        lb.push({ name, score }); lb.sort((a, b) => b.score - a.score); lb = lb.slice(0, 10);
        localStorage.setItem('tetris_mix_lb', JSON.stringify(lb));
    }
    function updateLB() {
        const lb = JSON.parse(localStorage.getItem('tetris_mix_lb') || '[]');
        const table = document.getElementById('lbTable');
        table.innerHTML = '<tr><th>#</th><th>Имя</th><th>Очки</th></tr>';
        lb.forEach((entry, i) => { const row = table.insertRow(); row.insertCell().textContent = i + 1; row.insertCell().textContent = entry.name; row.insertCell().textContent = entry.score; });
    }

    loadSettings();
    setTimeout(resizeCanvas, 100);
</script>
</body>
</html>
