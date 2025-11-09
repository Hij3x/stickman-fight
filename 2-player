<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Simple Stickman Fight</title>
  <style>
    *{box-sizing:border-box;margin:0;padding:0;font-family:sans-serif}
    body{background:#111;color:#f5f5f5;display:flex;justify-content:center;align-items:center;height:100vh}
    #wrapper{display:flex;flex-direction:column;align-items:center;gap:8px}
    canvas{border:2px solid #444;background:#222;border-radius:8px}
    .ui{width:800px;display:flex;justify-content:space-between;font-size:14px;margin-bottom:6px}
    .health-bar{width:230px;height:16px;border-radius:999px;border:2px solid #555;overflow:hidden;background:#333;position:relative}
    .health-fill{position:absolute;left:0;top:0;width:100%;height:100%;background:linear-gradient(90deg,#0f0,#ff0,#f00);transform-origin:left center}
  </style>
</head>
<body>
<div id="wrapper">
  <div class="ui">
    <div>
      <div>Player 1 (Blue)</div>
      <div class="health-bar"><div id="hp1" class="health-fill"></div></div>
      <div>Move: A/D, Jump: W, Punch: S</div>
    </div>
    <div style="text-align:right">
      <div>Player 2 (Red)</div>
      <div class="health-bar"><div id="hp2" class="health-fill"></div></div>
      <div>Move: ←/→, Jump: ↑, Punch: ↓</div>
    </div>
  </div>
  <canvas id="game" width="800" height="450"></canvas>
</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const hp1 = document.getElementById("hp1");
const hp2 = document.getElementById("hp2");
const keys = {};

window.addEventListener("keydown", e => { keys[e.code] = true; });
window.addEventListener("keyup", e => { keys[e.code] = false; });

const GROUND = canvas.height - 40;
const GRAVITY = 2000;
const MOVE = 350;
const JUMP = 800;

class Player {
  constructor(x, color, controls, facing) {
    this.x = x;
    this.y = GROUND;
    this.vx = 0;
    this.vy = 0;
    this.width = 30;
    this.height = 90;
    this.color = color;
    this.controls = controls;
    this.facing = facing;
    this.hpMax = 100;
    this.hp = 100;
    this.attackTime = 0;
    this.isAttacking = false;
    this.hasHit = false;
    this.dead = false;
  }
  reset(x, facing) {
    this.x = x;
    this.y = GROUND;
    this.vx = this.vy = 0;
    this.facing = facing;
    this.hp = this.hpMax;
    this.attackTime = 0;
    this.isAttacking = false;
    this.hasHit = false;
    this.dead = false;
  }
  update(dt) {
    if (this.dead) return;
    const L = keys[this.controls.left];
    const R = keys[this.controls.right];
    const U = keys[this.controls.up];
    const A = keys[this.controls.attack];

    if (L && !R) { this.vx = -MOVE; this.facing = -1; }
    else if (R && !L) { this.vx = MOVE; this.facing = 1; }
    else { this.vx *= 0.85; if (Math.abs(this.vx) < 5) this.vx = 0; }

    const onGround = this.y >= GROUND - 1;
    if (U && onGround) this.vy = -JUMP;

    if (A && this.attackTime <= 0) {
      this.attackTime = 0.18 + 0.22;
      this.hasHit = false;
    }
    if (this.attackTime > 0) {
      this.attackTime -= dt;
      this.isAttacking = this.attackTime > 0.22;
    } else this.isAttacking = false;

    this.vy += GRAVITY * dt;
    this.x += this.vx * dt;
    this.y += this.vy * dt;

    if (this.y > GROUND) { this.y = GROUND; this.vy = 0; }
    if (this.x < 20) this.x = 20;
    if (this.x > canvas.width - 50) this.x = canvas.width - 50;

    if (this.hp <= 0) { this.hp = 0; this.dead = true; }
  }
  attackBox() {
    const reach = 40;
    const h = 30;
    const cx = this.x + this.width / 2;
    const cy = this.y - this.height * 0.4;
    return {
      x: this.facing === 1 ? cx : cx - reach,
      y: cy - h / 2,
      width: reach,
      height: h
    };
  }
  draw(ctx) {
    const cx = this.x + this.width / 2;
    const headR = this.width * 0.7;
    const headY = this.y - this.height;
    const neckY = headY + headR * 2;
    const hipY = this.y - 15;

    ctx.strokeStyle = this.color;
    ctx.lineWidth = 5;
    ctx.lineCap = "round";

    // head
    ctx.beginPath();
    ctx.arc(cx, headY + headR, headR, 0, Math.PI * 2);
    ctx.stroke();

    // body
    ctx.beginPath();
    ctx.moveTo(cx, neckY);
    ctx.lineTo(cx, hipY);
    ctx.stroke();

    // legs
    const legLen = 35;
    ctx.beginPath();
    ctx.moveTo(cx, hipY);
    ctx.lineTo(cx - 18, hipY + legLen);
    ctx.moveTo(cx, hipY);
    ctx.lineTo(cx + 18, hipY + legLen);
    ctx.stroke();

    // arms
    const shoulderY = neckY + 8;
    const armLen = 30;

    // back arm
    ctx.beginPath();
    ctx.moveTo(cx, shoulderY);
    ctx.lineTo(cx - this.facing * armLen * 0.6, shoulderY + armLen * 0.4);
    ctx.stroke();

    // front arm / punch
    ctx.beginPath();
    ctx.moveTo(cx, shoulderY);
    if (this.isAttacking) {
      ctx.lineTo(cx + this.facing * (armLen + 12), shoulderY - 4);
    } else {
      ctx.lineTo(cx + this.facing * armLen * 0.7, shoulderY + armLen * 0.2);
    }
    ctx.stroke();
  }
}

const p1 = new Player(150, "#3aa8ff",
  {left:"KeyA", right:"KeyD", up:"KeyW", attack:"KeyS"}, 1);
const p2 = new Player(canvas.width - 180, "#ff5555",
  {left:"ArrowLeft", right:"ArrowRight", up:"ArrowUp", attack:"ArrowDown"}, -1);

let last = 0;
let gameOver = false;
let winner = "";

function overlap(a,b){
  return a.x < b.x + b.width &&
         a.x + a.width > b.x &&
         a.y < b.y + b.height &&
         a.y + a.height > b.y;
}

function tryHit(att, def) {
  if (!att.isAttacking || att.hasHit) return;
  const ab = att.attackBox();
  const db = {x:def.x, y:def.y-def.height, width:def.width, height:def.height};
  if (overlap(ab, db)) {
    att.hasHit = true;
    def.hp -= 12;
    def.vx = 400 * att.facing;
    def.vy = -400;
  }
}

function updateHealthBars() {
  hp1.style.transform = `scaleX(${Math.max(0, p1.hp / p1.hpMax)})`;
  hp2.style.transform = `scaleX(${Math.max(0, p2.hp / p2.hpMax)})`;
}

function resetGame() {
  p1.reset(150, 1);
  p2.reset(canvas.width - 180, -1);
  gameOver = false;
  winner = "";
  updateHealthBars();
}

function drawBackground() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.strokeStyle = "#555";
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(0, GROUND + 0.5);
  ctx.lineTo(canvas.width, GROUND + 0.5);
  ctx.stroke();
}

function drawGameOver() {
  if (!gameOver) return;
  ctx.save();
  ctx.fillStyle = "rgba(0,0,0,0.7)";
  ctx.fillRect(0,0,canvas.width,canvas.height);
  ctx.fillStyle = "#fff";
  ctx.textAlign = "center";
  ctx.font = "32px sans-serif";
  ctx.fillText(winner, canvas.width/2, canvas.height/2 - 30);
  ctx.font = "22px sans-serif";
  ctx.fillText("👑 KING — Made by OfficialYaman 👑", canvas.width/2, canvas.height/2 + 5);
  ctx.font = "18px sans-serif";
  ctx.fillText("Press R to Restart", canvas.width/2, canvas.height/2 + 35);
  ctx.restore();
}

function loop(ts) {
  if (!last) last = ts;
  const dt = (ts - last) / 1000;
  last = ts;

  if (keys["KeyR"]) resetGame();

  if (!gameOver) {
    p1.update(dt); p2.update(dt);
    tryHit(p1,p2); tryHit(p2,p1);
    updateHealthBars();

    if (p1.dead || p2.dead) {
      gameOver = true;
      if (p1.dead && p2.dead) winner = "Draw!";
      else if (p2.dead) winner = "Player 1 Wins!";
      else winner = "Player 2 Wins!";
    }
  }

  drawBackground();
  p1.draw(ctx);
  p2.draw(ctx);
  drawGameOver();
  requestAnimationFrame(loop);
}

resetGame();
requestAnimationFrame(loop);
</script>
</body>
</html>
