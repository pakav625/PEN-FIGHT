
</body>
<!DOCTYPE html>
<html lang="gu">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pen Fight: Final Fix</title>
    <style>
        body { margin: 0; background: #121212; font-family: sans-serif; color: white; overflow: hidden; touch-action: none; display: flex; flex-direction: column; align-items: center; justify-content: flex-start; height: 100vh; }
        #game-ui { width: 95%; display: none; justify-content: space-between; align-items: center; padding: 10px 0; z-index: 10; margin-top: 5px; }
        canvas { background: #4e342e; border: 8px solid #2e1b18; border-radius: 5px; display: none; box-shadow: 0 5px 20px rgba(0,0,0,0.8); margin-top: 5px; }
        .hud { background: rgba(0,0,0,0.8); padding: 6px 10px; border-radius: 5px; border: 1.5px solid #ffd700; font-weight: bold; font-size: 0.85rem; }
        #menu, #win-screen { text-align: center; z-index: 20; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 100%; }
        #win-screen { display: none; background: rgba(18,18,18,0.98); height: 100%; flex-direction: column; align-items: center; justify-content: center; }
        .btn { background: #ffd700; color: #000; padding: 15px 40px; border: none; border-radius: 50px; font-weight: bold; font-size: 1.2rem; cursor: pointer; margin: 10px; display: inline-block; width: 220px; box-shadow: 0 4px #b8860b; }
        .small-btn { background: #444; color: white; padding: 5px 12px; border: none; border-radius: 5px; font-size: 0.75rem; cursor: pointer; margin: 0 2px; }
        #confetti { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 15; display: none; }
    </style>
</head>
<body>

    <div id="game-ui">
        <div class="hud" id="p1-score">P1: 0</div>
        <div>
            <button class="small-btn" onclick="goBack()">BACK</button>
            <button class="small-btn" onclick="showHistory()">HISTORY</button>
        </div>
        <div class="hud" id="p2-score">P2: 0</div>
    </div>

    <div id="menu">
        <h1 style="color:#ffd700; margin: 0; font-size: 2.5rem;">PEN FIGHT</h1>
        <p style="color:#888;">Stable PvP Edition</p><br>
        <button class="btn" onclick="startPvP()">START MATCH</button>
    </div>

    <div id="win-screen">
        <h1 id="winner-text" style="color:#ffd700; font-size: 2.2rem;">PLAYER 1 WINS!</h1>
        <button class="btn" onclick="startPvP()">RESTART MATCH</button>
        <button class="btn" style="background:#444; color:white;" onclick="goBack()">MAIN MENU</button>
    </div>

    <canvas id="gameCanvas"></canvas>
    <canvas id="confetti"></canvas>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const confCanvas = document.getElementById('confetti');
const confCtx = confCanvas.getContext('2d');

let p1Sets = 0, p2Sets = 0, currentMatch = 1;
let turn = "p1", active = false, dragging = false;
let sX, sY, mX, mY, animId;
let lastWinner = localStorage.getItem('lastWin') || "No matches yet";
let roundWinner = null;

canvas.width = Math.min(window.innerWidth * 0.94, 420);
canvas.height = window.innerHeight * 0.82; 
confCanvas.width = window.innerWidth;
confCanvas.height = window.innerHeight;

const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function playSound(freq, type, dur) {
    try {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = type; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
        gain.gain.setValueAtTime(0.08, audioCtx.currentTime);
        gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + dur);
        osc.connect(gain); gain.connect(audioCtx.destination);
        osc.start(); osc.stop(audioCtx.currentTime + dur);
    } catch(e) {}
}

class Pen {
    constructor(x, y, color) {
        this.x = x; this.y = y; this.color = color;
        this.vx = 0; this.vy = 0; this.angle = 0; this.va = 0;
    }
    draw() {
        ctx.save(); ctx.translate(this.x, this.y); ctx.rotate(this.angle);
        ctx.fillStyle = this.color; ctx.beginPath(); ctx.roundRect(-7, -32, 14, 64, 4); ctx.fill();
        ctx.fillStyle = "#1a1a1a"; ctx.beginPath(); ctx.roundRect(-7.5, -33, 15, 20, 2); ctx.fill();
        ctx.fillStyle = "#bdc3c7"; ctx.fillRect(3, -22, 2.5, 15);
        ctx.fillStyle = "#222"; ctx.beginPath(); ctx.moveTo(-3, 32); ctx.lineTo(3, 32); ctx.lineTo(0, 40); ctx.fill();
        ctx.restore();
    }
    update() {
        this.x += this.vx; this.y += this.vy; this.angle += this.va;
        this.vx *= 0.96; this.vy *= 0.96; this.va *= 0.92;
        if (Math.hypot(this.vx, this.vy) < 0.18) { this.vx = 0; this.vy = 0; this.va = 0; }
    }
}

let p1 = new Pen(canvas.width/2, canvas.height - 80, "#2196F3");
let p2 = new Pen(canvas.width/2, 80, "#F44336");

function startPvP() {
    if (animId) cancelAnimationFrame(animId);
    p1Sets = 0; p2Sets = 0; currentMatch = 1; roundWinner = null;
    updateHUD();
    document.getElementById('menu').style.display='none';
    document.getElementById('win-screen').style.display='none';
    document.getElementById('game-ui').style.display='flex';
    canvas.style.display='block';
    confCanvas.style.display='none';
    resetPositions();
    active = true;
    loop();
}

function goBack() {
    if (animId) cancelAnimationFrame(animId);
    document.getElementById('menu').style.display='block';
    document.getElementById('win-screen').style.display='none';
    document.getElementById('game-ui').style.display='none';
    canvas.style.display='none';
    confCanvas.style.display='none';
}

function updateHUD() {
    document.getElementById('p1-score').innerText = "P1: " + p1Sets;
    document.getElementById('p2-score').innerText = "P2: " + p2Sets;
}

function resetPositions() {
    p1.x = canvas.width/2; p1.y = canvas.height - 80; p1.vx = 0; p1.vy = 0; p1.angle=0;
    p2.x = canvas.width/2; p2.y = 80; p2.vx = 0; p2.vy = 0; p2.angle=0;
    if (currentMatch === 1) turn = "p1";
    else if (currentMatch === 2) turn = "p2";
    else turn = (roundWinner === "Player 1") ? "p2" : "p1";
}

function resetRound(winner) {
    active = false;
    roundWinner = winner;
    if (winner === "Player 1") p1Sets++; else p2Sets++;
    playSound(550, 'triangle', 0.2);
    updateHUD();
    if (p1Sets >= 2 || p2Sets >= 2) {
        setTimeout(() => showWinScreen(winner), 400);
    } else {
        currentMatch++;
        setTimeout(() => { resetPositions(); active = true; }, 1000);
    }
}

function showWinScreen(winner) {
    active = false; lastWinner = winner; localStorage.setItem('lastWin', winner);
    playSound(750, 'sawtooth', 0.4);
    canvas.style.display = 'none';
    document.getElementById('game-ui').style.display = 'none';
    const ws = document.getElementById('win-screen');
    ws.style.display = 'flex';
    document.getElementById('winner-text').innerText = winner.toUpperCase() + " WINS!";
    confCanvas.style.display = 'block';
    startConfetti();
}

let particles = [];
function startConfetti() {
    particles = [];
    for(let i=0; i<80; i++) particles.push({x: Math.random()*confCanvas.width, y: -20, c: `hsl(${Math.random()*360}, 70%, 50%)`, s: Math.random()*4+2, v: Math.random()*2+2});
    confettiLoop();
}
function confettiLoop() {
    if (document.getElementById('win-screen').style.display !== 'flex') return;
    confCtx.clearRect(0,0,confCanvas.width, confCanvas.height);
    particles.forEach(p => {
        p.y += p.v; confCtx.fillStyle = p.c; confCtx.fillRect(p.x, p.y, p.s, p.s);
        if(p.y > confCanvas.height) p.y = -20;
    });
    requestAnimationFrame(confettiLoop);
}

function solveCollision() {
    let dx = p2.x - p1.x, dy = p2.y - p1.y, dist = Math.hypot(dx, dy);
    if (dist < 30) {
        playSound(350, 'square', 0.08);
        let angle = Math.atan2(dy, dx);
        let force = (Math.hypot(p1.vx, p1.vy) + Math.hypot(p2.vx, p2.vy) + 3.5) * 0.48;
        p1.vx = -Math.cos(angle) * force; p1.vy = -Math.sin(angle) * force;
        p2.vx = Math.cos(angle) * force; p2.vy = Math.sin(angle) * force;
        p1.va = (Math.random()-0.5)*0.5; p2.va = (Math.random()-0.5)*0.5;
    }
}

function loop() {
    if (canvas.style.display === 'none') return;
    animId = requestAnimationFrame(loop);
    ctx.clearRect(0,0,canvas.width, canvas.height);
    ctx.strokeStyle = "rgba(0,0,0,0.06)";
    for(let i=0; i<canvas.width; i+=25) { ctx.beginPath(); ctx.moveTo(i,0); ctx.lineTo(i,canvas.height); ctx.stroke(); }
    p1.update(); p2.update(); solveCollision();
    if (dragging) {
        let p = (turn === "p1") ? p1 : p2;
        let d = Math.min(Math.hypot(mX - sX, mY - sY), 140);
        ctx.beginPath(); ctx.setLineDash([4, 4]); ctx.moveTo(p.x, p.y);
        let angle = Math.atan2(mY - sY, mX - sX);
        ctx.lineTo(p.x - Math.cos(angle) * d, p.y - Math.sin(angle) * d);
        ctx.strokeStyle = "rgba(255,255,255,0.4)"; ctx.stroke(); ctx.setLineDash([]);
    }
    p1.draw(); p2.draw();
    if (active) {
        if (p1.x < 0 || p1.x > canvas.width || p1.y < 0 || p1.y > canvas.height) resetRound("Player 2");
        else if (p2.x < 0 || p2.x > canvas.width || p2.y < 0 || p2.y > canvas.height) resetRound("Player 1");
    }
}

function handleInput(x, y, end) {
    if (!active) return;
    let p = (turn === "p1") ? p1 : p2;
    if (!end) {
        if (Math.hypot(x - p.x, y - p.y) < 55) { dragging = true; sX = x; sY = y; mX = x; mY = y; }
    } else if (dragging) {
        let d = Math.min(Math.hypot(mX - sX, mY - sY), 140);
        let a = Math.atan2(mY - sY, mX - sX);
        p.vx = -Math.cos(a) * (d * 0.125); p.vy = -Math.sin(a) * (d * 0.125);
        dragging = false; turn = (turn === "p1") ? "p2" : "p1";
    }
}

canvas.addEventListener('mousedown', e => handleInput(e.offsetX, e.offsetY, false));
window.addEventListener('mousemove', e => { mX = e.offsetX; mY = e.offsetY; });
window.addEventListener('mouseup', e => handleInput(mX, mY, true));
canvas.addEventListener('touchstart', e => { let t = e.touches[0]; handleInput(t.clientX - canvas.offsetLeft, t.clientY - canvas.offsetTop, false); e.preventDefault(); }, {passive: false});
canvas.addEventListener('touchmove', e => { let t = e.touches[0]; mX = t.clientX - canvas.offsetLeft; mY = t.clientY - canvas.offsetTop; e.preventDefault(); }, {passive: false});
canvas.addEventListener('touchend', () => handleInput(mX, mY, true));

function showHistory() { alert("Last Match Winner: " + lastWinner); }
</script>
</body>
</html></html>
