<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Stickman 2-Player Fight</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
    }

    body {
      background: #111;
      color: #f5f5f5;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    #wrapper {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
    }

    canvas {
      border: 2px solid #444;
      background: linear-gradient(#222 50%, #1a1a1a 50%);
      border-radius: 8px;
      display: block;
    }

    .ui {
      width: 900px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 14px;
    }

    .health-bar {
      position: relative;
      width: 260px;
      height: 18px;
      border-radius: 999px;
      border: 2px solid #555;
      overflow: hidden;
      background: #333;
    }

    .health-bar-fill {
      position: absolute;
      height: 100%;
      left: 0;
      top: 0;
      background: linear-gradient(90deg, #0f0, #ff0, #f00);
      transform-origin: left center;
    }

    .label {
      font-weight: 600;
      margin-bottom: 2px;
    }

    .player-label {
      display: flex;
      flex-direction: column;
      gap: 2px;
    }

    .center-text {
      text-align: center;
      font-size: 13px;
      opacity: 0.8;
      line-height: 1.4;
    }

    .keys {
      font-size: 12px;
      opacity: 0.9;
    }
  </style>
</head>
<body>
<div id="wrapper">

  <div class="ui">
    <div class="player-label">
      <span class="label">Player 1 (Blue)</span>
      <div class="health-bar">
        <div id="hp1" class="health-bar-fill"></div>
      </div>
      <span class="keys">Move: W/A/D — Attack: S</span>
    </div>

    <div class="center-text">
      <div>Stickman Fight</div>
      <div>First to drop HP to 0 loses.</div>
      <div>Press <b>R</b> to restart.</div>
    </div>

    <div class="player-label" style="align-items: flex-end; text-align: right;">
      <span class="label">Player 2 (Red)</span>
      <div class="health-bar">
        <div id="hp2" class="health-bar-fill"></div>
      </div>
      <span class="keys">Move: ↑/←/→ — Attack: ↓</span>
    </div>
  </div>

  <canvas id="game" width="900" height="500"></canvas>
</div>

<script>
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');

  const hpBar1 = document.getElementById('hp1');
  const hpBar2 = document.getElementById('hp2');

  const keys = {};
  window.addEventListener('keydown', (e) => {
    keys[e.code] = true;
    // prevent page scrolling with arrows / space if you add space later
    if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(e.code)) {
      e.preventDefault();
    }
  });
  window.addEventListener('keyup', (e) => {
    keys[e.code] = false;
  });

  const GROUND_Y = canvas.height - 60;
  const GRAVITY = 2000;        // px/s^2
  const MOVE_SPEED = 380;      // px/s
  const JUMP_SPEED = 850;      // px/s
  const FRICTION = 0.82;

  class Player {
    constructor(options) {
      this.x = options.x;
      this.y = options.y;      // y is bottom position
      this.width = options.width;
      this.height = options.height;
      this.color = options.color;
      this.facing = options.facing; // 1 right, -1 left

      this.hpMax = options.hp || 100;
      this.hp = options.hp || 100;

      this.vx = 0;
      this.vy = 0;

      this.attackTime = 0;
      this.attackDuration = 0.18; // seconds
      this.attackCooldown = 0.25; // seconds
      this.isAttacking = false;
      this.hasDealtDamage = false;

      this.control = options.control;

      this.dead = false;
    }

    reset(x, facing) {
      this.x = x;
      this.y = GROUND_Y;
      this.vx = 0;
      this.vy = 0;
      this.hp = this.hpMax;
      this.facing = facing;
      this.attackTime = 0;
      this.isAttacking = false;
      this.hasDealtDamage = false;
      this.dead = false;
    }

    update(dt) {
      if (this.dead) return;

      // Horizontal movement
      let moveLeft = keys[this.control.left];
      let moveRight = keys[this.control.right];
      let moveUp = keys[this.control.up];
      let attackKey = keys[this.control.attack];

      if (moveLeft && !moveRight) {
        this.vx = -MOVE_SPEED;
        this.facing = -1;
      } else if (moveRight && !moveLeft) {
        this.vx = MOVE_SPEED;
        this.facing = 1;
      } else {
        this.vx *= FRICTION;
        if (Math.abs(this.vx) < 10) this.vx = 0;
      }

      // Jump (only if on ground)
      const onGround = this.y >= GROUND_Y - 1;
      if (moveUp && onGround) {
        this.vy = -JUMP_SPEED;
      }

      // Attack logic
      if (attackKey && this.attackTime <= 0) {
        this.attackTime = this.attackDuration + this.attackCooldown;
        this.hasDealtDamage = false;
      }

      if (this.attackTime > 0) {
        this.attackTime -= dt;
        if (this.attackTime <= 0) {
          this.attackTime = 0;
          this.isAttacking = false;
        } else if (this.attackTime <= this.attackCooldown) {
          this.isAttacking = false;
        } else {
          this.isAttacking = true;
        }
      } else {
        this.isAttacking = false;
      }

      // Gravity
      this.vy += GRAVITY * dt;

      // Apply velocity
      this.x += this.vx * dt;
      this.y += this.vy * dt;

      // Ground collision
      if (this.y > GROUND_Y) {
        this.y = GROUND_Y;
        this.vy = 0;
      }

      // Wall collision
      if (this.x < 20) {
        this.x = 20;
        this.vx = 0;
      }
      if (this.x + this.width > canvas.width - 20) {
        this.x = canvas.width - 20 - this.width;
        this.vx = 0;
      }

      if (this.hp <= 0) {
        this.hp = 0;
        this.dead = true;
      }
    }

    getAttackBox() {
      const range = 40;
      return {
        x: this.facing === 1 ? this.x + this.width : this.x - range,
        y: this.y - this.height + 10,
        width: range,
        height: this.height - 20
      };
    }

    draw(context) {
      // body
      context.fillStyle = this.color;
      context.fillRect(this.x, this.y - this.height, this.width, this.height);

      // simple head
      const headRadius = this.width * 0.55;
      context.beginPath();
      context.arc(this.x + this.width / 2, this.y - this.height - headRadius + 5, headRadius, 0, Math.PI * 2);
      context.fill();

      // attack box debug (optional: comment out if you don’t want to see it)
      if (this.isAttacking) {
        const atk = this.getAttackBox();
        context.save();
        context.globalAlpha = 0.2;
        context.fillStyle = "#fff";
        context.fillRect(atk.x, atk.y, atk.width, atk.height);
        context.restore();
      }
    }
  }

  const player1 = new Player({
    x: 150,
    y: GROUND_Y,
    width: 35,
    height: 90,
    color: "#3aa8ff",
    facing: 1,
    hp: 100,
    control: {
      left: "KeyA",
      right: "KeyD",
      up: "KeyW",
      attack: "KeyS"
    }
  });

  const player2 = new Player({
    x: canvas.width - 150 - 35,
    y: GROUND_Y,
    width: 35,
    height: 90,
    color: "#ff5555",
    facing: -1,
    hp: 100,
    control: {
      left: "ArrowLeft",
      right: "ArrowRight",
      up: "ArrowUp",
      attack: "ArrowDown"
    }
  });

  let lastTime = 0;
  let gameOver = false;
  let winnerText = "";

  function rectsOverlap(a, b) {
    return (
      a.x < b.x + b.width &&
      a.x + a.width > b.x &&
      a.y < b.y + b.height &&
      a.y + a.height > b.y
    );
  }

  function applyHit(attacker, defender) {
    if (attacker.hasDealtDamage) return;
    const atkBox = attacker.getAttackBox();
    const defBox = {
      x: defender.x,
      y: defender.y - defender.height,
      width: defender.width,
      height: defender.height
    };
    if (rectsOverlap(atkBox, defBox)) {
      attacker.hasDealtDamage = true;
      defender.hp -= 10;
      // knockback
      defender.vx = 450 * attacker.facing;
      defender.vy = -500;
    }
  }

  function updateHealthBars() {
    const p1Ratio = player1.hp / player1.hpMax;
    const p2Ratio = player2.hp / player2.hpMax;
    hpBar1.style.transform = `scaleX(${Math.max(0, p1Ratio)})`;
    hpBar2.style.transform = `scaleX(${Math.max(0, p2Ratio)})`;
  }

  function resetGame() {
    player1.reset(150, 1);
    player2.reset(canvas.width - 150 - player2.width, -1);
    winnerText = "";
    gameOver = false;
  }

  function drawBackground() {
    // clear
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // ground line
    ctx.strokeStyle = "#444";
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.moveTo(0, GROUND_Y + 0.5);
    ctx.lineTo(canvas.width, GROUND_Y + 0.5);
    ctx.stroke();

    // simple "arena" pillars
    ctx.fillStyle = "#333";
    ctx.fillRect(60, GROUND_Y - 80, 15, 80);
    ctx.fillRect(canvas.width - 75, GROUND_Y - 80, 15, 80);
  }

  function drawGameOver() {
    if (!gameOver) return;
    ctx.save();
    ctx.fillStyle = "rgba(0,0,0,0.6)";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = "#fff";
    ctx.textAlign = "center";

    ctx.font = "32px system-ui";
    ctx.fillText(winnerText, canvas.width / 2, canvas.height / 2 - 10);

    ctx.font = "18px system-ui";
    ctx.fillText("Press R to restart", canvas.width / 2, canvas.height / 2 + 24);
    ctx.restore();
  }

  function gameLoop(timestamp) {
    if (!lastTime) lastTime = timestamp;
    const dt = (timestamp - lastTime) / 1000; // seconds
    lastTime = timestamp;

    if (keys["KeyR"]) {
      resetGame();
    }

    if (!gameOver) {
      player1.update(dt);
      player2.update(dt);

      if (player1.isAttacking) applyHit(player1, player2);
      if (player2.isAttacking) applyHit(player2, player1);

      updateHealthBars();

      if (player1.dead || player2.dead) {
        gameOver = true;
        if (player1.dead && player2.dead) {
          winnerText = "Draw!";
        } else if (player2.dead) {
          winnerText = "Player 1 Wins!";
        } else {
          winnerText = "Player 2 Wins!";
        }
      }
    }

    drawBackground();
    player1.draw(ctx);
    player2.draw(ctx);
    drawGameOver();

    requestAnimationFrame(gameLoop);
  }

  resetGame();
  requestAnimationFrame(gameLoop);
</script>
</body>
</html>
