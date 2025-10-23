<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Car Racing Game</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    background: #222;
    font-family: Arial, sans-serif;
  }
  #startScreen {
    position: absolute;
    width: 100%;
    height: 100%;
    background: linear-gradient(to bottom, #333, #111);
    color: white;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: 2em;
    z-index: 2;
    text-align: center;
    padding: 20px;
  }
  #gameArea {
    width: 100%;
    height: 100vh;
    position: relative;
    overflow: hidden;
    background: #444;
  }
  .car {
    width: 50px;
    height: 100px;
    position: absolute;
    bottom: 20px;
    left: calc(50% - 25px);
    background: red;
    border-radius: 5px;
  }
  .enemy {
    width: 50px;
    height: 100px;
    position: absolute;
    background: yellow;
    border-radius: 5px;
  }
  .roadLine {
    position: absolute;
    width: 10px;
    height: 80px;
    background: white;
    left: 50%;
    transform: translateX(-50%);
  }
  #score {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-size: 20px;
    z-index: 1;
  }
  #controls {
    position: absolute;
    bottom: 10px;
    width: 100%;
    display: flex;
    justify-content: space-around;
    z-index: 3;
  }
  .btn {
    background: #ff6600;
    color: white;
    padding: 15px 25px;
    border: none;
    border-radius: 10px;
    font-size: 18px;
  }
</style>
</head>
<body>

<div id="startScreen">
  <h1>🚗 Car Racing Game</h1>
  <p>Use on-screen buttons or arrow keys to move.<br>Don't hit other cars!</p>
  <button id="startBtn" style="padding:10px 20px;font-size:20px;border:none;border-radius:10px;background:#ff6600;color:white;">Start Game</button>
</div>

<div id="gameArea">
  <div id="score">Score: 0</div>
  <div id="controls">
    <button class="btn" id="leftBtn">⬅️ Left</button>
    <button class="btn" id="rightBtn">➡️ Right</button>
  </div>
</div>

<script>
const startScreen = document.getElementById('startScreen');
const gameArea = document.getElementById('gameArea');
let player = { speed: 5, score: 0, x: 0, y: 0, inplay: false };
let keys = { ArrowLeft: false, ArrowRight: false, ArrowUp: false, ArrowDown: false };

document.addEventListener('keydown', e => { e.preventDefault(); keys[e.key] = true; });
document.addEventListener('keyup', e => { e.preventDefault(); keys[e.key] = false; });

document.getElementById('startBtn').addEventListener('click', start);

function attachButtonControls() {
  const leftBtn = document.getElementById('leftBtn');
  const rightBtn = document.getElementById('rightBtn');

  // left button
  leftBtn.addEventListener('touchstart', () => keys.ArrowLeft = true);
  leftBtn.addEventListener('touchend', () => keys.ArrowLeft = false);
  leftBtn.addEventListener('mousedown', () => keys.ArrowLeft = true);
  leftBtn.addEventListener('mouseup', () => keys.ArrowLeft = false);
  leftBtn.addEventListener('mouseleave', () => keys.ArrowLeft = false);

  // right button
  rightBtn.addEventListener('touchstart', () => keys.ArrowRight = true);
  rightBtn.addEventListener('touchend', () => keys.ArrowRight = false);
  rightBtn.addEventListener('mousedown', () => keys.ArrowRight = true);
  rightBtn.addEventListener('mouseup', () => keys.ArrowRight = false);
  rightBtn.addEventListener('mouseleave', () => keys.ArrowRight = false);
}

function start() {
  startScreen.style.display = 'none';
  gameArea.innerHTML = `
    <div id="score">Score: 0</div>
    <div id="controls">
      <button class="btn" id="leftBtn">⬅️ Left</button>
      <button class="btn" id="rightBtn">➡️ Right</button>
    </div>
  `;
  player.inplay = true;
  player.score = 0;
  createCar();
  for (let i = 0; i < 5; i++) createRoadLine(i * 150);
  for (let i = 0; i < 3; i++) createEnemy(i * 350);
  attachButtonControls();
  window.requestAnimationFrame(playGame);
}

function createCar() {
  let car = document.createElement('div');
  car.classList.add('car');
  gameArea.appendChild(car);
  player.x = car.offsetLeft;
  player.y = car.offsetTop;
  player.el = car;
}

function createEnemy(y) {
  let enemy = document.createElement('div');
  enemy.classList.add('enemy');
  enemy.style.top = y + 'px';
  enemy.style.left = Math.floor(Math.random() * (window.innerWidth - 60)) + 'px';
  gameArea.appendChild(enemy);
}

function createRoadLine(y) {
  let line = document.createElement('div');
  line.classList.add('roadLine');
  line.style.top = y + 'px';
  gameArea.appendChild(line);
}

function moveRoadLines() {
  document.querySelectorAll('.roadLine').forEach(line => {
    let top = parseInt(line.style.top);
    top += player.speed;
    if (top > window.innerHeight) top -= window.innerHeight + 100;
    line.style.top = top + 'px';
  });
}

function moveEnemies() {
  document.querySelectorAll('.enemy').forEach(enemy => {
    let top = parseInt(enemy.style.top);
    top += player.speed;
    if (top > window.innerHeight) {
      top = -200;
      enemy.style.left = Math.floor(Math.random() * (window.innerWidth - 60)) + 'px';
    }
    enemy.style.top = top + 'px';
    if (isCollide(player.el, enemy)) endGame();
  });
}

function isCollide(a, b) {
  let aRect = a.getBoundingClientRect();
  let bRect = b.getBoundingClientRect();
  return !(
    aRect.bottom < bRect.top ||
    aRect.top > bRect.bottom ||
    aRect.right < bRect.left ||
    aRect.left > bRect.right
  );
}

function playGame() {
  if (!player.inplay) return;
  moveRoadLines();
  moveEnemies();

  let car = player.el;
  let road = gameArea.getBoundingClientRect();

  if (keys.ArrowLeft && player.x > 0) player.x -= player.speed;
  if (keys.ArrowRight && player.x < road.width - 60) player.x += player.speed;
  if (keys.ArrowUp && player.y > 0) player.y -= player.speed;
  if (keys.ArrowDown && player.y < road.height - 120) player.y += player.speed;

  car.style.top = player.y + 'px';
  car.style.left = player.x + 'px';

  player.score++;
  document.getElementById('score').textContent = 'Score: ' + player.score;
  window.requestAnimationFrame(playGame);
}

function endGame() {
  player.inplay = false;
  startScreen.style.display = 'flex';
  startScreen.innerHTML = `
    <h1>Game Over 😢</h1>
    <p>Your Score: ${player.score}</p>
    <button id="startBtn" style="padding:10px 20px;font-size:20px;border:none;border-radius:10px;background:#ff6600;color:white;">Restart</button>
  `;
  document.getElementById('startBtn').addEventListener('click', start);
}
</script>
<!-- ad container fixed at the bottom -->
<div id="adContainer" style="
    position:fixed;
    bottom:0;
    left:0;
    width:100%;
    text-align:center;
    background:#000;
    z-index:5;">
    
    <!-- Paste your ad code here -->
    <script type="text/javascript">
      atOptions = {
        'key' : 'c6e7e29c2a5340c57f25279305492c90',
        'format' : 'iframe',
        'height' : 50,
        'width' : 320,
        'params' : {}
      };
    </script>
    <script type="text/javascript" src="//www.highperformanceformat.com/c6e7e29c2a5340c57f25279305492c90/invoke.js"></script>
</div>


</body>
</html>
