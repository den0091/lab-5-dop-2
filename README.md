<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8">
  <title>Спіймай куб</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      background: #111;
      color: white;
      font-family: Arial, sans-serif;
    }
    #gameWindow {
      position: relative;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
    }
    .cube {
      position: absolute;
      width: 50px;
      height: 50px;
      border-radius: 8px;
      cursor: pointer;
    }
    #controls {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      gap: 10px;
      z-index: 10;
    }
    button {
      padding: 10px 20px;
      background: #444;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 16px;
    }
    button:hover {
      background: #666;
    }
    #task {
      position: absolute;
      top: 10px;
      right: 10px;
      background: rgba(0,0,0,0.7);
      padding: 10px;
      border-radius: 8px;
      display: flex;
      align-items: center;
      gap: 10px;
    }
    #targetCube {
      width: 30px;
      height: 30px;
      border-radius: 5px;
    }
    #scoreDisplay {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(0,0,0,0.7);
      padding: 10px;
      border-radius: 8px;
      font-size: 18px;
    }
    /* Модальне вікно */
    #modal {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(30,30,30,0.95);
      color: #fff;
      padding: 40px 60px;
      border-radius: 15px;
      font-size: 24px;
      text-align: center;
      display: none;
      z-index: 1000;
      box-shadow: 0 0 30px rgba(255,255,255,0.5);
    }
    #modal button {
      margin-top: 20px;
      background: #ff4444;
    }
  </style>
</head>
<body>
<div id="gameWindow"></div>

<div id="controls">
  <button onclick="startGame()">Старт</button>
  <button onclick="pauseGame()">Пауза</button>
  <button onclick="slower()">Повільніше</button>
  <button onclick="faster()">Швидше</button>
  <button id="easyBtn" onclick="toggleEasyMode()">Полегшення: Вимкнено</button>
  <label>
    Швидкість:
    <input type="range" id="speedRange" min="1" max="10" value="3" onchange="setSpeed(this.value)">
  </label>
</div>

<div id="task">
  Завдання: <div id="targetCube"></div>
  <span id="attempts">Спроби: 10</span>
</div>

<div id="scoreDisplay">Попадання: 0</div>

<div id="modal">
  <div id="modalText"></div>
  <button onclick="closeModal()">OK</button>
</div>

<script>
const gameWindow = document.getElementById("gameWindow");
const colors = ["red", "green", "blue", "yellow", "purple"];
let cubes = [];
let vx = [];
let vy = [];
let speed = 3;
let running = false;
let paused = false;
let targetColor;
let attempts = 10;
let score = 0;
let easyMode = false;

// Модальне вікно
const modal = document.getElementById("modal");
const modalText = document.getElementById("modalText");

function showModal(text) {
  modalText.innerText = text;
  modal.style.display = "block";
}

function closeModal() {
  modal.style.display = "none";
}

// Перемикач легкого режиму
function toggleEasyMode() {
  easyMode = !easyMode;
  const btn = document.getElementById("easyBtn");
  if (easyMode) {
    btn.innerText = "Полегшення: Увімкнено";
    alert("Полегшений режим увімкнено! Наведи курсор на правильний куб — він автоматично буде спійманий.");
  } else {
    btn.innerText = "Полегшення: Вимкнено";
    alert("Полегшений режим вимкнено.");
  }
}

function createCubes() {
  cubes = [];
  vx = [];
  vy = [];
  gameWindow.innerHTML = "";

  const spacing = 80;
  const startX = (gameWindow.clientWidth - (colors.length * spacing)) / 2;
  const startY = 50;

  for (let i = 0; i < colors.length; i++) {
    let cube = document.createElement("div");
    cube.className = "cube";
    cube.style.background = colors[i];
    cube.style.left = startX + i * spacing + "px";
    cube.style.top = startY + "px";

    gameWindow.appendChild(cube);
    cubes.push(cube);

    vx.push((Math.random() < 0.5 ? -1 : 1) * (Math.random() * 2 + 1));
    vy.push((Math.random() < 0.5 ? -1 : 1) * (Math.random() * 2 + 1));

    // Клік на куб
    cube.onclick = () => checkCube(cube);

    // Автоспіймання при наведенні на правильний куб
    cube.addEventListener("mouseenter", () => {
      if (easyMode && cube.style.background === targetColor) {
        checkCube(cube);
      }
    });
  }
}

function setTarget() {
  targetColor = colors[Math.floor(Math.random() * colors.length)];
  document.getElementById("targetCube").style.background = targetColor;
}

function startGame() {
  running = true;
  paused = false;
  attempts = 10;
  score = 0;
  document.getElementById("attempts").innerText = "Спроби: " + attempts;
  document.getElementById("scoreDisplay").innerText = "Попадання: " + score;
  createCubes();
  setTarget();
  update();
}

function pauseGame() {
  paused = !paused;
  if (!paused) update();
}

function slower() {
  speed = Math.max(1, speed - 1);
  document.getElementById("speedRange").value = speed;
}

function faster() {
  speed = Math.min(10, speed + 1);
  document.getElementById("speedRange").value = speed;
}

function setSpeed(val) {
  speed = parseInt(val);
}

function checkCube(cube) {
  if (!running) return;
  if (cube.style.background === targetColor) {
    score++;
    document.getElementById("scoreDisplay").innerText = "Попадання: " + score;
    if (score >= 20) {
      showModal("Молодець!");
      running = false;
    } else {
      setTarget();
    }
  } else {
    attempts--;
    document.getElementById("attempts").innerText = "Спроби: " + attempts;
    if (attempts <= 0) {
      showModal("Треба ще потренуватись");
      running = false;
    }
  }
}

// Колізія між кубами
function update() {
  if (!running || paused) return;

  for (let i = 0; i < cubes.length; i++) {
    let cube = cubes[i];
    let x = cube.offsetLeft + vx[i]*speed*0.5;
    let y = cube.offsetTop + vy[i]*speed*0.5;

    if (x <= 0) { vx[i] = Math.abs(vx[i]); x = 0; }
    if (x >= gameWindow.clientWidth - 50) { vx[i] = -Math.abs(vx[i]); x = gameWindow.clientWidth - 50; }
    if (y <= 0) { vy[i] = Math.abs(vy[i]); y = 0; }
    if (y >= gameWindow.clientHeight - 50) { vy[i] = -Math.abs(vy[i]); y = gameWindow.clientHeight - 50; }

    for (let j = 0; j < cubes.length; j++) {
      if (i === j) continue;
      let other = cubes[j];
      let dx = x - other.offsetLeft;
      let dy = y - other.offsetTop;
      if (Math.abs(dx) < 50 && Math.abs(dy) < 50) {
        vx[i] *= -1; vy[i] *= -1;
        x += vx[i]*speed*0.5;
        y += vy[i]*speed*0.5;
      }
    }

    cube.style.left = x + "px";
    cube.style.top = y + "px";
  }

  requestAnimationFrame(update);
}
</script>
</body>
</html>

