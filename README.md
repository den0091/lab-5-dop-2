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
      flex-wrap: wrap;
      justify-content: center;
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
      padding: 10px 20px;
      border-radius: 8px;
      border: none;
      color: white;
      cursor: pointer;
      font-size: 18px;
    }
    #modal button:hover {
      background: #ff6666;
    }

    .menu-button {
      position: fixed;
      top: 50px;
      left: 20px;
      background-color: #444;
      color: #fff;
      border: none;
      padding: 10px 15px;
      border-radius: 5px;
      cursor: pointer;
      z-index: 1000;
    }
    .menu-button:hover {
      background-color: #666;
    }
    .menu {
      display: none;
      position: fixed;
      top: 90px;
      left: 20px;
      background-color: #2c2c2c;
      padding: 10px;
      border-radius: 5px;
      flex-direction: column;
      gap: 10px;
      z-index: 999;
    }
    .menu a {
      color: #1e90ff;
      text-decoration: none;
      padding: 5px 10px;
      background-color: #444;
      border-radius: 5px;
      display: block;
    }
    .menu a:hover {
      background-color: #666;
    }
  </style>
</head>
<body>

<!-- Кнопка Меню і випадаюче меню -->
<button class="menu-button" onclick="toggleMenu()">Меню</button>
<div class="menu" id="menu">
    <a href="laba5.html" target="_blank">Перейти в гру</a>
</div>

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

const modal = document.getElementById("modal");
const modalText = document.getElementById("modalText");

function toggleMenu() {
  const menu = document.getElementById('menu');
  menu.style.display = menu.style.display === 'flex' ? 'none' : 'flex';
}

function showModal(text) {
  modalText.innerHTML = text;
  modal.style.display = "block";
}

function closeModal() {
  modal.style.display = "none";
}

function toggleEasyMode() {
  easyMode = !easyMode;
  const btn = document.getElementById("easyBtn");
  btn.innerText = easyMode ? "Полегшення: Увімкнено" : "Полегшення: Вимкнено";
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

    cube.onclick = () => checkCube(cube);
    cube.addEventListener("mouseenter", () => {
      if (easyMode && cube.style.background === targetColor) checkCube(cube);
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

function slower() { speed = Math.max(1, speed - 1); document.getElementById("speedRange").value = speed; }
function faster() { speed = Math.min(10, speed + 1); document.getElementById("speedRange").value = speed; }
function setSpeed(val) { speed = parseInt(val); }

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

function update() {
  if (!running || paused) return;

  const cubeSize = cubes[0].offsetWidth;

  for (let i = 0; i < cubes.length; i++) {
    let cube = cubes[i];
    let x = cube.offsetLeft + vx[i] * speed * 0.5;
    let y = cube.offsetTop + vy[i] * speed * 0.5;

    // Відскок від країв з виправленням позиції
    if (x < 0) {
      x = 0;
      vx[i] *= -1;
    } else if (x + cubeSize > gameWindow.clientWidth) {
      x = gameWindow.clientWidth - cubeSize;
      vx[i] *= -1;
    }

    if (y < 0) {
      y = 0;
      vy[i] *= -1;
    } else if (y + cubeSize > gameWindow.clientHeight) {
      y = gameWindow.clientHeight - cubeSize;
      vy[i] *= -1;
    }

    cube.style.left = x + "px";
    cube.style.top = y + "px";
  }

  // Колізії між кубами
  for (let i = 0; i < cubes.length; i++) {
    for (let j = i + 1; j < cubes.length; j++) {
      const a = cubes[i];
      const b = cubes[j];

      const ax = a.offsetLeft;
      const ay = a.offsetTop;
      const bx = b.offsetLeft;
      const by = b.offsetTop;

      const dx = ax - bx;
      const dy = ay - by;
      const distance = Math.sqrt(dx * dx + dy * dy);

      if (distance < cubeSize) {
        // Простий обмін векторів
        [vx[i], vx[j]] = [vx[j], vx[i]];
        [vy[i], vy[j]] = [vy[j], vy[i]];

        // Зміщення, щоб уникнути залипання
        const overlap = cubeSize - distance;
        const shiftX = (dx / distance) * (overlap / 2);
        const shiftY = (dy / distance) * (overlap / 2);

        a.style.left = (ax + shiftX) + "px";
        a.style.top = (ay + shiftY) + "px";
        b.style.left = (bx - shiftX) + "px";
        b.style.top = (by - shiftY) + "px";
      }
    }
  }

  requestAnimationFrame(update);
}
</script>
</body>
</html>
